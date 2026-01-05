# Backend Development Rules

This document defines the rules and best practices for backend development with FastAPI, SQLModel, and SQLite.

## Core Principles

### 1. API-First Design
- **RESTful conventions** - Use proper HTTP methods and status codes
- **OpenAPI documentation** - All endpoints auto-documented
- **Pydantic validation** - Validate all inputs
- **Type hints everywhere** - Python 3.10+ type annotations
- **Async by default** - Use async/await for I/O operations

### 2. Database Abstraction
- **SQLModel ORM only** - No raw SQL unless absolutely necessary
- **Type-safe models** - Database schema defined with Python classes
- **Transactions** - Use for multi-step operations
- **Migrations** - Track schema changes with Alembic
- **Connection pooling** - Efficient database connections

### 3. No Frontend Dependencies
- **NEVER import frontend code**
- **JSON responses only** - No HTML rendering
- **CORS configured** - Allow frontend origin
- **Stateless API** - No session state on server (use JWT)
- **No file system access** outside project directory

## Project Structure

```
backend/
├── main.py                 # FastAPI app entry point
├── database.py             # Database connection and session
├── models.py               # SQLModel database models
├── schemas.py              # Pydantic request/response schemas
├── config.py               # Configuration and environment variables
├── dependencies.py         # Dependency injection functions
├── routers/                # API route handlers
│   ├── __init__.py
│   ├── items.py           # /api/items endpoints
│   ├── users.py           # /api/users endpoints
│   └── auth.py            # /api/auth endpoints
├── services/               # Business logic layer
│   ├── __init__.py
│   ├── item_service.py    # Item business logic
│   └── user_service.py    # User business logic
├── utils/                  # Utility functions
│   ├── __init__.py
│   ├── security.py        # Password hashing, JWT
│   └── validators.py      # Custom validators
├── tests/                  # Test files
│   ├── __init__.py
│   ├── test_items.py
│   └── test_users.py
├── requirements.txt        # Python dependencies
├── .env                    # Environment variables (not committed)
└── app.db                  # SQLite database (not committed)
```

## FastAPI Application Setup

### Main Application

```python
# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager

from database import init_db
from routers import items, users, auth
from config import settings

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: Initialize database
    init_db()
    yield
    # Shutdown: Cleanup if needed

app = FastAPI(
    title="My API",
    version="1.0.0",
    description="Full-stack application API",
    lifespan=lifespan
)

# CORS middleware - MUST be configured for frontend
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,  # ["http://localhost:5173"]
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(items.router, prefix="/api/items", tags=["items"])
app.include_router(users.router, prefix="/api/users", tags=["users"])
app.include_router(auth.router, prefix="/api/auth", tags=["auth"])

@app.get("/")
async def root():
    return {"message": "API is running"}

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

### Configuration

```python
# config.py
from pydantic_settings import BaseSettings
from typing import List

class Settings(BaseSettings):
    # Database
    DATABASE_URL: str = "sqlite:///./app.db"

    # CORS
    CORS_ORIGINS: List[str] = ["http://localhost:5173"]

    # Security
    SECRET_KEY: str = "your-secret-key-change-in-production"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30

    # Application
    DEBUG: bool = True

    class Config:
        env_file = ".env"
        case_sensitive = True

settings = Settings()
```

## Database Models (SQLModel)

### Model Definition

```python
# models.py
from sqlmodel import SQLModel, Field, Relationship
from typing import Optional, List
from datetime import datetime

class User(SQLModel, table=True):
    """User model - represents users table"""
    id: Optional[int] = Field(default=None, primary_key=True)
    email: str = Field(unique=True, index=True, max_length=255)
    username: str = Field(unique=True, index=True, max_length=50)
    hashed_password: str = Field(max_length=255)
    is_active: bool = Field(default=True)
    is_superuser: bool = Field(default=False)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # Relationships
    items: List["Item"] = Relationship(back_populates="owner")

