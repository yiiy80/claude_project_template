# Technology Stack Overview

This document provides a comprehensive overview of the technology stack used in this full-stack application.

## Stack Summary

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| Frontend Framework | React | 18+ | UI component library |
| Build Tool | Vite | 5+ | Fast development and building |
| Language | TypeScript | 5+ | Type-safe JavaScript |
| Styling | Tailwind CSS | 3+ | Utility-first CSS framework |
| Backend Framework | FastAPI | 0.100+ | Modern Python web framework |
| ORM | SQLModel | 0.0.14+ | SQL database ORM with Pydantic |
| Database | SQLite | 3+ | Embedded relational database |
| HTTP Client | Fetch API | Native | Frontend API communication |

## Frontend Stack

### React 18+
**Why React?**
- Component-based architecture
- Large ecosystem and community
- Excellent TypeScript support
- Virtual DOM for performance
- Hooks for state management

**Key Features Used:**
- Functional components with hooks
- Context API for global state
- React Router for navigation
- Suspense for code splitting
- Error boundaries for error handling

**Best Practices:**
- Use functional components exclusively
- Prefer hooks over class components
- Keep components small and focused
- Use custom hooks for reusable logic
- Implement proper memoization

### Vite 5+
**Why Vite?**
- Lightning-fast HMR (Hot Module Replacement)
- Native ES modules in development
- Optimized production builds
- Built-in TypeScript support
- Plugin ecosystem

**Configuration:**
```typescript
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      '/api': 'http://localhost:8000'
    }
  }
})
```

**Key Features:**
- Instant server start
- Fast HMR updates
- Optimized build with Rollup
- CSS code splitting
- Asset handling

### TypeScript 5+
**Why TypeScript?**
- Static type checking
- Better IDE support
- Catch errors at compile time
- Self-documenting code
- Refactoring confidence

**Configuration:**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  }
}
```

**Type Safety Rules:**
- No `any` types (use `unknown` if needed)
- Strict null checks enabled
- Explicit return types for functions
- Interface over type for objects
- Use generics for reusable code

### Tailwind CSS 3+
**Why Tailwind?**
- Utility-first approach
- No CSS file bloat
- Consistent design system
- Responsive design built-in
- Easy customization

**Configuration:**
```javascript
// tailwind.config.js
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: '#3b82f6',
        secondary: '#8b5cf6'
      }
    }
  }
}
```

**Best Practices:**
- Use utility classes directly in JSX
- Create component classes for repeated patterns
- Use @apply sparingly
- Leverage responsive modifiers (sm:, md:, lg:)
- Use dark mode variants when needed

## Backend Stack

### FastAPI 0.100+
**Why FastAPI?**
- Automatic API documentation (Swagger/OpenAPI)
- Built-in data validation with Pydantic
- Async support for high performance
- Type hints for better code quality
- Easy dependency injection

**Key Features:**
```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="My API", version="1.0.0")

# CORS for frontend
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/api/items/{item_id}")
async def get_item(item_id: int):
    # Automatic validation and documentation
    return {"id": item_id}
```

**Best Practices:**
- Use async/await for I/O operations
- Implement proper dependency injection
- Use Pydantic models for validation
- Add proper error handling
- Document all endpoints

### SQLModel 0.0.14+
**Why SQLModel?**
- Combines SQLAlchemy and Pydantic
- Single model definition for DB and API
- Type hints for database fields
- Automatic validation
- Easy migrations

**Model Definition:**
```python
from sqlmodel import SQLModel, Field
from typing import Optional
from datetime import datetime

