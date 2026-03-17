# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ECS 静态网站验证服务 - 用于验证阿里云 ECS 服务器和域名配置的简单 Python HTTP 服务器。

## Development Commands

```bash
# 本地开发
PORT=8000 python server.py

# ECS 部署
sudo .venv/bin/python server.py
```

## Architecture

- `server.py` - Python HTTP 服务器（使用标准库 http.server）
- `index.html` - 静态验证页面
- 监听 80 端口（生产）或 8000 端口（开发）
- 使用 uv 管理 Python 环境（3.10.12）