class Item(SQLModel, table=True):
    """Item model - represents items table"""
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(index=True, max_length=100)
    description: str = Field(max_length=500)
    price: float = Field(gt=0)  # Greater than 0
    is_available: bool = Field(default=True)
    owner_id: int = Field(foreign_key="user.id")
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # Relationships
    owner: Optional[User] = Relationship(back_populates="items")

class Category(SQLModel, table=True):
    """Category model - represents categories table"""
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(unique=True, index=True, max_length=50)
    description: Optional[str] = Field(default=None, max_length=200)
    created_at: datetime = Field(default_factory=datetime.utcnow)
```

### Database Connection

```python
# database.py
from sqlmodel import SQLModel, create_engine, Session
from config import settings

# Create engine
engine = create_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,  # Log SQL queries in debug mode
    connect_args={"check_same_thread": False}  # Needed for SQLite
)

def init_db():
    """Initialize database - create all tables"""
    SQLModel.metadata.create_all(engine)

def get_session():
    """Dependency for getting database session"""
    with Session(engine) as session:
        yield session
```

## Pydantic Schemas

### Request/Response Schemas

```python
# schemas.py
from pydantic import BaseModel, EmailStr, Field, validator
from typing import Optional
from datetime import datetime

# ============= Item Schemas =============

class ItemBase(BaseModel):
    """Base schema for Item - shared fields"""
    name: str = Field(..., min_length=1, max_length=100)
    description: str = Field(..., max_length=500)
    price: float = Field(..., gt=0)
    is_available: bool = True

class ItemCreate(ItemBase):
    """Schema for creating an item"""
    pass

class ItemUpdate(BaseModel):
    """Schema for updating an item - all fields optional"""
    name: Optional[str] = Field(None, min_length=1, max_length=100)
    description: Optional[str] = Field(None, max_length=500)
    price: Optional[float] = Field(None, gt=0)
    is_available: Optional[bool] = None

class ItemResponse(ItemBase):
    """Schema for item response"""
    id: int
    owner_id: int
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True  # Enable ORM mode

# ============= User Schemas =============

class UserBase(BaseModel):
    """Base schema for User"""
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)

class UserCreate(UserBase):
    """Schema for user registration"""
    password: str = Field(..., min_length=8, max_length=100)

    @validator('password')
    def validate_password(cls, v):
        if not any(char.isdigit() for char in v):
            raise ValueError('Password must contain at least one digit')
        if not any(char.isupper() for char in v):
            raise ValueError('Password must contaone uppercase letter')
        return v

class UserUpdate(BaseModel):
    """Schema for updating user"""
    email: Optional[EmailStr] = None
    username: Optional[str] = Field(None, min_length=3, max_length=50)
    password: Optional[str] = Field(None, min_length=8, max_length=100)

class UserResponse(UserBase):
    """Schema for user response - NEVER include password"""
    id: int
    is_active: bool
    is_superuser: bool
    created_at: datetime

    class Config:
        from_attributes = True

# ============= Auth Schemas =============

class Token(BaseModel):
    """Schema for JWT token response"""
    access_token: str
    token_type: str = "bearer"

class TokenData(BaseModel):
    """Schema for token payload"""
    user_id: Optional[int] = None

class LoginRequest(BaseModel):
    """Schema for login request"""
    email: EmailStr
    password: str
```

## API Routers

### Router Implementation

```python
# routers/items.py
from fastapi import APIRouter, Depends, HTTPException, status, Query
from sqlmodel import Session, select
from typing import List

from database import get_session
from models import Item, User
from schemas import ItemCreate, ItemUpdate, ItemResponse
from dependencies import get_current_user

router = APIRouter()

@router.get("/", response_model=List[ItemResponse])
async def get_items(
    skip: int = Query(0, ge=0),
    limit: int = Query(100, ge=1, le=100),
    session: Session = Depends(get_session)
):
    """Get all items with pagination"""
    statement = select(Item).offset(skip).limit(limit)
    items = session.exec(statement).all()
    return items

@router.get("/{item_id}", response_model=ItemResponse)
async def get_item(
    item_id: int,
    session: Session = Depends(get_session)
):
    """Get a specific item by ID"""
    item = session.get(Item, item_id)
    if not item:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item with id {item_id} not found"
        )
    return item

