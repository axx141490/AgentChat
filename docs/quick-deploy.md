# AgentChat 快速部署教程

约 **5 分钟** 完成本地 Docker 部署。

---

## 一、环境要求

- **Docker** 与 **Docker Compose**（或 Docker Desktop 自带 Compose）
- 内存建议 **4GB+**，磁盘 **10GB+**

---

## 二、部署步骤

### 1. 进入项目目录

```bash
cd /path/to/AgentChat   # 替换为你的项目路径
```

### 2. 准备环境变量

```bash
# 若没有 docker.env，从模板复制
cp docker/docker.env.example docker/docker.env

# 可选：编辑 docker.env 填写 API 密钥（不填也可先跑起来）
# OPENAI_API_KEY=sk-xxx
# QWEN_API_KEY=sk-xxx
```

### 3. 启动所有服务

```bash
docker compose -f docker/docker-compose.yml --env-file docker/docker.env up -d --build
```

首次会拉镜像并构建，约 3–10 分钟。之后启动约 30 秒。

### 4. 仅启动核心服务（可选）

若网络不稳定导致前端构建失败，可先只起后端与数据库：

```bash
docker compose -f docker/docker-compose.yml --env-file docker/docker.env up -d mysql redis backend
```

前端稍后再单独构建并启动：

```bash
docker compose -f docker/docker-compose.yml build frontend
docker compose -f docker/docker-compose.yml --env-file docker/docker.env up -d frontend
```

---

## 三、访问地址

| 服务       | 地址                          |
|------------|-------------------------------|
| 前端界面   | http://localhost:8090         |
| 后端 API   | http://localhost:7860         |
| API 文档   | http://localhost:7860/docs    |
| 健康检查   | http://localhost:7860/health  |

首次使用请在前端 **注册** 账号后再登录。

---

## 四、常用命令

```bash
# 查看运行状态
docker compose -f docker/docker-compose.yml ps

# 查看后端日志
docker compose -f docker/docker-compose.yml logs -f backend

# 重启后端
docker compose -f docker/docker-compose.yml restart backend

# 停止所有服务
docker compose -f docker/docker-compose.yml down
```

---

## 五、常见问题

### 1. 注册/登录提示「请检查网络连接」

- **原因**：前端请求未发到后端。  
- **处理**：确认 `src/frontend/src/utils/request.ts` 中 `baseURL` 使用 `import.meta.env.VITE_API_BASE_URL`（构建时已设为 `http://localhost:7860`），并已重新构建前端。

### 2. 浏览器报 CORS 错误

- **原因**：后端未允许前端源。  
- **处理**：后端已在 `main.py` 中为 `http://localhost:8090` 配置 CORS，重启后端即可：  
  `docker compose -f docker/docker-compose.yml restart backend`

### 3. 注册返回 500「系统错误，请重试」

- **原因**：后端连不上 MySQL（例如在 Docker 里仍用 `localhost`）。  
- **处理**：确保 `docker-compose.yml` 中 backend 有环境变量 `DATABASE_URL=mysql://agentchat_user:123456@mysql:3306/agentchat`，且 `settings.py` 中已支持用 `DATABASE_URL` 覆盖配置。重启后端后再试注册。

### 4. 前端构建时 npm install 失败（ECONNRESET）

- **原因**：网络不稳定。  
- **处理**：多试几次，或换网络/镜像后再执行：  
  `docker compose -f docker/docker-compose.yml build frontend`

### 5. 端口被占用

修改 `docker/docker-compose.yml` 中对应服务的 `ports`，例如：

```yaml
backend:
  ports:
    - "17860:7860"   # 宿主机端口可改
frontend:
  ports:
    - "18090:8090"
```

---

## 六、生产环境简要说明

- 使用生产配置：  
  `docker compose -f docker/docker-compose.yml -f docker/docker-compose.prod.yml up -d`
- 务必在 `docker.env` 中修改 `JWT_SECRET_KEY`、`MYSQL_PASSWORD` 等敏感配置。
- 对外暴露时建议前加 Nginx 并配置 HTTPS。

更多细节见 [docker/README.md](../docker/README.md) 与项目主 [README.md](../README.md)。
