# WeKnora API 接口文档

## 基础信息

### 服务地址

- **本地开发**: http://localhost:8080
- **Docker 部署**: http://localhost:8080

### API 版本

所有 API 路径前缀：`/api/v1`

### 认证方式

1. **JWT Token**: 在请求头中添加 `Authorization: Bearer <token>`
2. **API Key**: 在请求头中添加 `X-API-Key: <api-key>`

### 响应格式

```json
{
    "code": 0,
    "message": "success",
    "data": {},
    "timestamp": 1620000000
}
```

### 权限说明

| 角色 | 权限级别 | 说明 |
|------|----------|------|
| Viewer | 查看者 | 只读权限，可查看知识库、会话等 |
| Contributor | 贡献者 | 可创建、编辑知识库和知识 |
| Admin | 管理员 | 完整管理权限，可管理模型、存储等 |
| Owner | 所有者 | 租户最高权限，可管理成员、API Key |

## 认证接口

### 1. 用户登录

**POST** `/api/v1/auth/login`

**请求体**:
```json
{
    "email": "user@example.com",
    "password": "password"
}
```

### 2. 用户注册

**POST** `/api/v1/auth/register`

**请求体**:
```json
{
    "email": "user@example.com",
    "password": "password",
    "name": "John Doe"
}
```

### 3. 邀请注册

**POST** `/api/v1/auth/register-by-invite`

**请求体**:
```json
{
    "token": "invite-token",
    "email": "user@example.com",
    "password": "password",
    "name": "John Doe"
}
```

### 4. Token 刷新

**POST** `/api/v1/auth/refresh`

**请求头**: `Authorization: Bearer <refresh_token>`

### 5. 用户登出

**POST** `/api/v1/auth/logout`

### 6. 获取当前用户信息

**GET** `/api/v1/auth/me`

### 7. 更新用户偏好

**PUT** `/api/v1/auth/me/preferences`

### 8. 修改密码

**POST** `/api/v1/auth/change-password`

**请求体**:
```json
{
    "old_password": "old-password",
    "new_password": "new-password"
}
```

### 9. 切换租户

**POST** `/api/v1/auth/switch-tenant`

**请求体**:
```json
{
    "tenant_id": 1
}
```

### 10. OIDC 配置

**GET** `/api/v1/auth/oidc/config`

**GET** `/api/v1/auth/oidc/url`

**GET** `/api/v1/auth/oidc/callback`

## 知识库接口

### 1. 创建知识库

**POST** `/api/v1/knowledge-bases`

**权限**: Contributor+

**请求体**:
```json
{
    "name": "我的知识库",
    "description": "测试知识库",
    "type": "document",
    "chunk_size": 512,
    "chunk_overlap": 80,
    "vector_store_id": "vs-xxx"
}
```

**知识库类型**:
- `document`: 文档知识库
- `faq`: FAQ 知识库
- `wiki`: Wiki 知识库

### 2. 获取知识库列表

**GET** `/api/v1/knowledge-bases`

**权限**: Viewer+

**查询参数**:
- `page`: 页码（默认 1）
- `page_size`: 每页数量（默认 20）
- `type`: 类型筛选

### 3. 获取知识库详情

**GET** `/api/v1/knowledge-bases/:id`

**权限**: Viewer+，且对该知识库有读权限

### 4. 更新知识库

**PUT** `/api/v1/knowledge-bases/:id`

**权限**: KB 创建者或 Admin+

### 5. 删除知识库

**DELETE** `/api/v1/knowledge-bases/:id`

**权限**: KB 创建者或 Admin+

### 6. 置顶知识库

**PUT** `/api/v1/knowledge-bases/:id/pin`

**权限**: Viewer+，且对该知识库有读权限

### 7. 混合搜索

**POST** `/api/v1/knowledge-bases/:id/hybrid-search`

**GET** `/api/v1/knowledge-bases/:id/hybrid-search`

**权限**: Viewer+，且对该知识库有读权限

**请求体**:
```json
{
    "query": "搜索关键词",
    "top_k": 10
}
```

### 8. 拷贝知识库

**POST** `/api/v1/knowledge-bases/copy`

**权限**: Contributor+

### 9. 创建知识库副本

**POST** `/api/v1/knowledge-bases/:id/duplicate`

**权限**: Contributor+，且对源知识库有读权限

### 10. 获取复制进度

**GET** `/api/v1/knowledge-bases/copy/progress/:task_id`

**权限**: Viewer+

### 11. 获取可移动目标知识库列表

**GET** `/api/v1/knowledge-bases/:id/move-targets`

**权限**: Viewer+，且对该知识库有读权限

## 知识接口

### 1. 上传文件创建知识

**POST** `/api/v1/knowledge-bases/:id/knowledge/file`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

**请求体** (multipart/form-data):
- `file`: 文件
- `process_config`: 处理配置（JSON）

### 2. 从 URL 创建知识

**POST** `/api/v1/knowledge-bases/:id/knowledge/url`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

**请求体**:
```json
{
    "url": "https://example.com/doc.html",
    "process_config": {}
}
```

### 3. 创建手动知识

**POST** `/api/v1/knowledge-bases/:id/knowledge/manual`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 4. 获取知识列表

**GET** `/api/v1/knowledge-bases/:id/knowledge`

**权限**: Viewer+，且对该知识库有读权限

### 5. 获取知识详情

**GET** `/api/v1/knowledge/:id`

**权限**: Viewer+，且对该知识所在知识库有读权限

### 6. 更新知识

**PUT** `/api/v1/knowledge/:id`

