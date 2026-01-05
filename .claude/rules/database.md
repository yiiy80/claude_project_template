# Database Development Rules

This document defines the rules and best practices for database development with SQLite and SQLModel.

## Core Principles

### 1. ORM-First Approach
- **SQLModel for everything** - Use ORM, not raw SQL
- **Type-safe queries** - Leverage Python type hints
- **Declarative models** - Define schema with Python classes
- **Automatic validation** - Pydantic integration
- **Migration tracking** - Use Alembic for schema changes

### 2. Data Integrity
- **Constraints** - Use database constraints (unique, not null, foreign keys)
- **Transactions** - Atomic operations for data consistency
- **Indexes** - Optimize frequently queried fields
- **Cascading deletes** - Handle related data properly
- **Validation** - Both at database and application level

### 3. SQLite-Specific Considerations
- **Single file database** - Simple deployment
- **Limited concurrency** - One writer at a time
- **No ALTER TABLE** - Limited schema modification
- **WAL mode** - Better concurrent read performance
- **Migration path** - Plan for PostgreSQL if needed

## Database Setup

### Engine Configuration

```python
# database.py
from sqlmodel import SQLModel, create_engine, Session
from sqlalchemy.pool import StaticPool
from config import settings

# SQLite-specific connection arguments
connect_args = {
    "check_same_thread": False,  # Allow multiple threads
    "timeout": 30.0,              # Wait 30s for locks
}

# Create engine with proper configuration
engine = create_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,           # Log SQL in debug mode
    connect_args=connect_args,
    poolclass=StaticPool,          # Use static pool for SQLite
)

def init_db():
    """Initialize database - create all tables"""
    # Enable WAL mode for better concurrency
    with engine.connect() as conn:
        conn.execute("PRAGMA journal_mode=WAL")
        conn.execute("PRAGMA foreign_keys=ON")  # Enable foreign keys

    # Create all tables
    SQLModel.metadata.create_all(engine)

def get_session():
    """Dependency for getting database session"""
    with Session(engine) as session:
        yield session
```

### Environment Configuration

```python
# config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str = "sqlite:///./app.db"

    class Config:
        env_file = ".env"

settings = Settings()
```

```bash
# .env
DATABASE_URL=sqlite:///./app.db
```

## Model Definition

### Basic Model

```python
# models.py
from sqlmodel import SQLModel, Field
from typing import Optional
from datetime import datetime

class Item(SQLModel, table=True):
    """Item model - represents items table"""
    # Primary key
    id: Optional[int] = Field(default=None, primary_key=True)

    # Required fields
    name: str = Field(index=True, max_length=100)
    description: str = Field(max_length=500)
    price: float = Field(gt=0)  # Must be greater than 0

    # Optional fields with defaults
    is_available: bool = Field(default=True)
    quantity: int = Field(default=0, ge=0)  # Greater than or equal to 0

    # Timestamps
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

    # Indexes are automatically created for:
    # - Primary keys
    # - Foreign keys
    # - Fields with index=True
```

### Relationships

```python
from sqlmodel import Relationship
from typing import List, Optional

class User(SQLModel, table=True):
    """User model"""
    id: Optional[int] = Field(default=None, primary_key=True)
    email: str = Field(unique=True, index=True, max_length=255)
    username: str = Field(unique=True, index=True, max_length=50)

    # One-to-many relationship
    items: List["Item"] = Relationship(back_populates="owner")

class Item(SQLModel, table=True):
    """Item model"""
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(max_length=100)

    # Foreign key
    owner_id: int = Field(foreign_key="user.id")

    # Many-to-one relationship
    owner: Optional[User] = Relationship(back_populates="items")
```

### Many-to-Many Relationships

```python
# Link table for many-to-many
class ItemTagLink(SQLModel, table=True):
    """Link table between items and tags"""
    item_id: Optional[int] = Field(
        default=None,
        foreign_key="item.id",
        primary_key=True
    )
    tag_id: Optional[int] = Field(
        default=None,
        foreign_key="tag.id",
        primary_key=True
    )

class Item(SQLModel, table=True):
    """Item model"""
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(max_length=100)

    # Many-to-many relationship
    tags: List["Tag"] = Relationship(
        back_populates="items",
        link_model=ItemTagLink
    )

class Tag(SQLModel, table=True):
    """Tag model"""
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(unique=True, max_length=50)

    # Many-to-many relationship
    items: List[Item] = Relationship(
        back_populates="tags",
        link_model=ItemTagLink
    )
```

