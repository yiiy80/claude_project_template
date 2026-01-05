# 数据库规范 (SQLite/SQLModel)

## 概述

项目使用 SQLite 作为数据库，配合 SQLModel ORM 进行数据建模和操作。SQLite 轻量级、零配置，适合中小型应用开发和部署。

## 数据库设计原则

### 1. 规范化设计
- **第一范式 (1NF)**: 确保每列都是原子值
- **第二范式 (2NF)**: 消除部分依赖
- **第三范式 (3NF)**: 消除传递依赖

### 2. 命名规范
- 表名: snake_case (如 `user_profiles`, `product_categories`)
- 列名: snake_case (如 `user_id`, `created_at`, `is_active`)
- 主键: `id` (自增整数)
- 外键: `{table_name}_id` (如 `user_id`, `category_id`)

### 3. 必备字段
每个表应包含以下审计字段：
- `created_at`: 创建时间 (datetime)
- `updated_at`: 更新时间 (datetime)

## SQLModel 数据建模

### 模型定义规范

```python
# models/user.py
from sqlmodel import SQLModel, Field, Relationship
from typing import Optional, List
from datetime import datetime

class User(SQLModel, table=True):
    __tablename__ = "users"  # 明确指定表名

    # 主键
    id: Optional[int] = Field(default=None, primary_key=True)

    # 必需字段
    email: str = Field(unique=True, index=True, max_length=255)
    username: str = Field(unique=True, index=True, max_length=50)
    hashed_password: str = Field(max_length=255)

    # 可选字段
    full_name: Optional[str] = Field(default=None, max_length=100)
    bio: Optional[str] = Field(default=None)
    avatar_url: Optional[str] = Field(default=None)

    # 状态字段
    is_active: bool = Field(default=True)
    is_superuser: bool = Field(default=False)

    # 审计字段
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # 关联关系
    posts: List["Post"] = Relationship(back_populates="author")
    comments: List["Comment"] = Relationship(back_populates="author")

class Post(SQLModel, table=True):
    __tablename__ = "posts"

    id: Optional[int] = Field(default=None, primary_key=True)
    title: str = Field(max_length=200)
    content: str
    published: bool = Field(default=False)

    # 外键
    author_id: int = Field(foreign_key="users.id")

    # 审计字段
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # 关联关系
    author: User = Relationship(back_populates="posts")
    comments: List["Comment"] = Relationship(back_populates="post")

class Comment(SQLModel, table=True):
    __tablename__ = "comments"

    id: Optional[int] = Field(default=None, primary_key=True)
    content: str = Field(max_length=1000)

    # 外键
    author_id: int = Field(foreign_key="users.id")
    post_id: int = Field(foreign_key="posts.id")

    # 审计字段
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # 关联关系
    author: User = Relationship(back_populates="comments")
    post: Post = Relationship(back_populates="comments")
```

## 数据库配置

### 连接配置

```python
# database.py
from sqlmodel import SQLModel, create_engine, Session
from sqlalchemy.pool import StaticPool
from app.config import settings

# SQLite 连接 URL
DATABASE_URL = settings.database_url or "sqlite:///./app.db"

# 创建引擎
engine = create_engine(
    DATABASE_URL,
    connect_args={
        "check_same_thread": False,  # SQLite 多线程支持
    },
    poolclass=StaticPool,  # 连接池配置
    echo=settings.debug,   # 开发环境显示 SQL 语句
)

def create_tables():
    """创建所有数据表"""
    SQLModel.metadata.create_all(bind=engine)

def get_session():
    """获取数据库会话"""
    with Session(engine) as session:
        yield session

# 全局会话
SessionLocal = Session(engine)
```

### Alembic 迁移配置

```python
# alembic/env.py
import os
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
from sqlmodel import SQLModel

# 导入所有模型
from app.models import User, Post, Comment  # 导入所有模型

config = context.config
fileConfig(config.config_file_name)

target_metadata = SQLModel.metadata

def run_migrations_offline():
    """离线模式运行迁移"""
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online():
    """在线模式运行迁移"""
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

## 数据访问层

### Repository 模式

```python
# repositories/base.py
from typing import Generic, TypeVar, List, Optional
from sqlalchemy.orm import Session
from sqlmodel import SQLModel

