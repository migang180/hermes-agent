# Hermes Agent —— Windows 原生部署指南

> 🖥️ **本指南针对原生 Windows 部署**（直通本机，能读写文件、跑 PowerShell）。
> 如果你用 Docker 方案，请看 [HERMES_DOCKER_GUIDE.md](HERMES_DOCKER_GUIDE.md)。

---

## 部署概况

| 项 | 值 |
|---|---|
| 安装方式 | conda 环境 + 源码可编辑安装（`pip install -e .`） |
| 源码目录 | `f:\hermes` |
| conda 环境名 | `hermes`（位于 `F:\Anaconda3\envs\hermes`） |
| Python | 3.12 |
| 数据/配置目录 | `F:\hermes-data`（由环境变量 `HERMES_HOME` 指定） |
| LLM | DeepSeek（`deepseek-chat`） |
| C 盘占用 | 无（程序与数据全在 F 盘） |

---

## 🚀 启动

打开 **Anaconda Prompt**（或 PowerShell / VSCode 终端），两步：

```powershell
conda activate hermes
hermes
```

进入交互聊天界面，即可使用。此时 Hermes 直通你的 Windows 本机：
- 让它"把文件存到桌面" → 真存到你桌面
- 让它"跑个 PowerShell 命令" → 真在你本机跑
- 读写任意目录、管理系统，和 Claude Code 体验一致

> `HERMES_HOME=F:\hermes-data` 已用 `setx` 写进系统环境变量，新开终端会自动认，无需每次手设。

---

## 🌐 启动 Web 界面（前端 Dashboard）

如果想要浏览器版的界面（和之前 Docker 版同一个 dashboard，但更省心），另开一个终端：

```powershell
conda activate hermes
hermes dashboard
```

启动后浏览器访问：**http://127.0.0.1:9119**

> ⏱️ **首次启动**会现场编译前端（约 20–30 秒，看到 `HERMES_DASHBOARD_READY port=9119` 即就绪），之后再起就是秒开。
>
> 🔓 **不用登录**。原生版默认绑 `127.0.0.1`（loopback），自动绕过 auth 闸门——Docker 版那个"绑 0.0.0.0 → 触发 auth → 配账号密码 → 还 500"的破事这里完全没有。浏览器打开直接用。
>
> 🛑 这个终端窗口要**保持开着**，关掉 = Web 界面关闭。用 `Ctrl+C` 停。

Web 界面和终端 `hermes` 共享同一份配置/会话/记忆（都在 `F:\hermes-data`），两边随便用。

### 想开机自启 / 后台常驻

`hermes dashboard` 默认前台跑。要后台常驻可加 `--no-open` 并用 PowerShell 后台作业或计划任务，例如：

```powershell
Start-Job { conda activate hermes; hermes dashboard --no-open }
```

（日常用不必这么搞，需要时再起即可。）

---

## 常用命令

```powershell
# 先激活环境（每个新终端都要）
conda activate hermes

hermes                          # 交互聊天
hermes -z "你的问题"             # 一次性提问，不进交互
hermes model                    # 切模型 / provider（交互菜单）
hermes doctor                   # 自检环境
hermes tools                    # 配置工具开关
hermes config show              # 查看当前配置
hermes config edit              # 用编辑器打开 config.yaml
hermes dashboard                # 启动本地 Web 界面（可选）
hermes update                   # 升级 Hermes（注意：会拉官方源码，见下）
```

---

## 切换模型

DeepSeek 可用模型：`deepseek-chat`（默认）、`deepseek-reasoner`、`deepseek-v4-pro`、`deepseek-v4-flash`。

方式一：交互菜单
```powershell
hermes model
```

方式二：直接改 `F:\hermes-data\config.yaml`
```yaml
model:
  default: deepseek-v4-pro   # 改这里
  provider: deepseek
```

---

## 关键文件位置

| 文件 | 作用 |
|---|---|
| `F:\hermes-data\config.yaml` | 主配置（模型、provider 等） |
| `F:\hermes-data\.env` | API Key（`DEEPSEEK_API_KEY`） |
| `F:\hermes-data\sessions\` | 会话历史 |
| `F:\hermes-data\memories\` | 记忆库 |
| `F:\hermes-data\skills\` | 自定义技能 |
| `f:\hermes\` | Hermes 源码（可编辑安装，改了立即生效） |

---

## 更新

⚠️ **注意**：本部署是从源码可编辑安装的，`hermes update` 默认拉官方仓库，可能和你本地改动冲突。推荐两种方式：

**方式 A：用 git 拉取（推荐，可控）**
```powershell
cd f:\hermes
git stash              # 暂存你的本地改动（如 docker 配置）
git pull upstream main # 拉官方最新
git stash pop          # 恢复本地改动
# 依赖若有变化，重装：
conda activate hermes
pip install -e .
```

**方式 B：重新装依赖**
```powershell
conda activate hermes
cd f:\hermes
pip install -e .
```

---

## 与 Docker 方案的关系

两套独立、互不干扰：

- **原生版（本指南）**：直通本机，日常使用推荐
- **Docker 版**（`docker-compose.windows.yml`）：隔离容器，已停用，文件保留备用

两者**不要同时跑**，避免双开混淆。

---

## 故障排查

### `hermes` 命令找不到
没激活环境。先 `conda activate hermes`。

### `CondaError: Run 'conda init' before 'conda activate'`
PowerShell（或 cmd）第一次用 conda 要先初始化，**一次性操作**：
```powershell
conda init powershell      # 当前用的是 PowerShell
# conda init cmd.exe       # 用 cmd 的话跑这条
```
然后**关掉当前终端、重开一个**（必须重开，初始化才会生效）。之后再 `conda activate hermes` 就正常。已为你执行过，除非换新终端用户/新机器，否则不用再跑。

### 中文/特殊字符乱码
PowerShell 里设 UTF-8：
```powershell
$env:PYTHONUTF8=1
$env:PYTHONIOENCODING="utf-8"
```

### DeepSeek 报 API key 错误
检查 `F:\hermes-data\.env` 里 `DEEPSEEK_API_KEY=` 后面有没有正确的 key。

### 想把数据目录移到别处
改 `HERMES_HOME` 环境变量：
```powershell
setx HERMES_HOME "新的路径"
```
然后重启终端。原 `F:\hermes-data` 整个文件夹可以搬过去。

---

*最后更新: 2026-07-02*