## CRUD Operations

### Create

```python
from sqlmodel import Session

def create_item(session: Session, name: str, price: float) -> Item:
    """Create a new item"""
    item = Item(name=name, price=price)
    session.add(item)
    session.commit()
    session.refresh(item)  # Get the ID and defaults
    return item

# Bulk create
def create_items(session: Session, items_data: List[dict]) -> List[Item]:
    """Create multiple items"""
    items = [Item(**data) for data in items_data]
    session.add_all(items)
    session.commit()
    for item in items:
        session.refresh(item)
    return items
```

### Read

```python
from sqlmodel import select

def get_item(session: Session, item_id: int) -> Optional[Item]:
    """Get item by ID"""
    return session.get(Item, item_id)

def get_all_items(session: Session, skip: int = 0, limit: int = 100) -> List[Item]:
    """Get all items with pagination"""
    statement = select(Item).offset(skip).limit(limit)
    return session.exec(statement).all()

def get_items_by_owner(session: Session, owner_id: int) -> List[Item]:
    """Get items filtered by owner"""
    statement = select(Item).where(Item.owner_id == owner_id)
    return session.exec(statement).all()

def search_items(session: Session, query: str) -> List[Item]:
    """Search items by name"""
    statement = select(Item).where(Item.name.contains(query))
    return session.exec(statement).all()
```

### Update

```python
def update_item(session: Session, item_id: int, **kwargs) -> Optional[Item]:
    """Update an item"""
    item = session.get(Item, item_id)
    if not item:
        return None

    # Update only provided fields
    for key, value in kwargs.items():
        if hasattr(item, key):
            setattr(item, key, value)

    item.updated_at = datetime.utcnow()
    session.add(item)
    session.commit()
    session.refresh(item)
    return item
```

### Delete

```python
def delete_item(session: Session, item_id: int) -> bool:
    """Delete an item"""
    item = session.get(Item, item_id)
    if not item:
        return False

    session.delete(item)
    session.commit()
    return True

def delete_items_by_owner(session: Session, owner_id: int) -> int:
    """Delete all items by owner"""
    statement = select(Item).where(Item.owner_id == owner_id)
    items = session.exec(statement).all()

    for item in items:
        session.delete(item)

    session.commit()
    return len(items)
```

## Advanced Queries

### Filtering

```python
from sqlmodel import and_, or_, not_

# Multiple conditions with AND
statement = select(Item).where(
    and_(
        Item.price > 10,
        Item.is_available == True,
        Item.quantity > 0
    )
)

# Multiple conditions with OR
statement = select(Item).where(
    or_(
        Item.name.contains("laptop"),
        Item.description.contains("laptop")
    )
)

# NOT condition
statement = select(Item).where(
    not_(Item.is_available)
)

# Combining AND and OR
statement = select(Item).where(
    and_(
        Item.price > 10,
        or_(
            Item.name.contains("laptop"),
            Item.name.contains("computer")
        )
    )
)
```

### Sorting

```python
# Sort ascending
statement = select(Item).order_by(Item.price)

# Sort descending
statement = select(Item).order_by(Item.price.desc())

# Multiple sort fields
statement = select(Item).order_by(Item.category, Item.price.desc())
```

### Aggregation

```python
from sqlmodel import func

# Count
count = session.exec(select(func.count(Item.id))).one()

# Sum
total_price = session.exec(select(func.sum(Item.price))).one()

# Average
avg_price = session.exec(select(func.avg(Item.price))).one()

# Min/Max
min_price = session.exec(select(func.min(Item.price))).one()
max_price = session.exec(select(func.max(Item.price))).one()

# Group by
statement = select(
    Item.category,
    func.count(Item.id).label("count"),
    func.avg(Item.price).label("avg_price")
).group_by(Item.category)
results = session.exec(statement).all()
```

### Joins

```python
# Join with relationship
statement = select(Item, User).join(User)
results = session.exec(statement).all()

# Left join
statement = select(Item).join(User, isouter=True)
results = session.exec(statement).all()

# Filter on joined table
statement = select(Item).join(User).where(User.username == "john")
results = session.exec(statement).all()
```

## Transactions

### Basic Transaction

