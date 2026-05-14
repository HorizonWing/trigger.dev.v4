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