ModelType = TypeVar("ModelType", bound=SQLModel)

class BaseRepository(Generic[ModelType]):
    def __init__(self, model: type[ModelType], db: Session):
        self.model = model
        self.db = db

    def get(self, id: int) -> Optional[ModelType]:
        return self.db.get(self.model, id)

    def get_multi(self, skip: int = 0, limit: int = 100) -> List[ModelType]:
        return self.db.query(self.model).offset(skip).limit(limit).all()

    def create(self, obj_in: ModelType) -> ModelType:
        self.db.add(obj_in)
        self.db.commit()
        self.db.refresh(obj_in)
        return obj_in

    def update(self, obj: ModelType, obj_in: dict) -> ModelType:
        for field, value in obj_in.items():
            setattr(obj, field, value)
        self.db.commit()
        self.db.refresh(obj)
        return obj

    def delete(self, id: int) -> bool:
        obj = self.get(id)
        if obj:
            self.db.delete(obj)
            self.db.commit()
            return True
        return False
```

### 具体 Repository

```python
# repositories/user_repository.py
from typing import Optional
from sqlalchemy.orm import Session
from app.models import User
from .base import BaseRepository

class UserRepository(BaseRepository[User]):
    def __init__(self, db: Session):
        super().__init__(User, db)

    def get_by_email(self, email: str) -> Optional[User]:
        return self.db.query(User).filter(User.email == email).first()

    def get_by_username(self, username: str) -> Optional[User]:
        return self.db.query(User).filter(User.username == username).first()

    def get_active_users(self, skip: int = 0, limit: int = 100) -> list[User]:
        return (
            self.db.query(User)
            .filter(User.is_active == True)
            .offset(skip)
            .limit(limit)
            .all()
        )
