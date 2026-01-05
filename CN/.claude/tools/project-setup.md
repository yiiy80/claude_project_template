# 项目脚手架指令

## 概述

本指南提供完整的项目初始化和开发环境搭建指令，适用于 React + FastAPI + SQLite 的全栈项目。

## 前提条件

### 系统要求
- **操作系统**: Windows 10/11, macOS 10.15+, Ubuntu 18.04+
- **Node.js**: 18.0 或更高版本
- **Python**: 3.9 或更高版本
- **Git**: 2.0 或更高版本

### 环境检查

#### 检查 Node.js 和 npm
```bash
node --version
npm --version
```

#### 检查 Python 和 pip
```bash
python --version
pip --version
```

#### 检查 Git
```bash
git --version
```

## 项目初始化

### 1. 创建项目目录结构

```bash
# 创建项目根目录
mkdir my-fullstack-app
cd my-fullstack-app

# 创建前端目录
mkdir frontend
cd frontend

# 初始化前端项目
npm create vite@latest . -- --template react-ts
cd ..

# 创建后端目录
mkdir backend
cd backend

# 初始化后端项目
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 安装基础依赖
pip install fastapi uvicorn sqlmodel alembic python-multipart

# 创建项目结构
mkdir -p app/{models,routers,dependencies,services,utils,exceptions}
mkdir -p app/{models,routers,dependencies,services,utils,exceptions}
mkdir tests

# 创建基础文件
touch app/__init__.py
touch app/main.py
touch app/config.py
touch app/database.py
touch requirements.txt
touch README.md

cd ..
```

### 2. 初始化 Git 仓库

```bash
# 初始化 Git
git init

# 创建 .gitignore
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

# Virtual environments
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

# SQLite databases
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

# Logs
*.log
logs/
EOF

# 初始提交
git add .
git commit -m "Initial commit: Project setup"
```

## 前端配置 (React + Vite + TypeScript + Tailwind)

### 1. 安装依赖

```bash
cd frontend

# 安装核心依赖
npm install

# 安装额外依赖
npm install axios react-router-dom @types/react-router-dom
npm install @headlessui/react @heroicons/react  # UI 组件库
npm install react-hook-form @hookform/resolvers zod  # 表单处理
npm install @tanstack/react-query  # 数据获取
npm install lucide-react  # 图标库

# 安装开发依赖
npm install -D tailwindcss postcss autoprefixer
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
npm install -D prettier eslint-config-prettier eslint-plugin-prettier
npm install -D @testing-library/react @testing-library/jest-dom
```

### 2. 配置 Tailwind CSS

```bash
# 初始化 Tailwind
npx tailwindcss init -p

# 配置 tailwind.config.js
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
    },
  },
  plugins: [],
}
EOF
```

### 3. 配置 ESLint 和 Prettier

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

### 4. 配置 Vite

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

### 5. 配置 TypeScript

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

### 6. 设置基础项目结构

```bash
# 创建目录结构
mkdir -p src/{components,hooks,lib,pages,services,types,utils}

# 创建基础文件
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

## 后端配置 (FastAPI + SQLModel)

### 1. 配置 Python 环境

```bash
cd backend

# 激活虚拟环境
source venv/bin/activate  # Windows: venv\Scripts\activate

# 升级 pip
pip install --upgrade pip

# 安装基础依赖
pip install fastapi uvicorn[standard] sqlmodel alembic python-multipart
pip install python-jose[cryptography] passlib[bcrypt]
pip install python-dotenv

# 安装开发依赖
pip install pytest httpx black isort flake8 mypy

# 生成 requirements.txt
pip freeze > requirements.txt
```

### 2. 配置 SQLAlchemy 和 Alembic

```python
# database.py
from sqlmodel import SQLModel, create_engine
from sqlalchemy.pool import StaticPool
import os

DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./app.db")

engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False},
    poolclass=StaticPool,
)