```python
def transfer_items(
    session: Session,
    from_user_id: int,
    to_user_id: int,
    item_ids: List[int]
):
    """Transfer items between users (atomic operation)"""
    try:
        # All operations in one transaction
        for item_id in item_ids:
            item = session.get(Item, item_id)
            if not item or item.owner_id != from_user_id:
                raise ValueError(f"Invalid item {item_id}")
            item.owner_id = to_user_id
            item.updated_at = datetime.utcnow()
            session.add(item)

        session.commit()
    except Exception as e:
        session.rollback()
        raise
```

### Nested Transactions (Savepoints)

```python
def complex_operation(session: Session):
    """Complex operation with savepoints"""
    try:
        # Main transaction
        user = User(username="john", email="john@example.com")
        session.add(user)
        session.flush()  # Get user.id without committing

        # Savepoint
        with session.begin_nested():
            item1 = Item(name="Item 1", owner_id=user.id)
            session.add(item1)
            # This will rollback to savepoint if it fails

        # Another savepoint
        with session.begin_nested():
            item2 = Item(name="Item 2", owner_id=user.id)
            session.add(item2)

        session.commit()
    except Exception as e:
        session.rollback()
        raise
```

## Indexes

### Creating Indexes

```python
# Index on single column
class Item(SQLModel, table=True):
    name: str = Field(index=True)  # Creates index automatically

# Composite index (multiple columns)
from sqlalchemy import Index

class Item(SQLModel, table=True):
    __tablename__ = "items"

    category: str
    price: float

    __table_args__ = (
        Index("idx_category_price", "category", "price"),
    )

# Unique index
class User(SQLModel, table=True):
    email: str = Field(unique=True, index=True)  # Unique index
```

### When to Use Indexes

✅ **Create indexes for:**
- Primary keys (automatic)
- Foreign keys (automatic)
- Columns used in WHERE clauses frequently
- Columns used in JOIN conditions
- Columns used in ORDER BY
- Unique constraints

❌ **Avoid indexes for:**
- Small tables (< 1000 rows)
- Columns with low cardinality (few unique values)
- Columns that are frequently updated
- Wide columns (long text fields)

## Migrations with Alembic

### Setup

```bash
# Install Alembic
pip install alembic

# Initialize Alembic
alembic init alembic
```

### Configuration

```python
# alembic/env.py
from sqlmodel import SQLModel
from models import *  # Import all models
from config import settings

# Set target metadata
target_metadata = SQLModel.metadata

# Set database URL
def get_url():
    return settings.DATABASE_URL

config.set_main_option("sqlalchemy.url", get_url())
```

### Creating Migrations

```bash
# Auto-generate migration from model changes
alembic revision --autogenerate -m "Add items table"

# Create empty migration
alembic revision -m "Custom migration"

# Apply migrations
alembic upgrade head

# Rollback one migration
alembic downgrade -1

# Show current version
alembic current

# Show migration history
alembic history
```

### Manual Migration

```python
# alembic/versions/xxx_add_column.py
from alembic import op
import sqlalchemy as sa

def upgrade():
    """Add new column"""
    op.add_column('items', sa.Column('status', sa.String(50), nullable=True))

    # Set default value for existing rows
    op.execute("UPDATE items SET status = 'active' WHERE status IS NULL")

    # Make column non-nullable
    op.alter_column('items', 'status', nullable=False)

def downgrade():
    """Remove column"""
    op.drop_column('items', 'status')
```

## Performance Optimization

### Query Optimization

```python
# ❌ BAD: N+1 query problem
items = session.exec(select(Item)).all()
for item in items:
    print(item.owner.username)  # Separate query for each item!

# ✅ GOOD: Eager loading with join
from sqlmodel import selectinload

statement = select(Item).options(selectinload(Item.owner))
items = session.exec(statement).all()
for item in items:
    print(item.owner.username)  # No additional queries
```

### Batch Operations

```python
# ❌ BAD: Individual commits
for data in items_data:
    item = Item(**data)
    session.add(item)
    session.commit()  # Slow!

# ✅ GOOD: Batch commit
items = [Item(**data) for data in items_data]
session.add_all(items)
session.commit()  # Single commit
```

### Connection Pooling

```python
from sqlalchemy.pool import QueuePool

engine = create_engine(
    settings.DATABASE_URL,
    poolclass=QueuePool,
    pool_size=5,          # Number of connections to keep
    max_overflow=10,      # Max additional connections
    pool_timeout=30,      # Timeout waiting for connection
    pool_recycle=3600,    # Recycle connections after 1 hour
)
```

