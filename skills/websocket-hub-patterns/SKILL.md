---
name: websocket-hub-patterns
model: standard
description: 具有延迟 Redis 订阅、连接注册表和优雅关闭的水平扩展 WebSocket hub 模式。在构建跨多个实例扩展的实时 WebSocket 服务器时使用。触发词：WebSocket hub、WebSocket scaling、connection registry、Redis WebSocket、real-time gateway、horizontal scaling。
tags:
- Harbor
- Redis
- 容量规划
- 模式
---

# WebSocket Hub 模式

用于水平扩展 WebSocket 连接的生产模式，使用 Redis 支持的协调。


## 安装

### OpenClaw / Moltbot / Clawbot

```bash
npx clawhub@latest install websocket-hub-patterns
```


---

## 何时使用

- 实时双向通信
- 聊天应用、协作编辑
- 带客户端交互的实时仪表板
- 需要在多个网关实例之间进行水平扩展

---

## Hub 结构

```go
type Hub struct {
    // 本地状态
    connections   map[*Connection]bool
    subscriptions map[string]map[*Connection]bool // channel -> connections

    // 通道
    register   chan *Connection
    unregister chan *Connection
    broadcast  chan *Event

    // Redis 用于扩展
    redisClient  *redis.Client
    redisSubs    map[string]*goredis.PubSub
    redisSubLock sync.Mutex

    // 可选：分布式注册表
    connRegistry *ConnectionRegistry
    instanceID   string

    // 关闭
    done chan struct{}
    wg   sync.WaitGroup
}
```

---

## Hub 主循环

```go
func (h *Hub) Run() {
    for {
        select {
        case <-h.done:
            return

        case conn := <-h.register:
            h.connections[conn] = true
            if h.connRegistry != nil {
                h.connRegistry.RegisterConnection(ctx, conn.ID(), info)
            }

        case conn := <-h.unregister:
            if _, ok := h.connections[conn]; ok {
                if h.connRegistry != nil {
                    h.connRegistry.UnregisterConnection(ctx, conn.ID())
                }
                h.removeConnection(conn)
            }

        case event := <-h.broadcast:
            h.broadcastToChannel(event)
        }
    }
}
```

---

## 延迟 Redis 订阅

仅在第一个本地订阅者加入时订阅 Redis：

```go
func (h *Hub) subscribeToChannel(conn *Connection, channel string) error {
    // 添加到本地订阅
    if h.subscriptions[channel] == nil {
        h.subscriptions[channel] = make(map[*Connection]bool)
    }
    h.subscriptions[channel][conn] = true

    // 延迟：仅在第一个订阅者时订阅 Redis
    h.redisSubLock.Lock()
    defer h.redisSubLock.Unlock()

    if _, exists := h.redisSubs[channel]; !exists {
        pubsub := h.redisClient.Subscribe(context.Background(), channel)
        h.redisSubs[channel] = pubsub
        go h.forwardRedisMessages(channel, pubsub)
    }

    return nil
}

func (h *Hub) unsubscribeFromChannel(conn *Connection, channel string) {
    if subs, ok := h.subscriptions[channel]; ok {
        delete(subs, conn)

        // 没有本地订阅者时清理
        if len(subs) == 0 {
            delete(h.subscriptions, channel)
            h.closeRedisSubscription(channel)
        }
    }
}
```

---

## Redis 消息转发

```go
func (h *Hub) forwardRedisMessages(channel string, pubsub *goredis.PubSub) {
    ch := pubsub.Channel()
    for {
        select {
        case <-h.done:
            return
        case msg, ok := <-ch:
            if !ok {
                return
            }
            h.broadcast <- &Event{
                Channel: channel,
                Data:    []byte(msg.Payload),
            }
        }
    }
}

func (h *Hub) broadcastToChannel(event *Event) {
    subs := h.subscriptions[event.Channel]
    for conn := range subs {
        select {
        case conn.send <- event.Data:
            // 已发送
        default:
            // 缓冲区满 - 关闭慢客户端
            h.removeConnection(conn)
        }
    }
}
```

---

## 连接写入泵

```go
func (c *Connection) writePump() {
    ticker := time.NewTicker(54 * time.Second) // Ping 间隔
    defer func() {
        ticker.Stop()
        c.conn.Close()
    }()

    for {
        select {
        case message, ok := <-c.send:
            c.conn.SetWriteDeadline(time.Now().Add(10 * time.Second))
            if !ok {
                c.conn.WriteMessage(websocket.CloseMessage, []byte{})
                return
            }
            c.conn.WriteMessage(websocket.TextMessage, message)

            // 批量排空队列
            for i := 0; i < len(c.send); i++ {
                c.conn.WriteMessage(websocket.TextMessage, <-c.send)
            }

        case <-ticker.C:
            if err := c.conn.WriteMessage(websocket.PingMessage, nil); err != nil {
                return
            }
        }
    }
}
```

---

## 用于水平扩展的连接注册表

```go
type ConnectionRegistry struct {
    client     *redis.Client
    instanceID string
}

func (r *ConnectionRegistry) RegisterConnection(ctx context.Context, connID string, info ConnectionInfo) error {
    info.InstanceID = r.instanceID
    data, _ := json.Marshal(info)
    return r.client.Set(ctx, "ws:conn:"+connID, data, 2*time.Minute).Err()
}

func (r *ConnectionRegistry) HeartbeatInstance(ctx context.Context, connectionCount int) error {
    info := InstanceInfo{
        InstanceID:  r.instanceID,
        Connections: connectionCount,
    }
    data, _ := json.Marshal(info)
    return r.client.Set(ctx, "ws:instance:"+r.instanceID, data, 30*time.Second).Err()
}
```

---

## 优雅关闭

```go
func (h *Hub) Shutdown() {
    close(h.done)

    // 关闭所有 Redis 订阅
    h.redisSubLock.Lock()
    for channel, pubsub := range h.redisSubs {
        pubsub.Close()
        delete(h.redisSubs, channel)
    }
    h.redisSubLock.Unlock()

    // 关闭所有连接
    for conn := range h.connections {
        conn.Close()
    }

    h.wg.Wait()
}
```

---

## 决策树

| 情况 | 方法 |
|-----------|----------|
| 单实例 | 跳过 ConnectionRegistry |
| 多实例 | 启用 ConnectionRegistry |
| 没有 channel 订阅者 | 从 Redis 延迟取消订阅 |
| 慢客户端 | 缓冲区溢出时关闭 |
| 需要消息历史 | 使用 Redis Streams + Pub/Sub |

---

## 相关技能

- **Meta-skill:** [ai/skills/meta/realtime-dashboard/](../../meta/realtime-dashboard/) —— 完整的实时仪表板指南
- [dual-stream-architecture](../dual-stream-architecture/) —— 事件发布
- [resilient-connections](../resilient-connections/) —— 连接弹性

---

## 切勿

- **切勿在 conn.send 上阻塞** —— 使用带 default 的 select 来检测溢出
- **切勿跳过优雅关闭** —— 客户端需要 close frames
- **切勿跨 channel 共享 pubsub** —— 每个 channel 需要自己的订阅
- **切勿忘记实例心跳** —— 死亡实例会留下孤立连接
- **切勿在没有 ping/pong 的情况下发送** —— 负载均衡器会关闭 "空闲" 连接
