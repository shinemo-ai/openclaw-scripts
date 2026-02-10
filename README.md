# openclaw-scripts

一组用于**用 Docker Compose 运行 OpenClaw（Gateway + CLI）**的便捷脚本，包含初始化目录、交互式配置（onboard）以及调用 CLI 的封装。

## 你会得到什么

- **本机目录映射**：默认把配置写到 `~/.openclaw`，把工作区映射到 `~/.openclaw/workspace`
- **Gateway 容器**：对外暴露端口 `18789`（gateway）与 `18790`（bridge）
- **CLI 容器**：用 `./openclaw-cli.sh ...` 像本地命令一样运行 `openclaw` 的 CLI

## 依赖

- **Docker + Docker Compose（`docker compose`）**
- **bash**
- 用于生成 token（可选其一）：
  - `openssl`，或
  - `python3`

首次克隆后如果脚本没有可执行权限，可以执行：

```bash
chmod +x ./set_up.sh ./start.sh ./openclaw-cli.sh
```

## 快速开始

### 0) 准备镜像

默认使用镜像 `openclaw:local`（可通过环境变量覆盖）：

- **使用本地镜像**：确保你的机器上存在 `openclaw:local`
- **使用远程镜像**：运行前设置 `OPENCLAW_IMAGE`

示例（仅示意）：

```bash
export OPENCLAW_IMAGE="openclaw:local"
```

### 1) 准备认证信息（按需）

`docker-compose.yml` 里预留了以下环境变量（从宿主机透传进容器）：

- `CLAUDE_AI_SESSION_KEY`
- `CLAUDE_WEB_SESSION_KEY`
- `CLAUDE_WEB_COOKIE`

你可以在当前 shell 里 `export`，也可以用 `.env` 文件加载（不要提交到 git）：

```bash
set -a
source .env
set +a
```

### 2) 交互式初始化（推荐先跑一次）

运行 `set_up.sh` 会：

- 创建并映射目录：
  - `OPENCLAW_CONFIG_DIR`（默认 `~/.openclaw`）
  - `OPENCLAW_WORKSPACE_DIR`（默认 `~/.openclaw/workspace`）
- 生成 `OPENCLAW_GATEWAY_TOKEN`（若你没提前设置）
- 在 CLI 容器里执行 `onboard --no-install-daemon`

```bash
./set_up.sh
```

脚本会提示你在交互过程中如何选择（例如 bind 选 `lan`、auth 选 `token` 等）。

### 3) 启动 Gateway

仓库里的 `docker-compose.yml` 已定义 `openclaw-gateway` 服务。推荐用 `up -d` 常驻启动：

```bash
docker compose up -d openclaw-gateway
```

查看日志：

```bash
docker compose logs -f openclaw-gateway
```

停止/清理：

```bash
docker compose down
```

### 4) 运行 CLI

用 `openclaw-cli.sh` 传参，会在容器里执行 `node dist/index.js <args...>`：

```bash
./openclaw-cli.sh --help
```

例如（仅示意，以实际 CLI 命令为准）：

```bash
./openclaw-cli.sh onboard --no-install-daemon
```

## 脚本说明

- `set_up.sh`
  - **用途**：首次/重新配置（交互式 onboard），并生成/导出 `OPENCLAW_GATEWAY_TOKEN`
- `openclaw-cli.sh`
  - **用途**：把 `docker compose run --rm openclaw-cli ...` 封装成一个本地命令
- `start.sh`
  - **用途**：当前实现会启动一个 CLI 容器的交互会话（脚本里打印了 “Starting gateway”，但实际运行的是 `openclaw-cli` 服务）。
  - **建议**：要常驻启动 gateway，请使用上面的 `docker compose up -d openclaw-gateway`

## 可配置环境变量

这些变量会影响脚本/compose 行为（括号内为默认值）：

- **镜像**
  - `OPENCLAW_IMAGE`（`openclaw:local`）
- **目录映射**
  - `OPENCLAW_CONFIG_DIR`（`$HOME/.openclaw`）
  - `OPENCLAW_WORKSPACE_DIR`（`$HOME/.openclaw/workspace`）
- **端口**
  - `OPENCLAW_GATEWAY_PORT`（`18789`）
  - `OPENCLAW_BRIDGE_PORT`（`18790`）
- **绑定方式**
  - `OPENCLAW_GATEWAY_BIND`（`lan`）
- **鉴权**
  - `OPENCLAW_GATEWAY_TOKEN`（强烈建议你自行设置为随机值并妥善保管）
- **认证信息透传（按需）**
  - `CLAUDE_AI_SESSION_KEY` / `CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`

## 安全提示

- **不要把** `OPENCLAW_GATEWAY_TOKEN`、`CLAUDE_*`、`CLAUDE_WEB_COOKIE` **提交到仓库**。
- 脚本会对 `OPENCLAW_CONFIG_DIR` / `OPENCLAW_WORKSPACE_DIR` 执行 `chmod 777` 以避免容器内权限问题；若你对权限有要求，建议在本机上调整为更严格的权限模型。

## 常见问题

- **提示 `Docker Compose not available`**
  - 需要 `docker compose version` 可用（Docker Desktop/新版本 Docker Engine 一般自带）
- **端口冲突**
  - 通过 `OPENCLAW_GATEWAY_PORT` / `OPENCLAW_BRIDGE_PORT` 改端口后再启动
