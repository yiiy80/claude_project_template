# 后端规范 (FastAPI)

## 概述

后端采用 FastAPI 框架，基于 Python 的现代化异步 Web 框架，注重性能、类型安全和开发体验。

## 项目结构

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI 应用入口
│   ├── config.py            # 配置管理
│   ├── database.py          # 数据库连接
│   ├── models/              # 数据模型
│   │   ├── __init__.py
│   │   └── user.py          # 用户模型
│   ├── schemas/             # Pydantic 模式
│   │   ├── __init__.py
│   │   └── user.py          # 用户模式
│   ├── routers/             # API 路由
│   │   ├── __init__.py
│   │   ├── users.py         # 用户路由
│   │   └── auth.py          # 认证路由
│   ├── dependencies/        # 依赖注入
│   │   ├── __init__.py
│   │   ├── auth.py          # 认证依赖
│   │   └── database.py      # 数据库依赖
│   ├── services/            # 业务逻辑层
│   │   ├── __init__.py
│   │   └── user_service.py  # 用户服务
│   ├── utils/               # 工具函数
│   │   ├── __init__.py
│   │   └── security.py      # 安全工具
│   └── exceptions/          # 自定义异常
│       ├── __init__.py
│       └── http_exceptions.py
├── tests/                   # 测试文件
│   ├── __init__.py
│   ├── conftest.py          # 测试配置
│   └── test_users.py        # 用户测试
├── alembic/                 # 数据库迁移
│   ├── versions/
│   ├── env.py
│   └── alembic.ini
├── requirements.txt         # 依赖列表
└── README.md
```

## FastAPI 应用配置

### 应用入口

```python
# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import users, auth
from app.database import create_tables