**权限**: KB 创建者或 Admin+，且对该知识所在知识库有写权限

### 7. 删除知识

**DELETE** `/api/v1/knowledge/:id`

**权限**: KB 创建者或 Admin+，且对该知识所在知识库有写权限

### 8. 重新解析知识

**POST** `/api/v1/knowledge/:id/reparse`

**权限**: KB 创建者或 Admin+，且对该知识所在知识库有写权限

### 9. 取消解析

**POST** `/api/v1/knowledge/:id/cancel-parse`

**权限**: KB 创建者或 Admin+，且对该知识所在知识库有写权限

### 10. 下载知识文件

**GET** `/api/v1/knowledge/:id/download`

**权限**: Viewer+，且对该知识所在知识库有读权限

### 11. 预览知识文件

**GET** `/api/v1/knowledge/:id/preview`

**权限**: Viewer+，且对该知识所在知识库有读权限

### 12. 更新图片信息

**PUT** `/api/v1/knowledge/image/:id/:chunk_id`

**权限**: KB 创建者或 Admin+，且对该知识所在知识库有写权限

### 13. 搜索知识

**GET** `/api/v1/knowledge/search`

**权限**: Viewer+

### 14. 批量获取知识

**GET** `/api/v1/knowledge/batch`

**权限**: Viewer+

### 15. 批量重新解析

**POST** `/api/v1/knowledge/batch-reparse`

**权限**: Contributor+

### 16. 批量删除

**POST** `/api/v1/knowledge/batch-delete`

**权限**: Contributor+

### 17. 移动知识

**POST** `/api/v1/knowledge/move`

**权限**: Contributor+

### 18. 获取移动进度

**GET** `/api/v1/knowledge/move/progress/:task_id`

**权限**: Viewer+

### 19. 清空知识库内容

**DELETE** `/api/v1/knowledge-bases/:id/knowledge`

**权限**: Admin+，且对该知识库有写权限

## 标签接口

### 1. 获取知识库标签列表

**GET** `/api/v1/knowledge-bases/:id/tags`

**权限**: Viewer+，且对该知识库有读权限

### 2. 创建标签

**POST** `/api/v1/knowledge-bases/:id/tags`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 3. 更新标签

**PUT** `/api/v1/knowledge-bases/:id/tags/:tag_id`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 4. 删除标签

**DELETE** `/api/v1/knowledge-bases/:id/tags/:tag_id`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 5. 批量更新知识标签

**PUT** `/api/v1/knowledge/tags`

**权限**: Contributor+

## FAQ 接口

### 1. 获取 FAQ 条目列表

**GET** `/api/v1/knowledge-bases/:id/faq/entries`

**权限**: Viewer+，且对该知识库有读权限

### 2. 导出 FAQ 条目

**GET** `/api/v1/knowledge-bases/:id/faq/entries/export`

**权限**: Viewer+，且对该知识库有读权限

### 3. 获取单个 FAQ 条目

**GET** `/api/v1/knowledge-bases/:id/faq/entries/:entry_id`

**权限**: Viewer+，且对该知识库有读权限

### 4. 创建/更新 FAQ 条目

**POST** `/api/v1/knowledge-bases/:id/faq/entries`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 5. 创建单个 FAQ 条目

**POST** `/api/v1/knowledge-bases/:id/faq/entry`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 6. 更新 FAQ 条目

**PUT** `/api/v1/knowledge-bases/:id/faq/entries/:entry_id`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 7. 添加相似问题

**POST** `/api/v1/knowledge-bases/:id/faq/entries/:entry_id/similar-questions`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 8. 批量更新 FAQ 条目字段

**PUT** `/api/v1/knowledge-bases/:id/faq/entries/fields`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 9. 批量更新 FAQ 条目标签

**PUT** `/api/v1/knowledge-bases/:id/faq/entries/tags`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 10. 删除 FAQ 条目

**DELETE** `/api/v1/knowledge-bases/:id/faq/entries`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 11. 搜索 FAQ

**POST** `/api/v1/knowledge-bases/:id/faq/search`

**权限**: Viewer+，且对该知识库有读权限

### 12. 更新导入结果显示状态

**PUT** `/api/v1/knowledge-bases/:id/faq/import/last-result/display`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 13. 获取导入进度

**GET** `/api/v1/faq/import/progress/:task_id`

**权限**: Viewer+

## 分块接口

### 1. 获取知识分块列表

**GET** `/api/v1/chunks/:knowledge_id`

**权限**: Viewer+，且对该知识所在知识库有读权限

### 2. 通过分块 ID 获取分块

**GET** `/api/v1/chunks/by-id/:id`

**权限**: Viewer+，且对该分块所在知识库有读权限

### 3. 删除分块

**DELETE** `/api/v1/chunks/:knowledge_id/:id`

**权限**: KB 创建者或 Admin+，且对该知识所在知识库有写权限

### 4. 删除知识下的所有分块

**DELETE** `/api/v1/chunks/:knowledge_id`

**权限**: KB 创建者或 Admin+，且对该知识所在知识库有写权限

### 5. 更新分块

**PUT** `/api/v1/chunks/:knowledge_id/:id`

**权限**: KB 创建者或 Admin+，且对该知识所在知识库有写权限

### 6. 删除生成的问题

**DELETE** `/api/v1/chunks/by-id/:id/questions`

**权限**: KB 创建者或 Admin+，且对该分块所在知识库有写权限

### 7. 分块预览

**POST** `/api/v1/chunker/preview`

**权限**: Viewer+