def create_tables():
    SQLModel.metadata.create_all(bind=engine)
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

### 3. 初始化 Alembic

```bash
# 初始化 Alembic
alembic init alembic

# 配置 alembic.ini
cat > alembic.ini << 'EOF'
[alembic]
script_location = alembic
sqlalchemy.url = sqlite:///./app.db

[post_write_hooks]
EOF

# 配置 env.py
cat > alembic/env.py << 'EOF'
import os
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
from sqlmodel import SQLModel

config = context.config
fileConfig(config.config_file_name)

target_metadata = SQLModel.metadata

def run_migrations_offline():
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

### 4. 创建基础模型

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

### 5. 创建 Pydantic 模式

```python
# schemas/user.py
from pydantic import BaseModel, EmailStr
from typing import Optional
from datetime import datetime

class UserBase(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None

class UserCreate(UserBase):
    password: str

class UserUpdate(BaseModel):
    username: Optional[str] = None
    email: Optional[EmailStr] = None
    full_name: Optional[str] = None
    is_active: Optional[bool] = None

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

### 6. 配置 FastAPI 应用

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

# CORS 配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
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

### 7. 配置代码格式化工具

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

## 开发工作流

### 1. 启动开发服务器

#### 后端开发服务器
```bash
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

#### 前端开发服务器
```bash
cd frontend
npm run dev
```

### 2. 数据库迁移

```bash
cd backend
source venv/bin/activate

# 创建迁移
alembic revision --autogenerate -m "create users table"

# 执行迁移
alembic upgrade head
```

### 3. 代码格式化和检查

#### 前端
```bash
cd frontend
npm run lint
npm run format
```

#### 后端
```bash
cd backend
source venv/bin/activate
black .
isort .
flake8 .
```

### 4. 运行测试

#### 前端测试
```bash
cd frontend
npm test
```

#### 后端测试
```bash
cd backend
source venv/bin/activate
pytest
```

## 部署准备

### 1. 构建前端

```bash
cd frontend
npm run build
```

### 2. 配置生产环境

#### 环境变量文件
```bash
# .env
DATABASE_URL=sqlite:///./prod.db
SECRET_KEY=your-production-secret-key
DEBUG=False
```

#### 生产服务器启动
```bash
# 使用 Gunicorn + Uvicorn workers
pip install gunicorn
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

## 故障排除

### 常见问题

#### 1. 端口冲突
```bash
# 检查端口占用
netstat -tulpn | grep :8000
# 杀死进程
kill -9 <PID>
```

#### 2. 依赖问题
```bash
# 清理并重新安装依赖
cd backend && rm -rf venv && python -m venv venv && source venv/bin/activate && pip install -r requirements.txt
cd frontend && rm -rf node_modules && npm install
```

#### 3. 数据库连接问题
```bash
# 检查数据库文件权限
ls -la *.db
# 重置数据库
rm app.db && python -c "from app.database import create_tables; create_tables()"
```

### 调试技巧

#### 启用 SQL 日志
```python
# config.py
class Settings(BaseSettings):
    debug: bool = True
    # ...

# database.py
engine = create_engine(
    DATABASE_URL,
    echo=settings.debug,  # 显示 SQL 语句
)
```

#### 前端调试
```javascript
// src/lib/api.ts
if (process.env.NODE_ENV === 'development') {
  api.interceptors.request.use((config) => {
    console.log('API Request:', config)
    return config
  })
}
```

## 项目模板

### 下载项目模板

```bash
# 使用 GitHub 模板
npx create-react-app frontend --template typescript
# 或使用 Vite
npm create vite@latest frontend -- --template react-ts

# 后端使用 FastAPI 模板
pip install cookiecutter
cookiecutter https://github.com/tiangolo/full-stack-fastapi-postgresql
```

这个脚手架指令提供了完整的项目初始化和配置指南，确保开发环境的一致性和最佳实践的遵循。
