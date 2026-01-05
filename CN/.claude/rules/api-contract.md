# 前后端接口约定 (API Contract)

## 概述

本文档定义了前端和后端之间的API接口约定，确保前后端开发的一致性和可维护性。所有API接口必须遵循RESTful设计原则，并使用JSON作为数据交换格式。

## 基本约定

### 1. HTTP 方法

| 方法 | 用途 | 示例 |
|------|------|------|
| GET | 查询数据 | `GET /users` - 获取用户列表 |
| POST | 创建资源 | `POST /users` - 创建用户 |
| PUT | 更新资源 (完整替换) | `PUT /users/123` - 更新用户完整信息 |
| PATCH | 更新资源 (部分更新) | `PATCH /users/123` - 更新用户部分信息 |
| DELETE | 删除资源 | `DELETE /users/123` - 删除用户 |

### 2. 状态码规范

#### 成功响应 (2xx)
- `200 OK`: 请求成功
- `201 Created`: 资源创建成功
- `204 No Content`: 请求成功，但无返回内容

#### 客户端错误 (4xx)
- `400 Bad Request`: 请求参数错误
- `401 Unauthorized`: 未认证
- `403 Forbidden`: 权限不足
- `404 Not Found`: 资源不存在
- `409 Conflict`: 资源冲突 (如重复创建)
- `422 Unprocessable Entity`: 验证错误

#### 服务器错误 (5xx)
- `500 Internal Server Error`: 服务器内部错误

### 3. 响应格式

#### 标准响应结构

```json
{
  "success": true,
  "message": "操作成功",
  "data": {
    // 实际数据
  },
  "meta": {
    // 分页等元数据
  }
}
```

#### 错误响应结构

```json
{
  "success": false,
  "message": "错误信息",
  "error_code": "ERROR_CODE",
  "details": {
    // 详细错误信息
  }
}
```

#### 分页响应结构

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

## 认证和授权

### JWT 认证

#### 请求头
```
Authorization: Bearer <access_token>
```

#### Token 刷新
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

### 权限级别

#### 用户权限
- `read`: 读取权限
- `write`: 写入权限
- `admin`: 管理员权限

#### 资源权限检查
后端需要在每个受保护的端点检查用户权限，权限不足时返回 `403 Forbidden`。

## 数据格式约定

### 1. 时间格式
使用ISO 8601格式：
```json
{
  "created_at": "2024-01-01T12:00:00Z",
  "updated_at": "2024-01-01T12:30:00Z"
}
```

### 2. 布尔值
使用小写的 `true`/`false`：
```json
{
  "is_active": true,
  "is_deleted": false
}
```

### 3. 空值处理
- 数据库中的 `NULL` 在JSON中表示为 `null`
- 可选字段使用 `null` 表示未设置
- 空字符串 `""` 和 `null` 语义不同

### 4. 枚举值
使用字符串枚举：
```json
{
  "status": "active",  // 而不是数字或布尔值
  "role": "admin"
}
```

## API 端点设计

### 1. 资源命名

#### 基础资源
- 使用复数名词: `/users`, `/posts`, `/comments`
- 使用小写字母和连字符: `/user-profiles`, `/blog-posts`

#### 嵌套资源
```http
GET /users/{user_id}/posts        # 用户的文章
GET /posts/{post_id}/comments     # 文章的评论
GET /users/{user_id}/followers    # 用户的关注者
```

#### 动作资源
```http
POST /users/{user_id}/follow      # 关注用户
POST /posts/{post_id}/like        # 点赞文章
POST /posts/{post_id}/publish     # 发布文章
```

### 2. 查询参数

#### 分页参数
```
GET /users?page=1&size=20&sort=created_at&order=desc
```

#### 过滤参数
```
GET /users?status=active&role=admin&created_after=2024-01-01
```

#### 搜索参数
```
GET /users?search=john&fields=username,email,full_name
```

### 3. 请求体设计

#### 创建资源
```json
POST /users
{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "secure_password",
  "full_name": "John Doe"
}
```

#### 更新资源 (PUT - 完整替换)
```json
PUT /users/123
{
  "username": "johndoe",
  "email": "john@example.com",
  "full_name": "John Doe Updated",
  "is_active": true
}
```

#### 更新资源 (PATCH - 部分更新)
```json
PATCH /users/123
{
  "full_name": "John Doe Updated"
}
```

## 验证规则

### 1. 输入验证

#### 必填字段验证
```json
// 错误响应示例
{
  "success": false,
  "message": "验证失败",
  "error_code": "VALIDATION_ERROR",
  "details": {
    "username": ["必填字段"],
    "email": ["必填字段", "邮箱格式不正确"]
  }
}
```

#### 数据类型验证
- 字符串: 长度限制、格式验证
- 数字: 范围限制
- 日期: 格式验证
- 数组: 长度限制、元素类型验证

### 2. 业务规则验证

#### 唯一性检查
```json
// 用户名已存在
{
  "success": false,
  "message": "用户名已存在",
  "error_code": "USERNAME_EXISTS"
}
```

#### 引用完整性
```json
// 关联资源不存在
{
  "success": false,
  "message": "指定的用户不存在",
  "error_code": "USER_NOT_FOUND"
}
```

## 缓存策略