**请求体**:
```json
{
    "text": "测试文本内容...",
    "strategy": "auto",
    "chunk_size": 512,
    "chunk_overlap": 80
}
```

## 会话接口

### 1. 创建会话

**POST** `/api/v1/sessions`

**权限**: Viewer+

**请求体**:
```json
{
    "knowledge_base_ids": ["kb-xxx"],
    "agent_id": "agent-xxx",
    "title": "新会话"
}
```

### 2. 获取会话列表

**GET** `/api/v1/sessions`

**权限**: Viewer+

### 3. 获取会话详情

**GET** `/api/v1/sessions/:id`

**权限**: Viewer+

### 4. 更新会话

**PUT** `/api/v1/sessions/:id`

**权限**: Viewer+

### 5. 删除会话

**DELETE** `/api/v1/sessions/:id`

**权限**: Viewer+

### 6. 批量删除会话

**DELETE** `/api/v1/sessions/batch`

**权限**: Viewer+

### 7. 清空会话消息

**DELETE** `/api/v1/sessions/:id/messages`

**权限**: Viewer+

### 8. 生成会话标题

**POST** `/api/v1/sessions/:session_id/generate_title`

**权限**: Viewer+

### 9. 停止会话

**POST** `/api/v1/sessions/:session_id/stop`

**权限**: Viewer+

### 10. 置顶会话

**POST** `/api/v1/sessions/:session_id/pin`

**权限**: Viewer+

### 11. 取消置顶会话

**DELETE** `/api/v1/sessions/:id/pin`

**权限**: Viewer+

### 12. 继续活跃流

**GET** `/api/v1/sessions/continue-stream/:session_id`

**权限**: Viewer+

## 对话接口

### 1. RAG 问答

**POST** `/api/v1/knowledge-chat/:session_id`

**权限**: Viewer+

**请求体**:
```json
{
    "message": "用户问题",
    "stream": true,
    "knowledge_base_ids": ["kb-xxx"],
    "options": {
        "temperature": 0.7,
        "top_k": 30
    }
}
```

### 2. Agent 问答

**POST** `/api/v1/agent-chat/:session_id`

**权限**: Viewer+

**请求体**:
```json
{
    "message": "用户问题",
    "stream": true,
    "agent_id": "agent-xxx"
}
```

### 3. 知识检索（无需会话）

**POST** `/api/v1/knowledge-search`

**权限**: Viewer+

**请求体**:
```json
{
    "query": "搜索关键词",
    "knowledge_base_ids": ["kb-xxx"],
    "top_k": 10
}
```

## 消息接口

### 1. 加载消息

**GET** `/api/v1/messages/:session_id/load`

**权限**: Viewer+

### 2. 删除消息

**DELETE** `/api/v1/messages/:session_id/:id`

**权限**: Viewer+

### 3. 搜索消息

**POST** `/api/v1/messages/search`

**权限**: Viewer+

### 4. 获取聊天历史统计

**GET** `/api/v1/messages/chat-history-stats`

**权限**: Viewer+

## Agent 接口

### 1. 创建智能体

**POST** `/api/v1/agents`

**权限**: Contributor+

**请求体**:
```json
{
    "name": "我的智能体",
    "description": "测试智能体",
    "system_prompt": "你是一个专业助手...",
    "model_id": "model-xxx",
    "enabled_tools": ["web_search", "calculator"],
    "knowledge_base_ids": ["kb-xxx"],
    "type": "rag-qa"
}
```

### 2. 获取智能体列表

**GET** `/api/v1/agents`

**权限**: Viewer+

### 3. 获取智能体详情

**GET** `/api/v1/agents/:id`

**权限**: Viewer+

### 4. 更新智能体

**PUT** `/api/v1/agents/:id`

**权限**: Agent 创建者或 Admin+

### 5. 删除智能体

**DELETE** `/api/v1/agents/:id`

**权限**: Agent 创建者或 Admin+

### 6. 拷贝智能体

**POST** `/api/v1/agents/:id/copy`

**权限**: Contributor+

### 7. 获取占位符定义

**GET** `/api/v1/agents/placeholders`

**权限**: Viewer+

### 8. 获取智能体类型预设

**GET** `/api/v1/agents/type-presets`

**权限**: Viewer+

### 9. 获取建议问题

**GET** `/api/v1/agents/:id/suggested-questions`

**权限**: Viewer+

### 10. 处理工具审批

**POST** `/api/v1/agent/tool-approvals/:pending_id`

**权限**: Viewer+

### 11. 完成 MCP OAuth 授权

**POST** `/api/v1/agent/mcp-oauth-resolutions/:pending_id`

**权限**: Viewer+

### 12. 跳过 MCP OAuth 授权

**POST** `/api/v1/agent/mcp-oauth-resolutions/:pending_id/cancel`

**权限**: Viewer+

## 模型接口

### 1. 获取模型厂商列表

**GET** `/api/v1/models/providers`

**权限**: Viewer+

### 2. 创建模型

**POST** `/api/v1/models`

**权限**: Admin+

**请求体**:
```json
{
    "name": "GPT-4",
    "provider": "openai",
    "model": "gpt-4",
    "type": "chat",
    "api_base_url": "https://api.openai.com/v1",
    "credentials": {
        "api_key": "sk-xxx"
    }
}
```

### 3. 获取模型列表

**GET** `/api/v1/models`

**权限**: Viewer+

### 4. 获取单个模型

**GET** `/api/v1/models/:id`

**权限**: Viewer+

### 5. 更新模型

**PUT** `/api/v1/models/:id`

