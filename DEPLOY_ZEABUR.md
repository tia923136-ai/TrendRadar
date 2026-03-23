# TrendRadar Zeabur 部署指南

## 部署信息速查表

| 项目 | 值 |
|------|-----|
| 项目名 | TrendRadar |
| Docker 镜像 | `wantcat/trendradar:latest` |
| 镜像源 | Docker Hub（wantcat） |
| 部署方式 | Docker 镜像（预构建） |
| 目标平台 | Zeabur |
| GitHub 仓库 | tia923136-ai/TrendRadar |
| 当前分支 | master |

## 环境变量配置

部署到 Zeabur 时，在服务设置中添加以下环境变量：

```env
# 运行模式
RUN_MODE=cron

# 定时任务（每6小时执行）
CRON_SCHEDULE=0 */6 * * *

# 启动时立即执行一次
IMMEDIATE_RUN=true

# Web 服务
ENABLE_WEBSERVER=true
WEBSERVER_PORT=8080

# 时区
TZ=Asia/Shanghai
```

### 可选环境变量

如需启用通知和 AI 功能，添加以下变量到 Zeabur 服务设置：

```env
# 飞书通知（可选）
FEISHU_WEBHOOK_URL=https://open.feishu.cn/open-apis/bot/v2/hook/...

# Telegram 通知（可选）
TELEGRAM_BOT_TOKEN=<bot_token>
TELEGRAM_CHAT_ID=<chat_id>

# AI 分析（可选）
AI_ANALYSIS_ENABLED=false
AI_API_KEY=<api_key>
AI_MODEL=deepseek/deepseek-chat
```

## 快速部署步骤

### 方案 A：Zeabur Web 控制台（推荐首次部署）

1. **登录 Zeabur**
   - 打开 https://zeabur.com/dashboard
   - 登录或创建账号

2. **创建新服务**
   - 点击 "Create Service"
   - 选择 "Docker" 选项
   - 选择 "Pre-built Image"

3. **配置镜像**
   - 镜像地址：`wantcat/trendradar:latest`
   - 拉取策略：`Always`（每次部署拉取最新镜像）

4. **添加环境变量**
   - 在 "Environment Variables" 中添加上述 `env` 配置
   - 点击 "Save"

5. **配置网络和存储（可选）**
   - 如需持久化 `/app/config` 和 `/app/output`，创建卷挂载
   - 启用 "Public URL" 以访问 Web 服务

6. **部署**
   - 点击 "Deploy"
   - 等待容器启动（1-2 分钟）
   - 查看 "Logs" 确认启动无误

### 方案 B：Zeabur CLI（推荐自动化场景）

#### 1. 安装 Zeabur CLI

```bash
# macOS
brew install zeabur/brew/zeabur

# 或通过 npm
npm install -g zeabur

# 验证
zeabur version
```

#### 2. 认证

```bash
zeabur auth login
# 浏览器打开 Zeabur 登录页面，完成认证后返回终端
```

#### 3. 列出项目

```bash
zeabur project list
# 记录项目 ID（如果没有，通过 Zeabur Web 创建）
```

#### 4. 部署服务

```bash
cd /Users/tia/claude/02-📡信息中台/TrendRadar

# 首次部署
zeabur deploy \
  --project <project-id> \
  --name trendradar \
  --image wantcat/trendradar:latest \
  --port 8080

# 添加环境变量
zeabur service config set \
  --service trendradar \
  RUN_MODE=cron \
  CRON_SCHEDULE="0 */6 * * *" \
  IMMEDIATE_RUN=true \
  ENABLE_WEBSERVER=true \
  WEBSERVER_PORT=8080 \
  TZ=Asia/Shanghai
```

#### 5. 验证部署

```bash
# 查看服务状态
zeabur service list --project <project-id>

# 查看实时日志
zeabur service logs trendradar --project <project-id> -f

# 查看服务详情
zeabur service inspect trendradar --project <project-id>
```

## 本地验证（部署前可选）

如想在本地测试部署配置，可使用 Docker Compose：

```bash
cd /Users/tia/claude/02-📡信息中台/TrendRadar

# 创建 .env 文件
cat > .env << 'EOF'
RUN_MODE=cron
CRON_SCHEDULE=0 */6 * * *
IMMEDIATE_RUN=true
ENABLE_WEBSERVER=true
WEBSERVER_PORT=8080
TZ=Asia/Shanghai
EOF

# 启动容器
docker run -d \
  --name trendradar-test \
  --env-file .env \
  -p 8080:8080 \
  -v $(pwd)/config:/app/config \
  -v $(pwd)/output:/app/output \
  wantcat/trendradar:latest

# 查看日志
docker logs -f trendradar-test

# 验证 Web 服务
curl http://localhost:8080

# 清理
docker stop trendradar-test && docker rm trendradar-test
```

