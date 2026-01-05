# Project Setup Tool

This document provides commands and scripts for setting up and managing the full-stack project.

## Initial Setup

### Prerequisites

Ensure you have the following installed:
- **Node.js** 18+ and npm
- **Python** 3.10+
- **Git**

### Quick Start

```bash
# Clone or create project
git clone <repository-url>
cd <project-name>

# Run setup script
./init.sh
```

## Setup Script (init.sh)

Create this script in the project root:

```bash
#!/bin/bash
# init.sh - Project initialization script

set -e  # Exit on error

echo "🚀 Setting up full-stack project..."

# Check prerequisites
echo "📋 Checking prerequisites..."

if ! command -v node &> /dev/null; then
    echo "❌ Node.js is not installed. Please install Node.js 18+"
    exit 1
fi

if ! command -v python3 &> /dev/null; then
    echo "❌ Python 3 is not installed. Please install Python 3.10+"
    exit 1
fi

echo "✅ Prerequisites check passed"

# Backend setup
echo ""
echo "🐍 Setting up backend..."
cd backend

# Create virtual environment if it doesn't exist
if [ ! -d "venv" ]; then
    echo "Creating Python virtual environment..."
    python3 -m venv venv
fi

# Activate virtual environment
source venv/bin/activate

# Install dependencies
echo "Installing Python dependencies..."
pip install --upgrade pip
pip install -r requirements.txt

# Create .env file if it doesn't exist
if [ ! -f ".env" ]; then
    echo "Creating .env file..."
    cat > .env << EOF
DATABASE_URL=sqlite:///./app.db
SECRET_KEY=$(python3 -c 'import secrets; print(secrets.token_urlsafe(32))')
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
CORS_ORIGINS=["http://localhost:5173"]
DEBUG=True
EOF
    echo "✅ Created .env file with generated SECRET_KEY"
fi

# Initialize database
echo "Initializing database..."
python3 -c "from database import init_db; init_db()"

echo "✅ Backend setup complete"
cd ..

# Frontend setup
echo ""
echo "⚛️  Setting up frontend..."
cd frontend

# Install dependencies
echo "Installing Node.js dependencies..."
npm install

# Create .env file if it doesn't exist
if [ ! -f ".env" ]; then
    echo "Creating .env file..."
    cat > .env << EOF
VITE_API_BASE_URL=http://localhost:8000
EOF
    echo "✅ Created .env file"
fi

echo "✅ Frontend setup complete"
cd ..

echo ""
echo "✅ Setup complete!"
echo ""
echo "📚 Next steps:"
echo "  1. Start backend:  cd backend && source venv/bin/activate && uvicorn main:app --reload"
echo "  2. Start frontend: cd frontend && npm run dev"
echo "  3. Open browser:   http://localhost:5173"
echo "  4. API docs:       http://localhost:8000/docs"
echo ""
```

Make the script executable:

```bash
chmod +x init.sh
```

## Manual Setup

If you prefer to set up manually:

### Backend Setup

```bash
# Navigate to backend directory
cd backend

# Create virtual environment
python3 -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate
# On Windows:
venv\Scripts\activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt

# Create .env file
cat > .env << EOF
DATABASE_URL=sqlite:///./app.db
SECRET_KEY=your-secret-key-here-change-in-production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
CORS_ORIGINS=["http://localhost:5173"]
DEBUG=True
EOF

# Initialize database
python -c "from database import init_db; init_db()"
```

### Frontend Setup

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Create .env file
cat > .env << EOF
VITE_API_BASE_URL=http://localhost:8000
EOF
```

## Development Commands

### Backend

```bash
cd backend

# Activate virtual environment
source venv/bin/activate  # macOS/Linux
venv\Scripts\activate     # Windows

# Start development server
uvicorn main:app --reload

# Start with custom host/port
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Run tests
pytest

# Run tests with coverage
pytest --cov=. --cov-report=html

# Format code
black .

# Lint code
flake8 .

# Type check
mypy .

# Create database migration
alembic revision --autogenerate -m "Description"

# Apply migrations
alembic upgrade head

# Rollback migration
alembic downgrade -1

# Open database CLI
sqlite3 app.db
```

### Frontend

```bash
cd frontend

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Run linter
npm run lint

# Fix linting issues
npm run lint:fix

# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Type check
npm run type-check
```

## Project Structure Creation

### Create Backend Structure

```bash
mkdir -p backend/{routers,services,utils,tests}
cd backend

# Create main files
touch main.py database.py models.py schemas.py config.py dependencies.py

# Create router files
touch routers/__init__.py routers/items.py routers/users.py routers/auth.py

# Create service files
touch services/__init__.py services/item_service.py services/user_service.py

# Create utility files
touch utils/__init__.py utils/security.py utils/validators.py utils/exceptions.py

# Create test files
touch tests/__init__.py tests/conftest.py tests/test_items.py tests/test_users.py

