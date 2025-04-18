# Dify 管理 API 接口文档

本文档详细介绍 Dify 的管理 API，这些 API 用于管理 Dify 平台资源，包括应用、数据集、模型配置等。

## 目录

- [认证方式](#认证方式)
- [基本使用](#基本使用)
- [应用管理接口](#应用管理接口)
- [数据集接口](#数据集接口)
- [工作区接口](#工作区接口)
- [模型配置接口](#模型配置接口)
- [成员管理接口](#成员管理接口)
- [账户接口](#账户接口)
- [错误处理](#错误处理)

## 认证方式

管理 API 使用 Personal Access Token (PAT) 进行认证：

```
Authorization: Bearer {PERSONAL_ACCESS_TOKEN}
```

您可以在 Dify 平台的"设置" > "API 访问"中创建和管理您的 Personal Access Token。

## 基本使用

- 所有 API 请求应发送到 Dify 服务的基础 URL
- 请求和响应的内容类型均为 `application/json`
- 所有时间戳使用 UTC 时间的 Unix 时间戳（秒）

## 应用管理接口

### 获取应用列表

```
GET /api/workspaces/{workspace_id}/apps
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| page | Integer | 否 | 页码，默认为1 |
| limit | Integer | 否 | 每页数量，默认20，最大100 |
| name | String | 否 | 按名称筛选 |
| type | String | 否 | 应用类型筛选："chat" 或 "completion" |

**响应字段**：

| 字段 | 类型 | 说明 |
|-----|------|------|
| data | Array | 应用列表 |
| total | Integer | 总应用数 |
| page | Integer | 当前页码 |
| limit | Integer | 每页数量 |

**示例响应**：

```json
{
  "data": [
    {
      "id": "app-xxx",
      "name": "客服助手",
      "type": "chat",
      "icon": "emoji-happy",
      "icon_background": "#FFDD00",
      "description": "一个智能客服助理",
      "created_at": 1650000000
    }
  ],
  "total": 10,
  "page": 1,
  "limit": 20
}
```

### 创建应用

```
POST /api/workspaces/{workspace_id}/apps
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| name | String | 是 | 应用名称 |
| type | String | 是 | 应用类型："chat" 或 "completion" |
| icon | String | 否 | 图标标识 |
| icon_background | String | 否 | 图标背景色 |
| description | String | 否 | 应用描述 |

**响应字段**：返回创建的应用对象

### 获取应用详情

```
GET /api/apps/{app_id}
```

**请求参数**：无

**响应字段**：返回应用的详细信息

### 更新应用

```
PUT /api/apps/{app_id}
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| name | String | 否 | 应用名称 |
| icon | String | 否 | 图标标识 |
| icon_background | String | 否 | 图标背景色 |
| description | String | 否 | 应用描述 |

**响应字段**：返回更新后的应用对象

### 删除应用

```
DELETE /api/apps/{app_id}
```

**请求参数**：无

**响应字段**：

| 字段 | 类型 | 说明 |
|-----|------|------|
| result | String | 操作结果，成功为 "success" |

### 应用变量配置

```
PUT /api/apps/{app_id}/variables
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| variables | Array | 是 | 变量配置数组 |

**响应字段**：返回更新后的变量配置

## 数据集接口

### 获取数据集列表

```
GET /api/workspaces/{workspace_id}/datasets
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| page | Integer | 否 | 页码，默认为1 |
| limit | Integer | 否 | 每页数量，默认20，最大100 |
| name | String | 否 | 按名称筛选 |

**响应字段**：

| 字段 | 类型 | 说明 |
|-----|------|------|
| data | Array | 数据集列表 |
| total | Integer | 总数据集数 |
| page | Integer | 当前页码 |
| limit | Integer | 每页数量 |

### 创建数据集

```
POST /api/workspaces/{workspace_id}/datasets
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| name | String | 是 | 数据集名称 |
| description | String | 否 | 数据集描述 |
| indexing_technique | String | 是 | 索引技术："high_quality" 或 "economy" |

**响应字段**：返回创建的数据集对象

### 获取数据集详情

```
GET /api/datasets/{dataset_id}
```

**请求参数**：无

**响应字段**：返回数据集的详细信息

### 更新数据集

```
PATCH /api/datasets/{dataset_id}
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| name | String | 否 | 数据集名称 |
| description | String | 否 | 数据集描述 |

**响应字段**：返回更新后的数据集对象

### 删除数据集

```
DELETE /api/datasets/{dataset_id}
```

**请求参数**：无

**响应字段**：

| 字段 | 类型 | 说明 |
|-----|------|------|
| result | String | 操作结果，成功为 "success" |

### 上传文件到数据集

```
POST /api/datasets/{dataset_id}/documents/upload
```

**请求参数**：multipart/form-data 表单

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| file | File | 是 | 要上传的文件 |
| process_rule | Object | 是 | 处理规则 |

**响应字段**：返回上传任务信息

### 获取文档列表

```
GET /api/datasets/{dataset_id}/documents
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| page | Integer | 否 | 页码，默认为1 |
| limit | Integer | 否 | 每页数量，默认20，最大100 |
| status | String | 否 | 文档状态筛选 |

**响应字段**：

| 字段 | 类型 | 说明 |
|-----|------|------|
| data | Array | 文档列表 |
| total | Integer | 总文档数 |
| page | Integer | 当前页码 |
| limit | Integer | 每页数量 |

## 工作区接口

### 获取工作区信息

```
GET /api/workspaces/current
```

**请求参数**：无

**响应字段**：返回当前工作区的详细信息

### 更新工作区设置

```
PATCH /api/workspaces/{workspace_id}
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| name | String | 否 | 工作区名称 |
| plan | String | 否 | 工作区计划类型 |

**响应字段**：返回更新后的工作区信息

## 模型配置接口

### 获取模型供应商列表

```
GET /api/workspaces/{workspace_id}/model-providers
```

**请求参数**：无

**响应字段**：返回模型供应商列表

### 配置模型供应商

```
POST /api/workspaces/{workspace_id}/model-providers/{provider_name}/configure
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| api_key | String | 是 | API 密钥 |
| base_url | String | 否 | 自定义 API 基础 URL |

**响应字段**：返回配置结果

### 获取模型列表

```
GET /api/workspaces/{workspace_id}/models
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| provider | String | 否 | 按提供商筛选 |
| type | String | 否 | 模型类型筛选：text-generation, embedding 等 |

**响应字段**：返回模型列表

## 成员管理接口

### 获取成员列表

```
GET /api/workspaces/{workspace_id}/members
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| page | Integer | 否 | 页码，默认为1 |
| limit | Integer | 否 | 每页数量，默认20，最大100 |

**响应字段**：

| 字段 | 类型 | 说明 |
|-----|------|------|
| data | Array | 成员列表 |
| total | Integer | 总成员数 |
| page | Integer | 当前页码 |
| limit | Integer | 每页数量 |

### 邀请成员

```
POST /api/workspaces/{workspace_id}/members/invite
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| email | String | 是 | 受邀成员的电子邮件 |
| role | String | 是 | 角色："admin", "normal" |

**响应字段**：返回邀请信息

### 移除成员

```
DELETE /api/workspaces/{workspace_id}/members/{member_id}
```

**请求参数**：无

**响应字段**：

| 字段 | 类型 | 说明 |
|-----|------|------|
| result | String | 操作结果，成功为 "success" |

## 账户接口

### 获取当前用户信息

```
GET /api/account/profile
```

**请求参数**：无

**响应字段**：返回当前用户的详细信息

### 更新用户资料

```
PUT /api/account/profile
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| name | String | 否 | 用户名称 |
| avatar | String | 否 | 头像 URL |

**响应字段**：返回更新后的用户信息

### 管理 API 密钥

```
GET /api/account/api-keys
```

**请求参数**：无

**响应字段**：返回 API 密钥列表

### 创建 API 密钥

```
POST /api/account/api-keys
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|-----|------|------|------|
| name | String | 是 | 密钥名称 |
| role | String | 是 | 密钥角色："read" 或 "full" |

**响应字段**：返回创建的 API 密钥信息，包括密钥值（仅显示一次）

### 删除 API 密钥

```
DELETE /api/account/api-keys/{key_id}
```

**请求参数**：无

**响应字段**：

| 字段 | 类型 | 说明 |
|-----|------|------|
| result | String | 操作结果，成功为 "success" |

## 错误处理

所有 API 在遇到错误时会返回相应的 HTTP 状态码和 JSON 格式的错误信息：

```json
{
  "error": {
    "code": "error_code",
    "message": "详细错误信息",
    "details": { ... }  // 可选的错误详情
  }
}
```

**常见错误代码**：

| HTTP 状态码 | 错误代码 | 说明 |
|------------|---------|------|
| 400 | invalid_request | 请求参数无效 |
| 401 | authentication_failed | 认证失败 |
| 403 | permission_denied | 权限不足 |
| 404 | not_found | 资源不存在 |
| 429 | rate_limit_exceeded | 超出请求频率限制 |
| 500 | internal_server_error | 服务器内部错误 | 