**权限**: Admin+

### 6. 删除模型

**DELETE** `/api/v1/models/:id`

**权限**: Admin+

### 7. 调试模型

**POST** `/api/v1/models/:id/debug`

**权限**: Admin+

### 8. 更新模型凭证

**PUT** `/api/v1/models/:id/credentials`

**权限**: Admin+

### 9. 删除模型凭证字段

**DELETE** `/api/v1/models/:id/credentials/:field`

**权限**: Admin+

## MCP 服务接口

### 1. 创建 MCP 服务

**POST** `/api/v1/mcp-services`

**权限**: Admin+

### 2. 获取 MCP 服务列表

**GET** `/api/v1/mcp-services`

**权限**: Viewer+

### 3. 获取 MCP 服务详情

**GET** `/api/v1/mcp-services/:id`

**权限**: Viewer+

### 4. 更新 MCP 服务

**PUT** `/api/v1/mcp-services/:id`

**权限**: Admin+

### 5. 删除 MCP 服务

**DELETE** `/api/v1/mcp-services/:id`

**权限**: Admin+

### 6. 测试 MCP 服务连接

**POST** `/api/v1/mcp-services/:id/test`

**权限**: Admin+

### 7. 获取 MCP 服务工具

**GET** `/api/v1/mcp-services/:id/tools`

**权限**: Viewer+

### 8. 获取 MCP 服务资源

**GET** `/api/v1/mcp-services/:id/resources`

**权限**: Viewer+

### 9. 更新 MCP 凭证

**PUT** `/api/v1/mcp-services/:id/credentials`

**权限**: Admin+

### 10. 删除 MCP 凭证字段

**DELETE** `/api/v1/mcp-services/:id/credentials/:field`

**权限**: Admin+

### 11. 获取 MCP 工具审批列表

**GET** `/api/v1/mcp-services/:id/tool-approvals`

**权限**: Viewer+

### 12. 设置 MCP 工具审批

**PUT** `/api/v1/mcp-services/:id/tool-approvals/:tool_name`

**权限**: Admin+

### 13. 获取 OAuth 授权 URL

**POST** `/api/v1/mcp-services/:id/oauth/authorize-url`

**权限**: Viewer+

### 14. 获取 OAuth 状态

**GET** `/api/v1/mcp-services/:id/oauth/status`

**权限**: Viewer+

### 15. 撤销 OAuth Token

**DELETE** `/api/v1/mcp-services/:id/oauth/token`

**权限**: Viewer+

### 16. MCP OAuth 回调

**GET** `/api/v1/mcp-oauth/callback`

## 租户接口

### 1. 创建租户

**POST** `/api/v1/tenants`

**请求体**:
```json
{
    "name": "我的租户",
    "description": "测试租户"
}
```

### 2. 获取租户列表

**GET** `/api/v1/tenants`

**权限**: Viewer+

### 3. 获取租户详情

**GET** `/api/v1/tenants/:id`

**权限**: Viewer+，且必须是该租户成员

### 4. 更新租户

**PUT** `/api/v1/tenants/:id`

**权限**: Owner+

### 5. 删除租户

**DELETE** `/api/v1/tenants/:id`

**权限**: Owner+

### 6. 获取所有租户（跨租户）

**GET** `/api/v1/tenants/all`

**权限**: 跨租户超级管理员

### 7. 搜索租户（跨租户）

**GET** `/api/v1/tenants/search`

**权限**: 跨租户超级管理员

### 8. 获取租户 KV 配置

**GET** `/api/v1/tenants/kv/:key`

**权限**: Viewer+

### 9. 更新租户 KV 配置

**PUT** `/api/v1/tenants/kv/:key`

**权限**: Admin+

### 10. 创建 API Key

**POST** `/api/v1/tenants/:id/api-keys`

**权限**: Owner+

### 11. 获取 API Key 列表

**GET** `/api/v1/tenants/:id/api-keys`

**权限**: Owner+

### 12. 删除 API Key

**DELETE** `/api/v1/tenants/:id/api-keys/:key_id`

**权限**: Owner+

### 13. 获取 API 主体配置

**GET** `/api/v1/tenants/:id/api-principal-config`

**权限**: Owner+

### 14. 更新 API 主体配置

**PUT** `/api/v1/tenants/:id/api-principal-config`

**权限**: Owner+

### 15. 创建测试 Token

**POST** `/api/v1/tenants/:id/api-principal-test-token`

**权限**: Owner+

## 租户成员接口

### 1. 获取成员列表

**GET** `/api/v1/tenants/:id/members`

**权限**: Viewer+

### 2. 添加成员

**POST** `/api/v1/tenants/:id/members`

**权限**: Owner+

### 3. 更新成员角色

**PUT** `/api/v1/tenants/:id/members/:user_id`

**权限**: Owner+

### 4. 移除成员

**DELETE** `/api/v1/tenants/:id/members/:user_id`

**权限**: Owner+

### 5. 退出租户

**POST** `/api/v1/tenants/:id/leave`

**权限**: Viewer+

## 租户邀请接口

### 1. 获取我的邀请列表

**GET** `/api/v1/me/invitations`

### 2. 获取待处理邀请数量

**GET** `/api/v1/me/invitations/pending-count`

### 3. 接受邀请

**POST** `/api/v1/me/invitations/:inv_id/accept`

### 4. 拒绝邀请

**POST** `/api/v1/me/invitations/:inv_id/decline`

### 5. 获取租户邀请列表

**GET** `/api/v1/tenants/:id/invitations`

**权限**: Viewer+

