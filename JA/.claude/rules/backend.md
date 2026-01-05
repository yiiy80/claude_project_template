# バックエンド規範 (FastAPI)

## 概要

バックエンドは FastAPI フレームワークを採用し、パフォーマンス、型安全性、開発体験に重点を置いた現代的な Python 非同期 Web フレームワークです。

## プロジェクト構造

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI アプリケーションエントリ
│   ├── config.py            # 設定管理
│   ├── database.py          # データベース接続
│   ├── models/              # データモデル
│   │   ├── __init__.py
│   │   └── user.py          # ユーザーモデル
│   ├── schemas/             # Pydantic スキーマ
│   │   ├── __init__.py
│   │   └── user.py          # ユーザースキーマ
│   ├── routers/             # API ルーター
│   │   ├── __init__.py
│   │   ├── users.py         # ユーザールーター
│   │   └── auth.py          # 認証ルーター
│   ├── dependencies/        # 依存性注入
│   │   ├── __init__.py
│   │   ├── auth.py          # 認証依存性
│   │   └── database.py      # データベース依存性
│   ├── services/            # ビジネスロジックレイヤー
│   │   ├── __init__.py
│   │   └── user_service.py  # ユーザーサービス
│   ├── utils/               # ユーティリティ関数
│   │   ├── __init__.py
│   │   └── security.py      # セキュリティツール
│   └── exceptions/          # カスタム例外
│       ├── __init__.py
│       └── http_exceptions.py
├── tests/                   # テストファイル
│   ├── __init__.py
│   ├── conftest.py          # テスト設定
│   └── test_users.py        # ユーザーテスト
├── alembic/                 # データベースマイグレーション
│   ├── versions/
│   ├── env.py
│   └── alembic.ini
├── requirements.txt         # 依存関係リスト
└── README.md
```

## FastAPI アプリケーション設定

### アプリケーションエントリ

```python
# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import users, auth
from app.database import create_tables

app = FastAPI(
    title="プロジェクトAPI",
    description="プロジェクトバックエンドAPIドキュメント",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS 設定
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # フロントエンド開発サーバー
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# ルーター登録
app.include_router(auth.router, prefix="/auth", tags=["認証"])
app.include_router(users.router, prefix="/users", tags=["ユーザー"])

# 起動イベント
@app.on_event("startup")
async def startup_event():
    create_tables()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 設定管理

```python
# config.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    # データベース設定
    database_url: str = "sqlite:///./app.db"

    # JWT 設定
    secret_key: str = "your-secret-key"
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    # CORS 設定
    cors_origins: list = ["http://localhost:3000"]

    class Config:
        env_file = ".env"

settings = Settings()
```

## データモデル設計

### SQLModel 使用規範

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

## Pydantic スキーマ定義

### リクエストとレスポンススキーマ

```python
# schemas/user.py
from pydantic import BaseModel, EmailStr
from typing import Optional
from datetime import datetime

# ベースユーザースキーマ
class UserBase(BaseModel):
    email: EmailStr
    username: str
    full_name: Optional[str] = None

# ユーザー作成スキーマ
class UserCreate(UserBase):
    password: str

# ユーザー更新スキーマ
class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    username: Optional[str] = None
    full_name: Optional[str] = None
    is_active: Optional[bool] = None

# ユーザー応答スキーマ
class User(UserBase):
    id: int
    is_active: bool
    is_superuser: bool
    created_at: datetime
    updated_at: datetime

    class Config:
        orm_mode = True

# ユーザーリスト応答
class UserList(BaseModel):
    users: List[User]
    total: int
    page: int
    size: int
```

## API ルーター設計

### RESTful API 設計

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
    """ユーザーリストを取得"""
    if not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="権限が不足しています")

    users = user_service.get_users(db, skip=skip, limit=limit)
    return users

@router.post("/", response_model=schemas.User)
async def create_user(
    user: schemas.UserCreate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """新規ユーザーを作成"""
    if not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="権限が不足しています")

    # ユーザー名が既に存在するかチェック
    db_user = user_service.get_user_by_username(db, username=user.username)
    if db_user:
        raise HTTPException(status_code=400, detail="ユーザー名が既に存在します")

    # メールアドレスが既に存在するかチェック
    db_user = user_service.get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="メールアドレスが既に存在します")

    return user_service.create_user(db=db, user=user)

@router.get("/{user_id}", response_model=schemas.User)
async def get_user(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """単一ユーザーを取得"""
    user = user_service.get_user(db, user_id=user_id)
    if user is None:
        raise HTTPException(status_code=404, detail="ユーザーが存在しません")

    # ユーザーは自分の情報のみ閲覧可能、またはスーパーユーザーの場合
    if user.id != current_user.id and not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="権限が不足しています")

    return user

