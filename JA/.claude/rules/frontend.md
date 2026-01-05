# フロントエンド規範 (React/Vite/TypeScript/Tailwind CSS)

## 概要

フロントエンドは React + Vite + TypeScript + Tailwind CSS の現代的な技術スタックを採用し、型安全性、開発体験、パフォーマンス最適化に重点を置いています。

## プロジェクト構造

```
frontend/
├── src/
│   ├── components/          # 再利用可能コンポーネント
│   │   ├── ui/             # 基礎UIコンポーネント
│   │   └── features/       # ビジネス機能コンポーネント
│   ├── hooks/              # カスタムHooks
│   ├── lib/                # ユーティリティ関数と設定
│   ├── pages/              # ページコンポーネント
│   ├── services/           # APIサービスレイヤー
│   ├── types/              # TypeScript型定義
│   ├── utils/              # ユーティリティ関数
│   ├── App.tsx             # アプリケーションベースコンポーネント
│   ├── main.tsx            # アプリケーションエントリ
│   └── index.css           # グローバルスタイル
├── public/                 # 静的リソース
├── index.html              # HTMLテンプレート
├── package.json
├── tsconfig.json
├── tailwind.config.js
└── vite.config.ts
```

## React 規範

### コンポーネント設計原則

#### 1. 関数コンポーネント優先
```tsx
// ✅ 推奨：関数コンポーネント + Hooks
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

#### 2. コンポーネント分割
- 単一コンポーネントは200行を超えない
- 単一責任原則
- 再利用可能コンポーネントを `components/ui/` に抽出
- ビジネスコンポーネントを `components/features/` に配置

#### 3. Props 設計
```tsx
// ✅ 推奨：明確な型定義
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
  // コンポーネント実装
};
```

### Hooks 使用規範

#### 1. カスタム Hooks
```tsx
// ✅ 推奨：ビジネスロジックをカスタムHookに抽出
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

#### 2. Hooks 依存関係管理
- 常に完全な依存関係配列を追加
- パフォーマンス最適化のために `useCallback` と `useMemo` を使用
- 不要な再レンダリングを避ける

## TypeScript 規範

### 型定義

#### 1. インターフェース命名
```tsx
// ✅ 推奨：I プレフィックスまたは Props サフィックスを使用
interface User {
  id: string;
  name: string;
  email: string;
}

interface UserProfileProps {
  user: User;
  onEdit: (user: User) => void;
}

// API 応答型
interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}
```

#### 2. ジェネリクス使用
```tsx
// ✅ 推奨：再利用性を高めるためにジェネリクスを使用
interface SelectProps<T> {
  options: T[];
  value: T | null;
  onChange: (value: T) => void;
  renderOption: (option: T) => string;
}

const Select = <T,>({ options, value, onChange, renderOption }: SelectProps<T>) => {
  // コンポーネント実装
};
```

#### 3. ユニオン型と列挙型
```tsx
// ✅ 推奨：ユニオン型を使用
type ButtonVariant = 'primary' | 'secondary' | 'danger';
type ButtonSize = 'sm' | 'md' | 'lg';

// ✅ 推奨：const アサーションを使用
const BUTTON_VARIANTS = ['primary', 'secondary', 'danger'] as const;
type ButtonVariant = typeof BUTTON_VARIANTS[number];
```

### 厳格モード設定

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

## Tailwind CSS 規範

### スタイル組織化

#### 1. コンポーネントスタイル
```tsx
// ✅ 推奨：コンポーネント内で直接 Tailwind クラスを使用
const Card = ({ title, children }: CardProps) => {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 border border-gray-200">
      <h3 className="text-lg font-semibold text-gray-900 mb-4">{title}</h3>
      <div className="text-gray-700">{children}</div>
    </div>
  );
};
```

#### 2. レスポンシブデザイン
```tsx
// ✅ 推奨：レスポンシブプレフィックスを使用
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* コンテンツ */}
</div>
```

#### 3. カスタムスタイル
```tsx
// ✅ 推奨：複雑なスタイルのためにコンポーネントを作成するか @apply を使用
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

### Tailwind 設定

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

## Vite 設定と最適化

### 開発設定

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

### ビルド最適化

#### 1. コード分割
```tsx
// ✅ 推奨：ルートレベルのコード分割
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

#### 2. リソース最適化
- 画像圧縮と最適化
- CSS と JS コード圧縮
- 未使用コードを削除する Tree shaking

## 状態管理

### 軽量アプリケーション向け
中小規模アプリケーションでは以下を使用することを推奨：
- React useState/useReducer (コンポーネントレベル)
- Context API (グローバル状態)
- React Query (サーバー状態)

### 中大型アプリケーション向け
以下の導入を検討：
- Zustand
- Redux Toolkit
- Jotai

## テスト規範

### 単体テスト
```tsx
// ✅ 推奨：Testing Library を使用
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

## パフォーマンス最適化

### 1. React 最適化
- 不要な再レンダリングを避けるため React.memo を使用
- 計算を最適化するため useCallback と useMemo を使用
- key 属性を適切に使用

### 2. ビルド最適化
- コード分割と遅延読み込み
- Bundle 分析と最適化
- CDN リソース読み込み

### 3. 画像最適化
- 適切な画像形式を使用
- レスポンシブ画像
- 重要な画像以外は遅延読み込み

## コード規範ツール

### ESLint 設定
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
    // カスタム規則
  },
};
```

### Prettier 設定
```js
// .prettierrc.js
module.exports = {
  semi: false,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 100,
  tabWidth: 2,
};