### 6. 创建邀请

**POST** `/api/v1/tenants/:id/invitations`

**权限**: Owner+

### 7. 撤销邀请

**DELETE** `/api/v1/tenants/:id/invitations/:inv_id`

**权限**: Owner+

### 8. 创建邀请链接

**POST** `/api/v1/tenants/:id/invite-links`

**权限**: Owner+

### 9. 查找邀请

**POST** `/api/v1/auth/invitations/lookup`

## 审计日志接口

### 1. 获取租户审计日志

**GET** `/api/v1/tenants/:id/audit-log`

**权限**: Admin+

### 2. 获取系统审计日志

**GET** `/api/v1/system/admin/audit-log`

**权限**: System Admin

## 网络搜索接口

### 1. 获取搜索提供商

**GET** `/api/v1/web-search/providers`

**权限**: Viewer+

### 2. 获取提供商类型

**GET** `/api/v1/web-search-providers/types`

**权限**: Viewer+

### 3. 测试提供商

**POST** `/api/v1/web-search-providers/test`

**权限**: Admin+

### 4. 创建提供商配置

**POST** `/api/v1/web-search-providers`

**权限**: Admin+

### 5. 获取提供商列表

**GET** `/api/v1/web-search-providers`

**权限**: Viewer+

### 6. 获取提供商详情

**GET** `/api/v1/web-search-providers/:id`

**权限**: Viewer+

### 7. 更新提供商配置

**PUT** `/api/v1/web-search-providers/:id`

**权限**: Admin+

### 8. 删除提供商配置

**DELETE** `/api/v1/web-search-providers/:id`

**权限**: Admin+

### 9. 更新提供商凭证

**PUT** `/api/v1/web-search-providers/:id/credentials`

**权限**: Admin+

### 10. 删除提供商凭证字段

**DELETE** `/api/v1/web-search-providers/:id/credentials/:field`

**权限**: Admin+

### 11. 测试已保存的提供商

**POST** `/api/v1/web-search-providers/:id/test`

**权限**: Admin+

## 向量存储接口

### 1. 获取存储引擎类型

**GET** `/api/v1/vector-stores/types`

**权限**: Viewer+

### 2. 测试存储配置

**POST** `/api/v1/vector-stores/test`

**权限**: Admin+

### 3. 创建存储配置

**POST** `/api/v1/vector-stores`

**权限**: Admin+

### 4. 获取存储配置列表

**GET** `/api/v1/vector-stores`

**权限**: Viewer+

### 5. 获取存储配置详情

**GET** `/api/v1/vector-stores/:id`

**权限**: Viewer+

### 6. 更新存储配置

**PUT** `/api/v1/vector-stores/:id`

**权限**: Admin+

### 7. 删除存储配置

**DELETE** `/api/v1/vector-stores/:id`

**权限**: Admin+

### 8. 测试已保存的存储配置

**POST** `/api/v1/vector-stores/:id/test`

**权限**: Admin+

## 数据源接口

### 1. 获取可用连接器类型

**GET** `/api/v1/datasource/types`

**权限**: Viewer+

### 2. 验证凭证

**POST** `/api/v1/datasource/validate-credentials`

**权限**: Admin+

### 3. 创建数据源

**POST** `/api/v1/datasource`

**权限**: Admin+

### 4. 获取数据源列表

**GET** `/api/v1/datasource`

**权限**: Viewer+

### 5. 获取数据源详情

**GET** `/api/v1/datasource/:id`

**权限**: Viewer+

### 6. 更新数据源

**PUT** `/api/v1/datasource/:id`

**权限**: Admin+

### 7. 删除数据源

**DELETE** `/api/v1/datasource/:id`

**权限**: Admin+

### 8. 更新数据源凭证

**PUT** `/api/v1/datasource/:id/credentials`

**权限**: Admin+

### 9. 删除数据源凭证字段

**DELETE** `/api/v1/datasource/:id/credentials/:field`

**权限**: Admin+

### 10. 验证连接

**POST** `/api/v1/datasource/:id/validate`

**权限**: Admin+

### 11. 获取可用资源

**GET** `/api/v1/datasource/:id/resources`

**权限**: Admin+

### 12. 解析资源祖先

**POST** `/api/v1/datasource/:id/resource-ancestors`

**权限**: Admin+

### 13. 手动同步

**POST** `/api/v1/datasource/:id/sync`

**权限**: Admin+

### 14. 暂停数据源

**POST** `/api/v1/datasource/:id/pause`

**权限**: Admin+

### 15. 恢复数据源

**POST** `/api/v1/datasource/:id/resume`

**权限**: Admin+

### 16. 获取同步日志

**GET** `/api/v1/datasource/:id/logs`

**权限**: Viewer+

**GET** `/api/v1/datasource/logs/:log_id`

**权限**: Viewer+

## Wiki 接口

### 1. 获取页面列表

**GET** `/api/v1/knowledgebase/:kb_id/wiki/pages`

**权限**: Viewer+，且对该知识库有读权限

### 2. 创建页面

**POST** `/api/v1/knowledgebase/:kb_id/wiki/pages`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 3. 获取页面

**GET** `/api/v1/knowledgebase/:kb_id/wiki/pages/*slug`

**权限**: Viewer+，且对该知识库有读权限

### 4. 更新页面

**PUT** `/api/v1/knowledgebase/:kb_id/wiki/pages/*slug`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 5. 删除页面

**DELETE** `/api/v1/knowledgebase/:kb_id/wiki/pages/*slug`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 6. 移动页面