@router.post("/", response_model=ItemResponse, status_code=status.HTTP_201_CREATED)
async def create_item(
    item_data: ItemCreate,
    session: Session = Depends(get_session),
    current_user: User = Depends(get_current_user)
):
    """Create a new item"""
    item = Item(
        **item_data.model_dump(),
        owner_id=current_user.id
    )
    session.add(item)
    session.commit()
    session.refresh(item)
    return item

@router.put("/{item_id}", response_model=ItemResponse)
async def update_item(
    item_id: int,
    item_data: ItemUpdate,
    session: Session = Depends(get_session),
    current_user: User = Depends(get_current_user)
):
    """Update an existing item"""
    item = session.get(Item, item_id)
    if not item:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item with id {item_id} not found"
        )

    # Check ownership
    if item.owner_id != current_user.id and not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not authorized to update this item"
        )

    # Update only provided fields
    update_data = item_data.model_dump(exclude_unset=True)
    for key, value in update_data.items():
        setattr(item, key, value)

    item.updated_at = datetime.utcnow()
    session.add(item)
    session.commit()
    session.refresh(item)
    return item

@router.delete("/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_item(
    item_id: int,
    session: Session = Depends(get_session),
    current_user: User = Depends(get_current_user)
):
    """Delete an item"""
    item = session.get(Item, item_id)
    if not item:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item with id {item_id} not found"
        )

    # Check ownership
    if item.owner_id != current_user.id and not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not authorized to delete this item"
        )

    session.delete(item)
    session.commit()
    return None
```

## Business Logic (Services)

### Service Layer

```python
# services/item_service.py
from sqlmodel import Session, select
from typing import List, Optional
from datetime import datetime

from models import Item
from schemas import ItemCreate, ItemUpdate

class ItemService:
    """Business logic for items"""

    @staticmethod
    def get_all(
        session: Session,
        skip: int = 0,
        limit: int = 100,
        owner_id: Optional[int] = None
    ) -> List[Item]:
        """Get all items with optional filtering"""
        statement = select(Item)

        if owner_id is not None:
            statement = statement.where(Item.owner_id == owner_id)

        statement = statement.offset(skip).limit(limit)
        return session.exec(statement).all()

    @staticmethod
    def get_by_id(session: Session, item_id: int) -> Optional[Item]:
        """Get item by ID"""
        return session.get(Item, item_id)

    @staticmethod
    def create(
        session: Session,
        item_data: ItemCreate,
        owner_id: int
    ) -> Item:
        """Create a new item"""
        item = Item(**item_data.model_dump(), owner_id=owner_id)
        session.add(item)
        session.commit()
        session.refresh(item)
        return item

    @staticmethod
    def update(
        session: Session,
        item: Item,
        item_data: ItemUpdate
    ) -> Item:
        """Update an existing item"""
        update_data = item_data.model_dump(exclude_unset=True)
        for key, value in update_data.items():
            setattr(item, key, value)

        item.updated_at = datetime.utcnow()
        session.add(item)
        session.commit()
        session.refresh(item)
        return item

    @staticmethod
    def delete(session: Session, item: Item) -> None:
        """Delete an item"""
        session.delete(item)
        session.commit()

    @staticmethod
    def search(
        session: Session,
        query: str,
        skip: int = 0,
        limit: int = 100
    ) -> List[Item]:
        """Search items by name or description"""
        statement = select(Item).where(
            (Item.name.contains(query)) | (Item.description.contains(query))
        ).offset(skip).limit(limit)
        return session.exec(statement).all()
```

## Authentication & Security

### Security Utilities

```python
# utils/security.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext

