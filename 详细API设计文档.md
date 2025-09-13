# 企业知识库客服助理 - 详细API设计文档

## 目录
1. [API概述](#api概述)
2. [认证与权限](#认证与权限)
3. [用户管理API](#用户管理api)
4. [租户管理API](#租户管理api)
5. [文档管理API](#文档管理api)
6. [对话问答API](#对话问答api)
7. [工具集成API](#工具集成api)
8. [计费与统计API](#计费与统计api)
9. [系统配置API](#系统配置api)
10. [WebSocket实时通信](#websocket实时通信)

## API概述

### 基础信息
- **Base URL**: `https://api.enterprise-kb.com/v1`
- **协议**: HTTPS
- **数据格式**: JSON
- **字符编码**: UTF-8

### 统一响应格式
```json
{
  "ok": true,
  "code": 0,
  "message": "success",
  "data": {
    // 具体数据
  },
  "timestamp": "2024-01-01T12:00:00Z",
  "requestId": "uuid-string"
}
```

### 分页格式
```json
{
  "items": [],
  "pagination": {
    "page": 1,
    "size": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

## 认证与权限

### JWT令牌格式
```json
{
  "sub": "user-id",
  "tenant": "tenant-id",
  "roles": ["admin", "user"],
  "permissions": ["read:docs", "write:docs"],
  "exp": 1640995200,
  "iat": 1640908800
}
```

### 权限级别
- `super_admin`: 超级管理员
- `tenant_admin`: 租户管理员
- `knowledge_manager`: 知识管理员
- `agent`: 客服座席
- `user`: 普通用户

## 用户管理API

### 1. 用户注册
```http
POST /auth/register
Content-Type: application/json

{
  "email": "user@company.com",
  "password": "securePassword123",
  "firstName": "张",
  "lastName": "三",
  "tenantDomain": "company.com"
}
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "userId": "user-123",
    "email": "user@company.com",
    "status": "pending_verification"
  }
}
```

### 2. 用户登录
```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@company.com",
  "password": "securePassword123"
}
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "accessToken": "jwt-token",
    "refreshToken": "refresh-token",
    "expiresIn": 3600,
    "user": {
      "id": "user-123",
      "email": "user@company.com",
      "name": "张三",
      "roles": ["user"],
      "tenant": {
        "id": "tenant-123",
        "name": "ABC公司",
        "plan": "professional"
      }
    }
  }
}
```

### 3. SSO登录
```http
POST /auth/sso/saml
Content-Type: application/json

{
  "samlResponse": "base64-encoded-response",
  "relayState": "optional-state"
}
```

### 4. 获取用户信息
```http
GET /users/me
Authorization: Bearer <token>
```

### 5. 更新用户信息
```http
PUT /users/me
Authorization: Bearer <token>
Content-Type: application/json

{
  "firstName": "张",
  "lastName": "三丰",
  "avatar": "https://avatar-url.com/123.jpg",
  "preferences": {
    "language": "zh-CN",
    "timezone": "Asia/Shanghai",
    "notifications": {
      "email": true,
      "browser": false
    }
  }
}
```

## 租户管理API

### 1. 创建租户
```http
POST /tenants
Authorization: Bearer <super-admin-token>
Content-Type: application/json

{
  "name": "ABC公司",
  "domain": "abc.com",
  "plan": "professional",
  "settings": {
    "maxUsers": 100,
    "maxStorage": "10GB",
    "features": ["sso", "api_access", "advanced_analytics"]
  },
  "adminUser": {
    "email": "admin@abc.com",
    "firstName": "管理",
    "lastName": "员"
  }
}
```

### 2. 租户配置更新
```http
PUT /tenants/{tenantId}/settings
Authorization: Bearer <tenant-admin-token>
Content-Type: application/json

{
  "branding": {
    "logo": "https://logo-url.com/abc.png",
    "primaryColor": "#1890ff",
    "companyName": "ABC公司"
  },
  "sso": {
    "enabled": true,
    "provider": "saml",
    "metadata": "saml-metadata-xml"
  },
  "integrations": {
    "jira": {
      "enabled": true,
      "baseUrl": "https://abc.atlassian.net",
      "credentials": "encrypted-credentials"
    },
    "email": {
      "enabled": true,
      "smtpServer": "smtp.abc.com",
      "credentials": "encrypted-credentials"
    }
  }
}
```

## 文档管理API

### 1. 上传文档
```http
POST /docs/upload
Authorization: Bearer <token>
Content-Type: multipart/form-data

file: [binary-file-data]
metadata: {
  "title": "产品操作手册",
  "category": "manual",
  "tags": ["产品", "操作"],
  "language": "zh-CN",
  "department": "技术支持",
  "accessLevel": "internal"
}
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "docId": "doc-123",
    "fileName": "manual.pdf",
    "fileSize": 2048576,
    "status": "processing",
    "uploadedAt": "2024-01-01T12:00:00Z"
  }
}
```

### 2. 批量上传文档
```http
POST /docs/batch-upload
Authorization: Bearer <token>
Content-Type: application/json

{
  "files": [
    {
      "url": "https://storage.com/doc1.pdf",
      "metadata": {
        "title": "文档1",
        "category": "manual"
      }
    },
    {
      "url": "https://storage.com/doc2.docx",
      "metadata": {
        "title": "文档2",
        "category": "policy"
      }
    }
  ]
}
```

### 3. 获取文档列表
```http
GET /docs?page=1&size=20&category=manual&status=indexed&search=操作手册
Authorization: Bearer <token>
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "items": [
      {
        "id": "doc-123",
        "title": "产品操作手册",
        "fileName": "manual.pdf",
        "category": "manual",
        "tags": ["产品", "操作"],
        "status": "indexed",
        "chunksCount": 150,
        "fileSize": 2048576,
        "uploadedAt": "2024-01-01T12:00:00Z",
        "indexedAt": "2024-01-01T12:05:00Z",
        "metadata": {
          "language": "zh-CN",
          "department": "技术支持"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "size": 20,
      "total": 50,
      "totalPages": 3
    }
  }
}
```

### 4. 获取文档详情
```http
GET /docs/{docId}
Authorization: Bearer <token>
```

### 5. 更新文档元数据
```http
PUT /docs/{docId}
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "产品操作手册(更新版)",
  "category": "manual",
  "tags": ["产品", "操作", "2024"],
  "metadata": {
    "version": "2.0",
    "lastReviewed": "2024-01-01"
  }
}
```

### 6. 删除文档
```http
DELETE /docs/{docId}
Authorization: Bearer <token>
```

### 7. 获取文档处理状态
```http
GET /docs/{docId}/status
Authorization: Bearer <token>
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "status": "indexed",
    "progress": 100,
    "stages": [
      {
        "name": "upload",
        "status": "completed",
        "completedAt": "2024-01-01T12:00:00Z"
      },
      {
        "name": "text_extraction",
        "status": "completed",
        "completedAt": "2024-01-01T12:02:00Z"
      },
      {
        "name": "chunking",
        "status": "completed",
        "completedAt": "2024-01-01T12:03:00Z"
      },
      {
        "name": "embedding",
        "status": "completed",
        "completedAt": "2024-01-01T12:05:00Z"
      }
    ],
    "chunks": {
      "total": 150,
      "indexed": 150
    },
    "errors": []
  }
}
```

## 对话问答API

### 1. 创建对话
```http
POST /conversations
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "产品使用咨询",
  "context": {
    "department": "technical_support",
    "priority": "normal",
    "customer": {
      "id": "customer-123",
      "name": "李四",
      "email": "lisi@customer.com"
    }
  }
}
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "conversationId": "conv-123",
    "title": "产品使用咨询",
    "createdAt": "2024-01-01T12:00:00Z",
    "status": "active"
  }
}
```

### 2. 发送消息/查询
```http
POST /conversations/{conversationId}/messages
Authorization: Bearer <token>
Content-Type: application/json

