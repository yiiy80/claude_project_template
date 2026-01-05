# 技术栈总纲

## 概述

本项目采用现代化的全栈技术栈，旨在提供高效的开发体验、良好的类型安全性和优秀的性能表现。

## 核心技术栈

### 前端技术栈
- **React 18+**: 用户界面库，支持最新的并发特性
- **Vite**: 快速的构建工具，提供极佳的开发体验
- **TypeScript**: 提供类型安全和更好的开发体验
- **Tailwind CSS**: 实用优先的CSS框架，支持快速UI开发

### 后端技术栈
- **FastAPI**: 现代化的Python Web框架，基于Starlette和Pydantic
- **Python 3.9+**: 核心编程语言
- **Uvicorn**: ASGI服务器，用于生产环境部署

### 数据库技术栈
- **SQLite**: 轻量级关系型数据库，适合中小型应用
- **SQLModel**: 基于SQLAlchemy和Pydantic的ORM，支持类型提示

## 架构设计

### 前后端分离
```
┌─────────────────┐    HTTP/REST    ┌─────────────────┐
│   Frontend      │◄──────────────►│   Backend       │
│   (React)       │                │   (FastAPI)     │
└─────────────────┘                └─────────────────┘
         │                                 │
         └──────────────► SQLite ◄─────────┘
```

### 目录结构
```
project/
├── frontend/           # React前端应用
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.ts
├── backend/            # FastAPI后端应用
│   ├── app/
│   │   ├── main.py
│   │   ├── models.py
│   │   └── routers/
│   ├── requirements.txt
│   └── alembic/        # 数据库迁移
└── database/           # 数据库文件
    └── app.db
```

## 开发环境要求

### 系统要求
- Node.js 18+
- Python 3.9+
- Git

### 推荐工具
- VS Code (推荐插件: Python, TypeScript, Tailwind CSS)
- Git Bash / Terminal

## 包管理

### 前端依赖
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "typescript": "^5.0.0",
    "tailwindcss": "^3.3.0",
    "vite": "^4.3.0"
  }
}
```

### 后端依赖
```
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlmodel==0.0.14
alembic==1.12.1
pydantic==2.5.0
python-multipart==0.0.6
```

## 开发工作流

### 本地开发
1. 启动后端服务: `uvicorn app.main:app --reload`
2. 启动前端服务: `npm run dev`
3. 访问前端应用，API请求会代理到后端

### 生产部署
1. 构建前端: `npm run build`
2. 部署静态文件到Web服务器
3. 部署FastAPI应用到应用服务器

## 约定和规范

### 代码规范
- 使用ESLint + Prettier (前端)
- 使用Black + isort (后端)
- 遵循各语言的最佳实践

### 版本控制
- 使用Git进行版本控制
- 遵循语义化版本(SemVer)
- 使用Conventional Commits规范

### 测试策略
- 单元测试: Jest (前端), pytest (后端)
- 集成测试: API端点测试
- E2E测试: Playwright 或 Cypress

## 性能优化

### 前端优化
- 代码分割和懒加载
- 图片优化和资源压缩
- 使用React.memo和useMemo优化渲染

### 后端优化
- 数据库查询优化
- 缓存策略
- 异步处理

## 安全考虑

- 输入验证和清理
- CORS配置
- 敏感信息保护
- HTTPS强制使用

## 监控和日志

- 结构化日志记录
- 错误监控和告警
- 性能指标收集
- 用户行为分析