from config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Verify a password against a hash"""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """Hash a password"""
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    """Create a JWT access token"""
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)

    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(
        to_encode,
        settings.SECRET_KEY,
        algorithm=settings.ALGORITHM
    )
    return encoded_jwt

def decode_access_token(token: str) -> Optional[dict]:
    """Decode and verify a JWT token"""
    try:
        payload = jwt.decode(
            token,
            settings.SECRET_KEY,
            algorithms=[settings.ALGORITHM]
        )
        return payload
    except JWTError:
        return None
```

### Dependencies

```python
# dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlmodel import Session

from database import get_session
from models import User
from utils.security import decode_access_token

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/auth/login")

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    session: Session = Depends(get_session)
) -> User:
    """Get current authenticated user"""
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )

    payload = decode_access_token(token)
    if payload is None:
        raise credentials_exception

    user_id: int = payload.get("sub")
    if user_id is None:
        raise credentials_exception

    user = session.get(User, user_id)
    if user is None:
        raise credentials_exception

    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Inactive user"
        )

    return user

async def get_current_superuser(
    current_user: User = Depends(get_current_user)
) -> User:
    """Require superuser privileges"""
    if not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not enough privileges"
        )
    return current_user
```

## Error Handling

### Custom Exceptions

```python
# utils/exceptions.py
from fastapi import HTTPException, status

class NotFoundException(HTTPException):
    def __init__(self, detail: str = "Resource not found"):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=detail
        )

class UnauthorizedException(HTTPException):
    def __init__(self, detail: str = "Not authenticated"):
        super().__init__(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail=detail,
            headers={"WWW-Authenticate": "Bearer"}
        )

class ForbiddenException(HTTPException):
    def __init__(self, detail: str = "Not enough privileges"):
        super().__init__(
            status_code=status.HTTP_403_FORBIDDEN,
            detail=detail
        )

class BadRequestException(HTTPException):
    def __init__(self, detail: str = "Bad request"):
        super().__init__(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=detail
        )
```

### Global Exception Handler

```python
# main.py (add to existing file)
from fastapi import Request
from fastapi.responses import JSONResponse
from sqlalchemy.exc import IntegrityError

@app.exception_handler(IntegrityError)
async def integrity_error_handler(request: Request, exc: IntegrityError):
    """Handle database integrity errors"""
    return JSONResponse(
        status_code=status.HTTP_409_CONFLICT,
        content={"detail": "Database integrity error. Resource may already exist."}
    )

@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception):
    """Handle unexpected errors"""
    # Log the error
    print(f"Unexpected error: {exc}")

    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={"detail": "Internal server error"}
    )
```

## Database Migrations (Alembic)

### Setup Alembic

```bash
# Install alembic
pip install alembic

# Initialize alembic
alembic init alembic
```

### Configure Alembic

```python
# alembic/env.py
from sqlmodel import SQLModel
from models import *  # Import all models
from config import settings

# Set target metadata
target_metadata = SQLModel.metadata

# Set database URL
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)
```

### Create Migration

```bash
# Create a new migration
alembic revision --autogenerate -m "Add items table"

# Apply migrations
alembic upgrade head

# Rollback migration
alembic downgrade -1
```

## Testing

### Test Setup

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from sqlmodel import Session, create_engine, SQLModel
from sqlmodel.pool import StaticPool

from main import app
from database import get_session

@pytest.fixture(name="session")
def session_fixture():
    """Create a test database session"""
    engine = create_engine(
        "sqlite:///:memory:",
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    SQLModel.metadata.create_all(engine)
    with Session(engine) as session:
        yield session

@pytest.fixture(name="client")
def client_fixture(session: Session):
    """Create a test client"""
    def get_session_override():
        return session

    app.dependency_overrides[get_session] = get_session_override
    client = TestClient(app)
    yield client
    app.dependency_overrides.clear()
```

### Test Example

```python
# tests/test_items.py
from fastapi.testclient import TestClient
from sqlmodel import Session

from models import Item, User

def test_create_item(client: TestClient, session: Session):
    """Test creating an item"""
    # Create a test user
    user = User(email="test@example.com", username="testuser", hashed_password="hashed")
    session.add(user)
    session.commit()

    # Create item
    response = client.post(
        "/api/items/",
        json={
            "name": "Test Item",
            "description": "Test description",
            "price": 9.99,
            "is_available": True
        },
        headers={"Authorization": f"Bearer {get_test_token(user.id)}"}
    )

    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Test Item"
    assert data["price"] == 9.99

def test_get_items(client: TestClient, session: Session):
    """Test getting all items"""
    response = client.get("/api/items/")
    assert response.status_code == 200
    assert isinstance(response.json(), list)
```

## Best Practices Checklist

### For Every Endpoint
- [ ] Proper HTTP method (GET, POST, PUT, DELETE)
- [ ] Correct status cod, 201, 204, 400, 404, etc.)
- [ ] Request validation with Pydantic schema
- [ ] Response model defined
- [ ] Error handling implemented
- [ ] Authentication/authorization if needed
- [ ] Database session properly managed
- [ ] OpenAPI documentation (automatic)

### For Every Model
- [ ] Type hints for all fields
- [ ] Primary key defined
- [ ] Indexes on frequently queried fields
- [ ] Foreign keys for relationships
- [ ] Constraints (unique, not null, etc.)
- [ ] Default values where appropriate
- [ ] Timestamps (created_at, updated_at)

### For Every Service Function
- [ ] Type hints for parameters and return
- [ ] Docstring explaining purpose
- [ ] Error handling
- [ ] Transaction management if needed
- [ ] Unit tests written

## Common Patterns

### Pagination

```python
from fastapi import Query

@router.get("/items/")
async def get_items(
    skip: int = Query(0, ge=0, description="Number of items to skip"),
    limit: int = Query(100, ge=1, le=100, description="Max items to return"),
    session: Session = Depends(get_session)
):
    items = session.exec(select(Item).offset(skip).limit(limit)).all()
    total = session.exec(select(func.count(Item.id))).one()
    return {
        "items": items,
        "total": total,
        "skip": skip,
        "limit": limit
    }
```

### Filtering

```python
@router.get("/items/")
async def get_items(
    category_id: Optional[int] = Query(None),
    is_available: Optional[bool] = Query(None),
    min_price: Optional[float] = Query(None, ge=0),
    max_price: Optional[float] = Query(None, ge=0),
    session: Session = Depends(get_session)
):
    statement = select(Item)

    if category_id is not None:
        statement = statement.where(Item.category_id == category_id)
    if is_available is not None:
        statement = statement.where(Item.is_available == is_available)
    if min_price is not None:
        statement = statement.where(Item.price >= min_price)
    if max_price is not None:
        statement = statement.where(Item.price <= max_price)

    items = session.exec(statement).all()
    return items
```

### Transactions

```python
from sqlmodel import Session

def transfer_item(
    session: Session,
    item_id: int,
    from_user_id: int,
    to_user_id: int
):
    """Transfer item ownership (atomic operation)"
    try:
        # Get item
        item = session.get(Item, item_id)
        if not item or item.owner_id != from_user_id:
            raise ValueError("Invalid transfer")

        # Update ownership
        item.owner_id = to_user_id
        item.updated_at = datetime.utcnow()

        # Log transfer (example)
        log = TransferLog(
            item_id=item_id,
            from_user_id=from_user_id,
            to_user_id=to_user_id
        )

        session.add(item)
        session.add(log)
        session.commit()

    except Exception a       session.rollback()
        raise
```

## Anti-Patterns to Avoid

❌ **Raw SQL queries**
```python
session.exec("SELECT * FROM items")  # Avoid
```

❌ **Missing error handling**
```python
@router.get("/items/{item_id}")
async def get_item(item_id: int, session: Session = Depends(get_session)):
    return session.get(Item, item_id)  # What if not found?
```

❌ **No input validation**
```python
@router.post("/items/")
async def create_item(data: dict):  # Use Pydantic schema!
    pass
```

❌ **Returning sensitive data**
```python
@router.get("/users/{user_id}")
async def get_user(user_id: int):
    return user  # Includes hashed_password! Use response model
```

❌ **Not using dependency injection**
```python
def get_item(item_id: int):
    session = Session(engine)  # Don't create sessions manually
    # Use Depends(get_session) instead
```

## Summary

Follow these rules to build a robust, secure, and maintainable FastAPI backend that integrates seamlessly with the React frontend.
