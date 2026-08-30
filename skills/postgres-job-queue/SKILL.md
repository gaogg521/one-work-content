---
name: postgres-job-queue
model: standard
description: 基于 PostgreSQL 的 job queue，带有 priority scheduling、batch claiming 和 progress tracking。在构建无外部依赖的 job queue 时使用。触发词：PostgreSQL job queue、background jobs、task queue、priority queue、SKIP LOCKED。
tags:
- PostgreSQL
- RabbitMQ
---

# PostgreSQL Job Queue

使用 PostgreSQL 的生产级 job queue，带有 priority scheduling、batch claiming 和 progress tracking。

---

## 何时使用

- 需要 job queue 但想避免 Redis/RabbitMQ 依赖
- Jobs 需要基于 priority 的调度
- 长时间运行的 jobs 需要 progress visibility
- Jobs 应该在服务重启后存活

---

## Schema 设计

```sql
CREATE TABLE jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_type VARCHAR(50) NOT NULL,
    priority INT NOT NULL DEFAULT 100,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    data JSONB NOT NULL DEFAULT '{}',
    
    -- Progress tracking
    progress INT DEFAULT 0,
    current_stage VARCHAR(100),
    events_count INT DEFAULT 0,
    
    -- Worker tracking
    worker_id VARCHAR(100),
    claimed_at TIMESTAMPTZ,
    
    -- Timing
    created_at TIMESTAMPTZ DEFAULT NOW(),
    started_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ,
    
    -- Retry handling
    attempts INT DEFAULT 0,
    max_attempts INT DEFAULT 3,
    last_error TEXT,
    
    CONSTRAINT valid_status CHECK (
        status IN ('pending', 'claimed', 'running', 'completed', 'failed', 'cancelled')
    )
);

-- Critical: Partial index for fast claiming
CREATE INDEX idx_jobs_claimable ON jobs (priority DESC, created_at ASC) 
    WHERE status = 'pending';
CREATE INDEX idx_jobs_worker ON jobs (worker_id) 
    WHERE status IN ('claimed', 'running');
```

---

## 使用 SKIP LOCKED 的批量 Claim

```sql
CREATE OR REPLACE FUNCTION claim_job_batch(
    p_worker_id VARCHAR(100),
    p_job_types VARCHAR(50)[],
    p_batch_size INT DEFAULT 10
) RETURNS SETOF jobs AS $$
BEGIN
    RETURN QUERY
    WITH claimable AS (
        SELECT id
        FROM jobs
        WHERE status = 'pending'
          AND job_type = ANY(p_job_types)
          AND attempts < max_attempts
        ORDER BY priority DESC, created_at ASC
        LIMIT p_batch_size
        FOR UPDATE SKIP LOCKED  -- Critical: 跳过被锁定的行
    ),
    claimed AS (
        UPDATE jobs
        SET status = 'claimed',
            worker_id = p_worker_id,
            claimed_at = NOW(),
            attempts = attempts + 1
        WHERE id IN (SELECT id FROM claimable)
        RETURNING *
    )
    SELECT * FROM claimed;
END;
$$ LANGUAGE plpgsql;
```

---

## Go 实现

```go
const (
    PriorityExplicit   = 150  // 用户请求的
    PriorityDiscovered = 100  // 系统发现的
    PriorityBackfill   = 30   // 后台回填
)

type JobQueue struct {
    db       *pgx.Pool
    workerID string
}

func (q *JobQueue) Claim(ctx context.Context, types []string, batchSize int) ([]Job, error) {
    rows, err := q.db.Query(ctx,
        "SELECT * FROM claim_job_batch($1, $2, $3)",
        q.workerID, types, batchSize,
    )
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var jobs []Job
    for rows.Next() {
        var job Job
        if err := rows.Scan(&job); err != nil {
            return nil, err
        }
        jobs = append(jobs, job)
    }
    return jobs, nil
}

func (q *JobQueue) Complete(ctx context.Context, jobID uuid.UUID) error {
    _, err := q.db.Exec(ctx, `
        UPDATE jobs 
        SET status = 'completed',
            progress = 100,
            completed_at = NOW()
        WHERE id = $1`,
        jobID,
    )
    return err
}

func (q *JobQueue) Fail(ctx context.Context, jobID uuid.UUID, errMsg string) error {
    _, err := q.db.Exec(ctx, `
        UPDATE jobs 
        SET status = CASE 
                WHEN attempts >= max_attempts THEN 'failed' 
                ELSE 'pending' 
            END,
            last_error = $2,
            worker_id = NULL,
            claimed_at = NULL
        WHERE id = $1`,
        jobID, errMsg,
    )
    return err
}
```

---

## 过期 Job 恢复

```go
func (q *JobQueue) RecoverStaleJobs(ctx context.Context, timeout time.Duration) (int, error) {
    result, err := q.db.Exec(ctx, `
        UPDATE jobs 
        SET status = 'pending',
            worker_id = NULL,
            claimed_at = NULL
        WHERE status IN ('claimed', 'running')
          AND claimed_at < NOW() - $1::interval
          AND attempts < max_attempts`,
        timeout.String(),
    )
    if err != nil {
        return 0, err
    }
    return int(result.RowsAffected()), nil
}
```

---

## 决策树

| 场景 | 方案 |
|----------|----------|
| 需要保证投递 | PostgreSQL queue |
| 需要亚毫秒级延迟 | 改用 Redis |
| < 1000 jobs/sec | PostgreSQL 足够 |
| > 10000 jobs/sec | 添加 Redis 层 |
| 需要严格排序 | 每类型单 worker |

---

## 相关 Skills

- **Related:** [service-layer-architecture](../service-layer-architecture/) —— job handlers 的 Service patterns
- **Related:** [realtime/dual-stream-architecture](../../realtime/dual-stream-architecture/) —— 来自 jobs 的 Event publishing

---

## 绝对不要做的事

- **绝对不要先 SELECT 再 UPDATE** —— 竞态条件。使用 SKIP LOCKED。
- **绝对不要不带 SKIP LOCKED 就 claim** —— Workers 会死锁。
- **绝对不要存储大负载** —— 只存储引用。
- **绝对不要忘记 partial index** —— 没有它，claim 会很慢。
