# Claude 全栈项目规则体系

## 项目概述

这是一个基于 React + FastAPI + SQLite 的全栈应用项目，采用现代化的技术栈和最佳实践。

### 技术栈组合
- **前端**: React + Vite + TypeScript + Tailwind CSS
- **后端**: FastAPI (Python)
- **数据库**: SQLite + SQLModel
- **部署**: 适合中小型应用

### 规则体系结构

```
.claude/
├── CLAUDE.md                 # 主入口 (本文档)
├── .gitignore                # Git忽略规则
├── rules/
│   ├── stack.md              # 技术栈总纲
│   ├── frontend.md           # 前端规范 (React/Vite/TS/Tailwind)
│   ├── backend.md            # 后端规范 (FastAPI)
│   ├── database.md           # 数据库规范 (SQLite/SQLModel)
│   └── api-contract.md       # 前后端接口约定
└── tools/
    └── project-setup.md      # 项目脚手架指令
```

## 使用指南

### 对于 Claude (AI助手)

1. **严格遵循规则**: 在处理任何代码修改或新增功能时，必须先查阅相关规则文件
2. **前后端分离**: 明确区分前端和后端的职责边界，不得混淆
3. **接口约定优先**: API接口的设计必须遵循 `api-contract.md` 的约定
4. **最佳实践**: 遵循各技术栈的最佳实践和项目规范

### 对于开发者

1. **规则同步**: 开发前请阅读相关规则文件
2. **代码审查**: 确保新代码符合规则要求
3. **文档更新**: 规则更新时及时同步到代码实现

## 核心原则

### 1. 前后端分离
- 前端专注于用户界面和交互逻辑
- 后端专注于业务逻辑和数据处理
- 通过RESTful API进行通信

### 2. 类型安全
- 前端使用TypeScript确保类型安全
- 后端使用Pydantic进行数据验证

### 3. 现代化开发
- 使用最新的工具和框架
- 遵循行业最佳实践
- 注重开发体验和代码质量

### 4. 可维护性
- 清晰的代码结构
- 完善的文档
- 一致的命名规范

## 快速开始

1. 阅读 `rules/stack.md` 了解技术栈总纲
2. 根据需求查阅对应的规则文件
3. 参考 `tools/project-setup.md` 进行项目初始化

## 注意事项

- 本规则体系专为本项目定制，如有特殊需求可适当调整
- 规则会根据项目发展不断完善
- 建议定期review和更新规则内容
