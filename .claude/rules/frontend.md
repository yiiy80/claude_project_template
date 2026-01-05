# Frontend Development Rules

This document defines the rules and best practices for frontend development with React, Vite, TypeScript, and Tailwind CSS.

## Core Principles

### 1. Strict TypeScript
- **NO `any` types** - Use `unknown` if type is truly unknown
- **Explicit return types** for all functions
- **Strict null checks** - Handle undefined/null explicitly
- **Interface over type** for object shapes
- **Generics** for reusable components

### 2. Component Architecture
- **Functional components only** - No class components
- **Single responsibility** - One component, one purpose
- **Composition over inheritance**
- **Props interface** for every component
- **Default exports** for page components, named for others

### 3. No Backend Dependencies
- **NEVER import backend code** directly
- **All API calls** go through `services/` layer
- **Type definitions** mirror backend but are separate
- **No database access** from frontend
- **No file system operations**

## Project Structure

```
frontend/
├── src/
│   ├── components/          # Reusable UI components
│   │   ├── common/         # Generic components (Button, Input, etc.)
│   │   ├── features/       # Feature-specific components
│   │   └── layouts/        # Layout components (Header, Footer, etc.)
│   ├── pages/              # Page components (one per route)
│   ├── services/           # API communication layer
│   │   ├── api.ts         # Base API client
│   │   └── items.ts       # Resource-specific API calls
│   ├── types/              # TypeScript type definitions
│   │   ├── api.ts         # API response types
│   │   └── models.ts      # Domain model types
│   ├── hooks/              # Custom React hooks
│   │   ├── useApi.ts      # API call hook
│   │   └── useAuth.ts     # Authentication hook
│   ├── utils/              # Utility functions
│   │   ├── format.ts      # Formatting utilities
│   │   └── validation.ts  # Client-side validation
│   ├── contexts/           # React Context providers
│   ├── App.tsx             # Root component
│   ├── main.tsx            # Entry point
│   └── index.css           # Global styles (Tailwind imports)
├── public/                 # Static assets
├── index.html              # HTML template
├── vite.config.ts          # Vite configuration
├── tsconfig.json           # TypeScript configuration
├── tailwind.config.js      # Tailwind configuration
├── postcss.config.js       # PostCSS configuration
└── package.json            # Dependencies
```

## TypeScript Rules

### Type Definitions

```typescript
// ✅ GOOD: Explicit types
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: string; // ISO date string from API
}

interface ApiResponse<T> {
  data: T;
  message?: string;
}

// ✅ GOOD: Function with explicit return type
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error('Failed to fetch user');
  }
  return response.json();
}

// ❌ BAD: Using any
function processData(data: any) { // NEVER DO THIS
  return data.value;
}

// ✅ GOOD: Use unknown and type guard
function processData(data: unknown): string {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return String((data as { value: unknown }).value);
  }
  throw new Error('Invalid data');
}
```

### Component Props

```typescript
// ✅ GOOD: Props interface with explicit types
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  className?: string;
}

export function Button({
  children,
  onClick,
  variant = 'primary',
  disabled = false,
  className = ''
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant} ${className}`}
    >
      {children}
    </button>
  );
}

// ❌ BAD: No props interface
export function Button(props) { // NEVER DO THIS
  return <button {...props} />;
}
```

### Hooks Typing

```typescript
// ✅ GOOD: Typed hooks
function useItems() {
  const [items, setItems] = useState<Item[]>([]);
  const [loading, setLoading] = useState<boolean>(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    async function loadItems() {
      setLoading(true);
      try {
        const data = await fetchItems();
        setItems(data);
      } catch (err) {
        setError(err instanceof Error ? err : new Error('Unknown error'));
      } finally {
        setLoading(false);
      }
    }
    loadItems();
  }, []);

  return { items, loading, error };
}
```

## React Component Rules

### Component Structure

```typescript
// ✅ GOOD: Well-structured component
import { useState, useEffect } from 'react';
import { fetchItems } from '../services/items';
import type { Item } from '../types/models';

interface ItemListProps {
  categoryId?: number;
  onItemClick: (item: Item) => void;
}

