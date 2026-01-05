# CLAUDE.md

This file provides guidance to Claude Code when working with this full-stack application.

## Project Overview

This is a full-stack web application using:
- **Frontend**: React + Vite + TypeScript + Tailwind CSS
- **Backend**: FastAPI (Python)
- **Database**: SQLite with SQLModel ORM

The project is designed for autonomous agent development with strict separation of concerns between frontend and backend.

## Quick Start

```bash
# First time setup
./init.sh

# Development (runs both frontend and backend)
npm run dev          # Frontend on http://localhost:5173
cd backend && uvicorn main:app --reload  # Backend on http://localhost:8000
```

## Architecture Principles

### 1. Strict Frontend-Backend Separation
- Frontend NEVER directly accesses the database
- All data flows through REST API endpoints
- Backend NEVER renders UI components
- Communication only via JSON over HTTP

### 2. Single Source of Truth
- Database schema is defined in backend using SQLModel
- API contracts are documented in [rules/api-contract.md](.claude/rules/api-contract.md)
- Frontend types should mirror backend models

### 3. Autonomous Agent Compatibility
- Each component can be developed independently
- Clear interfaces between layers
- Comprehensive error handling at boundaries
- All changes must maintain backward compatibility

## Directory Structure

```
project-root/
├── .claude/                    # Claude rules and tools
│   ├── CLAUDE.md              # This file
│   ├── .gitignore             # Claude-specific ignores
│   ├── rules/                 # Development rules
│   │   ├── stack.md           # Technology stack overview
│   │   ├── frontend.md        # React/Vite/TS/Tailwind rules
│   │   ├── backend.md         # FastAPI rules
│   │   ├── database.md        # SQLite/SQLModel rules
│   │   └── api-contract.md    # API interface contracts
│   └── tools/
│       └── project-setup.md   # Setup commands
├── frontend/                   # React application
│   ├── src/
│   │   ├── components/        # React components
│   │   ├── pages/             # Page components
│   │   ├── services/          # API client functions
│   │   ├── types/             # TypeScript types
│   │   ├── hooks/             # Custom React hooks
│   │   └── utils/             # Utility functions
│   ├── public/                # Static assets
│   ├── package.json
│   ├── vite.config.ts
│   └── tsconfig.json
├── backend/                    # FastAPI application
│   ├── main.py                # FastAPI app entry point
│   ├── models.py              # SQLModel database models
│   ├── database.py            # Database connection
│   ├── routers/               # API route handlers
│   ├── schemas.py             # Pydantic schemas
│   ├── services/              # Business logic
│   └── requirements.txt
├── init.sh                     # Project setup script
└── README.md
```

## Core Rules

### When Working on Frontend
1. Read [rules/frontend.md](.claude/rules/frontend.md) first
2. NEVER import or use backend code directly
3. All backend communication via `services/` API clients
4. Use TypeScript strictly - no `any` types
5. Follow Tailwind CSS utility-first approach
6. Implement proper loading and error states

### When Working on Backend
1. Read [rules/backend.md](.claude/rules/backend.md) first
2. Define all models in `models.py` using SQLModel
3. Use Pydantic schemas for request/response validation
4. Implement proper error handling with HTTP status codes
5. Add CORS middleware for frontend communication
6. Document all endpoints with OpenAPI/Swagger

### When Working on Database
1. Read [rules/database.md](.claude/rules/database.md) first
2. Use SQLModel for all database operations
3. Never use raw SQL unless absolutely necessary
4. Implement proper migrations for schema changes
5. Add indexes for frequently queried fields
6. Use transactions for multi-step operations

### When Creating/Modifying APIs
1. Read [rules/api-contract.md](.claude/rules/api-contract.md) FIRST
2. Document the endpoint contract before implementation
3. Update both backend and frontend simultaneously
4. Maintain backward compatibility
5. Version APIs if breaking changes are needed
6. Test with actual HTTP requests

## Development Workflow

### Adding a New Feature

1. **Define the API Contract** ([rules/api-contract.md](.claude/rules/api-contract.md))
   ```markdown
   ## POST /api/items
   Create a new item
   Request: { name: string, description: string }
   Response: { id: number, name: string, description: string, created_at: string }
   ```

2. **Backend Implementation**
   - Add model to `backend/models.py`
   - Create router in `backend/routers/`
   - Add business logic to `backend/services/`
   - Test with FastAPI docs at http://localhost:8000/docs

