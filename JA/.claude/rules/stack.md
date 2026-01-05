# 技術スタック総綱

## 概要

本プロジェクトは、高効率の開発体験、優れた型安全性、そして優れたパフォーマンスを提供することを目的とした現代的な全栈技術スタックを採用しています。

## 核心技術スタック

### フロントエンド技術スタック
- **React 18+**: 最新の並行処理機能をサポートするユーザーインターフェースライブラリ
- **Vite**: 優れた開発体験を提供する高速ビルドツール
- **TypeScript**: 優れた開発体験を提供する型安全性
- **Tailwind CSS**: 迅速なUI開発を可能にする実用優先のCSSフレームワーク

### バックエンド技術スタック
- **FastAPI**: StarletteとPydanticに基づく現代的なPython Webフレームワーク
- **Python 3.9+**: 核心プログラミング言語
- **Uvicorn**: 生産環境デプロイ用のASGIサーバー

### データベース技術スタック
- **SQLite**: 中小規模アプリケーションに適した軽量リレーショナルデータベース
- **SQLModel**: SQLAlchemyとPydanticに基づき、型ヒントをサポートするORM

## アーキテクチャ設計

### フロントエンド・バックエンド分離
```
┌─────────────────┐    HTTP/REST    ┌─────────────────┐
│   Frontend      │◄──────────────►│   Backend       │
│   (React)       │                │   (FastAPI)     │
└─────────────────┘                └─────────────────┘
         │                                 │
         └──────────────► SQLite ◄─────────┘
```

### ディレクトリ構造
```
project/
├── frontend/           # Reactフロントエンドアプリケーション
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.ts
├── backend/            # FastAPIバックエンドアプリケーション
│   ├── app/
│   │   ├── main.py
│   │   ├── models.py
│   │   └── routers/
│   ├── requirements.txt
│   └── alembic/        # データベースマイグレーション
└── database/           # データベースファイル
    └── app.db
```

## 開発環境要件

### システム要件
- Node.js 18+
- Python 3.9+
- Git

### 推奨ツール
- VS Code (推奨プラグイン: Python, TypeScript, Tailwind CSS)
- Git Bash / Terminal

## パッケージ管理

### フロントエンド依存関係
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "typescript": "^5.0.0",
    "tailwindcss": "^3.3.0",
    "vite": "^4.3.0"
  }
}
```

### バックエンド依存関係
```
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlmodel==0.0.14
alembic==1.12.1
pydantic==2.5.0
python-multipart==0.0.6
```

## 開発ワークフロー

### ローカル開発
1. バックエンドサービス起動: `uvicorn app.main:app --reload`
2. フロントエンドサービス起動: `npm run dev`
3. フロントエンドアプリケーションにアクセスし、APIリクエストをバックエンドにプロキシ

### 生産デプロイ
1. フロントエンドビルド: `npm run build`
2. 静的ファイルをWebサーバーにデプロイ
3. FastAPIアプリケーションをアプリケーションサーバーにデプロイ

## 約定と規範

### コード規範
- ESLint + Prettier (フロントエンド)
- Black + isort (バックエンド)
- 各言語のベストプラクティスに従う

### バージョン管理
- Gitを使用したバージョン管理
- セマンティックバージョニング (SemVer) に従う
- Conventional Commits規範を使用

### テスト戦略
- 単体テスト: Jest (フロントエンド), pytest (バックエンド)
- 統合テスト: APIエンドポイントテスト
- E2Eテスト: Playwright または Cypress

## パフォーマンス最適化

### フロントエンド最適化
- コード分割と遅延読み込み
- 画像最適化とリソース圧縮
- React.memoとuseMemoを使用したレンダリング最適化

### バックエンド最適化
- データベースクエリ最適化
- キャッシュ戦略
- 非同期処理

## セキュリティ考慮事項

- 入力検証とクリーンアップ
- CORS設定
- 機密情報保護
- HTTPS強制使用

## 監視とログ

- 構造化ログ記録
- エラーモニタリングとアラート
- パフォーマンス指標収集
- ユーザー行動分析
