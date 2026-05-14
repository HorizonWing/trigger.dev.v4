# Trigger.dev v4 自托管部署指南

## 文件说明
- `docker-compose.yml` — 完整单机部署配置（含全部组件）
- `.env` — 环境变量配置（**必须修改标注项目**）
- `registry/auth.htpasswd` — 镜像仓库认证文件

## 部署前准备

### 1. 生成随机密钥
```bash
openssl rand -hex 32  # 运行3次，分别填入 SESSION_SECRET、MAGIC_LINK_SECRET、ENCRYPTION_KEY
```

### 2. 修改 .env
必须修改以下项目：
- APP_ORIGIN → 你的服务器 IP 或域名
- SESSION_SECRET / MAGIC_LINK_SECRET / ENCRYPTION_KEY
- POSTGRES_PASSWORD
- REGISTRY_PASSWORD
- OBJECT_STORE_ACCESS_KEY / OBJECT_STORE_SECRET_KEY

### 3. 更新镜像仓库密码
```bash
# 安装工具
sudo apt install apache2-utils -y

# 生成新的 htpasswd（替换 registry/auth.htpasswd）
htpasswd -Bbn registry-user 你的新密码 > registry/auth.htpasswd

# 同步修改 .env 中的 REGISTRY_PASSWORD
```

## 启动

```bash
# 首次启动
docker compose up -d

# 查看日志（获取首次登录的魔法链接）
docker compose logs -f webapp

# 登录内置镜像仓库（每台部署机器执行一次）
docker login -u registry-user localhost:5000
```

## 访问地址
- **Trigger.dev 仪表板**: http://你的IP:8030
- **MinIO 管理界面**: http://你的IP:9001（用 OBJECT_STORE_ACCESS_KEY/SECRET_KEY 登录）
- **镜像仓库**: localhost:5000（仅内部使用）

## 在项目中连接自托管实例

```bash
# 登录
npx trigger.dev@latest login -a http://你的IP:8030 --profile self-hosted

# 初始化项目
npx trigger.dev@latest init -p <project-ref> -a http://你的IP:8030

# 开发模式
npx trigger.dev@latest dev --profile self-hosted

# 部署
npx trigger.dev@latest deploy --profile self-hosted
```

## 升级版本

```bash
# 修改 .env 中的 TRIGGER_IMAGE_TAG=v4.x.x
# 然后重启
docker compose pull
docker compose up -d

# 同步更新 SDK
npx trigger.dev@latest update
```

## 端口说明
| 端口 | 服务 | 说明 |
|------|------|------|
| 8030 | Webapp | 主界面，对外开放 |
| 5000 | Registry | 镜像仓库，仅本机使用 |
| 9000 | MinIO API | 对象存储，仅内部 |
| 9001 | MinIO 管理界面 | 建议关闭公网访问 |

## Dokploy 使用说明
1. 在 Dokploy 创建 Docker Compose 服务
2. 将 docker-compose.yml 内容粘贴进去
3. 在 Environment 标签页填入 .env 中的所有变量
4. 上传 registry/auth.htpasswd 或通过 Volumes 挂载
5. 点击 Deploy

以下是 Trigger.dev v4 Self-hosting Limits 环境变量的完整中文翻译：

## 任务 Payload 限制

| 环境变量 | 必填 | 默认值 | 说明 |
|---|---|---|---|
| `TASK_PAYLOAD_OFFLOAD_THRESHOLD` | 否 | 524288（512KB） | 任务 Payload 超过此大小后将卸载至 S3 |
| `TASK_PAYLOAD_MAXIMUM_SIZE` | 否 | 3145728（3MB） | 单个任务 Payload 的最大体积 |
| `BATCH_TASK_PAYLOAD_MAXIMUM_SIZE` | 否 | 1000000（1MB） | 批量任务 Payload 的最大体积 |
| `TASK_RUN_METADATA_MAXIMUM_SIZE` | 否 | 262144（256KB） | 任务 Run 元数据的最大体积 |
| `MAX_BATCH_V2_TRIGGER_ITEMS` | 否 | 500 | 单次批量触发的最大条目数（旧版 v2 API） |
| `STREAMING_BATCH_MAX_ITEMS` | 否 | 1000 | 流式批量的最大条目数（v3 API，需要 SDK 4.3.1+） |
| `STREAMING_BATCH_ITEM_MAXIMUM_SIZE` | 否 | 3145728（3MB） | 流式批量中每条条目的最大体积 |
| `MAXIMUM_DEV_QUEUE_SIZE` | 否 | — | 开发环境队列的最大长度 |
| `MAXIMUM_DEPLOYED_QUEUE_SIZE` | 否 | — | 生产部署队列的最大长度 |

## OTel（OpenTelemetry）限制

| 环境变量 | 必填 | 默认值 | 说明 |
|---|---|---|---|
| `TRIGGER_OTEL_SPAN_ATTRIBUTE_COUNT_LIMIT` | 否 | 1024 | Span 的最大属性数量 |
| `TRIGGER_OTEL_LOG_ATTRIBUTE_COUNT_LIMIT` | 否 | 1024 | Log 的最大属性数量 |
| `TRIGGER_OTEL_SPAN_ATTRIBUTE_VALUE_LENGTH_LIMIT` | 否 | 131072 | Span 属性值的最大长度 |
| `TRIGGER_OTEL_LOG_ATTRIBUTE_VALUE_LENGTH_LIMIT` | 否 | 131072 | Log 属性值的最大长度 |
| `TRIGGER_OTEL_SPAN_EVENT_COUNT_LIMIT` | 否 | 10 | Span 的最大事件数量 |
| `TRIGGER_OTEL_LINK_COUNT_LIMIT` | 否 | 2 | Span 的最大 Link 数量 |
| `TRIGGER_OTEL_ATTRIBUTE_PER_LINK_COUNT_LIMIT` | 否 | 10 | 每个 Link 的最大属性数量 |
| `TRIGGER_OTEL_ATTRIBUTE_PER_EVENT_COUNT_LIMIT` | 否 | 10 | 每个 Event 的最大属性数量 |
| `SERVER_OTEL_SPAN_ATTRIBUTE_VALUE_LENGTH_LIMIT` | 否 | 8192 | 服务端 OTel Span 属性值的最大长度 |
