# Enhanced Gemini-to-Claude API Proxy v2.5.0

这是一个将 Google Gemini API 转换为 Claude API 格式的代理服务，支持 x-api-key 头部身份验证功能。

## 功能特性

- 🔄 将 Gemini API 转换为 Claude API 格式
- 🔐 可选的 x-api-key 头部身份验证
- 🛠️ 工具调用支持
- 📡 流式响应支持
- 🔧 自动错误处理和重试机制

## 环境变量配置

### 必需的环境变量

```bash
# Google Gemini API 密钥
GEMINI_API_KEY=AIza...
```

### 可选环境变量

```bash
# 身份验证密钥（可选）
AUTH_TOKEN=your-auth-token

# Gemini API Base URL（可选）
GEMINI_BASE_URL=https://your-custom-gemini-endpoint.com

# 模型配置
BIG_MODEL=gemini-2.5-pro
SMALL_MODEL=gemini-2.5-pro

# 服务器配置
HOST=0.0.0.0
PORT=8082
LOG_LEVEL=WARNING

# 请求配置
MAX_TOKENS_LIMIT=8192
REQUEST_TIMEOUT=90
MAX_RETRIES=2
MAX_STREAMING_RETRIES=12

# 流式响应配置
FORCE_DISABLE_STREAMING=false
EMERGENCY_DISABLE_STREAMING=false

# 调试选项
DEBUG_REQUESTS=false
LITELLM_DEBUG=false
```

## 身份验证

如果设置了 `AUTH_TOKEN` 环境变量，所有 API 端点都需要有效的 API key。在请求头中包含：

```bash
x-api-key: your-auth-token
```

如果未设置 `AUTH_TOKEN`，则跳过身份验证。

## 启动服务

```bash
# 安装依赖
pip install fastapi uvicorn litellm python-dotenv

# 设置环境变量
export GEMINI_API_KEY="your-gemini-api-key"
export AUTH_TOKEN="your-auth-token"  # 可选
export GEMINI_BASE_URL="https://your-custom-endpoint.com"  # 可选

# 启动服务
python server.py

# 或使用 uvicorn
uvicorn server:app --host 0.0.0.0 --port 8082
```

## API 端点

### 消息端点
```bash
POST /v1/messages
x-api-key: your-auth-token  # 如果设置了AUTH_TOKEN
Content-Type: application/json

{
  "model": "claude-3-sonnet-20240229",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user", 
      "content": "Hello!"
    }
  ]
}
```

### Token 计数
```bash
POST /v1/messages/count_tokens
x-api-key: your-auth-token  # 如果设置了AUTH_TOKEN
```

### 健康检查
```bash
GET /health
# 无需身份验证
```

## 使用示例

### cURL 示例
```bash
curl -X POST "http://localhost:8082/v1/messages" \
  -H "x-api-key: your-auth-token" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-3-sonnet-20240229",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### Python 示例
```python
import requests

headers = {
    "x-api-key": "your-auth-token",
    "Content-Type": "application/json"
}

data = {
    "model": "claude-3-sonnet-20240229", 
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello!"}]
}

response = requests.post(
    "http://localhost:8082/v1/messages",
    headers=headers,
    json=data
)

print(response.json())
```

## 错误处理

如果 API key 验证失败，服务会返回：

```json
{
  "type": "error",
  "error": {
    "type": "authentication_error",
    "message": "Invalid API key. Please check your x-api-key header."
  }
}
```

## 安全说明

- AUTH_TOKEN 仅用于身份验证，不会发送到任何外部服务
- 实际的 API 调用使用配置的 Gemini API key
- 如果未设置 AUTH_TOKEN，身份验证将被禁用
- 所有敏感信息都从日志中过滤
- 支持安全的环境变量配置

## 故障排除

1. **API key 错误**: 确保 `AUTH_TOKEN` 环境变量正确设置（如果需要认证）
2. **连接错误**: 检查 `GEMINI_API_KEY` 和网络连接
3. **流式响应问题**: 可以设置 `FORCE_DISABLE_STREAMING=true` 禁用流式响应
4. **认证问题**: 如果不需要认证，可以不设置 `AUTH_TOKEN` 环境变量
5. **自定义端点问题**: 如果使用自定义Gemini端点，确保 `GEMINI_BASE_URL` 设置正确

## 调试功能

### 启用详细调试

```bash
# 启用请求/响应详细日志
export DEBUG_REQUESTS=true

# 启用LiteLLM内部调试
export LITELLM_DEBUG=true

# 重启服务查看详细日志
python server.py
```

### 调试信息说明

- **DEBUG_REQUESTS=true**: 显示发送到Gemini的完整请求参数和响应信息
- **LITELLM_DEBUG=true**: 启用LiteLLM库的内部调试，显示HTTP请求详情
- 调试日志会显示实际的API端点URL、参数、模型映射等信息
- 自动检测并显示认证方式：`x-goog-api-key`（Gemini原生）或 `Authorization Bearer`（OpenAI风格）

### 认证方式说明

该代理服务会自动为不同的API使用正确的认证方式：
- **Gemini模型**: 使用 `x-goog-api-key` 头部（Google原生认证）
- **其他模型**: 使用 `Authorization: Bearer` 头部（OpenAI风格认证）

这确保了与各种API端点的兼容性，包括官方API和第三方代理服务。