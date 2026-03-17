# 技术架构文档

## 项目概述

基于 Next.js + FastAPI 的全栈 AI 应用架构，采用 DDD（领域驱动设计）模式，部署在单台 ECS 服务器（2C2G，40GB SSD）。

---

## 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                         用户浏览器                            │
│                    https://yourdomain.com                    │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS (443)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                      Nginx (反向代理)                         │
│  - SSL 终止                                                  │
│  - 请求路由                                                  │
│  - 静态资源缓存                                              │
└─────────┬───────────────────────────────┬───────────────────┘
          │                               │
          │ / (前端路由)                  │ /api/* (API 路由)
          ▼                               ▼
┌─────────────────────────┐    ┌──────────────────────────────┐
│   Next.js Server        │    │   FastAPI Backend (DDD)      │
│   Port: 3000            │───▶│   Port: 8000                 │
│                         │    │                              │
│  - SSR/SSG 渲染         │    │  - RESTful API               │
│  - API Routes (BFF)     │    │  - Domain Logic              │
│  - 前端路由             │    │  - Application Services      │
│  - 静态资源服务         │    │  - Infrastructure Layer      │
└─────────────────────────┘    └──────────┬───────────────────┘
                                          │
                                          ▼
                               ┌──────────────────────┐
                               │   数据层              │
                               │  - PostgreSQL/SQLite │
                               │  - Redis (缓存)      │
                               │  - Vector DB (RAG)   │
                               └──────────────────────┘
```

---

## 技术栈详解

### 前端层 (Next.js)

#### 核心技术
- **框架**: Next.js 15.x (App Router)
- **UI 库**: React 19
- **语言**: TypeScript 5.x
- **样式**: Tailwind CSS 3.x
- **状态管理**: Zustand / React Context
- **AI SDK**: Vercel AI SDK (`ai` 包)

#### 目录结构
```
frontend/
├── app/                    # Next.js App Router
│   ├── layout.tsx         # 全局布局
│   ├── page.tsx           # 首页
│   ├── chat/              # Chatbot 页面
│   │   └── page.tsx
│   ├── search/            # AI 搜索页面
│   │   └── page.tsx
│   └── api/               # API Routes (BFF 层)
│       ├── chat/
│       │   └── route.ts   # 代理到 FastAPI
│       └── search/
│           └── route.ts
├── components/            # React 组件
│   ├── ChatBot.tsx
│   ├── SearchBar.tsx
│   └── ui/
├── lib/
│   ├── api-client.ts     # API 请求封装
│   └── utils.ts
├── public/
├── package.json
├── next.config.js
└── tsconfig.json
```

---

## 后端层 (FastAPI + DDD)

### DDD 架构分层

```
┌─────────────────────────────────────────────────┐
│           Presentation Layer (接口层)            │
│  - REST API Controllers                         │
│  - Request/Response DTOs                        │
│  - API 路由定义                                  │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│         Application Layer (应用层)               │
│  - Use Cases / Application Services             │
│  - 业务流程编排                                  │
│  - DTO 转换                                      │
│  - 事务管理                                      │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│           Domain Layer (领域层)                  │
│  - Entities (实体)                              │
│  - Value Objects (值对象)                       │
│  - Domain Services (领域服务)                   │
│  - Domain Events (领域事件)                     │
│  - Repository Interfaces (仓储接口)             │
└────────────────┬────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────┐
│       Infrastructure Layer (基础设施层)          │
│  - Repository Implementations                   │
│  - External Services (LLM, Vector DB)           │
│  - Database Adapters                            │
│  - Message Queue                                │
└─────────────────────────────────────────────────┘
```

### 核心技术
- **框架**: FastAPI 0.115+
- **语言**: Python 3.10+
- **异步**: asyncio + uvicorn
- **ORM**: SQLAlchemy 2.0 (async)
- **依赖注入**: dependency-injector
- **AI 框架**: LangChain / LlamaIndex
- **验证**: Pydantic V2

---

### 生产级目录结构 (DDD)

```
backend/
├── src/
│   ├── main.py                          # FastAPI 应用入口
│   │
│   ├── presentation/                    # 接口层 (API Controllers)
│   │   ├── __init__.py
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── v1/                      # API 版本控制
│   │   │   │   ├── __init__.py
│   │   │   │   ├── router.py           # 路由聚合
│   │   │   │   ├── chat.py             # Chat API
│   │   │   │   ├── search.py           # Search API
│   │   │   │   └── health.py           # 健康检查
│   │   │   └── dependencies.py         # API 依赖注入
│   │   ├── schemas/                     # Request/Response DTOs
│   │   │   ├── __init__.py
│   │   │   ├── chat.py
│   │   │   ├── search.py
│   │   │   └── common.py
│   │   └── middleware/                  # 中间件
│   │       ├── __init__.py
│   │       ├── error_handler.py
│   │       ├── logging.py
│   │       └── cors.py
│   │
│   ├── application/                     # 应用层 (Use Cases)
│   │   ├── __init__.py
│   │   ├── use_cases/
│   │   │   ├── __init__.py
│   │   │   ├── chat/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── create_conversation.py
│   │   │   │   ├── send_message.py
│   │   │   │   └── get_conversation_history.py
│   │   │   └── search/
│   │   │       ├── __init__.py
│   │   │       ├── semantic_search.py
│   │   │       └── rag_query.py
│   │   ├── services/                    # 应用服务
│   │   │   ├── __init__.py
│   │   │   ├── chat_service.py
│   │   │   └── search_service.py
│   │   └── dto/                         # 内部 DTO
│   │       ├── __init__.py
│   │       └── chat_dto.py
│   │
│   ├── domain/                          # 领域层 (核心业务逻辑)
│   │   ├── __init__.py
│   │   ├── chat/                        # Chat 聚合根
│   │   │   ├── __init__.py
│   │   │   ├── entities/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── conversation.py     # 对话实体
│   │   │   │   └── message.py          # 消息实体
│   │   │   ├── value_objects/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── conversation_id.py
│   │   │   │   └── message_content.py
│   │   │   ├── services/
│   │   │   │   ├── __init__.py
│   │   │   │   └── conversation_service.py  # 领域服务
│   │   │   ├── events/
│   │   │   │   ├── __init__.py
│   │   │   │   └── message_created.py
│   │   │   └── repositories/            # 仓储接口
│   │   │       ├── __init__.py
│   │   │       └── conversation_repository.py
│   │   │
│   │   ├── search/                      # Search 聚合根
│   │   │   ├── __init__.py
│   │   │   ├── entities/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── document.py
│   │   │   │   └── search_result.py
│   │   │   ├── value_objects/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── embedding.py
│   │   │   │   └── relevance_score.py
│   │   │   ├── services/
│   │   │   │   ├── __init__.py
│   │   │   │   └── rag_service.py      # RAG 领域服务
│   │   │   └── repositories/
│   │   │       ├── __init__.py
│   │   │       └── document_repository.py
│   │   │
│   │   └── shared/                      # 共享领域对象
│   │       ├── __init__.py
│   │       ├── base_entity.py
│   │       ├── base_value_object.py
│   │       └── exceptions.py           # 领域异常
│   │
│   ├── infrastructure/                  # 基础设施层
│   │   ├── __init__.py
│   │   ├── persistence/                 # 持久化
│   │   │   ├── __init__.py
│   │   │   ├── database.py             # 数据库连接
│   │   │   ├── models/                 # ORM 模型
│   │   │   │   ├── __init__.py
│   │   │   │   ├── conversation.py
│   │   │   │   └── document.py
│   │   │   └── repositories/           # 仓储实现
│   │   │       ├── __init__.py
│   │   │       ├── conversation_repository_impl.py
│   │   │       └── document_repository_impl.py
│   │   │
│   │   ├── external_services/          # 外部服务
│   │   │   ├── __init__.py
│   │   │   ├── llm/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── base.py            # LLM 接口
│   │   │   │   ├── openai_client.py
│   │   │   │   └── anthropic_client.py
│   │   │   └── vector_store/
│   │   │       ├── __init__.py
│   │   │       ├── base.py
│   │   │       ├── chroma_client.py
│   │   │       └── qdrant_client.py
│   │   │
│   │   ├── cache/                      # 缓存
│   │   │   ├── __init__.py
│   │   │   ├── redis_client.py
│   │   │   └── cache_service.py
│   │   │
│   │   └── messaging/                  # 消息队列 (可选)
│   │       ├── __init__.py
│   │       └── event_bus.py
│   │
│   ├── shared/                         # 共享基础设施
│   │   ├── __init__.py
│   │   ├── config/                     # 配置管理
│   │   │   ├── __init__.py
│   │   │   ├── settings.py            # Pydantic Settings
│   │   │   └── logging_config.py
│   │   ├── security/                   # 安全
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   └── encryption.py
│   │   └── utils/                      # 工具函数
│   │       ├── __init__.py
│   │       ├── datetime_utils.py
│   │       └── text_utils.py
│   │
│   └── container.py                    # 依赖注入容器
│
├── tests/                              # 测试
│   ├── unit/                           # 单元测试
│   │   ├── domain/
│   │   ├── application/
│   │   └── infrastructure/
│   ├── integration/                    # 集成测试
│   │   └── api/
│   └── e2e/                           # 端到端测试
│
├── migrations/                         # 数据库迁移
│   └── alembic/
│
├── scripts/                            # 脚本
│   ├── init_db.py
│   └── seed_data.py
│
├── requirements/                       # 依赖管理
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
│
├── .env.example                        # 环境变量示例
├── pyproject.toml                      # 项目配置
├── pytest.ini                          # 测试配置
└── README.md
```

---

## DDD 核心概念说明

### 1. Presentation Layer (接口层)
**职责**: 处理 HTTP 请求/响应，不包含业务逻辑

**示例**: `presentation/api/v1/chat.py`
```python
from fastapi import APIRouter, Depends
from presentation.schemas.chat import CreateMessageRequest, MessageResponse
from application.use_cases.chat.send_message import SendMessageUseCase

router = APIRouter(prefix="/chat", tags=["chat"])

@router.post("/messages", response_model=MessageResponse)
async def send_message(
    request: CreateMessageRequest,
    use_case: SendMessageUseCase = Depends()
):
    """发送消息 - 仅处理 HTTP 层逻辑"""
    result = await use_case.execute(
        conversation_id=request.conversation_id,
        content=request.content
    )
    return MessageResponse.from_domain(result)
```

### 2. Application Layer (应用层)
**职责**: 编排业务流程，协调领域对象，不包含核心业务规则

**示例**: `application/use_cases/chat/send_message.py`
```python
from domain.chat.repositories.conversation_repository import ConversationRepository
from domain.chat.services.conversation_service import ConversationService
from infrastructure.external_services.llm.base import LLMClient

class SendMessageUseCase:
    def __init__(
        self,
        conversation_repo: ConversationRepository,
        conversation_service: ConversationService,
        llm_client: LLMClient
    ):
        self.conversation_repo = conversation_repo
        self.conversation_service = conversation_service
        self.llm_client = llm_client

    async def execute(self, conversation_id: str, content: str):
        # 1. 获取对话
        conversation = await self.conversation_repo.get_by_id(conversation_id)

        # 2. 添加用户消息 (领域逻辑)
        conversation.add_user_message(content)

        # 3. 调用 LLM
        ai_response = await self.llm_client.generate(
            messages=conversation.get_messages()
        )

        # 4. 添加 AI 回复 (领域逻辑)
        conversation.add_ai_message(ai_response)

        # 5. 持久化
        await self.conversation_repo.save(conversation)

        return conversation.get_last_message()
```

### 3. Domain Layer (领域层)
**职责**: 核心业务逻辑和规则，独立于技术实现

**示例**: `domain/chat/entities/conversation.py`
```python
from datetime import datetime
from domain.shared.base_entity import BaseEntity
from domain.chat.entities.message import Message
from domain.chat.value_objects.conversation_id import ConversationId

class Conversation(BaseEntity):
    """对话聚合根"""

    def __init__(
        self,
        id: ConversationId,
        messages: list[Message],
        created_at: datetime
    ):
        self.id = id
        self.messages = messages
        self.created_at = created_at

    def add_user_message(self, content: str) -> Message:
        """添加用户消息 - 领域规则"""
        if not content.strip():
            raise ValueError("消息内容不能为空")

        message = Message.create_user_message(content)
        self.messages.append(message)
        return message

    def add_ai_message(self, content: str) -> Message:
        """添加 AI 消息 - 领域规则"""
        if not self.messages or self.messages[-1].is_ai:
            raise ValueError("AI 消息必须跟在用户消息后")

        message = Message.create_ai_message(content)
        self.messages.append(message)
        return message

    def get_messages(self) -> list[Message]:
        """获取消息列表"""
        return self.messages.copy()

    def get_last_message(self) -> Message:
        """获取最后一条消息"""
        return self.messages[-1]
```

### 4. Infrastructure Layer (基础设施层)
**职责**: 技术实现细节，如数据库、外部 API

**示例**: `infrastructure/persistence/repositories/conversation_repository_impl.py`
```python
from sqlalchemy.ext.asyncio import AsyncSession
from domain.chat.repositories.conversation_repository import ConversationRepository
from domain.chat.entities.conversation import Conversation
from infrastructure.persistence.models.conversation import ConversationModel

class ConversationRepositoryImpl(ConversationRepository):
    """对话仓储实现"""

    def __init__(self, session: AsyncSession):
        self.session = session

    async def get_by_id(self, conversation_id: str) -> Conversation:
        """从数据库获取对话"""
        model = await self.session.get(ConversationModel, conversation_id)
        if not model:
            raise ValueError(f"Conversation {conversation_id} not found")
        return self._to_domain(model)

    async def save(self, conversation: Conversation) -> None:
        """保存对话到数据库"""
        model = self._to_model(conversation)
        self.session.add(model)
        await self.session.commit()

    def _to_domain(self, model: ConversationModel) -> Conversation:
        """ORM 模型转领域实体"""
        # 转换逻辑
        pass

    def _to_model(self, entity: Conversation) -> ConversationModel:
        """领域实体转 ORM 模型"""
        # 转换逻辑
        pass
```

---

## 依赖注入配置

**`src/container.py`**
```python
from dependency_injector import containers, providers
from infrastructure.persistence.database import Database
from infrastructure.external_services.llm.openai_client import OpenAIClient
from domain.chat.services.conversation_service import ConversationService
from application.use_cases.chat.send_message import SendMessageUseCase

class Container(containers.DeclarativeContainer):
    """依赖注入容器"""

    config = providers.Configuration()

    # 基础设施
    database = providers.Singleton(Database, db_url=config.database_url)

    llm_client = providers.Singleton(
        OpenAIClient,
        api_key=config.openai_api_key
    )

    # 仓储
    conversation_repository = providers.Factory(
        ConversationRepositoryImpl,
        session=database.provided.session
    )

    # 领域服务
    conversation_service = providers.Factory(
        ConversationService,
        repository=conversation_repository
    )

    # 用例
    send_message_use_case = providers.Factory(
        SendMessageUseCase,
        conversation_repo=conversation_repository,
        conversation_service=conversation_service,
        llm_client=llm_client
    )
```

---

## 数据流详解

### 完整请求流程 (DDD 视角)

```
HTTP Request
    ↓
[Presentation Layer]
    ├─ API Controller 接收请求
    ├─ 验证 Request DTO
    └─ 调用 Use Case
        ↓
[Application Layer]
    ├─ Use Case 编排流程
    ├─ 调用 Repository 获取实体
    ├─ 调用 Domain Service 执行业务逻辑
    ├─ 调用 Infrastructure Service (LLM)
    └─ 持久化结果
        ↓
[Domain Layer]
    ├─ Entity 执行核心业务规则
    ├─ Value Object 保证数据一致性
    └─ Domain Event 发布事件
        ↓
[Infrastructure Layer]
    ├─ Repository 实现数据持久化
    ├─ LLM Client 调用外部 API
    └─ Cache 缓存热点数据
        ↓
HTTP Response
```

---

## 配置管理

**`src/shared/config/settings.py`**
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # 应用配置
    app_name: str = "AI Backend"
    debug: bool = False

    # 数据库
    database_url: str

    # Redis
    redis_url: str = "redis://localhost:6379"

    # LLM
    openai_api_key: str
    openai_model: str = "gpt-4"

    # Vector DB
    vector_db_type: str = "chroma"  # chroma | qdrant
    chroma_path: str = "./data/chroma"

    class Config:
        env_file = ".env"

settings = Settings()
```

---

## 测试策略

### 1. 单元测试 (Domain Layer)
```python
# tests/unit/domain/test_conversation.py
def test_add_user_message():
    conversation = Conversation.create()
    message = conversation.add_user_message("Hello")

    assert len(conversation.messages) == 1
    assert message.content == "Hello"
    assert message.is_user

def test_cannot_add_empty_message():
    conversation = Conversation.create()

    with pytest.raises(ValueError):
        conversation.add_user_message("")
```

### 2. 集成测试 (Application Layer)
```python
# tests/integration/application/test_send_message_use_case.py
@pytest.mark.asyncio
async def test_send_message_use_case(
    conversation_repo_mock,
    llm_client_mock
):
    use_case = SendMessageUseCase(
        conversation_repo=conversation_repo_mock,
        llm_client=llm_client_mock
    )

    result = await use_case.execute(
        conversation_id="123",
        content="Hello"
    )

    assert result.content is not None
    conversation_repo_mock.save.assert_called_once()
```

### 3. E2E 测试 (API)
```python
# tests/e2e/test_chat_api.py
@pytest.mark.asyncio
async def test_send_message_api(client):
    response = await client.post(
        "/api/v1/chat/messages",
        json={
            "conversation_id": "123",
            "content": "Hello"
        }
    )

    assert response.status_code == 200
    assert "content" in response.json()
```

---

## 部署配置

### Systemd 服务

**`/etc/systemd/system/fastapi.service`**
```ini
[Unit]
Description=FastAPI Backend (DDD)
After=network.target postgresql.service redis.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/backend
Environment="PYTHONPATH=/var/www/backend/src"
Environment="ENV=production"
ExecStart=/var/www/backend/.venv/bin/uvicorn src.main:app \
    --host 0.0.0.0 \
    --port 8000 \
    --workers 2 \
    --log-config src/shared/config/logging_config.yaml

Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---

## 性能优化 (2C2G 环境)

### 1. 内存优化
- **SQLAlchemy**: 使用连接池 `pool_size=5, max_overflow=10`
- **Uvicorn**: 2 workers (避免过多进程)
- **Redis**: 限制最大内存 `maxmemory 256mb`

### 2. 查询优化
- 使用 `select_in_loading` 避免 N+1 查询
- 对热点数据使用 Redis 缓存
- 数据库索引优化

### 3. 异步处理
- 所有 I/O 操作使用 `async/await`
- LLM 调用使用流式响应
- 后台任务使用 `BackgroundTasks`

---

## 监控与日志

### 结构化日志
```python
import structlog

logger = structlog.get_logger()

logger.info(
    "message_sent",
    conversation_id=conversation_id,
    user_id=user_id,
    duration_ms=duration
)
```

### 健康检查
```python
@router.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "database": await check_database(),
        "redis": await check_redis(),
        "llm": await check_llm()
    }
```

---

## DDD 最佳实践

### 1. 聚合根设计
- 每个聚合根有明确的边界
- 通过 ID 引用其他聚合根
- 保持聚合根小而聚焦

### 2. 仓储模式
- 仓储接口定义在 Domain Layer
- 实现在 Infrastructure Layer
- 只对聚合根提供仓储

### 3. 领域事件
- 使用事件解耦聚合根
- 事件驱动异步处理
- 保证最终一致性

### 4. 值对象
- 不可变对象
- 通过值相等判断
- 封装验证逻辑

---

## 扩展路径

### 短期
- 添加 CQRS (读写分离)
- 集成事件溯源
- 添加 API 网关

### 中期
- 微服务拆分 (按聚合根)
- 消息队列 (RabbitMQ/Kafka)
- 分布式追踪

### 长期
- 事件驱动架构
- Kubernetes 部署
- 服务网格

---

## 参考资源

- [Domain-Driven Design (Eric Evans)](https://www.domainlanguage.com/ddd/)
- [Implementing DDD (Vaughn Vernon)](https://vaughnvernon.com/)
- [FastAPI Best Practices](https://github.com/zhanymkanov/fastapi-best-practices)
- [Python Clean Architecture](https://github.com/cosmic-python/book)