**PUT** `/api/v1/knowledgebase/:kb_id/wiki/move-page`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 7. 获取文件夹列表

**GET** `/api/v1/knowledgebase/:kb_id/wiki/folders`

**权限**: Viewer+，且对该知识库有读权限

### 8. 创建文件夹

**POST** `/api/v1/knowledgebase/:kb_id/wiki/folders`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 9. 更新文件夹

**PUT** `/api/v1/knowledgebase/:kb_id/wiki/folders/:folder_id`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 10. 删除文件夹

**DELETE** `/api/v1/knowledgebase/:kb_id/wiki/folders/:folder_id`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 11. 获取索引页面

**GET** `/api/v1/knowledgebase/:kb_id/wiki/index`

**权限**: Viewer+，且对该知识库有读权限

### 12. 获取日志

**GET** `/api/v1/knowledgebase/:kb_id/wiki/log`

**权限**: Viewer+，且对该知识库有读权限

### 13. 获取图谱

**GET** `/api/v1/knowledgebase/:kb_id/wiki/graph`

**权限**: Viewer+，且对该知识库有读权限

### 14. 获取统计

**GET** `/api/v1/knowledgebase/:kb_id/wiki/stats`

**权限**: Viewer+，且对该知识库有读权限

### 15. 搜索页面

**GET** `/api/v1/knowledgebase/:kb_id/wiki/search`

**权限**: Viewer+，且对该知识库有读权限

### 16. 重建链接

**POST** `/api/v1/knowledgebase/:kb_id/wiki/rebuild-links`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 17. 获取 Lint 结果

**GET** `/api/v1/knowledgebase/:kb_id/wiki/lint`

**权限**: Viewer+，且对该知识库有读权限

### 18. 自动修复

**POST** `/api/v1/knowledgebase/:kb_id/wiki/auto-fix`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 19. 获取问题列表

**GET** `/api/v1/knowledgebase/:kb_id/wiki/issues`

**权限**: Viewer+，且对该知识库有读权限

### 20. 更新问题状态

**PUT** `/api/v1/knowledgebase/:kb_id/wiki/issues/:issue_id/status`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

## 组织与分享接口

### 1. 创建组织

**POST** `/api/v1/organizations`

**权限**: Admin+

### 2. 获取我的组织列表

**GET** `/api/v1/organizations`

**权限**: Viewer+

### 3. 预览组织

**GET** `/api/v1/organizations/preview/:code`

**权限**: Viewer+

### 4. 加入组织

**POST** `/api/v1/organizations/join`

**权限**: Admin+

### 5. 搜索组织

**GET** `/api/v1/organizations/search`

**权限**: Viewer+

### 6. 通过 ID 加入组织

**POST** `/api/v1/organizations/join-by-id`

**权限**: Admin+

### 7. 获取组织详情

**GET** `/api/v1/organizations/:id`

**权限**: Viewer+

### 8. 更新组织

**PUT** `/api/v1/organizations/:id`

**权限**: Admin+

### 9. 删除组织

**DELETE** `/api/v1/organizations/:id`

**权限**: Admin+

### 10. 退出组织

**POST** `/api/v1/organizations/:id/leave`

**权限**: Admin+

### 11. 请求升级角色

**POST** `/api/v1/organizations/:id/request-upgrade`

**权限**: Admin+

### 12. 生成邀请码

**POST** `/api/v1/organizations/:id/invite-code`

**权限**: Admin+

### 13. 搜索租户（邀请用）

**GET** `/api/v1/organizations/:id/search-tenants`

**权限**: Admin+

### 14. 邀请成员

**POST** `/api/v1/organizations/:id/invite`

**权限**: Admin+

### 15. 获取组织成员

**GET** `/api/v1/organizations/:id/members`

**权限**: Viewer+

### 16. 更新成员角色

**PUT** `/api/v1/organizations/:id/members/:tenant_id`

**权限**: Admin+

### 17. 移除成员

**DELETE** `/api/v1/organizations/:id/members/:tenant_id`

**权限**: Admin+

### 18. 获取加入请求

**GET** `/api/v1/organizations/:id/join-requests`

**权限**: Admin+

### 19. 审核加入请求

**PUT** `/api/v1/organizations/:id/join-requests/:request_id/review`

**权限**: Admin+

### 20. 获取组织分享列表

**GET** `/api/v1/organizations/:id/shares`

**权限**: Viewer+

### 21. 获取组织 Agent 分享列表

**GET** `/api/v1/organizations/:id/agent-shares`

**权限**: Viewer+

### 22. 获取组织共享知识库

**GET** `/api/v1/organizations/:id/shared-knowledge-bases`

**权限**: Viewer+

### 23. 获取组织共享 Agent

**GET** `/api/v1/organizations/:id/shared-agents`

**权限**: Viewer+

### 24. 分享知识库到组织

**POST** `/api/v1/knowledge-bases/:id/shares`

**权限**: KB 创建者或 Admin+

### 25. 获取知识库分享列表

**GET** `/api/v1/knowledge-bases/:id/shares`

**权限**: Viewer+

### 26. 更新分享权限

**PUT** `/api/v1/knowledge-bases/:id/shares/:share_id`

**权限**: KB 创建者或 Admin+

### 27. 移除分享

**DELETE** `/api/v1/knowledge-bases/:id/shares/:share_id`

**权限**: KB 创建者或 Admin+

### 28. 分享 Agent 到组织

**POST** `/api/v1/agents/:id/shares`

**权限**: Agent 创建者或 Admin+

### 29. 获取 Agent 分享列表