{
  "content": "如何重置产品密码？",
  "type": "user",
  "context": {
    "attachments": [],
    "urgent": false
  },
  "options": {
    "enableFunctionCalls": true,
    "maxTokens": 2000,
    "temperature": 0.3,
    "includeSourcesInResponse": true
  }
}
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "messageId": "msg-123",
    "content": "根据产品操作手册，重置密码的步骤如下：\n\n1. 登录管理后台\n2. 进入用户管理页面\n3. 选择需要重置密码的用户\n4. 点击"重置密码"按钮\n\n我已经为您创建了一个技术支持工单来跟进此问题。",
    "type": "assistant",
    "sources": [
      {
        "docId": "doc-123",
        "title": "产品操作手册",
        "chunk": "密码重置功能位于用户管理模块...",
        "similarity": 0.89,
        "page": 15
      }
    ],
    "functionCalls": [
      {
        "name": "createJiraIssue",
        "parameters": {
          "projectKey": "SUPPORT",
          "summary": "用户密码重置咨询",
          "description": "客户咨询产品密码重置流程"
        },
        "result": {
          "issueId": "SUPPORT-456",
          "url": "https://jira.company.com/browse/SUPPORT-456"
        }
      }
    ],
    "metadata": {
      "responseTime": 1.2,
      "tokensUsed": 350,
      "confidence": 0.91
    },
    "createdAt": "2024-01-01T12:00:00Z"
  }
}
```

### 3. 获取对话历史
```http
GET /conversations/{conversationId}/messages?page=1&size=20
Authorization: Bearer <token>
```

### 4. 流式对话 (Server-Sent Events)
```http
POST /conversations/{conversationId}/messages/stream
Authorization: Bearer <token>
Content-Type: application/json

