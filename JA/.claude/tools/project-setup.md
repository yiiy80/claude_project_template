# プロジェクト足場指令

## 概要

このガイドは React + FastAPI + SQLite の全栈プロジェクト向けの完全なプロジェクト初期化と開発環境構築指令を提供します。

## 前提条件

### システム要件
- **OS**: Windows 10/11, macOS 10.15+, Ubuntu 18.04+
- **Node.js**: 18.0 以上
- **Python**: 3.9 以上
- **Git**: 2.0 以上

### 環境チェック

#### Node.js と npm のチェック
```bash
node --version
npm --version
```

#### Python と pip のチェック
```bash
python --version
pip --version
```

#### Git のチェック
```bash
git --version
```

## プロジェクト初期化

### 1. プロジェクトディレクトリ構造の作成

```bash
# プロジェクトルートディレクトリを作成
mkdir my-fullstack-app
cd my-fullstack-app

# フロントエンドディレクトリを作成
mkdir frontend
cd frontend

# フロントエンドプロジェクトを初期化
npm create vite@latest . -- --template react-ts
cd ..

# バックエンドディレクトリを作成
mkdir backend
cd backend

# バックエンドプロジェクトを初期化
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 基礎依存関係をインストール
pip install fastapi uvicorn sqlmodel alembic python-multipart

# プロジェクト構造を作成
mkdir -p app/{models,routers,dependencies,services,utils,exceptions}
mkdir -p app/{models,routers,dependencies,services,utils,exceptions}
mkdir tests

# 基礎ファイルを作成
touch app/__init__.py
touch app/main.py
touch app/config.py
touch app/database.py
touch requirements.txt
touch README.md

cd ..
```

### 2. Git リポジトリの初期化

```bash
# Git を初期化
git init

# .gitignore を作成
cat > .gitignore << EOF
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# 仮想環境
venv/
env/
ENV/
.venv/
.env

# Node.js
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.npm
.yarn-integrity

# SQLite データベース
*.db
*.sqlite
*.sqlite3

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# ログ
*.log
logs/
EOF

# 初期コミット
git add .
git commit -m "Initial commit: Project setup"
```

## フロントエンド設定 (React + Vite + TypeScript + Tailwind)

### 1. 依存関係のインストール

```bash
cd frontend

# 核心依存関係をインストール
npm install

# 追加依存関係をインストール
npm install axios react-router-dom @types/react-router-dom
npm install @headlessui/react @heroicons/react  # UI コンポーネントライブラリ
npm install react-hook-form @hookform/resolvers zod  # フォーム処理
npm install @tanstack/react-query  # データ取得
npm install lucide-react  # アイコンライブラリ

# 開発依存関係をインストール
npm install -D tailwindcss postcss autoprefixer
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
npm install -D prettier eslint-config-prettier eslint-plugin-prettier
npm install -D @testing-library/react @testing-library/jest-dom
```

### 2. Tailwind CSS の設定

```bash
# Tailwind を初期化
npx tailwindcss init -p

# tailwind.config.js を設定
cat > tailwind.config.js << 'EOF'
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        },
      },
      fontFamily: {
        sans: ['Inter', 'ui-sans-serif', 'system-ui'],
      },
    },
  },
  plugins: [],
}
EOF
```

### 3. ESLint と Prettier の設定

```javascript
// .eslintrc.js
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:prettier/recommended',
  ],
  parser: '@typescript-eslint/parser',
  plugins: ['react', '@typescript-eslint'],
  rules: {
    'react/react-in-jsx-scope': 'off',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
}
```

```javascript
// .prettierrc.js
module.exports = {
  semi: false,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 100,
  tabWidth: 2,
}
```

### 4. Vite の設定

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
})
```

### 5. TypeScript の設定

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 6. 基礎プロジェクト構造の設定

```bash
# ディレクトリ構造を作成
mkdir -p src/{components,hooks,lib,pages,services,types,utils}

# 基礎ファイルを作成
cat > src/index.css << 'EOF'
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    font-family: 'Inter', system-ui, sans-serif;
  }
}
EOF

cat > src/lib/api.ts << 'EOF'
import axios from 'axios'

const api = axios.create({
  baseURL: '/api',
  timeout: 10000,
})

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default api
EOF

cat > src/types/index.ts << 'EOF'
export interface User {
  id: number
  username: string
  email: string
  full_name?: string
  is_active: boolean
  created_at: string
}

export interface ApiResponse<T> {
  success: boolean
  message: string
  data: T
  meta?: {
    page?: number
    size?: number
    total?: number
    total_pages?: number
  }
}
EOF
```

## バックエンド設定 (FastAPI + SQLModel)

### 1. Python 環境の設定

```bash
cd backend

# 仮想環境をアクティブ化
source venv/bin/activate  # Windows: venv\Scripts\activate

# pip をアップグレード
pip install --upgrade pip

# 基礎依存関係をインストール
pip install fastapi uvicorn[standard] sqlmodel alembic python-multipart
pip install python-jose[cryptography] passlib[bcrypt]
pip install python-dotenv

# 開発依存関係をインストール
pip install pytest httpx black isort flake8 mypy

# requirements.txt を生成
pip freeze > requirements.txt
```

### 2. SQLAlchemy と Alembic の設定

```python
# database.py
from sqlmodel import SQLModel, create_engine, Session
from sqlalchemy.pool import StaticPool
import os

DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./app.db")

# エンジンを作成
engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False},
    poolclass=StaticPool,
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

```python
# config.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    database_url: str = "sqlite:///./app.db"
    secret_key: str = "your-secret-key-here"
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    class Config:
        env_file = ".env"

settings = Settings()
```

### 3. Alembic の初期化

```bash
# Alembic を初期化
alembic init alembic

# alembic.ini を設定
cat > alembic.ini << 'EOF'
[alembic]
script_location = alembic
sqlalchemy.url = sqlite:///./app.db

[post_write_hooks]
EOF

# env.py を設定
cat > alembic/env.py << 'EOF'
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
EOF
```

### 4. 基礎モデルの作成

```python
# models/user.py
from sqlmodel import SQLModel, Field
from typing import Optional
from datetime import datetime

class User(SQLModel, table=True):
    __tablename__ = "users"

    id: Optional[int] = Field(default=None, primary_key=True)
    username: str = Field(unique=True, index=True, max_length=50)
    email: str = Field(unique=True, index=True, max_length=255)
    hashed_password: str = Field(max_length=255)
    full_name: Optional[str] = Field(default=None, max_length=100)
    is_active: bool = Field(default=True)
    is_superuser: bool = Field(default=False)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
```

### 5. Pydantic スキーマの作成

```python
# schemas/user.py
from pydantic import BaseModel, EmailStr
from typing import Optional
from datetime import datetime

# ベースユーザースキーマ
class UserBase(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None

# ユーザー作成スキーマ
class UserCreate(UserBase):
    password: str

# ユーザー更新スキーマ
class UserUpdate(BaseModel):
    username: Optional[str] = None
    email: Optional[EmailStr] = None
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

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Optional[str] = None
```

### 6. FastAPI アプリケーションの設定

```python
# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.database import create_tables
from app.routers import users, auth

app = FastAPI(
    title="FullStack App API",
    description="FullStack App Backend API",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS 設定
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
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

### 7. コードフォーマットツールの設定

```ini
# pyproject.toml
[tool.black]
line-length = 88
target-version = ['py39']
include = '\.pyi?$'
extend-exclude = '''
/(
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

[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88
known_first_party = ["app"]
```

## 開発ワークフロー

### 1. 開発サーバーの起動

#### バックエンド開発サーバー
```bash
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

#### フロントエンド開発サーバー
```bash
cd frontend
npm run dev
```

### 2. データベースマイグレーション

```bash
cd backend
source venv/bin/activate

# マイグレーションを作成
alembic revision --autogenerate -m "create users table"

# マイグレーションを実行
alembic upgrade head
```

### 3. コードフォーマットとチェック

#### フロントエンド
```bash
cd frontend
npm run lint
npm run format
```

#### バックエンド
```bash
cd backend
source venv/bin/activate
black .
isort .
flake8 .
```

### 4. テスト実行

#### フロントエンドテスト
```bash
cd frontend
npm test
```

#### バックエンドテスト
```bash
cd backend
source venv/bin/activate
pytest
```

## デプロイ準備

### 1. フロントエンドのビルド

```bash
cd frontend
npm run build
```

### 2. 生産環境の設定

#### 環境変数ファイル
```bash
# .env
DATABASE_URL=sqlite:///./prod.db
SECRET_KEY=your-production-secret-key
DEBUG=False
```

#### 生産サーバー起動
```bash
# Gunicorn + Uvicorn workers を使用
pip install gunicorn
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

## トラブルシューティング

### 一般的な問題

#### 1. ポート競合
```bash
# ポート使用状況を確認
netstat -tulpn | grep :8000
# プロセスを強制終了
kill -9 <PID>
```

#### 2. 依存関係の問題
```bash
# 依存関係をクリーンアップして再インストール
cd backend && rm -rf venv && python -m venv venv && source venv/bin/activate && pip install -r requirements.txt
cd frontend && rm -rf node_modules && npm install
```

#### 3. データベース接続の問題
```bash
# データベースファイルの権限を確認
ls -la *.db
# データベースをリセット
rm app.db && python -c "from app.database import create_tables; create_tables()"
```

### デバッグのヒント

#### SQL ログの有効化
```python
# config.py
class Settings(BaseSettings):
    debug: bool = True
    # ...

# database.py
engine = create_engine(
    DATABASE_URL,
    echo=settings.debug,  # SQL 文を表示
)
```

#### フロントエンドデバッグ
```javascript
// src/lib/api.ts
if (process.env.NODE_ENV === 'development') {
  api.interceptors.request.use((config) => {
    console.log('API Request:', config)
    return config
  })
}
```

## プロジェクトテンプレート

### プロジェクトテンプレートのダウンロード

```bash
# GitHub テンプレートを使用
npx create-react-app frontend --template typescript
# または Vite を使用
npm create vite@latest frontend -- --template react-ts

# バックエンドは FastAPI テンプレートを使用
pip install cookiecutter
cookiecutter https://github.com/tiangolo/full-stack-fastapi-postgresql
```

この足場指令は完全なプロジェクト初期化と設定ガイドを提供し、開発環境の一貫性とベストプラクティスの順守を確保します。