**GET** `/api/v1/agents/:id/shares`

**权限**: Agent 创建者或 Admin+

### 30. 移除 Agent 分享

**DELETE** `/api/v1/agents/:id/shares/:share_id`

**权限**: Agent 创建者或 Admin+

### 31. 获取共享知识库列表

**GET** `/api/v1/shared-knowledge-bases`

**权限**: Viewer+

### 32. 获取共享 Agent 列表

**GET** `/api/v1/shared-agents`

**权限**: Viewer+

### 33. 设置共享 Agent 禁用状态

**POST** `/api/v1/shared-agents/disabled`

**权限**: Admin+

## IM 渠道接口

### 1. 创建 IM 渠道

**POST** `/api/v1/agents/:id/im-channels`

**权限**: Admin+

### 2. 获取 Agent 的 IM 渠道列表

**GET** `/api/v1/agents/:id/im-channels`

**权限**: Viewer+

### 3. 获取所有 IM 渠道

**GET** `/api/v1/im-channels`

**权限**: Viewer+

### 4. 更新 IM 渠道

**PUT** `/api/v1/im-channels/:id`

**权限**: Admin+

### 5. 删除 IM 渠道

**DELETE** `/api/v1/im-channels/:id`

**权限**: Admin+

### 6. 切换 IM 渠道状态

**POST** `/api/v1/im-channels/:id/toggle`

**权限**: Admin+

### 7. 获取微信二维码

**POST** `/api/v1/wechat/qrcode`

**权限**: Admin+

### 8. 轮询微信二维码状态

**POST** `/api/v1/wechat/qrcode/status`

**权限**: Admin+

### 9. IM 回调

**GET** `/api/v1/im/callback/:channel_id`

**POST** `/api/v1/im/callback/:channel_id`

## 嵌入渠道接口

### 1. 创建嵌入渠道

**POST** `/api/v1/agents/:id/embed-channels`

**权限**: Admin+

### 2. 获取 Agent 的嵌入渠道列表

**GET** `/api/v1/agents/:id/embed-channels`

**权限**: Viewer+

### 3. 获取所有嵌入渠道

**GET** `/api/v1/embed-channels`

**权限**: Viewer+

### 4. 获取嵌入渠道详情

**GET** `/api/v1/embed-channels/:channel_id`

**权限**: Viewer+

### 5. 更新嵌入渠道

**PUT** `/api/v1/embed-channels/:channel_id`

**权限**: Admin+

### 6. 删除嵌入渠道

**DELETE** `/api/v1/embed-channels/:channel_id`

**权限**: Admin+

### 7. 轮换嵌入 Token

**POST** `/api/v1/embed-channels/:channel_id/rotate-token`

**权限**: Admin+

### 8. 生成预览会话

**POST** `/api/v1/embed-channels/:channel_id/preview-session`

**权限**: Viewer+

### 9. 获取嵌入渠道统计

**GET** `/api/v1/embed-channels/:channel_id/stats`

**权限**: Viewer+

## 嵌入公开接口

### 1. 交换嵌入会话

**POST** `/api/v1/embed/:channel_id/exchange`

### 2. 获取嵌入配置

**GET** `/api/v1/embed/:channel_id/config`

### 3. 获取建议问题

**GET** `/api/v1/embed/:channel_id/suggested-questions`

### 4. 获取分块

**GET** `/api/v1/embed/:channel_id/chunks/:chunk_id`

### 5. 创建嵌入会话

**POST** `/api/v1/embed/:channel_id/sessions`

### 6. RAG 问答

**POST** `/api/v1/embed/:channel_id/knowledge-chat/:session_id`

### 7. Agent 问答

**POST** `/api/v1/embed/:channel_id/agent-chat/:session_id`

### 8. 加载消息

**GET** `/api/v1/embed/:channel_id/messages/:session_id/load`

### 9. 停止会话

**POST** `/api/v1/embed/:channel_id/sessions/:session_id/stop`

### 10. 转发 Webhook 事件

**POST** `/api/v1/embed/:channel_id/sessions/:session_id/events`

### 11. 解析 MCP OAuth

**POST** `/api/v1/embed/:channel_id/sessions/:session_id/mcp-oauth-resolutions/:pending_id`

### 12. 取消 MCP OAuth

**POST** `/api/v1/embed/:channel_id/sessions/:session_id/mcp-oauth-resolutions/:pending_id/cancel`

### 13. 获取 MCP OAuth 授权 URL

**POST** `/api/v1/embed/:channel_id/sessions/:session_id/mcp-services/:id/oauth/authorize-url`

### 14. 获取 MCP OAuth 状态

**GET** `/api/v1/embed/:channel_id/sessions/:session_id/mcp-services/:id/oauth/status`

### 15. 解析工具审批

**POST** `/api/v1/embed/:channel_id/sessions/:session_id/tool-approvals/:pending_id`

### 16. 服务文件

**GET** `/api/v1/embed/:channel_id/files`

## 技能接口

### 1. 获取技能列表

**GET** `/api/v1/skills`

**权限**: Viewer+

## 用户收藏接口

### 1. 获取收藏列表

**GET** `/api/v1/user/favorites`

**权限**: Viewer+

### 2. 添加收藏

**POST** `/api/v1/user/favorites`

**权限**: Viewer+

### 3. 移除收藏

**DELETE** `/api/v1/user/favorites/:type/:id`

**权限**: Viewer+

## 初始化接口

### 1. 获取 KB 当前配置

**GET** `/api/v1/initialization/config/:kbId`

**权限**: Viewer+，且对该知识库有读权限

