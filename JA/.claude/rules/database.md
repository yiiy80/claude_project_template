# データベース規範 (SQLite/SQLModel)

## 概要

プロジェクトは SQLite をデータベースとして使用し、SQLModel ORM でデータモデリングと操作を行います。SQLite は軽量でゼロ設定で、中小規模アプリケーションの開発とデプロイに適しています。

## データベース設計原則

### 1. 正規化設計
- **第一正規形 (1NF)**: 各列が原子値であることを確保
- **第二正規形 (2NF)**: 部分依存を排除
- **第三正規形 (3NF)**: 推移的依存を排除

### 2. 命名規範
- テーブル名: snake_case (例: `user_profiles`, `product_categories`)
- 列名: snake_case (例: `user_id`, `created_at`, `is_active`)
- 主キー: `id` (自動増分整数)
- 外部キー: `{table_name}_id` (例: `user_id`, `category_id`)

### 3. 必須フィールド
各テーブルには以下の監査フィールドを含める：
- `created_at`: 作成時間 (datetime)
- `updated_at`: 更新時間 (datetime)

## SQLModel データモデリング

### モデル定義規範

```python
# models/user.py
from sqlmodel import SQLModel, Field, Relationship
from typing import Optional, List
from datetime import datetime

class User(SQLModel, table=True):
    __tablename__ = "users"  # テーブル名を明示的に指定

    # 主キー
    id: Optional[int] = Field(default=None, primary_key=True)

    # 必須フィールド
    email: str = Field(unique=True, index=True, max_length=255)
    username: str = Field(unique=True, index=True, max_length=50)
    hashed_password: str = Field(max_length=255)

    # オプションフィールド
    full_name: Optional[str] = Field(default=None, max_length=100)
    bio: Optional[str] = Field(default=None)
    avatar_url: Optional[str] = Field(default=None)

    # ステータスフィールド
    is_active: bool = Field(default=True)
    is_superuser: bool = Field(default=False)

    # 監査フィールド
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # 関連関係
    posts: List["Post"] = Relationship(back_populates="author")
    comments: List["Comment"] = Relationship(back_populates="author")

class Post(SQLModel, table=True):
    __tablename__ = "posts"

    id: Optional[int] = Field(default=None, primary_key=True)
    title: str = Field(max_length=200)
    content: str
    published: bool = Field(default=False)

    # 外部キー
    author_id: int = Field(foreign_key="users.id")

    # 監査フィールド
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # 関連関係
    author: User = Relationship(back_populates="posts")
    comments: List["Comment"] = Relationship(back_populates="post")

class Comment(SQLModel, table=True):
    __tablename__ = "comments"

    id: Optional[int] = Field(default=None, primary_key=True)
    content: str = Field(max_length=1000)

    # 外部キー
    author_id: int = Field(foreign_key="users.id")
    post_id: int = Field(foreign_key="posts.id")

    # 監査フィールド
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # 関連関係
    author: User = Relationship(back_populates="comments")
    post: Post = Relationship(back_populates="comments")
```

## データベース設定

### 接続設定

```python
# database.py
from sqlmodel import SQLModel, create_engine, Session
from sqlalchemy.pool import StaticPool
from app.config import settings

# SQLite 接続 URL
DATABASE_URL = settings.database_url or "sqlite:///./app.db"

# エンジンを作成
engine = create_engine(
    DATABASE_URL,
    connect_args={
        "check_same_thread": False,  # SQLite マルチスレッドサポート
    },
    poolclass=StaticPool,  # 接続プール設定
    echo=settings.debug,   # 開発環境で SQL 文を表示
)

def create_tables():
    """全てのデータテーブルを作成"""
    SQLModel.metadata.create_all(bind=engine)

def get_session():
    """データベースセッションを取得"""
    with Session(engine) as session:
        yield session

# グローバルセッション
SessionLocal = Session(engine)
```

### Alembic マイグレーション設定