export function ItemList({ categoryId, onItemClick }: ItemListProps) {
  const [items, setItems] = useState<Item[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function loadItems() {
      try {
        setLoading(true);
        const data = await fetchItems(categoryId);
        setItems(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Failed to load items');
      } finally {
        setLoading(false);
      }
    }
    loadItems();
  }, [categoryId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div className="text-red-500">{error}</div>;
  if (items.length === 0) return <div>No items found</div>;

  return (
    <div className="grid gap-4">
      {items.map(item => (
        <ItemCard
          key={item.id}
          item={item}
          onClick={() => onItemClick(item)}
        />
      ))}
    </div>
  );
}
```

### State Management

```typescript
// ✅ GOOD: Local state for component-specific data
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}

// ✅ GOOD: Context for shared state
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    const userData = await loginApi(email, password);
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

### Custom Hooks

```typescript
// ✅ GOOD: Reusable API hook
interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => Promise<void>;
}

function useApi<T>(
  apiCall: () => Promise<T>,
  dependencies: unknown[] = []
): UseApiResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const result = await apiCall();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  }, [apiCall]);

  useEffect(() => {
    fetchData();
  }, dependencies);

  return { data, loading, error, refetch: fetchData };
}

// Usage
function UserProfile({ userId }: { userId: number }) {
  const { data: user, loading, error } = useApi(
    () => fetchUser(userId),
    [userId]
  );

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return <div>User not found</div>;

  return <div>{user.name}</div>;
}
```

## API Communication

### Service Layer Structure

```typescript
// services/api.ts - Base API client
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000';

export class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public data?: unknown
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

async function handleResponse<T>(response: Response): Promise<T> {
  if (!response.ok) {
    const errorData = await response.json().catch(() => ({}));
    throw new ApiError(
      errorData.detail || 'Request failed',
      response.status,
      errorData
    );
  }
  return response.json();
}

export async function apiGet<T>(endpoint: string): Promise<T> {
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
    },
  });
  return handleResponse<T>(response);
}

export async function apiPost<T, D = unknown>(
  endpoint: string,
  data: D
): Promise<T> {
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(data),
  });
  return handleResponse<T>(response);
}

export async function apiPut<T, D = unknown>(
  endpoint: string,
  data: D
): Promise<T> {
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(data),
  });
  return handleResponse<T>(response);
}

export async function apiDelete<T>(endpoint: string): Promise<T> {
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    method: 'DELETE',
    headers: {
      'Content-Type': 'application/json',
    },
  });
  return handleResponse<T>(response);
}
```

```typescript
// services/items.ts - Resource-specific API
import { apiGet, apiPost, apiPut, apiDelete } from './api';
import type { Item, CreateItemRequest, UpdateItemRequest } from '../types/models';

export async function fetchItems(categoryId?: number): Promise<Item[]> {
  const query = categoryId ? `?category_id=${categoryId}` : '';
  return apiGet<Item[]>(`/api/items${query}`);
}

export async function fetchItem(id: number): Promise<Item> {
  return apiGet<Item>(`/api/items/${id}`);
}

export async function createItem(data: CreateItemRequest): Promise<Item> {
  return apiPost<Item, CreateItemRequest>('/api/items', data);
}

export async function updateItem(
  id: number,
  data: UpdateItemRequest
): Promise<Item> {
  return apiPut<Item, UpdateItemRequest>(`/api/items/${id}`, data);
}

export async function deleteItem(id: number): Promise<void> {
  return apiDelete<void>(`/api/items/${id}`);
}
```

### Error Handling

```typescript
// ✅ GOOD: Comprehensive error handling
async function handleSubmit(data: FormData) {
  try {
    setLoading(true);
    setError(null);
    await createItem(data);
    setSuccess(true);
    navigate('/items');
  } catch (err) {
    if (err instanceof ApiError) {
      if (err.status === 400) {
        setError('Invalid data. Please check your input.');
      } else if (err.status === 401) {
        setError('Please log in to continue.');
        navigate('/login');
      } else if (err.status === 403) {
        setError('You do not have permission to perform this action.');
      } else if (err.status === 404) {
        setError('Resource not found.');
      } else if (err.status >= 500) {
        setError('Server error. Please try again later.');
      } else {
        setError(err.message);
      }
    } else {
      setError('An unexpected error occurred.');
      console.error('Unexpected error:', err);
    }
  } finally {
    setLoading(false);
  }
}
```

## Tailwind CSS Rules

### Utility-First Approach

```typescript
// ✅ GOOD: Utility classes directly in JSX
function Card({ title, children }: CardProps) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
      <h2 className="text-2xl font-bold text-gray-800 mb-4">{title}</h2>
      <div className="text-gray-600">{children}</div>
    </div>
  );
}

// ✅ GOOD: Responsive design
function Hero() {
  return (
    <div className="container mx-auto px-4">
      <h1 className="text-3xl md:text-4xl lg:text-5xl font-bold">
        Welcome
      </h1>
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {/* Content */}
      </div>
    </div>
  );
}

// ✅ GOOD: Conditional classes
function Button({ variant, disabled }: ButtonProps) {
  const baseClasses = "px-4 py-2 rounded font-medium transition-colors";
  const variantClasses = {
    primary: "bg-blue-500 hover:bg-blue-600 text-white",
    secondary: "bg-gray-200 hover:bg-gray-300 text-gray-800",
  };
  const disabledClasses = "opacity-50 cursor-not-allowed";

  return (
    <button
      className={`
        ${baseClasses}
        ${variantClasses[variant]}
        ${disabled ? disabledClasses : ''}
      `}
      disabled={disabled}
    >
      Click me
    </button>
  );
}

// ✅ BETTER: Use clsx or classnames library
import clsx from 'clsx';

function Button({ variant, disabled }: ButtonProps) {
  return (
    <button
      className={clsx(
        "px-4 py-2 rounded font-medium transition-colors",
        {
          "bg-blue-500 hover:bg-blue-600 text-white": variant === 'primary',
          "bg-gray-200 hover:bg-gray-300 text-gray-800": variant === 'secondary',
          "opacity-50 cursor-not-allowed": disabled,
        }
      )}
      disabled={disabled}
    >
      Click me
    </button>
  );
}
```

### Custom Configuration

```javascript
// tailwind.config.js
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          100: '#dbeafe',
          500: '#3b82f6',
          600: '#2563eb',
          900: '#1e3a8a',
        },
      },
      spacing: {
        '128': '32rem',
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
    },
  },
  plugins: [],
}
```

## Form Handling

```typescript
// ✅ GOOD: Controlled form with validation
interface FormData {
  name: string;
  email: string;
  message: string;
}

interface FormErrors {
  name?: string;
  email?: string;
  message?: string;
}

function ContactForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    email: '',
    message: '',
  });
  const [errors, setErrors] = useState<FormErrors>({});
  const [submitting, setSubmitting] = useState(false);

  function validateForm(): boolean {
    const newErrors: FormErrors = {};

    if (!formData.name.trim()) {
      newErrors.name = 'Name is required';
    }

    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) {
      newErrors.email = 'Invalid email format';
    }

    if (!formData.message.trim()) {
      newErrors.message = 'Message is required';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();

    if (!validateForm()) return;

    try {
      setSubmitting(true);
      await submitContactForm(formData);
      alert('Form submitted successfully!');
      setFormData({ name: '', email: '', message: '' });
    } catch (err) {
      alert('Failed to submit form');
    } finally {
      setSubmitting(false);
    }
  }

  function handleChange(
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    // Clear error when user starts typing
    if (errors[name as keyof FormErrors]) {
      setErrors(prev => ({ ...prev, [name]: undefined }));
    }
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label htmlFor="name" className="block text-sm font-medium mb-1">
          Name
        </label>
        <input
          type="text"
          id="name"
          name="name"
          value={formData.name}
          onChange={handleChange}
          className={clsx(
            "w-full px-3 py-2 border rounded-md",
            errors.name ? "border-red-500" : "border-gray-300"
          )}
        />
        {errors.name && (
          <p className="text-red-500 text-sm mt-1">{errors.name}</p>
        )}
      </div>

      <div>
        <label htmlFor="email" className="block text-sm font-medium mb-1">
          Email
        </label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          className={clsx(
            "w-full px-3 py-2 border rounded-md",
            errors.email ? "border-red-500" : "border-gray-300"
          )}
        />
        {errors.email && (
          <p className="text-red-500 text-sm mt-1">{errors.email}</p>
        )}
      </div>

      <div>
        <label htmlFor="message" className="block text-sm font-medium mb-1">
          Message
        </label>
        <textarea
          id="message"
          name="message"
          value={formData.message}
          onChange={handleChange}
          rows={4}
          className={clsx(
            "w-full px-3 py-2 border rounded-md",
            errors.message ? "border-red-500" : "border-gray-300"
          )}
        />
        {errors.message && (
          <p className="text-red-500 text-sm mt-1">{errors.message}</p>
        )}
      </div>

      <button
        type="submit"
        disabled={submitting}
        className="w-full bg-blue-500 text-white py-2 rounded-md hover:bg-blue-600 disabled:opacity-50"
      >
        {submitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

## Performance Optimization

### Code Splitting

```typescript
// ✅ GOOD: Lazy load pages
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const HomePage = lazy(() => import('./pages/HomePage'));
const AboutPage = lazy(() => import('./pages/AboutPage'));
const ItemsPage = lazy(() => import('./pages/ItemsPage'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/about" element={<AboutPage />} />
          <Route path="/items" element={<ItemsPage />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### Memoization

```typescript
// ✅ GOOD: Memoize expensive computations
function ItemList({ items, filter }: ItemListProps) {
  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  return (
    <div>
      {filteredItems.map(item => (
        <ItemCard key={item.id} item={item} />
      ))}
    </div>
  );
}

// ✅ GOOD: Memoize callbacks
function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []); // Stable reference

  return <Child onClick={handleClick} />;
}

const Child = memo(({ onClick }: { onClick: () => void }) => {
  return <button onClick={onClick}>Click me</button>;
});
```

## Testing

```typescript
// ✅ GOOD: Component test
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { ItemList } from './ItemList';
import { fetchItems } from '../services/items';

jest.mock('../services/items');

describe('ItemList', () => {
  it('displays loading state initially', () => {
    render(<ItemList />);
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });

  it('displays items after loading', async () => {
    const mockItems = [
      { id: 1, name: 'Item 1' },
      { id: 2, name: 'Item 2' },
    ];
    (fetchItems as jest.Mock).mockResolvedValue(mockItems);

    render(<ItemList />);

    await waitFor(() => {
      expect(screen.getByText('Item 1')).toBeInTheDocument();
      expect(screen.getByText('Item 2')).toBeInTheDocument();
    });
  });

  it('displays error message on failure', async () => {
    (fetchItems as jest.Mock).mockRejectedValue(new Error('Failed'));

    render(<ItemList />);

    await waitFor(() => {
      expect(screen.getByText(/failed/i)).toBeInTheDocument();
    });
  });
});
```

## Common Patterns

### Loading States

```typescript
function DataComponent() {
  const { data, loading, error } = useApi(fetchData);

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-500" />
      </div>
    );
  }

  if (error) {
    return (
      <div className="bg-red-50 border border-red-200 rounded-md p-4">
        <p className="text-red-800">{error.message}</p>
      </div>
    );
  }

  if (!data) {
    return (
      <div className="text-center text-gray-500 py-8">
        No data available
      </div>
    );
  }

  return <div>{/* Render data */}</div>;
}
```

### Modal Pattern

```typescript
interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

function Modal({ isOpen, onClose, title, children }: ModalProps) {
  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center">
      <div
        className="absolute inset-0 bg-black bg-opacity-50"
        onClick={onClose}
      />
      <div className="relative bg-white rounded-lg shadow-xl max-w-md w-full mx-4 p-6">
        <div className="flex justify-between items-center mb-4">
          <h2 className="text-xl font-bold">{title}</h2>
          <button
            onClick={onClose}
            className="text-gray-500 hover:text-gray-700"
          >
            ✕
          </button>
        </div>
        <div>{children}</div>
      </div>
    </div>
  );
}
```

## Checklist for Every Component

- [ ] TypeScript types defined for all props
- [ ] No `any` types used
- [ ] Loading state implemented
- [ ] Error handling implemented
- [ ] Empty state handled
- [ ] Accessibility attributes added (aria-*, role)
- [ ] Responsive design with Tailwind
- [ ] API calls go through services layer
- [ ] No direct backend imports
- [ ] Component is focused and reusable
- [ ] Proper key props for lists
- [ ] Event handlers are memoized if needed
- [ ] Form inputs are controlled
- [ ] Validation implemented for forms

## Anti-Patterns to Avoid

❌ **Direct backend imports**
```typescript
import { database } from '../../backend/database'; // NEVER
```

❌ **Using `any` type**
```typescript
function process(data: any) { // NEVER
```

❌ **Inline styles instead of Tailwind**
```typescript
<div style={{ color: 'red' }}> // Avoid, use Tailwind
```

❌ **Uncontrolled forms**
```typescript
<input defaultValue="test" /> // Prefer controlled
```

❌ **Missing error boundaries**
```typescript
// Always wrap risky components in error boundaries
```

❌ **Prop drilling**
```typescript
// Use Context API for deeply nested props
```

❌ **Mutating state directly**
```typescript
items.push(newItem); // NEVER
setItems(items); // This won't trigger re-render
// Use: setItems([...items, newItem])
```

## Summary

Follow these rules to maintain a clean, type-safe, and maintainable frontend codebase that works seamlessly with the FastAPI backend.