### Database Maintenance

```python
# Vacuum database (reclaim space)
with engine.connect() as conn:
    conn.execute("VACUUM")

# Analyze database (update statistics)
with engine.connect() as conn:
    conn.execute("ANALYZE")

# Check integrity
with engine.connect() as conn:
    result = conn.execute("PRAGMA integrity_check")
    print(result.fetchall())
```

## SQLite-Specific Features

### Enabling WAL Mode

```python
# Enable Write-Ahead Logging for better concurrency
with engine.connect() as conn:
    conn.execute("PRAGMA journal_mode=WAL")
```

### Foreign Key Support

```python
# Enable foreign key constraints (disabled by default in SQLite)
with engine.connect() as conn:
    conn.execute("PRAGMA foreign_keys=ON")
```

### Performance Settings

```python
with engine.connect() as conn:
    # Increase cache size (in KB)
    conn.execute("PRAGMA cache_size=-64000")  # 64MB

    # Synchronous mode (faster but less safe)
    conn.execute("PRAGMA synchronous=NORMAL")

    # Temp store in memory
    conn.execute("PRAGMA temp_store=MEMORY")
```

## Error Handling

### Common Errors

```python
from sqlalchemy.exc import IntegrityError, OperationalError

try:
    session.add(item)
    session.commit()
except IntegrityError as e:
    session.rollback()
    # Handle unique constraint violation, foreign key violation, etc.
    if "UNIQUE constraint failed" in str(e):
        raise ValueError("Item already exists")
    elif "FOREIGN KEY constraint failed" in str(e):
        raise ValueError("Invalid foreign key reference")
    else:
        raise
except OperationalError as e:
    session.rollback()
    # Handle database locked, disk full, etc.
    if "database is locked" in str(e):
        raise ValueError("Database is busy, please try again")
    else:
        raise
```

## Testing

### Test Database Setup

```python
# tests/conftest.py
import pytest
from sqlmodel import Session, create_engine, SQLModel
from sqlmodel.pool import StaticPool

@pytest.fixture(name="session")
def session_fixture():
    """Create in-memory test database"""
    engine = create_engine(
        "sqlite:///:memory:",
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    SQLModel.metadata.create_all(engine)

    with Session(engine) as session:
        yield session
```

### Test Example

```python
def test_create_item(session: Session):
    """Test creating an item"""
    item = Item(name="Test Item", price=9.99)
    session.add(item)
    session.commit()
    session.refresh(item)

    assert item.id is not None
    assert item.name == "Test Item"
    assert item.price == 9.99

def test_unique_constraint(session: Session):
    """Test unique constraint"""
    user1 = User(email="test@example.com", username="test")
    session.add(user1)
    session.commit()

    user2 = User(email="test@example.com", username="test2")
    session.add(user2)

    with pytest.raises(IntegrityError):
        session.commit()
```

## Best Practices Checklist

### For Every Model
- [ ] Primary key defined
- [ ] Indexes on frequently queried fields
- [ ] Foreign keys for relationships
- [ ] Constraints (unique, not null, check)
- [ ] Default values where appropriate
- [ ] Timestamps (created_at, updated_at)
- [ ] Type hints for all fields
- [ ] Docstring explaining purpose

### For Every Query
- [ ] Use ORM, not raw SQL
- [ ] Handle None/empty results
- [ ] Use pagination for large datasets
- [ ] Eager load relationships if needed
- [ ] Add appropriate indexes
- [ ] Use transactions for multi-step operations
- [ ] Handle database errors gracefully

### For Production
- [ ] Enable WAL mode
- [ ] Enable foreign keys
- [ ] Set up regular backups
- [ ] Monitor database size
- [ ] Run VACUUM periodically
- [ ] Consider migration to PostgreSQL if needed
- [ ] Implement connection pooling
- [ ] Add query logging for debugging

## Migration to PostgreSQL

When your application outgrows SQLite, migrate to PostgreSQL:

```python
# Change DATABASE_URL
DATABASE_URL = "postgresql://user:password@localhost/dbname"

# Update engine configuration
engine = create_engine(
    DATABASE_URL,
    pool_size=20,
    max_overflow=0,
)

# Most SQLModel code works unchanged!
# Only SQLite-specific features need updating
```

## Summary

Follow these rules to build a robust, performant, and maintainable database layer that integrates seamlessly with FastAPI and supports the React frontend.
