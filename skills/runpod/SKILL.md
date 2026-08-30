---
name: runpod
description: 管理RunPod GPU云实例——创建、启动、停止、通过SSH和API连接Pod。适用于RunPod基础设施、GPU实例工作，或需要SSH访问远程GPU机器时。处理Pod生命周期、SSH代理连接、文件系统挂载和API查询。需安装runpodctl（brew install runpod/runpodctl/runpodctl）。
---

# RunPod Skill

管理 RunPod GPU cloud instances, SSH connections, and filesystem access.

## 先决条件

```bash
brew install runpod/runpodctl/runpodctl
runpodctl config --apiKey "your-api-key"
```

**SSH Key:** runpodctl manages SSH keys in `~/.runpod/ssh/`:

```bash
runpodctl ssh add-key
```

查看 and 管理 keys at: https://console.runpod.io/user/settings

**Mount script 配置:**
The mount script checks `~/.ssh/runpod_key` first, then falls back 迁移到 runpodctl's default key. Override with:
```bash
export RUNPOD_SSH_KEY="$HOME/.runpod/ssh/RunPod-Key"
```

Host keys are stored separately in `~/.runpod/ssh/known_hosts` (isolated from your main SSH config). Uses `StrictHostKeyChe`StrictHostKeyChe`king=accept-new`on reconnect while allowing new RunPod instances.

## Quick 参考

```bash
runpodctl get pod                    # List pods
runpodctl get pod <id>               # Get pod details
runpodctl start pod <id>             # Start pod
runpodctl stop pod <id>              # Stop pod
runpodctl ssh connect <id>           # Get SSH command
runpodctl send <file>                # Send file to pod
runpodctl receive <code>             # Receive file from pod
```

## Common Operations

### 创建 Pod

```bash
# Without volume
runpodctl create pod --name "my-pod" --gpuType "NVIDIA GeForce RTX 4090" --imageName "runpod/pytorch:1.0.2-cu1281-torch280-ubuntu2404"

# With volume (100GB at /workspace)
runpodctl create pod --name "my-pod" --gpuType "NVIDIA GeForce RTX 4090" --imageName "runpod/pytorch:1.0.2-cu1281-torch280-ubuntu2404" --volumeSize 100 --volumePath "/workspace"
```

**重要:** When using a volume (`--volumeSize`), always specify `--v`--v`lumePath`. Without it:
```
error creating container: ... invalid mount config for type "volume": field Target must not be empty
```

### SSH 迁移到 Pod

```bash
# Get SSH command
runpodctl ssh connect <pod_id>

# Connect directly (copy command from above)
ssh -p <port> root@<ip> -i ~/.ssh/runpod_key
```

### Mount Pod Filesystem (SSHFS)

```bash
./scripts/mount_pod.sh <pod_id> [base_dir]
```

Mounts pod 迁移到 `~/pods/<pod_id>` by default.

**Access files:**
```bash
ls ~/pods/<pod_id>/
cat ~/pods/<pod_id>/workspace/my-project/train.py
```

**Unmount:**
```bash
fusermount -u ~/pods/<pod_id>
```

## Helper Script

| Script | Purpose |
|--------|---------|
| `mount_pod.sh` | Mount pod filesystem via SSHFS (no runpodctl equivalent) |

## Web Service Access

Proxy URLs:
```
https://<pod_id>-<port>.proxy.runpod.net
```

Common ports:
- **8188**: ComfyUI
- **7860**: Gradio
- **8888**: Jupyter
- **8080**: Dev tools