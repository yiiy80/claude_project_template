# フロントエンド・バックエンドインターフェース約定 (API Contract)

## 概要

この文書はフロントエンドとバックエンド間のAPIインターフェース約定を定義し、前後端開発の一貫性と保守性を確保します。すべてのAPIインターフェースはRESTful設計原則に従い、JSONをデータ交換形式として使用します。

## 基本約定

### 1. HTTP メソッド

| メソッド | 用途 | 例 |
|------|------|------|
| GET | データのクエリ | `GET /users` - ユーザーリストを取得 |
| POST | リソースの作成 | `POST /users` - ユーザーを作成 |
| PUT | リソースの更新 (完全置換) | `PUT /users/123` - ユーザー完全情報を更新 |
| PATCH | リソースの更新 (部分更新) | `PATCH /users/123` - ユーザー部分情報を更新 |
| DELETE | リソースの削除 | `DELETE /users/123` - ユーザーを削除 |

### 2. ステータスコード規範

#### 成功レスポンス (2xx)
- `200 OK`: リクエスト成功
- `201 Created`: リソース作成成功
- `204 No Content`: リクエスト成功、ただし返却内容なし

#### クライアントエラー (4xx)
- `400 Bad Request`: リクエストパラメータエラー
- `401 Unauthorized`: 未認証
- `403 Forbidden`: 権限不足
- `404 Not Found`: リソースが存在しない
- `409 Conflict`: リソース競合 (重複作成など)
- `422 Unprocessable Entity`: 検証エラー

#### サーバーエラー (5xx)
- `500 Internal Server Error`: サーバー内部エラー

### 3. レスポンス形式

#### 標準レスポンス構造

```json
{
  "success": true,
  "message": "操作成功",
  "data": {
    // 実際のデータ
  },
  "meta": {
    // ページングなどのメタデータ
  }
}
```

#### エラーレスポンス構造

```json
{
  "success": false,
  "message": "エラーメッセージ",
  "error_code": "ERROR_CODE",
  "details": {
    // 詳細エラー情報
  }
}
```

#### ページングレスポンス構造

```json
{
  "success": true,
  "data": [...],
  "meta": {
    "page": 1,
    "size": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_prev": false
  }
}
```

## 認証と認可

### JWT 認証

#### リクエストヘッダー
```
Authorization: Bearer <access_token>
```

#### Token 更新
```json
POST /auth/refresh
{
  "refresh_token": "refresh_token_string"
}

Response:
{
  "success": true,
  "data": {
    "access_token": "new_access_token",
    "token_type": "bearer",
    "expires_in": 1800
  }
}
```

### 権限レベル

#### ユーザー権限
- `read`: 読み取り権限
- `write`: 書き込み権限
- `admin`: 管理者権限

#### リソース権限チェック
バックエンドは各保護されたエンドポイントでユーザーの権限をチェックし、権限不足時は `403 Forbidden` を返却。

## データ形式約定

### 1. 時間形式
ISO 8601形式を使用：
```json
{
  "created_at": "2024-01-01T12:00:00Z",
  "updated_at": "2024-01-01T12:30:00Z"
}
```

### 2. ブール値
小文字の `true`/`false` を使用：
```json
{
  "is_active": true,
  "is_deleted": false
}
```

### 3. 空値処理
- データベースの `NULL` はJSONで `null` として表現
- オプションフィールドは設定されていないことを表すために `null` を使用
- 空文字列 `""` と `null` は意味的に異なる

### 4. 列挙値
文字列列挙を使用：
```json
{
  "status": "active",  // 数値やブール値ではなく
  "role": "admin"
}
```

## API エンドポイント設計

### 1. リソース命名

#### 基礎リソース
- 複数名詞を使用: `/users`, `/posts`, `/comments`
- 小文字とハイフンを使用: `/user-profiles`, `/blog-posts`

#### ネストリソース
```http
GET /users/{user_id}/posts        # ユーザーの記事
GET /posts/{post_id}/comments     # 記事のコメント
GET /users/{user_id}/followers    # ユーザーのフォロワー
```

#### アクションリソース
```http
POST /users/{user_id}/follow      # ユーザーをフォロー
POST /posts/{post_id}/like        # 記事にいいね
POST /posts/{post_id}/publish     # 記事を公開
```

