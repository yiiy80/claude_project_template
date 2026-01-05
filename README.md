📁 Complete Directory Structure   .claude/
├── CLAUDE.md                 # Main entry point with project overview
├── .gitignore               # Ignores __pycache__, venv, node_modules, *.db, .env
├── rules/
│   ├── stack.md             # Technology stack overview (React, Vite, FastAPI, SQLite)
│   ├── frontend.md          # Frontend rules (React/Vite/TypeScript/Tailwind)
│   ├── backend.md           # Backend rules (FastAPI/SQLModel)
│   ├── database.md          # Database rules (SQLite/SQLModel/Alembic)
│   └── api-contract.md      # API interface contracts (CRITICAL!)
└── tools/
    └── project-setup.md     # Project setup commands and scripts
  🎯 Key Features 1. Strict Separation of Concerns • Frontend NEVER imports backend code • Backend NEVER renders UI • All communication via REST API • Clear boundaries between layers  2. API-First Development • api-contract.md is the single source of truth • Document endpoints BEFORE implementation • TypeScript and Python types stay in sync • Version breaking changes properly  3. Type Safety Everywhere • No any types in TypeScript • Python type hints required • SQLModel for type-safe database operations • Pydantic validation on all inputs  4. Autonomous Agent Compatible • Each component can be developed independently • Clear interfaces and contracts • Comprehensive error handling • Backward compatibility maintained  5. Production Ready • Security best practices • Performance optimization guidelines • Testing strategies • Deployment instructions  📚 How to Use For New Features: 1. Read api-contract.md first 2. Document the API endpoint 3. Implement backend (models → routers → services) 4. Implement frontend (types → services → components) 5. Test end-to-end  For Claude Agents: • Claude will automatically read these rules when working in the project • Rules enforce best practices and prevent common mistakes • API contracts ensure frontend/backend stay in sync • Clear guidelines for every layer of the stack  🚀 Quick Start   # Initialize project
./init.sh

# Start development
cd backend && uvicorn main:app --reload  # Terminal 1
cd frontend && npm run dev                # Terminal 2
  All files are now in place and ready to guide development! The rules are comprehensive, practical, and designed specifically for long-running autonomous agents working on full-stack applications.