{
  "content": "如何重置产品密码？",
  "type": "user"
}
```

**响应流**:
```
data: {"type": "start", "messageId": "msg-123"}

data: {"type": "content", "chunk": "根据产品操作手册，"}

data: {"type": "content", "chunk": "重置密码的步骤如下："}

data: {"type": "sources", "sources": [...]}

data: {"type": "function_call", "name": "createJiraIssue", "parameters": {...}}

data: {"type": "function_result", "result": {...}}

data: {"type": "end", "metadata": {...}}
```

### 5. 获取对话列表
```http
GET /conversations?page=1&size=20&status=active&search=密码
Authorization: Bearer <token>
```

### 6. 更新对话状态
```http
PUT /conversations/{conversationId}
Authorization: Bearer <token>
Content-Type: application/json

{
  "status": "closed",
  "resolution": "问题已解决",
  "tags": ["已解决", "密码重置"]
}
```

## 工具集成API

### 1. 配置Jira集成
```http
POST /integrations/jira/config
Authorization: Bearer <tenant-admin-token>
Content-Type: application/json

{
  "baseUrl": "https://company.atlassian.net",
  "username": "api-user@company.com",
  "apiToken": "api-token",
  "defaultProject": "SUPPORT",
  "issueTypes": ["Bug", "Task", "Support"],
  "customFields": {
    "customerEmail": "customfield_10001",
    "priority": "customfield_10002"
  }
}
```

### 2. 测试集成连接
```http
POST /integrations/{integration}/test
Authorization: Bearer <token>
```

### 3. 执行工具操作
```http
POST /tools/execute
Authorization: Bearer <token>
Content-Type: application/json

{
  "tool": "jira.createIssue",
  "parameters": {
    "projectKey": "SUPPORT",
    "summary": "客户无法登录系统",
    "description": "客户反馈登录时提示用户名密码错误",
    "issueType": "Support",
    "priority": "High",
    "assignee": "support-agent@company.com",
    "customFields": {
      "customerEmail": "customer@email.com"
    }
  },
  "context": {
    "conversationId": "conv-123",
    "userId": "user-123"
  }
}
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "executionId": "exec-123",
    "tool": "jira.createIssue",
    "status": "completed",
    "result": {
      "issueId": "SUPPORT-789",
      "issueKey": "SUPPORT-789",
      "url": "https://company.atlassian.net/browse/SUPPORT-789",
      "status": "Open"
    },
    "executedAt": "2024-01-01T12:00:00Z"
  }
}
```

### 4. 获取工具执行历史
```http
GET /tools/executions?page=1&size=20&tool=jira.createIssue&status=completed
Authorization: Bearer <token>
```

### 5. 获取可用工具列表
```http
GET /tools/available
Authorization: Bearer <token>
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "tools": [
      {
        "name": "jira.createIssue",
        "displayName": "创建Jira工单",
        "description": "在Jira中创建新的工单",
        "category": "ticketing",
        "enabled": true,
        "schema": {
          "type": "object",
          "properties": {
            "projectKey": {"type": "string", "required": true},
            "summary": {"type": "string", "required": true},
            "description": {"type": "string"},
            "issueType": {"type": "string", "enum": ["Bug", "Task", "Support"]},
            "priority": {"type": "string", "enum": ["Low", "Medium", "High", "Critical"]}
          }
        }
      },
      {
        "name": "email.send",
        "displayName": "发送邮件",
        "description": "通过企业邮箱发送邮件",
        "category": "communication",
        "enabled": true,
        "schema": {
          "type": "object",
          "properties": {
            "to": {"type": "array", "items": {"type": "string"}},
            "subject": {"type": "string", "required": true},
            "body": {"type": "string", "required": true},
            "cc": {"type": "array", "items": {"type": "string"}},
            "attachments": {"type": "array", "items": {"type": "string"}}
          }
        }
      }
    ]
  }
}
```

## 计费与统计API

### 1. 获取使用统计
```http
GET /billing/usage?startDate=2024-01-01&endDate=2024-01-31&granularity=daily
Authorization: Bearer <token>
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "period": {
      "start": "2024-01-01",
      "end": "2024-01-31"
    },
    "summary": {
      "apiCalls": 15420,
      "storageUsed": "2.5GB",
      "vectorSearches": 8230,
      "documentProcessed": 145,
      "activeUsers": 28
    },
    "daily": [
      {
        "date": "2024-01-01",
        "apiCalls": 520,
        "vectorSearches": 280,
        "documentProcessed": 5,
        "activeUsers": 12
      }
    ],
    "limits": {
      "apiCalls": 50000,
      "storage": "10GB",
      "users": 100
    },
    "overage": {
      "apiCalls": 0,
      "storage": 0
    }
  }
}
```

### 2. 获取计费详情
```http
GET /billing/invoices?page=1&size=12
Authorization: Bearer <tenant-admin-token>
```

### 3. 更新订阅计划
```http
PUT /billing/subscription
Authorization: Bearer <tenant-admin-token>
Content-Type: application/json

