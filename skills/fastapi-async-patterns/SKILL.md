---
name: fastapi-async-patterns
description: 用于 FastAPI 异步模式相关问题，适用于构建高性能 API、处理并发请求和各类异步操作。
allowed-tools:
  - Bash
  - Read
---

# FastAPI 异步模式（Async Patterns）

掌握 FastAPI 中的异步模式（async patterns），用于构建高性能、高并发且资源利用率优秀的 API。

## 基础异步路由处理（Basic Async Route Handlers）

理解 FastAPI 中异步（async）与同步（sync）端点的差异和适用场景。

```python
from fastapi import FastAPI
import time
import asyncio

app = FastAPI()

# Sync endpoint (blocks the event loop)
@app.get('/sync')
def sync_endpoint():
    time.sleep(1)  # Blocks the entire server
    return {'message': 'Completed after 1 second'}

# Async endpoint (non-blocking)
@app.get('/async')
async def async_endpoint():
    await asyncio.sleep(1)  # Other requests can be handled
    return {'message': 'Completed after 1 second'}

# CPU-bound work (use sync)
@app.get('/cpu-intensive')
def cpu_intensive():
    result = sum(i * i for i in range(10000000))
    return {'result': result}

# I/O-bound work (use async)
@app.get('/io-intensive')
async def io_intensive():
    async with httpx.AsyncClient() as client:
        response = await client.get('https://api.example.com/data')
        return response.json()
```

## 异步数据库操作（Async Database Operations）

结合常见 ORM / 库的异步数据库访问模式。

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy import select
import asyncpg
from motor.motor_asyncio import AsyncIOMotorClient
from tortoise import Tortoise
from tortoise.contrib.fastapi import register_tortoise

app = FastAPI()

# SQLAlchemy async setup
DATABASE_URL = 'postgresql+asyncpg://user:pass@localhost/db'
engine = create_async_engine(DATABASE_URL, echo=True, future=True)
AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

@app.get('/users/{user_id}')
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(User).where(User.id == user_id))
    user = result.scalar_one_or_none()
    if not user:
        raise HTTPException(status_code=404, detail='User not found')
    return user

# Direct asyncpg (lower level, faster)
async def get_asyncpg_pool():
    pool = await asyncpg.create_pool(
        'postgresql://user:pass@localhost/db',
        min_size=10,
        max_size=20
    )
    try:
        yield pool
    finally:
        await pool.close()

@app.get('/users-fast/{user_id}')
async def get_user_fast(user_id: int, pool = Depends(get_asyncpg_pool)):
    async with pool.acquire() as conn:
        row = await conn.fetchrow(
            'SELECT * FROM users WHERE id = $1', user_id
        )
        if not row:
            raise HTTPException(status_code=404, detail='User not found')
        return dict(row)

# MongoDB with Motor
mongo_client = AsyncIOMotorClient('mongodb://localhost:27017')
db = mongo_client.mydatabase

@app.get('/documents/{doc_id}')
async def get_document(doc_id: str):
    document = await db.collection.find_one({'_id': doc_id})
    if not document:
        raise HTTPException(status_code=404, detail='Document not found')
    return document

@app.post('/documents')
async def create_document(data: dict):
    result = await db.collection.insert_one(data)
    return {'id': str(result.inserted_id)}

# Tortoise ORM async
register_tortoise(
    app,
    db_url='postgres://user:pass@localhost/db',
    modules={'models': ['app.models']},
    generate_schemas=True,
    add_exception_handlers=True,
)

from tortoise.models import Model
from tortoise import fields

class UserModel(Model):
    id = fields.IntField(pk=True)
    name = fields.CharField(max_length=255)
    email = fields.CharField(max_length=255)

@app.get('/tortoise-users/{user_id}')
async def get_tortoise_user(user_id: int):
    user = await UserModel.get_or_none(id=user_id)
    if not user:
        raise HTTPException(status_code=404, detail='User not found')
    return user
```

## 后台任务（Background Tasks）

执行「即发即弃」（fire-and-forget）任务，而不阻塞主请求响应流程。

```python
from fastapi import BackgroundTasks, FastAPI
import asyncio
from datetime import datetime

app = FastAPI()

