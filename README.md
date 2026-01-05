# Claude Project Template

A comprehensive project template designed specifically for Claude AI assistants, providing a complete development rule system for full-stack applications.

## 📋 Overview

This template establishes a robust framework for AI-assisted development, featuring:

- **Multi-language Support**: English (primary), Chinese (CN/), Japanese (JA/)
- **Technology Stack**: React + Vite + TypeScript + Tailwind CSS (Frontend) | FastAPI + SQLModel + SQLite (Backend)
- **Modular Architecture**: Clear separation between frontend, backend, and database layers
- **API-First Design**: Contract-driven development with strict interface definitions
- **Type Safety**: Comprehensive TypeScript and Python typing throughout
- **Production Ready**: Security, performance, and deployment guidelines included

## 📁 Project Structure

```
.claude/
├── CLAUDE.md                 # Main entry point with project overview
├── .gitignore               # Git ignore rules for Python, Node.js, SQLite
├── rules/
│   ├── stack.md             # Technology stack overview
│   ├── frontend.md          # Frontend development rules (React/Vite/TS/Tailwind)
│   ├── backend.md           # Backend development rules (FastAPI/SQLModel)
│   ├── database.md          # Database design rules (SQLite/SQLModel/Alembic)
│   └── api-contract.md      # API interface contracts (CRITICAL)
└── tools/
    └── project-setup.md     # Project initialization and setup commands

CN/                           # Chinese language version
├── README.md                 # Chinese documentation
└── .claude/                  # Chinese rule files

JA/                           # Japanese language version
├── README.md                 # Japanese documentation
└── .claude/                  # Japanese rule files
```

## 🎯 Key Features

### 1. Strict Separation of Concerns
- Frontend NEVER imports backend code
- Backend NEVER renders UI
- All communication via REST API
- Clear boundaries between layers

### 2. API-First Development
- `api-contract.md` is the single source of truth
- Document endpoints BEFORE implementation
- TypeScript and Python types stay synchronized
- Version breaking changes properly managed

### 3. Type Safety Everywhere
- No `any` types in TypeScript
- Python type hints required
- SQLModel for type-safe database operations
- Pydantic validation on all inputs

### 4. Autonomous Agent Compatible
- Each component can be developed independently
- Clear interfaces and contracts
- Comprehensive error handling
- Backward compatibility maintained

### 5. Production Ready
- Security best practices
- Performance optimization guidelines
- Testing strategies
- Deployment instructions

## 📚 How to Use

### For New Features
1. Read `api-contract.md` first
2. Document the API endpoint
3. Implement backend (models → routers → services)
4. Implement frontend (types → services → components)
5. Test end-to-end

### For Claude Agents
- Claude will automatically read these rules when working in the project
- Rules enforce best practices and prevent common mistakes
- API contracts ensure frontend/backend synchronization
- Clear guidelines for every layer of the stack

## 🚀 Quick Start

```bash
# Initialize project
./init.sh

# Start development
# Terminal 1 - Backend
cd backend && uvicorn main:app --reload

# Terminal 2 - Frontend
cd frontend && npm run dev
```

## 🌐 Language Versions

- **English** (Primary): Main README and rule files
- **中文 (CN/)**: Complete Chinese translation
- **日本語 (JA/)**: Complete Japanese translation

## 📄 Rule Files

| File | Description |
|------|-------------|
| `CLAUDE.md` | Project overview and rule system entry point |
| `rules/stack.md` | Technology stack architecture and configuration |
| `rules/frontend.md` | React + Vite + TypeScript + Tailwind CSS guidelines |
| `rules/backend.md` | FastAPI framework development standards |
| `rules/database.md` | SQLite + SQLModel database design and optimization |
| `rules/api-contract.md` | Frontend-backend API interface contracts |
| `tools/project-setup.md` | Project initialization and development environment setup |

All files are now in place and ready to guide development! The rules are comprehensive, practical, and designed specifically for long-running autonomous agents working on full-stack applications.