```python
# alembic/env.py
import os
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
from sqlmodel import SQLModel

# 全てのモデルをインポート
from app.models import User, Post, Comment  # 全てのモデルをインポート

config = context.config
fileConfig(config.config_file_name)

target_metadata = SQLModel.metadata

def run_migrations_offline():
    """オフラインモードでマイグレーションを実行"""
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
    """オンラインモードでマイグレーションを実行"""
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

## データアクセスレイヤー

### Repository パターン

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

## クエリ最適化

### インデックス戦略

```python
# 自動作成されるインデックス
class Article(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    title: str = Field(index=True)  # 自動的にインデックスを作成
    author_id: int = Field(foreign_key="users.id", index=True)  # 外部キーは自動的にインデックス
    published: bool = Field(default=False)
    published_at: Optional[datetime] = Field(default=None, index=True)
    category: str = Field(index=True)

# 複合インデックス
from sqlalchemy import Index
Index('idx_article_category_published', Article.category, Article.published)
```

### 関連データのプリロード

```python
# N+1 クエリ問題を解決
from sqlalchemy.orm import selectinload, joinedload

# 方法1: selectinload - 一対多関係に適する
def get_users_with_posts(db: Session):
    return db.query(User).options(selectinload(User.posts)).all()

# 方法2: joinedload - 一対一または多対一関係に適する
def get_posts_with_authors(db: Session):
    return db.query(Post).options(joinedload(Post.author)).all()

# 方法3: 手動プリロード
def get_user_with_posts(db: Session, user_id: int):
    user = db.query(User).filter(User.id == user_id).first()
    if user:
        # 関連データを明示的に読み込み
        db.query(Post).filter(Post.author_id == user_id).all()
    return user
```

### ページングクエリ

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
        # ページサイズを制限
        size = min(size, max_size)

        # オフセットを計算
        offset = (page - 1) * size

        # 総数を計算
        total = query.count()

        # ページングデータを取得
        items = query.offset(offset).limit(size).all()

        # 総ページ数を計算
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

## データ検証

### Pydantic 統合

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
            raise ValueError('タイトルを空にすることはできません')
        if len(v) > 200:
            raise ValueError('タイトルの長さは200文字を超えることはできません')
        return v

    @validator('content')
    def content_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError('内容を空にすることはできません')
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

## トランザクション管理

### 明示的トランザクション

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
        """記事作成時に著者の権限をチェック - トランザクションを使用"""
        try:
            # トランザクションを開始
            with self.db.begin():
                # 著者が存在し、アクティブであるかをチェック
                author = self.user_repo.get(author_id)
                if not author or not author.is_active:
                    raise ValueError("無効な著者")

                # カテゴリが許可されているかをチェック
                if article.category not in ALLOWED_CATEGORIES:
                    raise ValueError("サポートされていないカテゴリ")

                # 記事を作成
                article_data = article.dict()
                article_data['author_id'] = author_id
                new_article = self.article_repo.create(article_data)

                # 著者の記事数を更新 (フィールドがあると仮定)
                # author.article_count += 1
                # self.user_repo.update(author, {'article_count': author.article_count})

            return new_article

        except Exception as e:
            # トランザクションは自動的にロールバックされる
            raise e
```

### 依存性注入におけるトランザクション

```python
# dependencies/database.py
from sqlalchemy.orm import Session
from app.database import SessionLocal

def get_db():
    """依存性注入データベースセッション"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
        # 注意: ここではコミット/ロールバックを明示的に行わない
        # FastAPI が自動的にトランザクションを処理する

# routers/articles.py
@router.post("/", response_model=schemas.Article)
async def create_article(
    article: schemas.ArticleCreate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    # ここで実行される全てのデータベース操作は同じトランザクション内にある
    # 例外が発生した場合、トランザクションは自動的にロールバックされる
    return article_service.create_article(db, article, current_user.id)
```

## データベースマイグレーション

### マイグレーション作成

```bash
# Alembic を初期化
alembic init alembic

# マイグレーションファイルを作成
alembic revision --autogenerate -m "create users table"

# マイグレーションを実行
alembic upgrade head

# マイグレーションをロールバック
alembic downgrade -1
```

### マイグレーションファイル例

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

## バックアップと復元

### SQLite バックアップ戦略

```python
# utils/backup.py
import sqlite3
import shutil
from datetime import datetime
from pathlib import Path
from app.config import settings

def create_backup():
    """データベースバックアップを作成"""
    db_path = Path(settings.database_url.replace("sqlite:///", ""))
    backup_dir = Path("backups")
    backup_dir.mkdir(exist_ok=True)

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_path = backup_dir / f"backup_{timestamp}.db"

    # SQLite バックアップ
    shutil.copy2(db_path, backup_path)

    # 古いバックアップをクリーンアップ (最新10個を保持)
    backups = sorted(backup_dir.glob("backup_*.db"), reverse=True)
    for old_backup in backups[10:]:
        old_backup.unlink()

    return backup_path

def restore_backup(backup_path: str):
    """バックアップからデータベースを復元"""
    db_path = Path(settings.database_url.replace("sqlite:///", ""))
    backup_path = Path(backup_path)

    if not backup_path.exists():
        raise FileNotFoundError(f"バックアップファイルが存在しません: {backup_path}")

    # 復元前に現在のデータベースのバックアップを作成
    create_backup()

    # データベースを復元
    shutil.copy2(backup_path, db_path)

def vacuum_database():
    """データベースを圧縮し、スペースを解放"""
    db_path = settings.database_url.replace("sqlite:///", "")
    conn = sqlite3.connect(db_path)
    conn.execute("VACUUM")
    conn.close()
```

## パフォーマンス監視

### クエリパフォーマンス分析

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

        # 遅いクエリを記録
        if process_time > 1.0:  # 1秒を超えるクエリ
            print(f"遅いクエリ: {request.method} {request.url} - {process_time:.2f}s")

        response.headers["X-Process-Time"] = str(process_time)
        return response
```

### データベース接続監視

```python
# monitoring/database.py
from sqlalchemy import event
from app.database import engine

# 接続プール統計
@event.listens_for(engine, "connect")
def connect(dbapi_connection, connection_record):
    print("データベース接続が確立されました")

@event.listens_for(engine, "close")
def close(dbapi_connection, connection_record):
    print("データベース接続が閉じられました")

# クエリ統計
@event.listens_for(engine, "before_execute")
def before_execute(conn, clauseelement, multiparams, params):
    print(f"クエリを実行: {clauseelement}")

@event.listens_for(engine, "after_execute")
def after_execute(conn, clauseelement, multiparams, params, result):
    print("クエリ実行が完了しました")
```

## セキュリティ考慮事項

### SQLインジェクション防止
- 常にパラメータ化クエリを使用 (SQLModel/SQLAlchemy が自動的に処理)
- 文字列連結でSQLを構築しない
- ホワイトリストでユーザー入力を検証

### データ暗号化
- 機密データを暗号化して保存
- パスワードに強力なハッシュアルゴリズムを使用 (bcrypt)
- API Key と機密設定に環境変数を使用

### アクセス制御
- 最小権限の原則を実施
- ロールベースアクセス制御 (RBAC) を使用
- 機密操作の監査ログを記録

## テスト戦略

### データベーステスト

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.database import Base

@pytest.fixture(scope="session")
def test_engine():
    """テストデータベースエンジン"""
    engine = create_engine("sqlite:///./test.db", connect_args={"check_same_thread": False})
    Base.metadata.create_all(bind=engine)
    yield engine
    Base.metadata.drop_all(bind=engine)

@pytest.fixture(scope="function")
def db_session(test_engine):
    """テストデータベースセッション"""
    connection = test_engine.connect()
    transaction = connection.begin()
    session = sessionmaker(bind=connection)()

    yield session

    session.close()
    transaction.rollback()
    connection.close()
```

### モデルテスト

```python
# tests/test_models.py
import pytest
from app.models import User, Post

def test_user_creation(db_session):
    """ユーザー作成をテスト"""
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
    """ユーザーと記事の関連関係をテスト"""
    user = User(
        email="author@example.com",
        username="author",
        hashed_password="hashed_password"
    )
    db_session.add(user)

    post = Post(
        title="テスト記事",
        content="記事内容",
        author_id=user.id
    )
    db_session.add(post)
    db_session.commit()

    # 関連関係を検証
    assert len(user.posts) == 1
    assert user.posts[0].title == "テスト記事"
    assert post.author.username == "author"