## 部署后验证检查清单

部署完成后，按以下步骤验证：

- [ ] **容器状态**：Zeabur 控制台显示 "Running"
- [ ] **日志检查**：无 ERROR 或 CRITICAL 日志
- [ ] **Web 服务**：能访问 `https://<service-domain>.zeabur.app`
- [ ] **首次执行**：日志中出现 "Task triggered at ..." 信息
- [ ] **Cron 任务**：等待 6 小时验证定时执行，或查看日志中的 Supercronic 确认
- [ ] **输出目录**：Zeabur 文件管理器中 `/app/output` 有生成的报告
- [ ] **配置文件**：`/app/config/config.yaml` 和 `/app/frequency_words.txt` 已加载

## 常见问题与排查

### Q: 镜像拉取失败或超时

**症状**：部署卡在 "Pulling image" 或显示 timeout

**解决方案**：
1. 检查 Zeabur 所在区域网络连接
2. 尝试切换 Zeabur 地域（如从香港切到新加坡）
3. 确认 Docker Hub 镜像存在：`docker pull wantcat/trendradar:latest`

### Q: Cron 任务不执行

**症状**：日志中没有 "Task triggered at ..."

**排查步骤**：
1. 验证 CRON_SCHEDULE 格式（5 字段 Cron 表达式）
2. 查看 Supercronic 错误日志
3. 检查容器时区是否正确（TZ=Asia/Shanghai）
4. 查看 `/app/config/config.yaml` 是否正确加载

```bash
# 在 Zeabur 中通过容器终端验证 Cron 表达式
docker exec <container-id> /usr/local/bin/supercronic -test /app/crontab
```

### Q: Web 服务端口无法访问

**症状**：访问服务 URL 返回 502 或超时

**排查步骤**：
1. 确认 ENABLE_WEBSERVER=true
2. 检查 WEBSERVER_PORT 是否为 8080
3. 查看容器日志中 Web 服务启动日志
4. 确认 Zeabur 的 "Public URL" 已启用

### Q: 配置文件修改无法生效

**症状**：修改 config.yaml 后，容器读取的仍是旧配置

**解决方案**：
1. 通过 Zeabur 文件管理器直接修改 `/app/config/config.yaml`
2. 或重新部署：`zeabur redeploy trendradar`
3. 检查容器是否挂载了该文件（如已挂载，修改只对新容器生效）

### Q: 内存或 CPU 使用过高

**症状**：Zeabur 显示容器 OOM（Out of Memory）或 CPU 持续 100%

**解决方案**：
1. 在 Zeabur 控制台扩展资源（CPU/Memory）
2. 减少 CRON_SCHEDULE 频率（如改为每 12 小时）
3. 检查 AI_ANALYSIS 是否启用（消耗资源）
4. 查看日志中是否有内存泄漏

## 更新部署

### 更新镜像版本

```bash
# 通过 CLI
zeabur redeploy trendradar --project <project-id>

# 通过 Web 控制台
# 进入服务设置 → 更改镜像版本（如从 latest 改为特定版本号）
# 点击 "Deploy" 触发重新部署
```

### 更新环境变量

```bash
# 通过 CLI
zeabur service config set \
  --service trendradar \
  CRON_SCHEDULE="0 0,12 * * *"  # 改为每天 00:00 和 12:00 执行

# 通过 Web 控制台
# 直接修改 "Environment Variables" 并保存，自动重启容器
```

## 相关文件

- **本地配置**：`docker/.env` - Docker Compose 本地测试用
- **Docker 镜像构建**：`docker/Dockerfile` - 镜像构建配置
- **启动脚本**：`docker/entrypoint.sh` - 容器启动脚本
- **项目配置**：`config/config.yaml` - TrendRadar 业务配置
- **频词库**：`config/frequency_words.txt` - 趋势词过滤库

## 支持与反馈

如遇到部署问题，可查阅：
1. Zeabur 官方文档：https://docs.zeabur.com/
2. TrendRadar GitHub Issues：https://github.com/tia923136-ai/TrendRadar/issues
3. 容器日志详情：Zeabur 控制台 → Service → Logs

---

**最后更新**：2026-03-24
**部署状态**：待部署