app = FastAPI(
    title="项目API",
    description="项目后端API文档",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS 配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # 前端开发服务器
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 路由注册
app.include_router(auth.router, prefix="/auth", tags=["认证"])
app.include_router(users.router, prefix="/users", tags=["用户"])

# 启动事件
@app.on_event("startup")
async def startup_event():
    create_tables()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 配置管理

```python
# config.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    # 数据库配置
    database_url: str = "sqlite:///./app.db"

    # JWT 配置
    secret_key: str = "your-secret-key"
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    # CORS 配置
    cors_origins: list = ["http://localhost:3000"]

    class Config:
        env_file = ".env"

settings = Settings()
```

## 数据模型设计

### SQLModel 使用规范

```python
# models/user.py
from sqlmodel import SQLModel, Field, Relationship
from typing import Optional, List
from datetime import datetime

class User(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    email: str = Field(unique=True, index=True)
    username: str = Field(unique=True, index=True)
    hashed_password: str
    full_name: Optional[str] = None
    is_active: bool = Field(default=True)
    is_superuser: bool = Field(default=False)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # 关联关系
    posts: List["Post"] = Relationship(back_populates="author")

class Post(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    title: str
    content: str
    author_id: int = Field(foreign_key="user.id")
    created_at: datetime = Field(default_factory=datetime.utcnow)

    # 关联关系
    author: User = Relationship(back_populates="posts")
```

## Pydantic 模式定义

### 请求和响应模式

```python
# schemas/user.py
from pydantic import BaseModel, EmailStr
from typing import Optional
from datetime import datetime

# 基础用户模式
class UserBase(BaseModel):
    email: EmailStr
    username: str
    full_name: Optional[str] = None

# 用户创建模式
class UserCreate(UserBase):
    password: str

# 用户更新模式
class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    username: Optional[str] = None
    full_name: Optional[str] = None
    is_active: Optional[bool] = None

# 用户响应模式
class User(UserBase):
    id: int
    is_active: bool
    is_superuser: bool
    created_at: datetime
    updated_at: datetime

    class Config:
        orm_mode = True

# 用户列表响应
class UserList(BaseModel):
    users: List[User]
    total: int
    page: int
    size: int
```

## API 路由设计

### RESTful API 设计

```python
# routers/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session
from typing import List
from app import models, schemas
from app.dependencies import get_db, get_current_user
from app.services import user_service

router = APIRouter()

@router.get("/", response_model=List[schemas.User])
async def get_users(
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """获取用户列表"""
    if not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="权限不足")

    users = user_service.get_users(db, skip=skip, limit=limit)
    return users

@router.post("/", response_model=schemas.User)
async def create_user(
    user: schemas.UserCreate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """创建新用户"""
    if not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="权限不足")

    # 检查用户名是否已存在
    db_user = user_service.get_user_by_username(db, username=user.username)
    if db_user:
        raise HTTPException(status_code=400, detail="用户名已存在")

    # 检查邮箱是否已存在
    db_user = user_service.get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="邮箱已存在")

    return user_service.create_user(db=db, user=user)

@router.get("/{user_id}", response_model=schemas.User)
async def get_user(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """获取单个用户"""
    user = user_service.get_user(db, user_id=user_id)
    if user is None:
        raise HTTPException(status_code=404, detail="用户不存在")

    # 用户只能查看自己的信息，除非是超级用户
    if user.id != current_user.id and not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="权限不足")

    return user

@router.put("/{user_id}", response_model=schemas.User)
async def update_user(
    user_id: int,
    user_update: schemas.UserUpdate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """更新用户信息"""
    user = user_service.get_user(db, user_id=user_id)
    if user is None:
        raise HTTPException(status_code=404, detail="用户不存在")

    # 用户只能更新自己的信息，除非是超级用户
    if user.id != current_user.id and not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="权限不足")

    return user_service.update_user(db=db, user=user, user_update=user_update)

@router.delete("/{user_id}")
async def delete_user(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """删除用户"""
    if not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="权限不足")

    user = user_service.get_user(db, user_id=user_id)
    if user is None:
        raise HTTPException(status_code=404, detail="用户不存在")

    user_service.delete_user(db=db, user=user)
    return {"message": "用户已删除"}
```

## 依赖注入

### 数据库依赖

```python
# dependencies/database.py
from sqlalchemy.orm import Session
from app.database import SessionLocal

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 认证依赖

```python
# dependencies/auth.py
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt
from app import models, schemas
from app.config import settings
from app.services import user_service

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/token")

def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> models.User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )

    try:
        payload = jwt.decode(token, settings.secret_key, algorithms=[settings.algorithm])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = schemas.TokenData(username=username)
    except JWTError:
        raise credentials_exception

    user = user_service.get_user_by_username(db, username=token_data.username)
    if user is None:
        raise credentials_exception

    if not user.is_active:
        raise HTTPException(status_code=400, detail="用户已被禁用")

    return user

def get_current_active_superuser(
    current_user: models.User = Depends(get_current_user),
) -> models.User:
    if not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN, detail="权限不足"
        )
    return current_user
```

## 业务逻辑层

### 服务类设计

```python
# services/user_service.py
from sqlalchemy.orm import Session
from app import models, schemas
from app.utils.security import get_password_hash, verify_password

class UserService:
    @staticmethod
    def get_users(db: Session, skip: int = 0, limit: int = 100):
        return db.query(models.User).offset(skip).limit(limit).all()

    @staticmethod
    def get_user(db: Session, user_id: int):
        return db.query(models.User).filter(models.User.id == user_id).first()

    @staticmethod
    def get_user_by_email(db: Session, email: str):
        return db.query(models.User).filter(models.User.email == email).first()

    @staticmethod
    def get_user_by_username(db: Session, username: str):
        return db.query(models.User).filter(models.User.username == username).first()

    @staticmethod
    def create_user(db: Session, user: schemas.UserCreate):
        hashed_password = get_password_hash(user.password)
        db_user = models.User(
            email=user.email,
            username=user.username,
            hashed_password=hashed_password,
            full_name=user.full_name
        )
        db.add(db_user)
        db.commit()
        db.refresh(db_user)
        return db_user

    @staticmethod
    def update_user(db: Session, user: models.User, user_update: schemas.UserUpdate):
        update_data = user_update.dict(exclude_unset=True)
        for field, value in update_data.items():
            setattr(user, field, value)
        db.commit()
        db.refresh(user)
        return user

    @staticmethod
    def delete_user(db: Session, user: models.User):
        db.delete(user)
        db.commit()

    @staticmethod
    def authenticate_user(db: Session, username: str, password: str):
        user = UserService.get_user_by_username(db, username)
        if not user:
            return False
        if not verify_password(password, user.hashed_password):
            return False
        return user