# Simple background task
async def send_email(email: str, message: str):
    await asyncio.sleep(2)  # Simulate email sending
    print(f'Email sent to {email}: {message}')

@app.post('/send-email')
async def send_email_endpoint(
    email: str,
    message: str,
    background_tasks: BackgroundTasks
):
    background_tasks.add_task(send_email, email, message)
    return {'status': 'Email will be sent in background'}

# Multiple background tasks
async def log_activity(user_id: int, action: str):
    await asyncio.sleep(0.5)
    print(f'[{datetime.now()}] User {user_id} performed: {action}')

async def update_analytics(action: str):
    await asyncio.sleep(1)
    print(f'Analytics updated for action: {action}')

@app.post('/users/{user_id}/action')
async def perform_action(
    user_id: int,
    action: str,
    background_tasks: BackgroundTasks
):
    # Add multiple tasks
    background_tasks.add_task(log_activity, user_id, action)
    background_tasks.add_task(update_analytics, action)
    return {'status': 'Action logged'}

# Background cleanup
async def cleanup_temp_files(file_path: str):
    await asyncio.sleep(60)  # Wait before cleanup
    import os
    if os.path.exists(file_path):
        os.remove(file_path)
        print(f'Cleaned up: {file_path}')

@app.post('/upload')
async def upload_file(
    file: UploadFile,
    background_tasks: BackgroundTasks
):
    temp_path = f'/tmp/{file.filename}'
    with open(temp_path, 'wb') as f:
        content = await file.read()
        f.write(content)

    # Schedule cleanup
    background_tasks.add_task(cleanup_temp_files, temp_path)
    return {'filename': file.filename, 'path': temp_path}
```

## WebSocket 处理（WebSocket Handling）

实现实时双向通信（real-time bidirectional communication）相关模式。

```python
from fastapi import WebSocket, WebSocketDisconnect, Depends
from typing import List
import json

app = FastAPI()

# Simple WebSocket
@app.websocket('/ws')
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f'Echo: {data}')
    except WebSocketDisconnect:
        print('Client disconnected')

# WebSocket with authentication
async def get_current_user_ws(websocket: WebSocket):
    token = websocket.query_params.get('token')
    if not token or not verify_token(token):
        await websocket.close(code=1008)  # Policy violation
        raise HTTPException(status_code=401, detail='Unauthorized')
    return decode_token(token)

@app.websocket('/ws/authenticated')
async def authenticated_websocket(
    websocket: WebSocket,
    user = Depends(get_current_user_ws)
):
    await websocket.accept()
    try:
        await websocket.send_text(f'Welcome {user["name"]}')
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f'{user["name"]}: {data}')
    except WebSocketDisconnect:
        print(f'User {user["name"]} disconnected')

# Broadcasting to multiple connections
class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)

manager = ConnectionManager()

@app.websocket('/ws/chat/{client_id}')
async def chat_endpoint(websocket: WebSocket, client_id: str):
    await manager.connect(websocket)
    await manager.broadcast(f'Client {client_id} joined the chat')
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast(f'Client {client_id}: {data}')
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast(f'Client {client_id} left the chat')