3. **Frontend Implementation**
   - Add TypeScript types to `frontend/src/types/`
   - Create API client in `frontend/src/services/`
   - Build UI components in `frontend/src/components/`
   - Create page in `frontend/src/pages/`

4. **Verification**
   - Test full flow from UI to database
   - Check error handling
   - Verify loading states
   - Test edge cases

### Making Changes

1. **Read relevant rule files first**
2. **Check API contracts** if touching endpoints
3. **Update both sides** if changing interfaces
4. **Test thoroughly** before marking complete
5. **Commit with clear messages**

## Technology Stack Details

See [rules/stack.md](.claude/rules/stack.md) for comprehensive technology stack documentation.

## Common Commands

```bash
# Frontend
cd frontend
npm install              # Install dependencies
npm run dev             # Start dev server (port 5173)
npm run build           # Production build
npm run preview         # Preview production build
npm run lint            # Run ESLint

# Backend
cd backend
pip install -r requirements.txt  # Install dependencies
uvicorn main:app --reload        # Start dev server (port 8000)
pytest                           # Run tests
python -m alembic upgrade head   # Run migrations

# Database
cd backend
python -c "from database import init_db; init_db()"  # Initialize database
sqlite3 app.db                                        # Open database CLI
```

## Error Handling Strategy

### Frontend
- Use try-catch for all API calls
- Display user-friendly error messages
- Log errors to console for debugging
- Implement retry logic for transient failures
- Show loading states during async operations

### Backend
- Use FastAPI's HTTPException for API errors
- Return appropriate HTTP status codes
- Include error details in response body
- Log errors with context
- Validate all inputs with Pydantic

### Database
- Use transactions for data consistency
- Handle constraint violations gracefully
- Implement proper connection pooling
- Add retry logic for lock timeouts
- Log all database errors

## Security Considerations

1. **Input Validation**: Validate all inputs on backend with Pydantic
2. **SQL Injection**: Use SQLModel ORM, never raw SQL with user input
3. **CORS**: Configure properly for production
4. **Authentication**: Implement JWT or session-based auth
5. **Environment Variables**: Use `.env` for secrets, never commit
6. **Rate Limiting**: Add rate limiting for public endpoints

## Testing Strategy

### Frontend Testing
- Unit tests for utility functions
- Component tests with React Testing Library
- Integration tests for API interactions
- E2E tests with Playwright

### Backend Testing
- Unit tests for business logic
- Integration tests for API endpoints
- Database tests with test database
- Load tests for performance

## Performance Optimization

### Frontend
- Code splitting with React.lazy()
- Memoization with useMemo/useCallback
- Virtual scrolling for long lists
- Image optimization
- Bundle size monitoring

### Backend
- Database query optimization
- Response caching
- Connection pooling
- Async operations where possible
- Pagination for large datasets

### Database
- Proper indexing
- Query optimization
- Connection pooling
- Regular VACUUM for SQLite
- Consider migration to PostgreSQL for scale

## Deployment

### Frontend
- Build: `npm run build`
- Deploy `dist/` folder to static hosting
- Configure environment variables
- Set up CDN for assets

### Backend
- Use production ASGI server (Gunicorn + Uvicorn)
- Set up reverse proxy (Nginx)
- Configure environment variables
- Set up database backups
- Monitor logs and metrics

## Troubleshooting

### Frontend won't connect to backend
- Check CORS configuration in backend
- Verify API base URL in frontend config
- Check browser console for errors
- Ensure backend is running on correct port

### Database locked errors
- Close all connections properly
- Use connection pooling
- Implement retry logic
- Check for long-running transactions

### Type errors in frontend
- Ensure types match API contracts
- Update types when backend changes
- Use strict TypeScript configuration
- Validate API responses at runtime

## Additional Resources

- [Stack Overview](.claude/rules/stack.md)
- [Frontend Rules](.claude/rules/frontend.md)
- [Backend Rules](.claude/rules/backend.md)
- [Database Rules](.claude/rules/database.md)
- [API Contracts](.claude/rules/api-contract.md)
- [Project Setup](.claude/tools/project-setup.md)

## Important Notes for Claude

1. **Always read the relevant rule files before making changes**
2. **Never mix frontend and backend concerns**
3. **Update API contracts when changing interfaces**
4. **Test changes thoroughly before marking complete**
5. **Maintain backward compatibility**
6. **Follow the established patterns in the codebase**
7. **Ask for clarification if requirements are ambiguous**
8. **Document all new APIs in api-contract.md**
9. **Use TypeScript strictly - no shortcuts**
10. **Implement proper error handling at all layers**