### 2. 初始化 KB

**POST** `/api/v1/initialization/initialize/:kbId`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 3. 更新 KB 配置

**PUT** `/api/v1/initialization/config/:kbId`

**权限**: KB 创建者或 Admin+，且对该知识库有写权限

### 4. 检查 Ollama 状态

**GET** `/api/v1/initialization/ollama/status`

**权限**: Viewer+

### 5. 获取 Ollama 模型列表

**GET** `/api/v1/initialization/ollama/models`

**权限**: Viewer+

### 6. 检查 Ollama 模型

**POST** `/api/v1/initialization/ollama/models/check`

**权限**: Admin+

### 7. 下载 Ollama 模型

**POST** `/api/v1/initialization/ollama/models/download`

**权限**: Admin+

### 8. 获取下载进度

**GET** `/api/v1/initialization/ollama/download/progress/:taskId`

**权限**: Viewer+

### 9. 获取下载任务列表

**GET** `/api/v1/initialization/ollama/download/tasks`

**权限**: Viewer+

### 10. 检查远程模型

**POST** `/api/v1/initialization/remote/check`

**权限**: Admin+

### 11. 测试嵌入模型

**POST** `/api/v1/initialization/embedding/test`

**权限**: Admin+

### 12. 检查重排序模型

**POST** `/api/v1/initialization/rerank/check`

**权限**: Admin+

### 13. 检查 ASR 模型

**POST** `/api/v1/initialization/asr/check`

**权限**: Admin+

### 14. 测试多模态功能

**POST** `/api/v1/initialization/multimodal/test`

**权限**: Admin+

### 15. 抽取文本关系

**POST** `/api/v1/initialization/extract/text-relation`

**权限**: Admin+

### 16. 抽取标签

**POST** `/api/v1/initialization/extract/fabri-tag`

**权限**: Admin+

### 17. 抽取文本

**POST** `/api/v1/initialization/extract/fabri-text`

**权限**: Admin+

## 系统接口

### 1. 获取系统信息

**GET** `/api/v1/system/info`

**权限**: Viewer+

### 2. 获取解析引擎列表

**GET** `/api/v1/system/parser-engines`

**权限**: Viewer+

### 3. 检查解析引擎

**POST** `/api/v1/system/parser-engines/check`

**权限**: Admin+

### 4. 重新连接 DocReader

**POST** `/api/v1/system/docreader/reconnect`

**权限**: Admin+

### 5. 获取存储引擎状态

**GET** `/api/v1/system/storage-engine-status`

**权限**: Viewer+

### 6. 检查存储引擎

**POST** `/api/v1/system/storage-engine-check`

**权限**: Admin+

### 7. 健康检查

**GET** `/health`

## 系统管理员接口

### 1. 提升为系统管理员

**POST** `/api/v1/system/admin/promote`

**权限**: System Admin

### 2. 撤销系统管理员

**POST** `/api/v1/system/admin/revoke`

**权限**: System Admin

### 3. 获取系统管理员列表

**GET** `/api/v1/system/admin/list`

**权限**: System Admin

### 4. 获取系统设置列表

**GET** `/api/v1/system/admin/settings`

**权限**: System Admin

### 5. 获取系统设置

**GET** `/api/v1/system/admin/settings/:key`

**权限**: System Admin

### 6. 更新系统设置

**PUT** `/api/v1/system/admin/settings/:key`

**权限**: System Admin

### 7. 重置系统设置

**DELETE** `/api/v1/system/admin/settings/:key`

**权限**: System Admin

### 8. 应用默认存储配额到所有租户

**POST** `/api/v1/system/admin/tenants/apply-default-storage-quota`

**权限**: System Admin

## 评估接口

### 1. 执行评估

**POST** `/api/v1/evaluation`

**权限**: Admin+

### 2. 获取评估结果

**GET** `/api/v1/evaluation`

**权限**: Viewer+

## 文件服务接口

### 1. 获取文件

**GET** `/api/v1/files?file_path=<provider://...>`

### 2. 获取预签名文件

**GET** `/api/v1/files/presigned?file_path=<provider://...>&tenant_id=<id>&expires=<unix>&sig=<hmac>`

### 3. 获取预签名预览

**GET** `/api/v1/files/presigned-preview?file_path=<provider://...>`

**权限**: Admin+

## WeKnoraCloud 接口

### 1. 保存凭证

**POST** `/api/v1/weknoracloud/credentials`

**权限**: Admin+

### 2. 获取状态

**GET** `/api/v1/models/weknoracloud/status`

**权限**: Viewer+

## 错误码

| 错误码 | 含义 |
|--------|------|
| 0 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未授权 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 409 | 冲突 |
| 500 | 服务器错误 |

## API Key 权限范围

| 能力 | 说明 |
|------|------|
| `read` | 只读访问 |
| `write` | 写入访问 |
| `manage_kbs` | 管理知识库 |
| `manage_models` | 管理模型 |
| `manage_mcp_services` | 管理 MCP 服务 |
| `manage_web_search` | 管理网络搜索 |
| `manage_vector_stores` | 管理向量存储 |
| `manage_data_sources` | 管理数据源 |
| `manage_channels` | 管理渠道 |
| `manage_spaces` | 管理组织空间 |
| `manage_tenant_settings` | 管理租户设置 |
| `manage_members` | 管理成员 |
| `run_evaluations` | 执行评估 |
| `chat` | 对话能力 |
| `message_history` | 消息历史 |
| `read_agents` | 读取 Agent |
| `manage_agents` | 管理 Agent |