### HTTP 缓存头

#### 静态资源缓存
```
Cache-Control: public, max-age=31536000
ETag: "resource-etag"
Last-Modified: Mon, 01 Jan 2024 00:00:00 GMT
```

#### API 响应缓存
```
Cache-Control: private, max-age=300
ETag: "api-response-etag"
```

### 条件请求
```http
GET /users/123
If-None-Match: "etag-value"

Response: 304 Not Modified
```

## 错误处理

### 1. 统一错误格式

```json
{
  "success": false,
  "message": "用户友好的错误信息",
  "error_code": "SPECIFIC_ERROR_CODE",
  "details": {
    "field_name": ["具体的字段错误信息"],
    "another_field": ["另一个字段的错误"]
  },
  "trace_id": "唯一请求追踪ID"
}
```

### 2. 错误代码规范

#### 认证错误
- `INVALID_CREDENTIALS`: 用户名或密码错误
- `TOKEN_EXPIRED`: 令牌过期
- `TOKEN_INVALID`: 令牌无效
- `INSUFFICIENT_PERMISSIONS`: 权限不足

#### 验证错误
- `VALIDATION_ERROR`: 通用验证错误
- `REQUIRED_FIELD_MISSING`: 必填字段缺失
- `INVALID_FORMAT`: 格式错误
- `VALUE_OUT_OF_RANGE`: 值超出范围

#### 业务错误
- `RESOURCE_NOT_FOUND`: 资源不存在
- `RESOURCE_ALREADY_EXISTS`: 资源已存在
- `OPERATION_NOT_ALLOWED`: 操作不允许
- `QUOTA_EXCEEDED`: 配额超出

### 3. 错误日志记录

后端需要记录详细的错误信息，包括：
- 请求ID
- 用户ID (如果已认证)
- 请求参数
- 错误堆栈
- 时间戳

## API 版本控制

### 1. URL 路径版本控制

```
GET /api/v1/users
GET /api/v2/users
```

### 2. Accept Header 版本控制

```
Accept: application/vnd.api.v1+json
Accept: application/vnd.api.v2+json
```

### 3. 版本兼容性

- 新版本API必须向后兼容
- 废弃的API需要在响应头中标记
- 提供迁移指南

## 性能优化

### 1. 分页

#### 游标分页 (推荐用于大数据集)
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

#### 偏移分页 (适用于小数据集)
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

### 2. 字段选择

```json
GET /users?fields=id,username,email,created_at
GET /users/123?fields=username,email
```

### 3. 关联数据包含

```json
GET /posts?include=author,comments
GET /posts/123?include=author,tags
```

## 安全性

### 1. 输入清理

- 防止XSS攻击: 对用户输入进行HTML编码
- 防止SQL注入: 使用参数化查询
- 文件上传: 验证文件类型、大小限制

### 2. 速率限制

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
X-RateLimit-Retry-After: 60
```

### 3. CORS 配置

```json
{
  "origins": ["https://yourdomain.com"],
  "methods": ["GET", "POST", "PUT", "DELETE"],
  "headers": ["Authorization", "Content-Type"],
  "credentials": true
}
```

## 测试约定

### 1. API 测试

#### 单元测试
- 测试单个API端点的功能
- 使用模拟数据和依赖注入
- 验证响应格式和状态码

#### 集成测试
- 测试API之间的交互
- 验证数据一致性
- 测试错误处理流程

### 2. 契约测试

#### Consumer-Driven Contracts
```javascript
// 前端契约测试
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

## 文档和维护

### 1. API 文档

使用 OpenAPI/Swagger 自动生成API文档：
- 每个端点必须有完整的描述
- 请求/响应示例
- 参数验证规则
- 错误响应示例

### 2. 变更管理

#### API 变更流程
1. 评估影响范围
2. 通知所有客户端
3. 提供迁移指南
4. 实现新版本
5. 逐步弃用旧版本

#### 向后兼容性保证
- 新增字段默认为可选
- 枚举值只增不减
- 响应格式保持一致

### 3. 监控和告警

#### 性能监控
- 响应时间
- 错误率
- 吞吐量

#### 业务监控
- API 调用频率
- 用户活跃度
- 功能使用情况

## 常用API模式

### 1. CRUD 操作

```http
# 创建
POST   /resources
# 读取
GET    /resources
GET    /resources/{id}
# 更新
PUT    /resources/{id}
PATCH  /resources/{id}
# 删除
DELETE /resources/{id}
```

### 2. 批量操作

```http
# 批量创建
POST   /resources/batch
# 批量更新
PUT    /resources/batch
# 批量删除
DELETE /resources/batch
```

### 3. 搜索和过滤

```http
# 简单搜索
GET /resources?search=keyword
# 高级搜索
GET /resources?filters[name]=john&filters[age]=25&sort=name&order=asc
```

### 4. 文件上传

```http
# 单文件上传
POST /upload
Content-Type: multipart/form-data

# 多文件上传
POST /uploads
Content-Type: multipart/form-data
```

## 总结

遵循这些约定可以确保：
- 前后端开发的一致性
- API的可维护性和扩展性
- 良好的开发体验
- 系统的稳定性和安全性

所有API变更都需要经过review，并更新相应的文档和测试。