@router.put("/{user_id}", response_model=schemas.User)
async def update_user(
    user_id: int,
    user_update: schemas.UserUpdate,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """ユーザー情報を更新"""
    user = user_service.get_user(db, user_id=user_id)
    if user is None:
        raise HTTPException(status_code=404, detail="ユーザーが存在しません")

    # ユーザーは自分の情報のみ更新可能、またはスーパーユーザーの場合
    if user.id != current_user.id and not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="権限が不足しています")

    return user_service.update_user(db=db, user=user, user_update=user_update)

@router.delete("/{user_id}")
async def delete_user(
    user_id: int,
    db: Session = Depends(get_db),
    current_user: models.User = Depends(get_current_user)
):
    """ユーザーを削除"""
    if not current_user.is_superuser:
        raise HTTPException(status_code=403, detail="権限が不足しています")

    user = user_service.get_user(db, user_id=user_id)
    if user is None:
        raise HTTPException(status_code=404, detail="ユーザーが存在しません")

    user_service.delete_user(db=db, user=user)
    return {"message": "ユーザーが削除されました"}
```

## 依存性注入

### データベース依存性

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

### 認証依存性

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
        raise HTTPException(status_code=400, detail="ユーザーが無効化されています")

    return user

def get_current_active_superuser(
    current_user: models.User = Depends(get_current_user),
) -> models.User:
    if not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN, detail="権限が不足しています"
        )
    return current_user
```

## ビジネスロジックレイヤー

### サービスクラス設計

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

## セキュリティツール

### パスワード処理

```python
# utils/security.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from app.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """パスワードを検証"""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """パスワードハッシュを取得"""
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    """アクセス令牌を作成"""
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.access_token_expire_minutes)

    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.secret_key, algorithm=settings.algorithm)
    return encoded_jwt
```

## 例外処理

### カスタム例外

```python
# exceptions/http_exceptions.py
from fastapi import HTTPException, status

class UserNotFoundException(HTTPException):
    def __init__(self, user_id: int):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"ユーザー {user_id} が存在しません"
        )

class UserAlreadyExistsException(HTTPException):
    def __init__(self, field: str, value: str):
        super().__init__(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"{field} '{value}' は既に存在します"
        )

class InsufficientPermissionsException(HTTPException):
    def __init__(self):
        super().__init__(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="権限が不足しています"
        )

class InvalidCredentialsException(HTTPException):
    def __init__(self):
        super().__init__(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="ユーザー名またはパスワードが正しくありません",
            headers={"WWW-Authenticate": "Bearer"}
        )
```

## データベース設定

### SQLModel 設定

```python
# database.py
from sqlmodel import SQLModel, create_engine, Session
from app.config import settings

# SQLite 接続 URL
DATABASE_URL = settings.database_url or "sqlite:///./app.db"

# エンジンを作成
engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False}  # SQLite マルチスレッドサポート
)

def create_tables():
    """全てのテーブルを作成"""
    SQLModel.metadata.create_all(bind=engine)

def get_session():
    """データベースセッションを取得"""
    with Session(engine) as session:
        yield session

# グローバルセッション
SessionLocal = Session(engine)
```

## テスト規範

### テスト設定

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.database import Base
from app.config import settings

# テストデータベース
TEST_DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(TEST_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@pytest.fixture(scope="function")
def db():
    """テストデータベースセッション"""
    Base.metadata.create_all(bind=engine)
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(bind=engine)
```

### API テスト例

```python
# tests/test_users.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_user():
    """ユーザー作成をテスト"""
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
    """ユーザーリスト取得をテスト"""
    response = client.get("/users/")
    assert response.status_code == 200

    data = response.json()
    assert isinstance(data, list)
```

## デプロイとパフォーマンス最適化

### Uvicorn 設定

```bash
# 生産環境起動コマンド
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### パフォーマンス最適化提案

1. **データベース最適化**
   - インデックスを合理的に使用
   - N+1 クエリ問題を回避
   - 接続プールを使用

2. **キャッシュ戦略**
   - ホットデータを Redis でキャッシュ
   - API レスポンスキャッシュ

3. **非同期処理**
   - 非同期データベース操作を使用
   - バックグラウンドタスク処理

4. **監視とログ**
   - 構造化ログ記録
   - パフォーマンス指標監視
   - エラートラッキング

## コード規範

### Black コードフォーマット

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

### isort インポートソート

```ini
# pyproject.toml
[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88
known_first_party = ["app"]
```

### 命名規範

- クラス名: PascalCase (例: `UserService`)
- 関数名: snake_case (例: `get_user_by_id`)
- 変数名: snake_case (例: `user_data`)
- 定数: UPPER_CASE (例: `MAX_RETRY_COUNT`)

- ファイル名: snake_case (例: `user_service.py`)
