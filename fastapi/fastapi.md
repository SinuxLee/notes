

项目结构

```py
project/
|
|---app/
	|
    |--- main.py
    |--- api/ 	   # 基于 Starlette(base anyio) 异步Web框架
    |--- model/    # 业务的核心实体
    |--- schema/   # pydantic 基于类型的数据验证，核心验证逻辑用 Rust 开发 
    |--- service/  # 业务逻辑
    |--- database/ # 数据库
```

特性

- 依赖注入
- 身份认证 JWT
- **CORS**, GZip
- Background Task



接口定义

```py
# router 拆分
router = APIRouter()
@router.get('/foo')
async def foo():
    return 'foo'

app.include_route(router, prefix='player')
```





### WSGI（Web Server Gateway Interface）

支持同步的 Web 应用程序

常见的 WSGI 应用框架包括 Django、Flask 和 Bottle

WSGI服务：**uWSGI**, **Gunicorn**

### ASGI（Asynchronous Server Gateway Interface）

支持异步 Web 应用程序，基于 async/await

常见的 ASGI 应用框架包括 FastAPI, Starlette 和 Tornado

ASGI服务：**Uvicorn**, **Hypercorn**, **Daphne**