{
  "planId": "pro-100",
  "billingCycle": "monthly",
  "paymentMethod": "credit_card"
}
```

## 系统配置API

### 1. 获取系统配置
```http
GET /settings
Authorization: Bearer <token>
```

### 2. 更新系统配置
```http
PUT /settings
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "general": {
    "siteName": "ABC企业知识库",
    "supportEmail": "support@abc.com",
    "maxFileSize": "50MB",
    "allowedFileTypes": ["pdf", "docx", "txt", "md"]
  },
  "ai": {
    "defaultModel": "gpt-4",
    "maxTokens": 4000,
    "temperature": 0.3,
    "enableFunctionCalls": true
  },
  "security": {
    "passwordMinLength": 8,
    "sessionTimeout": 3600,
    "enableMFA": true,
    "ipWhitelist": ["192.168.1.0/24"]
  }
}
```

### 3. 获取系统健康状态
```http
GET /health
```

**响应**:
```json
{
  "ok": true,
  "code": 0,
  "data": {
    "status": "healthy",
    "timestamp": "2024-01-01T12:00:00Z",
    "services": {
      "database": "healthy",
      "redis": "healthy",
      "vectorDB": "healthy",
      "pythonRAG": "healthy",
      "externalAPIs": {
        "jira": "healthy",
        "email": "healthy"
      }
    },
    "metrics": {
      "responseTime": "150ms",
      "throughput": "500 req/min",
      "errorRate": "0.1%"
    }
  }
}
```

## WebSocket实时通信

### 连接建立
```javascript
const ws = new WebSocket('wss://api.enterprise-kb.com/ws/conversations/{conversationId}');
ws.headers = {
  'Authorization': 'Bearer <token>'
};
```

### 消息格式
```json
{
  "type": "message",
  "event": "new_message",
  "data": {
    "messageId": "msg-123",
    "content": "新消息内容",
    "sender": "assistant",
    "timestamp": "2024-01-01T12:00:00Z"
  }
}
```

### 事件类型
- `new_message`: 新消息
- `typing_start`: 开始输入
- `typing_stop`: 停止输入
- `conversation_updated`: 对话更新
- `function_call_started`: 工具调用开始
- `function_call_completed`: 工具调用完成

## 错误处理

### 错误响应格式
```json
{
  "ok": false,
  "code": 1001,
  "message": "参数验证失败",
  "errors": [
    {
      "field": "email",
      "message": "邮箱格式不正确"
    }
  ],
  "timestamp": "2024-01-01T12:00:00Z",
  "requestId": "uuid-string"
}
```

### 常见错误码
| Code | HTTP Status | 含义 | 处理建议 |
|------|------------|------|----------|
| 0 | 200 | 成功 | - |
| 1001 | 400 | 参数错误 | 检查请求参数 |
| 1101 | 401 | 未授权 | 重新登录 |
| 1102 | 403 | 权限不足 | 联系管理员 |
| 1103 | 429 | 请求过于频繁 | 稍后重试 |
| 2001 | 503 | 向量库不可用 | 稍后重试 |
| 3001 | 500 | LLM调用失败 | 重试或简化问题 |
| 4001 | 502 | Jira API失败 | 检查配置 |
| 5001 | 500 | 数据库错误 | 联系技术支持 |

这个API设计文档提供了完整的企业知识库系统接口规范，支持文档管理、智能问答、工具集成等核心功能，确保系统的可扩展性和易用性。