### 2. クエリパラメータ

#### ページングパラメータ
```
GET /users?page=1&size=20&sort=created_at&order=desc
```

#### フィルタリングパラメータ
```
GET /users?status=active&role=admin&created_after=2024-01-01
```

#### 検索パラメータ
```
GET /users?search=john&fields=username,email,full_name
```

### 3. リクエストボディ設計

#### リソース作成
```json
POST /users
{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "secure_password",
  "full_name": "John Doe"
}
```

#### リソース更新 (PUT - 完全置換)
```json
PUT /users/123
{
  "username": "johndoe",
  "email": "john@example.com",
  "full_name": "John Doe Updated",
  "is_active": true
}
```

#### リソース更新 (PATCH - 部分更新)
```json
PATCH /users/123
{
  "full_name": "John Doe Updated"
}
```

## 検証規則

### 1. 入力検証

#### 必須フィールド検証
```json
// エラーレスポンス例
{
  "success": false,
  "message": "検証失敗",
  "error_code": "VALIDATION_ERROR",
  "details": {
    "username": ["必須フィールド"],
    "email": ["必須フィールド", "メール形式が正しくありません"]
  }
}
```

#### データ型検証
- 文字列: 長さ制限、形式検証
- 数字: 範囲制限
- 日付: 形式検証
- 配列: 長さ制限、要素型検証

### 2. ビジネス規則検証

#### 一意性チェック
```json
// ユーザー名が既に存在
{
  "success": false,
  "message": "ユーザー名が既に存在します",
  "error_code": "USERNAME_EXISTS"
}
```

#### 参照整合性
```json
// 関連リソースが存在しない
{
  "success": false,
  "message": "指定されたユーザーが存在しません",
  "error_code": "USER_NOT_FOUND"
}
```

## キャッシュ戦略

### HTTP キャッシュヘッダー

#### 静的リソースキャッシュ
```
Cache-Control: public, max-age=31536000
ETag: "resource-etag"
Last-Modified: Mon, 01 Jan 2024 00:00:00 GMT
```

#### API レスポンスキャッシュ
```
Cache-Control: private, max-age=300
ETag: "api-response-etag"
```

### 条件リクエスト
```http
GET /users/123
If-None-Match: "etag-value"

Response: 304 Not Modified
```

## エラー処理

### 1. 統一エラー形式

```json
{
  "success": false,
  "message": "ユーザーフレンドリーなエラーメッセージ",
  "error_code": "SPECIFIC_ERROR_CODE",
  "details": {
    "field_name": ["具体的なフィールドエラー情報"],
    "another_field": ["別のフィールドのエラー"]
  },
  "trace_id": "一意のリクエスト追跡ID"
}
```

### 2. エラーコード規範

#### 認証エラー
- `INVALID_CREDENTIALS`: ユーザー名またはパスワードが正しくない
- `TOKEN_EXPIRED`: トークンが期限切れ
- `TOKEN_INVALID`: トークンが無効
- `INSUFFICIENT_PERMISSIONS`: 権限が不足

#### 検証エラー
- `VALIDATION_ERROR`: 汎用検証エラー
- `REQUIRED_FIELD_MISSING`: 必須フィールドが欠落
- `INVALID_FORMAT`: 形式エラー
- `VALUE_OUT_OF_RANGE`: 値が範囲外

#### ビジネスエラー
- `RESOURCE_NOT_FOUND`: リソースが存在しない
- `RESOURCE_ALREADY_EXISTS`: リソースが既に存在
- `OPERATION_NOT_ALLOWED`: 操作が許可されていない
- `QUOTA_EXCEEDED`: クォータ超過

### 3. エラーログ記録

バックエンドは以下の詳細エラー情報を記録：
- リクエストID
- ユーザーID (認証済みの場合)
- リクエストパラメータ
- エラースタック
- タイムスタンプ

## API バージョン管理

### 1. URL パスバージョン管理

```
GET /api/v1/users
GET /api/v2/users
```

### 2. Accept Header バージョン管理

```
Accept: application/vnd.api.v1+json
Accept: application/vnd.api.v2+json
```

### 3. バージョン互換性

- 新しいバージョンのAPIは後方互換性を保つ
- 非推奨のAPIはレスポンスヘッダーでマーク
- 移行ガイドを提供