```

## 查询优化

### 索引策略

```python
# 自动创建的索引
class Article(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    title: str = Field(index=True)  # 自动创建索引
    author_id: int = Field(foreign_key="users.id", index=True)  # 外键自动索引
    published: bool = Field(default=False)
    published_at: Optional[datetime] = Field(default=None, index=True)
    category: str = Field(index=True)

# 复合索引
from sqlalchemy import Index
Index('idx_article_category_published', Article.category, Article.published)
```

### 预加载关联数据

```python
# 解决 N+1 查询问题
from sqlalchemy.orm import selectinload, joinedload

# 方法1: selectinload - 适用于一对多关系
def get_users_with_posts(db: Session):
    return db.query(User).options(selectinload(User.posts)).all()

# 方法2: joinedload - 适用于一对一或多对一关系
def get_posts_with_authors(db: Session):
    return db.query(Post).options(joinedload(Post.author)).all()

# 方法3: 手动预加载
def get_user_with_posts(db: Session, user_id: int):
    user = db.query(User).filter(User.id == user_id).first()
    if user:
        # 显式加载关联数据
        db.query(Post).filter(Post.author_id == user_id).all()
    return user
```

### 分页查询

```python
# repositories/mixins/pagination.py
from typing import List, Dict, Any
from sqlalchemy.orm import Query

class PaginationMixin:
    @staticmethod
    def paginate(
        query: Query,
        page: int = 1,
        size: int = 20,
        max_size: int = 100
    ) -> Dict[str, Any]:
        # 限制每页大小
        size = min(size, max_size)

        # 计算偏移量
        offset = (page - 1) * size

        # 获取总数
        total = query.count()

        # 获取分页数据
        items = query.offset(offset).limit(size).all()

        # 计算总页数
        total_pages = (total + size - 1) // size

        return {
            "items": items,
            "total": total,
            "page": page,
            "size": size,
            "total_pages": total_pages,
            "has_next": page < total_pages,
            "has_prev": page > 1,
        }
```

## 数据验证

### Pydantic 集成

```python
# schemas/article.py
from pydantic import BaseModel, validator
from typing import Optional
from datetime import datetime

class ArticleBase(BaseModel):
    title: str
    content: str
    category: str
    published: bool = False

    @validator('title')
    def title_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError('标题不能为空')
        if len(v) > 200:
            raise ValueError('标题长度不能超过200字符')
        return v

    @validator('content')
    def content_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError('内容不能为空')
        return v

class ArticleCreate(ArticleBase):
    pass

class ArticleUpdate(BaseModel):
    title: Optional[str] = None
    content: Optional[str] = None
    category: Optional[str] = None
    published: Optional[bool] = None

class Article(ArticleBase):
    id: int
    author_id: int
    created_at: datetime
    updated_at: datetime

    class Config:
        orm_mode = True
```

## 事务管理

### 显式事务

```python
# services/article_service.py
from sqlalchemy.orm import Session
from app.repositories import ArticleRepository, UserRepository
from app.schemas import ArticleCreate

class ArticleService:
    def __init__(self, db: Session):
        self.db = db
        self.article_repo = ArticleRepository(db)
        self.user_repo = UserRepository(db)

    def create_article_with_author_check(self, article: ArticleCreate, author_id: int):
        """创建文章时检查作者权限 - 使用事务"""
        try:
            # 开始事务
            with self.db.begin():
                # 检查作者是否存在且活跃
                author = self.user_repo.get(author_id)
                if not author or not author.is_active:
                    raise ValueError("无效的作者")

                # 检查分类是否允许
                if article.category not in ALLOWED_CATEGORIES:
                    raise ValueError("不支持的分类")

                # 创建文章
                article_data = article.dict()
                article_data['author_id'] = author_id
                new_article = self.article_repo.create(article_data)

                # 更新作者的文章计数 (假设有此字段)
                # author.article_count += 1
                # self.user_repo.update(author, {'article_count': author.article_count})

            return new_article

        except Exception as e:
            # 事务会自动回滚
            raise e
```

### 依赖注入中的事务

```python
# dependencies/database.py
from sqlalchemy.orm import Session
from app.database import SessionLocal

def get_db():
    """依赖注入数据库会话"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
        # 注意: 这里没有显式提交/回滚
        # FastAPI 会自动处理事务

# routers/articles.py
@router.post("/", response_model=schemas.Article)
async def create_article(
    article: schemas.ArticleCreate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    # 在这里执行的所有数据库操作都在同一个事务中
    # 如果发生异常，事务会自动回滚
    return article_service.create_article(db, article, current_user.id)
```

## 数据库迁移

### 创建迁移

```bash
# 初始化 alembic
alembic init alembic

# 创建迁移文件
alembic revision --autogenerate -m "create users table"

# 执行迁移
alembic upgrade head

# 回滚迁移
alembic downgrade -1
```

### 迁移文件示例

```python
# alembic/versions/001_create_users_table.py
"""create users table

Revision ID: 001
Revises:
Create Date: 2024-01-01 00:00:00.000000

"""
from alembic import op
import sqlalchemy as sa

revision = '001'
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('email', sa.String(length=255), nullable=False),
        sa.Column('username', sa.String(length=50), nullable=False),
        sa.Column('hashed_password', sa.String(length=255), nullable=False),
        sa.Column('full_name', sa.String(length=100), nullable=True),
        sa.Column('is_active', sa.Boolean(), nullable=False, default=True),
        sa.Column('is_superuser', sa.Boolean(), nullable=False, default=False),
        sa.Column('created_at', sa.DateTime(), nullable=False),
        sa.Column('updated_at', sa.DateTime(), nullable=False),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('email'),
        sa.UniqueConstraint('username')
    )
    op.create_index('ix_users_email', 'users', ['email'], unique=False)
    op.create_index('ix_users_username', 'users', ['username'], unique=False)

def downgrade():
    op.drop_index('ix_users_username', table_name='users')
    op.drop_index('ix_users_email', table_name='users')
    op.drop_table('users')
```

## 备份和恢复

### SQLite 备份策略

```python
# utils/backup.py
import sqlite3
import shutil
from datetime import datetime
from pathlib import Path
from app.config import settings

def create_backup():
    """创建数据库备份"""
    db_path = Path(settings.database_url.replace("sqlite:///", ""))
    backup_dir = Path("backups")
    backup_dir.mkdir(exist_ok=True)

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_path = backup_dir / f"backup_{timestamp}.db"

    # SQLite 备份
    shutil.copy2(db_path, backup_path)

    # 清理旧备份 (保留最近10个)
    backups = sorted(backup_dir.glob("backup_*.db"), reverse=True)
    for old_backup in backups[10:]:
        old_backup.unlink()

    return backup_path

def restore_backup(backup_path: str):
    """从备份恢复数据库"""
    db_path = Path(settings.database_url.replace("sqlite:///", ""))
    backup_path = Path(backup_path)

    if not backup_path.exists():
        raise FileNotFoundError(f"备份文件不存在: {backup_path}")

    # 恢复前创建当前数据库的备份
    create_backup()

    # 恢复数据库
    shutil.copy2(backup_path, db_path)

def vacuum_database():
    """压缩数据库，回收空间"""
    db_path = settings.database_url.replace("sqlite:///", "")
    conn = sqlite3.connect(db_path)
    conn.execute("VACUUM")
    conn.close()
```

## 性能监控

### 查询性能分析

```python
# middleware/performance.py
import time
from fastapi import Request, Response
from starlette.middleware.base import BaseHTTPMiddleware

class PerformanceMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()

        response = await call_next(request)

        process_time = time.time() - start_time

        # 记录慢查询
        if process_time > 1.0:  # 超过1秒的查询
            print(f"慢查询: {request.method} {request.url} - {process_time:.2f}s")

        response.headers["X-Process-Time"] = str(process_time)
        return response
```

### 数据库连接监控

```python
# monitoring/database.py
from sqlalchemy import event
from app.database import engine

# 连接池统计
@event.listens_for(engine, "connect")
def connect(dbapi_connection, connection_record):
    print("数据库连接已建立")

@event.listens_for(engine, "close")
def close(dbapi_connection, connection_record):
    print("数据库连接已关闭")

# 查询统计
@event.listens_for(engine, "before_execute")
def before_execute(conn, clauseelement, multiparams, params):
    print(f"执行查询: {clauseelement}")

@event.listens_for(engine, "after_execute")
def after_execute(conn, clauseelement, multiparams, params, result):
    print("查询执行完成")
```

## 安全考虑

### SQL 注入防护
- 始终使用参数化查询 (SQLModel/SQLAlchemy 自动处理)
- 避免字符串拼接构建 SQL
- 使用白名单验证用户输入

### 数据加密
- 敏感数据使用加密存储
- 密码使用强哈希算法 (bcrypt)
- API Key 和敏感配置使用环境变量

### 访问控制
- 实施最小权限原则
- 使用角色-based 访问控制 (RBAC)
- 审计敏感操作日志

## 测试策略

### 数据库测试

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.database import Base

@pytest.fixture(scope="session")
def test_engine():
    """测试数据库引擎"""
    engine = create_engine("sqlite:///./test.db", connect_args={"check_same_thread": False})
    Base.metadata.create_all(bind=engine)
    yield engine
    Base.metadata.drop_all(bind=engine)

@pytest.fixture(scope="function")
def db_session(test_engine):
    """测试数据库会话"""
    connection = test_engine.connect()
    transaction = connection.begin()
    session = sessionmaker(bind=connection)()

    yield session

    session.close()
    transaction.rollback()
    connection.close()
```

### 模型测试

```python
# tests/test_models.py
import pytest
from app.models import User, Post

def test_user_creation(db_session):
    """测试用户创建"""
    user = User(
        email="test@example.com",
        username="testuser",
        hashed_password="hashed_password"
    )

    db_session.add(user)
    db_session.commit()

    assert user.id is not None
    assert user.email == "test@example.com"
    assert user.is_active == True

def test_user_post_relationship(db_session):
    """测试用户和文章的关联关系"""
    user = User(
        email="author@example.com",
        username="author",
        hashed_password="hashed_password"
    )
    db_session.add(user)

    post = Post(
        title="测试文章",
        content="文章内容",
        author_id=user.id
    )
    db_session.add(post)
    db_session.commit()

    # 验证关联关系
    assert len(user.posts) == 1
    assert user.posts[0].title == "测试文章"
    assert post.author.username == "author"