user_service = UserService()
```

## 安全工具

### 密码处理

```python
# utils/security.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from app.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证密码"""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """获取密码哈希"""
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    """创建访问令牌"""
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.access_token_expire_minutes)

    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.secret_key, algorithm=settings.algorithm)
    return encoded_jwt
```

## 异常处理

### 自定义异常

```python
# exceptions/http_exceptions.py
from fastapi import HTTPException, status

class UserNotFoundException(HTTPException):
    def __init__(self, user_id: int):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"用户 {user_id} 不存在"
        )

class UserAlreadyExistsException(HTTPException):
    def __init__(self, field: str, value: str):
        super().__init__(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"{field} '{value}' 已存在"
        )

class InsufficientPermissionsException(HTTPException):
    def __init__(self):
        super().__init__(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="权限不足"
        )

class InvalidCredentialsException(HTTPException):
    def __init__(self):
        super().__init__(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="用户名或密码错误",
            headers={"WWW-Authenticate": "Bearer"}
        )
```

## 数据库配置

### SQLModel 配置

```python
# database.py
from sqlmodel import SQLModel, create_engine, Session
from app.config import settings

# 创建数据库引擎
engine = create_engine(
    settings.database_url,
    connect_args={"check_same_thread": False}  # SQLite 特有配置
)

def create_tables():
    """创建所有表"""
    SQLModel.metadata.create_all(bind=engine)

def get_session():
    """获取数据库会话"""
    with Session(engine) as session:
        yield session

# 创建会话
SessionLocal = Session(engine)
```

## 测试规范

### 测试配置

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.database import Base
from app.config import settings

# 测试数据库
TEST_DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(TEST_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@pytest.fixture(scope="function")
def db():
    """测试数据库会话"""
    Base.metadata.create_all(bind=engine)
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(bind=engine)
```

### API 测试示例

```python
# tests/test_users.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_user():
    """测试创建用户"""
    user_data = {
        "email": "test@example.com",
        "username": "testuser",
        "password": "testpassword",
        "full_name": "Test User"
    }

    response = client.post("/users/", json=user_data)
    assert response.status_code == 201

    data = response.json()
    assert data["email"] == user_data["email"]
    assert data["username"] == user_data["username"]
    assert "id" in data

def test_get_users():
    """测试获取用户列表"""
    response = client.get("/users/")
    assert response.status_code == 200

    data = response.json()
    assert isinstance(data, list)
```

## 部署和性能优化

### Uvicorn 配置

```bash
# 生产环境启动命令
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### 性能优化建议

1. **数据库优化**
   - 合理使用索引
   - 避免 N+1 查询问题
   - 使用连接池

2. **缓存策略**
   - 使用 Redis 缓存热点数据
   - API 响应缓存

3. **异步处理**
   - 使用异步数据库操作
   - 后台任务处理

4. **监控和日志**
   - 结构化日志记录
   - 性能指标监控
   - 错误追踪

## 代码规范

### Black 代码格式化

```ini
# pyproject.toml
[tool.black]
line-length = 88
target-version = ['py39']
include = '\.pyi?$'
extend-exclude = '''
/(
  # directories
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | build
  | dist
)/
'''
```

### isort 导入排序

```ini
# pyproject.toml
[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88
known_first_party = ["app"]
```

### 命名规范

- 类名: PascalCase (如 `UserService`)
- 函数名: snake_case (如 `get_user_by_id`)
- 变量名: snake_case (如 `user_data`)
- 常量: UPPER_CASE (如 `MAX_RETRY_COUNT`)

- 文件名: snake_case (如 `user_service.py`)
