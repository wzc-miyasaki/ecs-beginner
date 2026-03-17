# ECS 前后端分离 Demo - 本地运行指南

## 快速启动（3 步）

### 1. 安装依赖

```bash
cd /Users/zec/Repo/ecs-beginner
source .venv/bin/activate
uv pip install fastapi uvicorn
```

### 2. 启动后端 API

```bash
# 新开终端 1
cd /Users/zec/Repo/ecs-beginner
source .venv/bin/activate
.venv/bin/python api_server.py
```

看到 `Uvicorn running on http://0.0.0.0:8080` 表示启动成功。

### 3. 启动 Nginx

```bash
# 新开终端 2
nginx -c /Users/zec/Repo/ecs-beginner/nginx.conf
```

## 测试验证

### 方式 1：浏览器测试（推荐）

打开浏览器访问：http://localhost:8888

点击页面上的 **"测试后端 API"** 按钮，应该看到：
```
✅ API 响应成功
消息: API 服务运行正常
时间: 2026-03-14
服务器: ECS Backend
```

### 方式 2：命令行测试

```bash
# 测试前端页面
curl http://localhost:8888

# 测试 API 代理
curl http://localhost:8888/api/test

# 测试后端直连
curl http://localhost:8080/api/test
```

## 停止服务

```bash
# 停止 Nginx
nginx -s stop

# 停止后端 API（在运行 api_server.py 的终端按 Ctrl+C）
# 或者
ps aux | grep api_server
kill <pid>
```

## 架构说明

```
浏览器 → Nginx (8888端口)
         ├── / → 静态 HTML
         └── /api → FastAPI (8080端口)
```

## 常见问题

**Q: Nginx 启动失败？**
```bash
# 检查是否已有 Nginx 进程
ps aux | grep nginx
# 停止旧进程
nginx -s stop
# 或强制停止
killall nginx
```

**Q: 端口被占用？**
```bash
# 查看端口占用
lsof -i :8888
lsof -i :8080
```

**Q: API 请求失败？**
- 确认后端服务已启动：`curl http://localhost:8080/api/test`
- 确认 Nginx 已启动：`ps aux | grep nginx`
