# Dify API 开发指南

Dify 提供了全面的 API 服务，让开发者能够将 Dify 的强大功能集成到自己的应用程序中。本指南将帮助您了解如何使用 Dify 的 API 进行开发。

## 目录

- [API 概述](#api-概述)
- [认证与授权](#认证与授权)
- [API 密钥管理](#api-密钥管理)
- [常用 API 接口](#常用-api-接口)
- [SDK 使用](#sdk-使用)
- [示例代码](#示例代码)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)

## API 概述

Dify 提供了两种类型的 API：

1. **应用 API**：用于在您自己的应用中集成特定的 Dify 应用
2. **管理 API**：用于管理 Dify 平台资源，如应用、数据集等

这两种 API 使用不同的认证机制和端点，根据您的需求选择合适的 API 类型。

## 认证与授权

### 应用 API 认证

应用 API 使用 API 密钥进行认证。每个 Dify 应用都有自己的 API 密钥：

1. 在 Dify 控制台中打开您的应用
2. 点击右上角的**设置**
3. 选择**API 访问**
4. 在此处您可以找到应用的 API 密钥

在 API 请求中，使用以下方式添加密钥：

```
headers: {
  'Authorization': 'Bearer {API_SECRET_KEY}',
  'Content-Type': 'application/json'
}
```

### 管理 API 认证

管理 API 使用 Personal Access Token 进行认证：

1. 在 Dify 控制台中，点击右上角的用户头像
2. 选择**设置**
3. 选择**API 密钥**
4. 创建新的访问令牌

在 API 请求中，使用以下方式添加令牌：

```
headers: {
  'Authorization': 'Bearer {PERSONAL_ACCESS_TOKEN}',
  'Content-Type': 'application/json'
}
```

## API 密钥管理

### 创建 API 密钥

1. 对于应用 API 密钥：
   - 在应用的**API 访问**页面中可以查看和重置密钥
   - 每个应用有两个密钥：API_SECRET_KEY（用于后端请求）和 API_KEY（用于前端请求）

2. 对于个人访问令牌：
   - 在用户设置的**API 密钥**页面创建
   - 创建时需要设置名称和过期时间
   - 创建后务必保存令牌，因为它只会显示一次

### 密钥安全最佳实践

- 不要在客户端代码中硬编码 API_SECRET_KEY
- 定期轮换 API 密钥
- 使用环境变量存储密钥
- 为不同的应用场景使用不同的密钥
- 设置合理的密钥过期时间

## 常用 API 接口

### 应用 API

#### 对话类应用

1. **创建对话**
   ```
   POST /api/chat-messages
   ```

2. **获取对话消息历史**
   ```
   GET /api/chat-messages?conversation_id={conversation_id}
   ```

3. **发送消息**
   ```
   POST /api/chat-messages
   Body: {
     "inputs": {},
     "query": "你好",
     "conversation_id": "optional-conversation-id",
     "response_mode": "streaming" // 或 "blocking"
   }
   ```

#### 文本生成应用

```
POST /api/completion-messages
Body: {
  "inputs": {},
  "query": "写一篇关于人工智能的文章"
}
```

### 管理 API

1. **获取应用列表**
   ```
   GET /api/management/apps
   ```

2. **创建应用**
   ```
   POST /api/management/apps
   ```

3. **获取数据集列表**
   ```
   GET /api/management/datasets
   ```

4. **上传文件到数据集**
   ```
   POST /api/management/datasets/{dataset_id}/documents/upload
   ```

## SDK 使用

Dify 提供了多种语言的 SDK，简化了 API 的使用：

### JavaScript SDK

安装：
```bash
npm install dify-client
```

使用示例：
```javascript
import { DifyClient } from 'dify-client';

// 创建客户端实例
const client = new DifyClient({
  apiKey: 'your-api-key',
  baseURL: 'https://api.dify.ai',  // 或者自托管的 API 地址
});

// 聊天应用
async function chatExample() {
  // 创建会话
  const response = await client.chat({
    query: '你好，Dify！',
    inputs: {},
    responseMode: 'streaming',
    onMessageStream: (chunk) => {
      console.log(chunk.answer);
    }
  });
  
  console.log('会话 ID:', response.conversationId);
}

// 文本生成应用
async function completionExample() {
  const response = await client.completion({
    query: '写一篇关于人工智能的短文',
    inputs: {}
  });
  
  console.log('生成内容:', response.text);
}
```

### Python SDK

安装：
```bash
pip install dify-client
```

使用示例：
```python
from dify_client import DifyClient

# 创建客户端实例
client = DifyClient(
    api_key="your-api-key",
    base_url="https://api.dify.ai"  # 或者自托管的 API 地址
)

# 聊天应用
def chat_example():
    response = client.chat(
        query="你好，Dify！",
        inputs={},
        response_mode="blocking"
    )
    
    print("回答:", response.message)
    print("会话 ID:", response.conversation_id)

# 文本生成应用
def completion_example():
    response = client.completion(
        query="写一篇关于人工智能的短文",
        inputs={}
    )
    
    print("生成内容:", response.text)
```

## 示例代码

### 构建聊天机器人

#### 前端（React）

```jsx
import { useState, useEffect } from 'react';
import { DifyClient } from 'dify-client';

const client = new DifyClient({
  apiKey: process.env.REACT_APP_DIFY_API_KEY,
  baseURL: process.env.REACT_APP_DIFY_API_URL,
});

function ChatBot() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [conversationId, setConversationId] = useState(null);
  const [loading, setLoading] = useState(false);

  const sendMessage = async () => {
    if (!input.trim()) return;
    
    const userMessage = { role: 'user', content: input };
    setMessages([...messages, userMessage]);
    setInput('');
    setLoading(true);
    
    let assistantMessage = { role: 'assistant', content: '' };
    setMessages([...messages, userMessage, assistantMessage]);
    
    try {
      const response = await client.chat({
        query: input,
        inputs: {},
        conversation_id: conversationId,
        response_mode: 'streaming',
        onMessageStream: (chunk) => {
          assistantMessage.content = chunk.answer;
          setMessages([...messages, userMessage, { ...assistantMessage }]);
        }
      });
      
      setConversationId(response.conversationId);
    } catch (error) {
      console.error('聊天出错:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="chat-container">
      <div className="messages">
        {messages.map((msg, index) => (
          <div key={index} className={`message ${msg.role}`}>
            {msg.content}
          </div>
        ))}
        {loading && <div className="loading">正在思考...</div>}
      </div>
      <div className="input-area">
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="在这里输入消息..."
          onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
        />
        <button onClick={sendMessage}>发送</button>
      </div>
    </div>
  );
}

export default ChatBot;
```

#### 后端（Node.js）

```javascript
const express = require('express');
const axios = require('axios');
const dotenv = require('dotenv');

dotenv.config();
const app = express();
app.use(express.json());

// 配置
const DIFY_API_URL = process.env.DIFY_API_URL;
const DIFY_API_KEY = process.env.DIFY_API_SECRET_KEY;

// 代理 Dify API 请求
app.post('/api/chat', async (req, res) => {
  try {
    const { query, conversationId, inputs } = req.body;
    
    const response = await axios.post(
      `${DIFY_API_URL}/api/chat-messages`, 
      {
        query,
        conversation_id: conversationId,
        inputs: inputs || {},
        response_mode: 'blocking',
      },
      {
        headers: {
          'Authorization': `Bearer ${DIFY_API_KEY}`,
          'Content-Type': 'application/json',
        },
      }
    );
    
    res.json(response.data);
  } catch (error) {
    console.error('API 请求错误:', error.response?.data || error.message);
    res.status(500).json({ error: '聊天请求失败' });
  }
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
  console.log(`服务器运行在端口 ${PORT}`);
});
```

## 最佳实践

### 性能优化

1. **使用流式响应**：对于聊天应用，尽可能使用流式响应（streaming）模式，提供更好的用户体验
2. **会话管理**：妥善管理和存储会话 ID，避免创建过多新会话
3. **错误处理**：实现健壮的错误处理机制，包括重试逻辑和用户友好的错误消息
4. **并发请求限制**：控制向 Dify API 发送的并发请求数量，避免触发限流

### 安全最佳实践

1. **不要在前端存储 API_SECRET_KEY**：只在服务器端使用 API_SECRET_KEY
2. **实现代理服务**：使用后端服务代理 Dify API 请求，而不是直接从前端调用
3. **加密敏感数据**：在存储对话历史或用户输入时加密敏感信息
4. **输入验证**：在发送到 Dify API 之前验证和清理用户输入
5. **实施速率限制**：为您的应用添加速率限制，防止 API 滥用

## 常见问题

### 调用 API 返回错误码，如何排查？

1. 检查认证令牌是否有效且未过期
2. 确认 API URL 是否正确
3. 验证请求体格式是否符合要求
4. 查看错误响应中的详细信息
5. 检查您的账户额度是否充足

### 如何处理流式响应中的错误？

流式响应可能在中间出现错误。实现一个错误处理机制：

```javascript
client.chat({
  // ... 其他参数
  response_mode: 'streaming',
  onMessageStream: (chunk) => {
    // 处理正常数据
  },
  onError: (error) => {
    console.error('流式响应错误:', error);
    // 向用户显示错误消息
  }
});
```

### API 请求超时怎么办？

1. 增加客户端的超时设置
2. 对于长时间运行的请求，考虑使用异步处理模式
3. 实现指数退避重试机制

### 如何在移动应用中使用 Dify API？

1. 不要在移动客户端直接使用 API_SECRET_KEY
2. 创建自己的后端服务作为 Dify API 的代理
3. 从移动应用调用您的后端 API，再由后端调用 Dify API
4. 在移动应用和后端之间实现适当的认证机制 