# Create requirements.txt
cat > requirements.txt << EOF
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlmodel==0.0.14
pydantic==2.5.0
pydantic-settings==2.1.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
alembic==1.13.0
pytest==7.4.3
pytest-cov==4.1.0
httpx==0.25.2
EOF
```

### Create Frontend Structure

```bash
mkdir -p frontend/src/{components/{common,features,layouts},pages,services,types,hooks,utils,contexts}
cd frontend

# Create main files
touch src/main.tsx src/App.tsx src/index.css

# Create component files
touch src/components/common/Button.tsx
touch src/components/common/Input.tsx
touch src/components/layouts/Header.tsx
touch src/components/layouts/Footer.tsx

# Create page files
touch src/pages/HomePage.tsx
touch src/pages/ItemsPage.tsx
touch src/pages/LoginPage.tsx

# Create service files
touch src/services/api.ts
touch src/services/items.ts
touch src/services/auth.ts

# Create type files
touch src/types/models.ts
touch src/types/api.ts

# Create hook files
touch src/hooks/useApi.ts
touch src/hooks/useAuth.ts

# Create utility files
touch src/utils/format.ts
touch src/utils/validation.ts

# Create package.json
cat > package.json << EOF
{
  "name": "frontend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0",
    "clsx": "^2.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.43",
    "@types/react-dom": "^18.2.17",
    "@typescript-eslint/eslint-plugin": "^6.14.0",
    "@typescript-eslint/parser": "^6.14.0",
    "@vitejs/plugin-react": "^4.2.1",
    "autoprefixer": "^10.4.16",
    "eslint": "^8.55.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.5",
    "postcss": "^8.4.32",
    "tailwindcss": "^3.3.6",
    "typescript": "^5.2.2",
    "vite": "^5.0.8"
  }
}
EOF

# Create vite.config.ts
cat > vite.config.ts << EOF
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      }
    }
  }
})
EOF

# Create tsconfig.json
cat > tsconfig.json << EOF
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
    "noImplicitReturns": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
EOF

# Create tailwind.config.js
cat > tailwind.config.js << EOF
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
EOF

# Create postcss.config.js
cat > postcss.config.js << EOF
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
EOF

# Create index.html
cat > index.html << EOF
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Full-Stack App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
EOF
```

## Docker Setup (Optional)

### Backend Dockerfile

```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Frontend Dockerfile

```dockerfile
# frontend/Dockerfile
FROM node:18-alpine as build

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Build application
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=sqlite:///./app.db
      - SECRET_KEY=${SECRET_KEY}
    volumes:
      - ./backend:/app
      - backend-data:/app/data

  frontend:
    build: ./frontend
    ports:
      - "5173:80"
    depends_on:
      - backend
    environment:
      - VITE_API_BASE_URL=http://localhost:8000

volumes:
  backend-data:
```

## Environment Variables

### Backend (.env)

```bash
# Database
DATABASE_URL=sqlite:///./app.db

# Security
SECRET_KEY=your-secret-key-change-in-production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# CORS
CORS_ORIGINS=["http://localhost:5173"]

# Application
DEBUG=True
```

### Frontend (.env)

```bash
# API Configuration
VITE_API_BASE_URL=http://localhost:8000

# Application
VITE_APP_TITLE=My Full-Stack App
```

## Troubleshooting

### Backend Issues

**Database locked error:**
```bash
# Stop all running instances
pkill -f uvicorn

# Delete database and recreate
rm app.db
python -c "from database import init_db; init_db()"
```

**Module not found:**
```bash
# Ensure virtual environment is activated
source venv/bin/activate

# Reinstall dependencies
pip install -r requirements.txt
```

**CORS errors:**
```bash
# Check CORS_ORIGINS in .env matches frontend URL
# Restart backend after changing .env
```

### Frontend Issues

**Module not found:**
```bash
# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

**Build errors:**
```bash
# Clear cache and rebuild
rm -rf dist .vite
npm run build
```

**API connection errors:**
```bash
# Check VITE_API_BASE_URL in .env
# Ensure backend is running
# Check browser console for CORS errors
```

## Production Deployment
### Backend

```bash
# Install production dependencies
pip install gunicorn

# Run with Gunicorn
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000

# Or use systemd service
sudo systemctl start myapp-backend
```

### Frontend

```bash
# Build for production
npm run build

# Deploy dist/ folder to:
# - Vercel
# - Netlify
# - Cloudflare Pages
# - AWS S3 + CloudFront
# - Your own server with Nginx
```

## Useful Commands

### Database

```bash
# Backup database
cp app.db app.db.backup

# Restore database
cp app.db.backup app.db

# View database schema
sqlite3 app.db ".schema"

# Export data
sqlite3 app.db ".dump" > backup.sql

# Import data
sqlite3 app.db < backup.sql
```

### Git

```bash
# Initial commit
git init
git add .
git commit -m "Initial commit"

# Create .gitignore
cat > .gitignore << EOF
# Python
__pycache__/
*.py[cod]
venv/
*.db

# Node
node_modules/
dist/

# Environment
.env
.env.local

# IDE
.vscode/
.idea/
EOF
```

## Summary

This setup provides a complete development environment for the React + FastAPI + SQLite full-stack application. Follow the quick start guide for the fastest setup, or use manual commands for more control.
