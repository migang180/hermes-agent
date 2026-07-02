# Hermes Agent 本地部署指南

> 基于源码 `f:/hermes/`，Docker Desktop 部署，DeepSeek 作为 LLM 后端。

---

## 环境

| 项 | 值 |
|---|---|
| 系统 | Windows 11 + Docker Desktop |
| 源码路径 | `f:/hermes/` |
| Docker 镜像 | `nousresearch/hermes-agent:latest` |
| LLM | DeepSeek (`deepseek-chat` / `deepseek-v4-pro`) |
| API Key | `DEEPSEEK_API_KEY=<your-deepseek-api-key>` |

---

## 快速启动

```powershell
# 1. 确保 Docker Desktop 在运行

# 2. 进入项目目录
cd f:\hermes

# 3. 启动 Gateway + Dashboard
docker compose -f docker-compose.windows.yml up -d

# 4. 查看状态
docker ps --filter "name=hermes"
```

启动后访问 Dashboard：**http://127.0.0.1:9119**

---

## 停止

```powershell
cd f:\hermes
docker compose -f docker-compose.windows.yml down
```

---

## 查看日志

```powershell
# Gateway 日志
docker logs hermes -f --tail 50

# Dashboard 日志
docker logs hermes-dashboard -f --tail 50
```

---

## 关键文件

| 文件 | 作用 |
|---|---|
| `f:/hermes/docker-compose.windows.yml` | Windows Docker Compose 配置 |
| `f:/hermes/.env` | API Key（docker-compose 自动加载） |
| `%USERPROFILE%/.hermes/config.yaml` | Hermes 主配置（模型、Provider 等） |
| `%USERPROFILE%/.hermes/skills/` | 自定义技能存放目录 |
| `%USERPROFILE%/.hermes/plugins/` | 自定义插件存放目录 |

---

## 当前配置

**Provider**: `deepseek`（原生支持，源码位于 `plugins/model-providers/deepseek/`）

**默认模型**: `deepseek-chat`（V3，非思考模式）

**可用模型列表**:
| 模型 | 说明 |
|---|---|
| `deepseek-chat` | V3，无思考模式 |
| `deepseek-reasoner` | R1，旧版推理 |
| `deepseek-v4-pro` | V4 Pro，支持推理 |
| `deepseek-v4-flash` | V4 Flash，支持推理 |

切换模型可以在 Dashboard 里操作，或修改 `%USERPROFILE%/.hermes/config.yaml`：

```yaml
model:
  default: deepseek-v4-pro   # 改这里
  provider: deepseek
```

---

## 更新上游源码

```powershell
cd f:\hermes

# 先停止容器
docker compose -f docker-compose.windows.yml down

# 拉取最新代码
git pull origin main

# 重新启动
docker compose -f docker-compose.windows.yml up -d
```

如果 `git pull` 有冲突，按照 `HERMES_DEV.md`（见下文）的策略处理。

---

## 二次开发建议

**不改源码**：自定义内容放进 `%USERPROFILE%/.hermes/` 下的 `skills/` 或 `plugins/`，Docker 卷已挂载，`git pull` 不受影响。

**必须改源码**：参考 `f:/hermes/HERMES_DEV.md` 了解 Git 分支/Fork/Worktree 策略。

---

## 服务架构

```
┌─────────────────────────────────────────┐
│  Docker Desktop                          │
│                                          │
│  ┌──────────────┐  ┌──────────────────┐ │
│  │   hermes     │  │ hermes-dashboard │ │
│  │  (gateway)   │  │   (Web UI)       │ │
│  │              │  │   :9119          │ │
│  └──────┬───────┘  └──────────────────┘ │
│         │                                │
│         │ DeepSeek API                   │
│         ▼                                │
│  api.deepseek.com                        │
└─────────────────────────────────────────┘
```

- **Gateway** — Agent 主进程，处理对话、工具调用、技能执行
- **Dashboard** — Web 管理界面，`http://127.0.0.1:9119`

---

## 故障排查

### 容器起不来

```powershell
docker compose -f docker-compose.windows.yml down
docker compose -f docker-compose.windows.yml up -d
```

### Dashboard 打不开

```powershell
docker logs hermes-dashboard --tail 30
```

看到 `HERMES_DASHBOARD_READY` 就说明好了，如果没有，等几秒再试。

### API Key 不对

确认 `f:/hermes/.env` 里的 Key 和 `docker-compose.windows.yml` 中 `DEEPSEEK_API_KEY` 环境变量一致。

---

*最后更新: 2026-07-02*