class Item(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    description: str
    price: float = Field(gt=0)
    created_at: datetime = Field(default_factory=datetime.utcnow)
```

**Best Practices:**
- Use Field() for constraints and indexes
- Define relationships with Relationship()
- Use Optional for nullable fields
- Add validators for complex logic
- Keep models in separate file

### SQLite 3+
**Why SQLite?**
- Zero configuration
- Serverless architecture
- Single file database
- ACID compliant
- Perfect for small to medium apps

**Connection Setup:**
```python
from sqlmodel import create_engine, Session

DATABASE_URL = "sqlite:///./app.db"
engine = create_engine(DATABASE_URL, echo=True)

def get_session():
    with Session(engine) as session:
        yield session
```

**Limitations to Know:**
- No concurrent writes (use connection pooling)
- Limited ALTER TABLE support
- No built-in replication
- Consider PostgreSQL for production scale

**Best Practices:**
- Enable WAL mode for better concurrency
- Use transactions for consistency
- Regular VACUUM for optimization
- Backup regularly
- Add indexes for performance

## Development Tools

### Package Managers
- **Frontend**: npm (or pnpm/yarn)
- **Backend**: pip with requirements.txt

### Code Quality
- **Frontend**: ESLint + Prettier
- **Backend**: Black + Flake8 + mypy

### Testing
- **Frontend**: Vitest + React Testing Library
- **Backend**: pytest + httpx

### Version Control
- Git with conventional commits
- Feature branch workflow
- Pull request reviews

## Architecture Patterns

### Frontend Patterns
1. **Component Structure**
   ```
   components/
   ├── common/        # Reusable UI components
   ├── features/      # Feature-specific components
   └── layouts/       # Page layouts
   ```

2. **State Management**
   - Local state: useState
   - Shared state: Context API
   - Server state: React Query (optional)

3. **API Communication**
   ```typescript
   // services/api.ts
   export async function fetchItems() {
     const response = await fetch('/api/items')
     if (!response.ok) throw new Error('Failed to fetch')
     return response.json()
   }
   ```

### Backend Patterns
1. **Layered Architecture**
   ```
   routers/     # HTTP endpoints
   services/    # Business logic
   models.py    # Database models
   schemas.py   # Request/response schemas
   ```

2. **Dependency Injection**
   ```python
   from fastapi import Depends

   def get_db():
       db = SessionLocal()
       try:
           yield db
       finally:
           db.close()

   @app.get("/items")
   def read_items(db: Session = Depends(get_db)):
       return db.query(Item).all()
   ```

3. **Error Handling**
   ```python
   from fastapi import HTTPException

   @app.get("/items/{item_id}")
   def get_item(item_id: int, db: Session = Depends(get_db)):
       item = db.get(Item, item_id)
       if not item:
           raise HTTPException(status_code=404, detail="Item not found")
       return item
   ```

## Communication Flow

```
User Browser
    ↓
React App (Port 5173)
    ↓ HTTP/JSON
FastAPI (Port 8000)
    ↓ SQLModel ORM
SQLite Database (app.db)
```

### Request Flow Example
1. User clicks button in React component
2. Component calls API service function
3. Service makes fetch() request to FastAPI
4. FastAPI validates request with Pydantic
5. FastAPI queries database with SQLModel
6. SQLModel returns Python objects
7. FastAPI serializes to JSON
8. React receives response and updates UI

## Environment Configuration

### Frontend (.env)
```bash
VITE_API_BASE_URL=http://localhost:8000
VITE_APP_TITLE=My App
```

### Backend (.env)
```bash
DATABASE_URL=sqlite:///./app.db
SECRET_KEY=your-secret-key-here
CORS_ORIGINS=http://localhost:5173
```

## Performance Considerations

### Frontend
- Code splitting with React.lazy()
- Image optimization
- Debounce user inputs
- Virtual scrolling for lists
- Memoization with useMemo/useCallback

### Backend
- Database connection pooling
- Query optimization with indexes
- Response caching
- Async operations
- Pagination for large datasets

### Database
- Proper indexing strategy
- Query optimization
- Regular VACUUM
- WAL mode for concurrency
- Consider read replicas for scale

## Security Best Practices

1. **Input Validation**: Pydantic on backend, Zod on frontend
2. **SQL Injection**: Use ORM, never raw SQL with user input
3. **XSS Protection**: React escapes by default
4. **CORS**: Configure properly for production
5. **Authentication**: JWT or session-based
6. **Environment Variables**: Never commit secrets
7. **HTTPS**: Always use in production
8. **Rate Limiting**: Implement for public APIs

## Scalability Path

### When to Upgrade
- **SQLite → PostgreSQL**: >100 concurrent users
- **Single Server → Load Balancer**: High traffic
- **Monolith → Microservices**: Complex domains
- **REST → GraphQL**: Complex data requirements

### Migration Strategy
1. Start with this stack for MVP
2. Monitor performance metrics
3. Identify bottlenecks
4. Upgrade components incrementally
5. Maintain backward compatibility

## Recommended Libraries

### Frontend
- **Routing**: react-router-dom
- **Forms**: react-hook-form + zod
- **HTTP**: axios or native fetch
- **State**: zustand or jotai (if Context isn't enough)
- **UI Components**: shadcn/ui or headlessui
- **Icons**: lucide-react or heroicons
- **Date**: date-fns or dayjs

### Backend
- **Validation**: pydantic (built-in)
- **Auth**: python-jose + passlib
- **Testing**: pytest + httpx
- **Migrations**: alembic
- **Environment**: python-dotenv
- **CORS**: fastapi.middleware.cors (built-in)

## Development Workflow

1. **Start Backend**: `cd backend && uvicorn main:app --reload`
2. **Start Frontend**: `cd frontend && npm run dev`
3. **Access App**: http://localhost:5173
4. **API Docs**: http://localhost:8000/docs
5. **Database**: `sqlite3 backend/app.db`

## Production Deployment

### Frontend
- Build: `npm run build`
- Deploy: Vercel, Netlify, or Cloudflare Pages
- CDN: Automatic with most platforms

### Backend
- Server: Gunicorn + Uvicorn workers
- Reverse Proxy: Nginx or Caddy
- Platform: Railway, Render, or DigitalOcean
- Database: Upgrade to PostgreSQL

### Database
- Backup: Regular automated backups
- Monitoring: Query performance tracking
- Scaling: Read replicas if needed

## Conclusion

This stack provides:
- ✅ Fast development with Vite and FastAPI
- ✅ Type safety with TypeScript and Pydantic
- ✅ Modern UI with React and Tailwind
- ✅ Simple deployment with SQLite
- ✅ Clear upgrade path for scaling
- ✅ Excellent developer experience
- ✅ Strong community support

Perfect for MVPs, internal tools, and small to medium applications.