## パフォーマンス最適化

### 1. ページング

#### カーソルページング (大規模データセットに推奨)
```json
GET /posts?cursor=eyJpZCI6MTIzfQ==&limit=20

Response:
{
  "data": [...],
  "meta": {
    "next_cursor": "eyJpZCI6MTQzfQ==",
    "has_next": true
  }
}
```

#### オフセットページング (小規模データセットに適する)
```json
GET /posts?page=1&size=20

Response:
{
  "data": [...],
  "meta": {
    "page": 1,
    "size": 20,
    "total": 150,
    "total_pages": 8
  }
}
```

### 2. フィールド選択

```json
GET /users?fields=id,username,email,created_at
GET /users/123?fields=username,email
```

### 3. 関連データ包含

```json
GET /posts?include=author,comments
GET /posts/123?include=author,tags
```

## セキュリティ

### 1. 入力クリーンアップ

- XSS攻撃防止: ユーザー入力にHTMLエンコードを実施
- SQLインジェクション防止: パラメータ化クエリを使用
- ファイルアップロード: ファイルタイプ、大小制限を検証

### 2. レート制限

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
X-RateLimit-Retry-After: 60
```

### 3. CORS 設定

```json
{
  "origins": ["https://yourdomain.com"],
  "methods": ["GET", "POST", "PUT", "DELETE"],
  "headers": ["Authorization", "Content-Type"],
  "credentials": true
}
```

## テスト約定

### 1. API テスト

#### 単体テスト
- 個別APIエンドポイントの機能をテスト
- モックデータと依存性注入を使用
- レスポンス形式とステータスコードを検証

#### 統合テスト
- API間の相互作用をテスト
- データ一貫性を検証
- エラー処理フローをテスト

### 2. 契約テスト

#### Consumer-Driven Contracts
```javascript
// フロントエンド契約テスト
const userContract = {
  request: {
    method: 'GET',
    path: '/users/{id}',
    headers: {
      'Authorization': 'Bearer {token}'
    }
  },
  response: {
    status: 200,
    body: {
      id: '{id}',
      username: 'string',
      email: 'email',
      created_at: 'date'
    }
  }
};
```

## ドキュメントと保守

### 1. API ドキュメント

OpenAPI/Swagger を使用して自動生成APIドキュメント：
- 各エンドポイントに完全な説明を記載
- リクエスト/レスポンス例
- パラメータ検証規則
- エラーレスポンス例

### 2. 変更管理

#### API 変更フロー
1. 影響範囲を評価
2. すべてのクライアントに通知
3. 移行ガイドを提供
4. 新しいバージョンを実装
5. 古いバージョンを段階的に廃止

#### 後方互換性保証
- 新しいフィールドはデフォルトでオプション
- 列挙値は増やすのみ
- レスポンス形式を一貫して保持

### 3. 監視とアラート

#### パフォーマンス監視
- レスポンスタイム
- エラー率
- スループット

#### ビジネス監視
- API 呼び出し頻度
- ユーザーアクティブ度
- 機能使用状況

## 一般的なAPIパターン

### 1. CRUD 操作

```http
# 作成
POST   /resources
# 読み取り
GET    /resources
GET    /resources/{id}
# 更新
PUT    /resources/{id}
PATCH  /resources/{id}
# 削除
DELETE /resources/{id}
```

### 2. バッチ操作

```http
# バッチ作成
POST   /resources/batch
# バッチ更新
PUT    /resources/batch
# バッチ削除
DELETE /resources/batch
```

### 3. 検索とフィルタリング

```http
# シンプル検索
GET /resources?search=keyword
# 高度検索
GET /resources?filters[name]=john&filters[age]=25&sort=name&order=asc
```

### 4. ファイルアップロード

```http
# 単一ファイルアップロード
POST /upload
Content-Type: multipart/form-data

# 複数ファイルアップロード
POST /uploads
Content-Type: multipart/form-data
```

## まとめ

これらの約定に従うことで：
- 前後端開発の一貫性
- APIの保守性と拡張性
- 優れた開発体験
- システムの安定性とセキュリティ

すべてのAPI変更はレビューを経て、関連ドキュメントとテストを更新する必要があります。
