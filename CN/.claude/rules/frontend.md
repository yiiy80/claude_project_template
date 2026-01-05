# 前端规范 (React/Vite/TypeScript/Tailwind CSS)

## 概述

前端采用 React + Vite + TypeScript + Tailwind CSS 的现代化技术栈，注重类型安全、开发体验和性能优化。

## 项目结构

```
frontend/
├── src/
│   ├── components/          # 可复用组件
│   │   ├── ui/             # 基础UI组件
│   │   └── features/       # 业务功能组件
│   ├── hooks/              # 自定义Hooks
│   ├── lib/                # 工具函数和配置
│   ├── pages/              # 页面组件
│   ├── services/           # API服务层
│   ├── types/              # TypeScript类型定义
│   ├── utils/              # 工具函数
│   ├── App.tsx             # 应用根组件
│   ├── main.tsx            # 应用入口
│   └── index.css           # 全局样式
├── public/                 # 静态资源
├── index.html              # HTML模板
├── package.json
├── tsconfig.json
├── tailwind.config.js
└── vite.config.ts
```

## React 规范

### 组件设计原则

#### 1. 函数组件优先
```tsx
// ✅ 推荐：函数组件 + Hooks
const UserProfile: React.FC<UserProfileProps> = ({ userId }) => {
  const { data: user, loading } = useUser(userId);

  if (loading) return <LoadingSpinner />;

  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};
```

#### 2. 组件拆分
- 单个组件不超过 200 行
- 职责单一原则
- 可复用组件提取到 `components/ui/`
- 业务组件放在 `components/features/`

#### 3. Props 设计
```tsx
// ✅ 推荐：明确的类型定义
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  onClick: () => void;
}

const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'md',
  disabled = false,
  onClick,
}) => {
  // 组件实现
};
```

### Hooks 使用规范

#### 1. 自定义 Hooks
```tsx
// ✅ 推荐：将业务逻辑提取到自定义Hook
const useUser = (userId: string) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await api.getUser(userId);
        setUser(response.data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  return { user, loading, error };
};
```

#### 2. Hooks 依赖管理
- 总是添加完整的依赖数组
- 使用 `useCallback` 和 `useMemo` 优化性能
- 避免不必要的重新渲染

## TypeScript 规范

### 类型定义

#### 1. 接口命名
```tsx
// ✅ 推荐：使用 I 前缀或 Props 后缀
interface User {
  id: string;
  name: string;
  email: string;
}

interface UserProfileProps {
  user: User;
  onEdit: (user: User) => void;
}

// API 响应类型
interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}
```

#### 2. 泛型使用
```tsx
// ✅ 推荐：使用泛型提高复用性
interface SelectProps<T> {
  options: T[];
  value: T | null;
  onChange: (value: T) => void;
  renderOption: (option: T) => string;
}

const Select = <T,>({ options, value, onChange, renderOption }: SelectProps<T>) => {
  // 组件实现
};
```

#### 3. 联合类型和枚举
```tsx
// ✅ 推荐：使用联合类型
type ButtonVariant = 'primary' | 'secondary' | 'danger';
type ButtonSize = 'sm' | 'md' | 'lg';

// ✅ 推荐：使用 const 断言
const BUTTON_VARIANTS = ['primary', 'secondary', 'danger'] as const;
type ButtonVariant = typeof BUTTON_VARIANTS[number];
```

### 严格模式配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

## Tailwind CSS 规范

### 样式组织

#### 1. 组件样式
```tsx
// ✅ 推荐：直接在组件中使用 Tailwind 类
const Card = ({ title, children }: CardProps) => {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 border border-gray-200">
      <h3 className="text-lg font-semibold text-gray-900 mb-4">{title}</h3>
      <div className="text-gray-700">{children}</div>
    </div>
  );
};
```

#### 2. 响应式设计
```tsx
// ✅ 推荐：使用响应式前缀
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* 内容 */}
</div>
```

#### 3. 自定义样式
```tsx
// ✅ 推荐：为复杂样式创建组件或使用 @apply
const Button = ({ variant, children }: ButtonProps) => {
  const baseStyles = "px-4 py-2 rounded-md font-medium transition-colors focus:outline-none focus:ring-2";

  const variants = {
    primary: "bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500",
    secondary: "bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-500",
  };

  return (
    <button className={`${baseStyles} ${variants[variant]}`}>
      {children}
    </button>
  );
};
```

### Tailwind 配置

```js
// tailwind.config.js
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
        },
      },
      fontFamily: {
        sans: ['Inter', 'ui-sans-serif', 'system-ui'],
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

## Vite 配置和优化

### 开发配置

```ts
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
})
```

### 构建优化

#### 1. 代码分割
```tsx
// ✅ 推荐：路由级别的代码分割
const UserManagement = lazy(() => import('./pages/UserManagement'));
const Reports = lazy(() => import('./pages/Reports'));

const App = () => {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/users" element={<UserManagement />} />
        <Route path="/reports" element={<Reports />} />
      </Routes>
    </Suspense>
  );
};
```

#### 2. 资源优化
- 图片压缩和优化
- CSS 和 JS 代码压缩
- Tree shaking 移除未使用代码

## 状态管理

### 轻量级应用
对于中小型应用，推荐使用：
- React useState/useReducer (组件级别)
- Context API (全局状态)
- React Query (服务端状态)

### 中大型应用
考虑引入：
- Zustand
- Redux Toolkit
- Jotai

## 测试规范

### 单元测试
```tsx
// ✅ 推荐：使用 Testing Library
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders children correctly', () => {
    render(<Button onClick={() => {}}>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

## 性能优化

### 1. React 优化
- 使用 React.memo 避免不必要的重新渲染
- 使用 useCallback 和 useMemo 优化计算
- 合理使用 key 属性

### 2. 打包优化
- 代码分割和懒加载
- Bundle 分析和优化
- CDN 资源加载

### 3. 图片优化
- 使用适当的图片格式
- 响应式图片
- 懒加载非关键图片

## 代码规范工具

### ESLint 配置
```js
// .eslintrc.js
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
  ],
  parser: '@typescript-eslint/parser',
  plugins: ['react', '@typescript-eslint'],
  rules: {
    // 自定义规则
  },
};
```

### Prettier 配置
```js
// .prettierrc.js
module.exports = {
  semi: false,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 100,
  tabWidth: 2,
};
