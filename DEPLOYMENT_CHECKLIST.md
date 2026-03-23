# TrendRadar Zeabur 部署检查清单

## 部署前检查

### 环境准备
- [ ] Zeabur 账号已创建（zeabur.com）
- [ ] GitHub 账号已关联
- [ ] Docker Hub 镜像 `wantcat/trendradar:latest` 可拉取
- [ ] 本地 Git 工作树干净（无未提交改动）

### 项目检查
- [ ] 项目分支：`master`（当前：✓）
- [ ] 远程仓库：`origin/master`（当前：✓）
- [ ] Git 状态：干净（当前：✓）
- [ ] 配置文件：`config/config.yaml` 已准备
- [ ] 频词库：`config/frequency_words.txt` 已准备

### 认证与权限
- [ ] Zeabur 项目 ID 已获取（如未创建项目，从控制台创建）
- [ ] GitHub 仓库权限确认（tia923136-ai/TrendRadar）
- [ ] Zeabur CLI 已安装（可选，仅用于 CLI 部署）

---

## 部署方案选择

### 推荐路径

**首选：Zeabur Web 控制台** ✓
- 操作步骤：5-10 分钟
- 无需命令行
- 实时日志查看
- 推荐首次部署者

**备选：Zeabur CLI** （需安装 CLI）
- 操作步骤：3-5 分钟
- 支持自动化脚本
- 推荐频繁更新者

**可选：本地 Docker 测试**
- 预部署验证
- 环境配置测试
- 推荐开发者

---

## 部署步骤

### 方案 A：Web 控制台部署（推荐）

#### 第 1 步：创建服务
```
打开 https://zeabur.com/dashboard
→ 选择项目（或创建新项目 "TrendRadar"）
→ 点击 "Create Service"
→ 选择 "Docker"
```

#### 第 2 步：配置镜像
```
镜像地址：wantcat/trendradar:latest
镜像拉取：Always（每次部署拉取最新）
镜像源：Docker Hub
```

#### 第 3 步：配置环境变量
在 Zeabur 服务页面添加：

| 变量名 | 值 |
|-------|-----|
| RUN_MODE | cron |
| CRON_SCHEDULE | 0 */6 * * * |
| IMMEDIATE_RUN | true |
| ENABLE_WEBSERVER | true |
| WEBSERVER_PORT | 8080 |
| TZ | Asia/Shanghai |

#### 第 4 步：配置卷存储（可选）
```
路径 1：/app/config → 持久化存储
路径 2：/app/output → 持久化存储
```

#### 第 5 步：部署
```
点击 "Deploy"
等待容器启动（1-2 分钟）
查看 "Logs" 标签，确认无错误
```

#### 第 6 步：验证
```
检查日志：
  ✓ "Starting TrendRadar"
  ✓ "Cron scheduler initialized"
  ✓ "Task triggered at ..." (如 IMMEDIATE_RUN=true)
```

---

### 方案 B：CLI 部署（可选）

#### 前置：安装 CLI
```bash
# macOS
brew install zeabur/brew/zeabur

# 或 npm
npm install -g zeabur

# 验证
zeabur version
```

#### 第 1 步：登录
```bash
zeabur auth login
# 浏览器打开，完成认证后返回终端
```

#### 第 2 步：部署
```bash
cd /Users/tia/claude/02-📡信息中台/TrendRadar

zeabur deploy \
  --project <project-id> \
  --name trendradar \
  --image wantcat/trendradar:latest \
  --port 8080
```

#### 第 3 步：配置环境变量
```bash
zeabur service config set \
  --service trendradar \
  RUN_MODE=cron \
  CRON_SCHEDULE="0 */6 * * *" \
  IMMEDIATE_RUN=true \
  ENABLE_WEBSERVER=true \
  WEBSERVER_PORT=8080 \
  TZ=Asia/Shanghai
```

#### 第 4 步：验证
```bash
zeabur service logs trendradar -f
```

---

### 方案 C：本地验证（可选，推荐在部署前执行）