# WebSocket with JSON messages
@app.websocket('/ws/json')
async def json_websocket(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_json()
            message_type = data.get('type')

            if message_type == 'ping':
                await websocket.send_json({'type': 'pong'})
            elif message_type == 'message':
                await websocket.send_json({
                    'type': 'response',
                    'data': f'Received: {data.get("content")}'
                })
    except WebSocketDisconnect:
        print('Client disconnected')
```

## Server-Sent Events（SSE）

从服务端到客户端的「单向流式」推送（one-way streaming）。

```python
from fastapi import FastAPI
from sse_starlette.sse import EventSourceResponse
import asyncio

app = FastAPI()

@app.get('/sse')
async def sse_endpoint():
    async def event_generator():
        for i in range(10):
            await asyncio.sleep(1)
            yield {
                'event': 'message',
                'data': f'Message {i}'
            }

    return EventSourceResponse(event_generator())

# SSE with real-time updates
@app.get('/sse/updates')
async def sse_updates():
    async def update_generator():
        while True:
            # Simulate fetching updates
            await asyncio.sleep(2)
            update = await fetch_latest_update()
            yield {
                'event': 'update',
                'data': json.dumps(update)
            }

    return EventSourceResponse(update_generator())

# SSE with heartbeat
@app.get('/sse/heartbeat')
async def sse_heartbeat():
    async def heartbeat_generator():
        try:
            while True:
                await asyncio.sleep(30)
                yield {
                    'event': 'heartbeat',
                    'data': datetime.now().isoformat()
                }
        except asyncio.CancelledError:
            print('SSE connection closed')

    return EventSourceResponse(heartbeat_generator())
```

## 流式响应（Streaming Responses）

用于大文件或动态生成内容的流式输出。

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import io
import csv

app = FastAPI()

# Stream large file
@app.get('/download/{filename}')
async def download_file(filename: str):
    async def file_stream():
        with open(f'/data/{filename}', 'rb') as f:
            while chunk := f.read(8192):
                yield chunk

    return StreamingResponse(
        file_stream(),
        media_type='application/octet-stream',
        headers={'Content-Disposition': f'attachment; filename={filename}'}
    )

# Stream generated CSV
@app.get('/export/users')
async def export_users():
    async def csv_stream():
        output = io.StringIO()
        writer = csv.writer(output)

        # Write header
        writer.writerow(['ID', 'Name', 'Email'])
        yield output.getvalue()
        output.truncate(0)
        output.seek(0)

        # Stream users in batches
        offset = 0
        batch_size = 100
        while True:
            users = await fetch_users_batch(offset, batch_size)
            if not users:
                break

            for user in users:
                writer.writerow([user.id, user.name, user.email])
                yield output.getvalue()
                output.truncate(0)
                output.seek(0)

            offset += batch_size

    return StreamingResponse(
        csv_stream(),
        media_type='text/csv',
        headers={'Content-Disposition': 'attachment; filename=users.csv'}
    )

# Stream generated content
@app.get('/generate/report')
async def generate_report():
    async def report_stream():
        yield b'<html><body><h1>Report</h1>'

        for section in ['users', 'orders', 'analytics']:
            await asyncio.sleep(0.5)  # Simulate processing
            data = await fetch_section_data(section)
            yield f'<h2>{section.title()}</h2>'.encode()
            yield f'<pre>{data}</pre>'.encode()

        yield b'</body></html>'

    return StreamingResponse(report_stream(), media_type='text/html')
```

## 并发请求处理（Concurrent Request Handling）

针对多个操作/请求进行并行处理的模式。

```python
from fastapi import FastAPI
import asyncio
import httpx

app = FastAPI()

# Parallel API calls
@app.get('/aggregate/user/{user_id}')
async def aggregate_user_data(user_id: int):
    async with httpx.AsyncClient() as client:
        # Fetch from multiple sources in parallel
        profile_task = client.get(f'https://api.example.com/users/{user_id}')
        posts_task = client.get(f'https://api.example.com/users/{user_id}/posts')
        comments_task = client.get(f'https://api.example.com/users/{user_id}/comments')

        profile, posts, comments = await asyncio.gather(
            profile_task,
            posts_task,
            comments_task
        )

        return {
            'profile': profile.json(),
            'posts': posts.json(),
            'comments': comments.json()
        }

# Parallel database queries
@app.get('/dashboard')
async def get_dashboard(db: AsyncSession = Depends(get_db)):
    # Execute multiple queries in parallel
    users_query = db.execute(select(User).limit(10))
    orders_query = db.execute(select(Order).limit(10))
    stats_query = db.execute(select(func.count(User.id)))

    users, orders, stats = await asyncio.gather(
        users_query,
        orders_query,
        stats_query
    )

    return {
        'users': users.scalars().all(),
        'orders': orders.scalars().all(),
        'total_users': stats.scalar()
    }

# Race condition (first to complete wins)
@app.get('/fastest-price/{product_id}')
async def get_fastest_price(product_id: str):
    async with httpx.AsyncClient() as client:
        tasks = [
            client.get(f'https://store1.com/price/{product_id}'),
            client.get(f'https://store2.com/price/{product_id}'),
            client.get(f'https://store3.com/price/{product_id}')
        ]

        done, pending = await asyncio.wait(
            tasks,
            return_when=asyncio.FIRST_COMPLETED
        )

        # Cancel pending requests
        for task in pending:
            task.cancel()

        result = done.pop().result()
        return result.json()
```

## 异步上下文管理器（Async Context Managers）

使用异步上下文管理器进行资源管理（连接、客户端等）的生命周期控制。

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
import asyncio

# Async context manager for lifespan events
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    print('Starting up...')
    db_pool = await create_db_pool()
    redis_client = await create_redis_client()

    # Store in app state
    app.state.db_pool = db_pool
    app.state.redis = redis_client

    yield

    # Shutdown
    print('Shutting down...')
    await db_pool.close()
    await redis_client.close()

app = FastAPI(lifespan=lifespan)

# Custom async context manager
class AsyncDatabaseSession:
    def __init__(self, pool):
        self.pool = pool
        self.conn = None

    async def __aenter__(self):
        self.conn = await self.pool.acquire()
        return self.conn

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.pool.release(self.conn)
        if exc_type is not None:
            # Handle exception
            await self.conn.rollback()
        return False

@app.get('/data')
async def get_data():
    async with AsyncDatabaseSession(app.state.db_pool) as conn:
        result = await conn.fetch('SELECT * FROM data')
        return result
```

## 连接池管理（Connection Pooling）

为数据库与 HTTP 客户端等提供高效的连接复用和管理。

```python
from fastapi import FastAPI, Depends
import asyncpg
import httpx
from typing import AsyncGenerator

app = FastAPI()

# Database connection pool
class DatabasePool:
    def __init__(self):
        self.pool = None

    async def create_pool(self):
        self.pool = await asyncpg.create_pool(
            'postgresql://user:pass@localhost/db',
            min_size=10,
            max_size=20,
            command_timeout=60,
            max_queries=50000,
            max_inactive_connection_lifetime=300
        )

    async def close_pool(self):
        await self.pool.close()

    async def get_connection(self):
        async with self.pool.acquire() as connection:
            yield connection

db_pool = DatabasePool()

@app.on_event('startup')
async def startup():
    await db_pool.create_pool()

@app.on_event('shutdown')
async def shutdown():
    await db_pool.close_pool()

@app.get('/users')
async def get_users(conn = Depends(db_pool.get_connection)):
    rows = await conn.fetch('SELECT * FROM users')
    return [dict(row) for row in rows]

# HTTP client pool
class HTTPClientPool:
    def __init__(self):
        self.client = None

    async def get_client(self) -> AsyncGenerator[httpx.AsyncClient, None]:
        if self.client is None:
            self.client = httpx.AsyncClient(
                limits=httpx.Limits(max_keepalive_connections=20, max_connections=100),
                timeout=httpx.Timeout(10.0)
            )
        yield self.client

    async def close(self):
        if self.client:
            await self.client.aclose()

http_pool = HTTPClientPool()

@app.get('/external-api')
async def call_external_api(client: httpx.AsyncClient = Depends(http_pool.get_client)):
    response = await client.get('https://api.example.com/data')
    return response.json()
```

## 性能优化（Performance Optimization）

利用异步模式实现更优的性能和吞吐量。

```python
from fastapi import FastAPI
import asyncio
from functools import lru_cache

app = FastAPI()

# Cache expensive async operations
from aiocache import Cache
from aiocache.serializers import JsonSerializer

cache = Cache(Cache.MEMORY, serializer=JsonSerializer())

@app.get('/expensive-data/{key}')
async def get_expensive_data(key: str):
    # Check cache first
    cached = await cache.get(key)
    if cached:
        return {'data': cached, 'cached': True}

    # Expensive operation
    await asyncio.sleep(2)
    data = compute_expensive_result(key)

    # Store in cache
    await cache.set(key, data, ttl=300)
    return {'data': data, 'cached': False}

# Batch operations
@app.post('/users/batch')
async def create_users_batch(users: List[UserCreate], db = Depends(get_db)):
    # Create users in batch (more efficient than one-by-one)
    user_objects = [User(**user.dict()) for user in users]
    db.add_all(user_objects)
    await db.flush()
    return user_objects

# Debouncing with asyncio
class Debouncer:
    def __init__(self, delay: float):
        self.delay = delay
        self.task = None

    async def debounce(self, coro):
        if self.task:
            self.task.cancel()

        async def delayed():
            await asyncio.sleep(self.delay)
            await coro

        self.task = asyncio.create_task(delayed())
        await self.task

debouncer = Debouncer(delay=1.0)

# Prefetching related data
@app.get('/posts/{post_id}')
async def get_post_with_relations(post_id: int, db = Depends(get_db)):
    # Fetch post and related data in parallel
    post_task = db.get(Post, post_id)
    comments_task = db.execute(
        select(Comment).where(Comment.post_id == post_id)
    )
    author_task = db.execute(
        select(User).where(User.id == Post.author_id)
    )

    post, comments_result, author_result = await asyncio.gather(
        post_task, comments_task, author_task
    )

    return {
        'post': post,
        'comments': comments_result.scalars().all(),
        'author': author_result.scalar_one()
    }
```

## 何时使用此技能

在以下情况使用 fastapi-async-patterns：

- 构建处理大量并发请求的高吞吐量 API
- 处理 I/O 密集型操作（数据库、外部 API、文件操作）
- 实现实时功能（WebSockets, SSE）
- 并行处理多个操作
- 流式传输大型数据集或文件
- 构建与其他服务通信的微服务
- 优化 API 响应时间和资源使用
- 处理后台任务而不阻塞响应

## FastAPI 异步最佳实践

1. **使用 async 进行 I/O** - 始终对数据库、HTTP 请求和文件操作使用 async
2. **避免阻塞调用** - 永远不要在 async 函数中使用阻塞调用（如 time.sleep, requests 库）
3. **连接池** - 为数据库和 HTTP 客户端使用连接池
4. **适当清理** - 始终使用 try/finally 或异步上下文管理器清理资源
5. **并发操作** - 尽可能使用 asyncio.gather 进行并行操作
6. **后台任务** - 对即发即弃的操作使用 BackgroundTasks
7. **流式传输大数据** - 对大文件或生成的内容使用 StreamingResponse
8. **超时处理** - 对所有外部调用设置超时以防止挂起
9. **错误传播** - 在异步代码中正确处理异常
10. **监控性能** - 使用如 aiomonitor 之类的工具调试异步问题

## FastAPI 异步常见陷阱

1. **阻塞事件循环** - 在 async 函数中使用同步 I/O 会严重影响性能
2. **遗漏 await** - 忘记在 async 函数上使用 await 会导致协程警告
3. **创建过多任务** - 生成无限任务可能会耗尽资源
4. **未关闭连接** - 未关闭的数据库/HTTP 连接会导致资源泄漏
5. **混合同步和异步** - 不正确的混合会导致事件循环问题
6. **竞态条件** - 异步代码中的共享状态没有适当的锁定
7. **超时问题** - 外部调用没有超时可能会挂起服务器
8. **内存泄漏** - 永不完成的后台任务会累积
9. **错误吞没** - 后台任务和事件处理程序中的静默失败
10. **死锁** - 异步依赖或锁中的循环等待

## 环境配置与 .env 约定

为保证不同服务与环境之间的配置方式一致，建议在项目根目录维护统一的 `.env.example` 文件，并遵循以下约定：

- **.env.example 使用规范**
  - 仅存放**键名与示例值**，不得包含真实密码、Access Token 等敏感信息。
  - 开发者本地使用时，从 `.env.example` 复制为 `.env`，再按实际环境修改。
  - 推荐统一的基础字段：
    - `APP_ENV`：环境标识，例如 `local` / `dev` / `prod`
    - `APP_DEBUG`：是否开启调试，例如 `true` / `false`
    - `LOG_LEVEL`：日志级别，例如 `INFO` / `DEBUG` / `ERROR`
    - `APP_NAME`: 项目的name,英文和字母,在redis保存数据的时候，需要把APP_NAME作为key的前缀这样可以更加好的保证数据的隔离

- **数据库与缓存配置示例（建议字段命名）**
  - **PostgreSQL（SQLAlchemy + asyncpg）**
    - 统一使用：`DATABASE_URL=postgresql+asyncpg://user:password@host:5432/db_name`
    - 若项目存在多个独立数据库，可加后缀：`DATABASE_URL_ANALYTICS`、`DATABASE_URL_AUTH` 等。
  - **MySQL（SQLAlchemy + aiomysql）**
    - 统一使用：`MYSQL_URL=mysql+aiomysql://user:password@host:3306/db_name?charset=utf8mb4`
    - 或拆分字段（如已有存量代码）：`MYSQL_HOST` / `MYSQL_PORT` / `MYSQL_USER` / `MYSQL_PASSWORD` / `MYSQL_DB`
  - **Redis（异步客户端）**
    - 推荐统一使用 URI 形式：`REDIS_URL=redis://user:password@host:6379/0`
    - 如区分不同用途，可增加前缀：`REDIS_URL_CACHE`、`REDIS_URL_SESSION` 等。
  - **MongoDB（Motor）**
    - 推荐统一使用：`MONGODB_URI=mongodb://user:password@host:27017/db_name`
    - 若使用集群，可采用：`MONGODB_URI=mongodb+srv://user:password@cluster-host/db_name`

> 建议：不同服务/模块之间尽量复用同一套环境变量命名（如统一使用 `DATABASE_URL`、`REDIS_URL`、`MONGODB_URI`），避免出现多种拼写和命名风格，降低维护成本。

## 常用依赖与推荐包

为提高可维护性，推荐在整个项目中对同一类功能统一使用以下包（如无特别原因，不混用多种实现）：

- **Web 框架与服务**
  - `fastapi`：统一用于构建 API / WebSocket / SSE 等 HTTP 服务。
  - `uvicorn[standard]`：统一用作 ASGI Server（开发与生产环境均优先使用 uvicorn）。

- **HTTP 客户端**
  - `httpx`：统一作为 HTTP 客户端，外部 HTTP 调用优先使用 `httpx.AsyncClient`，避免在异步代码中使用 `requests`。

- **数据库与 ORM**
  - `sqlalchemy`：关系型数据库 ORM，统一使用 SQLAlchemy Async（`AsyncSession` 等）进行异步访问。
  - `asyncpg`：PostgreSQL 异步驱动，配合 `sqlalchemy` 或直接使用连接池。
  - `aiomysql`：MySQL 异步驱动，推荐搭配 SQLAlchemy 使用 `mysql+aiomysql://` DSN。
  - `alembic`：数据库迁移工具，与 SQLAlchemy 配套。
  - `tortoise-orm`：如需要轻量级异步 ORM，可在项目中统一使用 Tortoise，而不要混用多种 ORM。

- **缓存与存储**
  - `redis`（4.x+，支持 asyncio）：统一用作 Redis 客户端，不再新引入旧版 `aioredis`；若已有存量 `aioredis` 代码，可在迁移期内保持一致后逐步替换。
  - `aiocache`：统一用于异步缓存（如内存缓存或 Redis 缓存），避免在不同模块引入多种缓存实现。
  - `motor`：统一用于 MongoDB 异步客户端（`AsyncIOMotorClient`）。

- **文件与配置**
  - `python-dotenv`：统一用于加载 `.env` / `.env.example` 中的环境变量。
  - `aiofiles`：统一用于异步文件读写，避免在 `async` 上下文中使用同步 `open`。

> 上述依赖清单可根据项目实际情况增删，但约定的原则是：**同一类能力（HTTP 客户端、Redis 客户端、ORM 等）项目内尽量只保留一套主方案**，减少心智负担和维护成本。

## 资源

- [FastAPI Async Documentation](https://fastapi.tiangolo.com/async/)
- [Python asyncio Documentation](https://docs.python.org/3/library/asyncio.html)
- [SQLAlchemy Async Guide](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)
- [HTTPX Async Client](https://www.python-httpx.org/async/)
- [AsyncPG Documentation](https://magicstack.github.io/asyncpg/)
- [Motor (MongoDB Async)](https://motor.readthedocs.io/)
- [WebSockets in FastAPI](https://fastapi.tiangolo.com/advanced/websockets/)
- [Server-Sent Events with Starlette](https://github.com/sysid/sse-starlette)

## 注意事项

1. **关于文档输出**：默认不要主动生成文档内容，除非用户明确提出需求。后端服务的启动步骤说明以及 `README.md` 文档属于例外场景，在相关上下文中可以按需输出。
2. **关于测试代码**：默认不要编写或输出测试代码（如单元测试、集成测试等）。只有在用户明确要求提供测试代码时，才输出，并且需在回复中**显式说明**「下面的是测试代码」或含义等价的提示。
