# ECS 前后端分离 Demo

前后端分离架构演示：Nginx + 静态前端 + FastAPI 后端。

## 项目结构

```
ecs-beginner/
├── frontend/           # 前端静态页面
│   ├── index.html
│   ├── server.py
│   └── README.md
├── backend/            # 后端 API 服务
│   ├── src/
│   │   └── backend/
│   │       ├── __init__.py
│   │       └── main.py
│   ├── pyproject.toml
│   ├── .python-version
│   ├── .gitignore
│   └── README.md
├── nginx.conf          # Nginx 配置
└── README.md
```

## 架构说明

```
Nginx (80端口)
├── / → 前端静态文件 (frontend/)
└── /api → 后端 API (8080端口)
```

## 本地开发

### 1. 后端开发

```bash
cd backend
uv venv
source .venv/bin/activate
uv pip install -e .

# 启动后端
python -m backend.main
```

后端运行在 http://localhost:8080

### 2. 前端开发

```bash
cd frontend
PORT=8000 python server.py
```

前端运行在 http://localhost:8000

### 3. 使用 Nginx 代理（完整演示）

```bash
# 启动后端
cd backend && source .venv/bin/activate && python -m backend.main &

# 启动 Nginx
nginx -c /Users/zec/Repo/ecs-beginner/nginx.conf
```

访问 http://localhost:8888

## ECS 部署

### 1. 环境准备

```bash
# SSH 登录 ECS
ssh root@<your-ecs-ip>

# 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.cargo/env

# 安装 Nginx
yum install -y nginx  # CentOS/AliyunOS
# 或
apt install -y nginx  # Ubuntu/Debian
```

### 2. 部署代码

```bash
# Clone 代码
git clone <your-repo-url>
cd ecs-beginner

# 创建虚拟环境并安装依赖
uv venv
source .venv/bin/activate
uv pip install -e .
```

### 3. 配置 Nginx

```bash
# 编辑 nginx.conf，修改 root 路径为实际路径
# 例如：root /root/ecs-beginner;

# 复制配置文件
cp nginx.conf /etc/nginx/conf.d/ecs-beginner.conf

# 测试配置
nginx -t

# 重启 Nginx
systemctl restart nginx
systemctl enable nginx
```

### 4. 启动后端服务

```bash
# 后台运行 API 服务
nohup .venv/bin/python api_server.py > api.log 2>&1 &
```

### 5. 配置防火墙

确保阿里云 ECS 安全组已开放 80 端口入站规则。

## 验证

```bash
# 本地测试
curl http://localhost

# 公网 IP 测试
curl http://<your-ecs-ip>

# 域名测试
curl http://<your-domain>
```

浏览器访问：http://<your-domain>，点击"测试后端 API"按钮。

## 停止服务

```bash
# 停止 API 服务
ps aux | grep api_server.py
kill <pid>

# 停止 Nginx
systemctl stop nginx
```