```bash
cd /Users/tia/claude/02-📡信息中台/TrendRadar

# 1. 创建 .env 文件
cat > .env << 'EOF'
RUN_MODE=cron
CRON_SCHEDULE=0 */6 * * *
IMMEDIATE_RUN=true
ENABLE_WEBSERVER=true
WEBSERVER_PORT=8080
TZ=Asia/Shanghai
EOF

# 2. 启动容器
docker run -d \
  --name trendradar-test \
  --env-file .env \
  -p 8080:8080 \
  -v $(pwd)/config:/app/config \
  -v $(pwd)/output:/app/output \
  wantcat/trendradar:latest

# 3. 查看日志（Ctrl+C 退出）
docker logs -f trendradar-test

# 4. 验证 Web 服务
curl http://localhost:8080

# 5. 清理
docker stop trendradar-test && docker rm trendradar-test
```

---

## 部署后验证

### 功能检查清单

| 项目 | 检查内容 | 通过 |
|------|---------|------|
| **容器状态** | Zeabur 显示 "Running" | [ ] |
| **日志检查** | 无 ERROR/CRITICAL 日志 | [ ] |
| **Web 服务** | 能访问 `https://<domain>.zeabur.app` | [ ] |
| **首次执行** | 日志中出现 "Task triggered at ..." | [ ] |
| **Cron 定时** | 确认 Supercronic 加载成功 | [ ] |
| **输出目录** | `/app/output` 有生成的报告文件 | [ ] |
| **配置加载** | `/app/config/config.yaml` 已加载 | [ ] |
| **时区** | 日志时间戳与 Asia/Shanghai 一致 | [ ] |

### 期望日志输出

```
2024-03-24 12:34:56,789 [INFO] Starting TrendRadar application
2024-03-24 12:34:57,890 [INFO] Cron scheduler initialized with schedule: 0 */6 * * *
2024-03-24 12:34:58,901 [INFO] Web server started on 0.0.0.0:8080
2024-03-24 12:35:00,012 [INFO] Task triggered at 2024-03-24 12:35:00
2024-03-24 12:35:01,123 [INFO] Processing trend analysis...
```

---

## 故障排查

### 常见问题快速索引

| 症状 | 原因 | 解决方案 |
|------|------|---------|
| 镜像拉取超时 | 网络问题或 Docker Hub 连接差 | 切换 Zeabur 地域，或检查镜像名是否正确 |
| 容器立即退出 | 启动命令错误或配置缺失 | 查看日志，确认 CRON_SCHEDULE 格式 |
| Cron 任务不执行 | 环境变量未生效 | 检查 CRON_SCHEDULE 和 RUN_MODE，重新部署 |
| Web 服务 502 | 端口配置错误或服务未启动 | 确认 ENABLE_WEBSERVER=true 和 WEBSERVER_PORT=8080 |
| 配置文件无法读取 | 卷未挂载或路径错误 | 通过 Zeabur 文件管理器验证文件存在 |
| 内存溢出（OOM） | 资源不足 | 扩展 CPU 和内存，或减少任务频率 |

详见 `DEPLOY_ZEABUR.md` 的"常见问题与排查"章节。

---

## 部署完成后

### 代码提交
```bash
git add DEPLOY_ZEABUR.md zeabur.json DEPLOYMENT_CHECKLIST.md
git commit -m "docs: add Zeabur deployment guides"
git push origin master
```

### 后续维护
- 监控 Zeabur 服务日志
- 定期检查 Cron 执行状况
- 根据需要调整 CRON_SCHEDULE
- 配置生效时间：修改环境变量后自动重启容器（3-5 秒）

### 相关文件路径
- **部署指南**：`/Users/tia/claude/02-📡信息中台/TrendRadar/DEPLOY_ZEABUR.md`
- **配置文件**：`/Users/tia/claude/02-📡信息中台/TrendRadar/zeabur.json`
- **本地测试**：`/Users/tia/claude/02-📡信息中台/TrendRadar/docker/.env`

---

**状态**：待部署
**最后更新**：2026-03-24
**部署者**：Claude Code
