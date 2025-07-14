# ✅ Function / Call 报文协议详解（Function Calling Protocol）

所谓 Function/Call 报文协议，通常指的是大语言模型（LLM）调用外部函数、插件或工具时使用的一种结构化 JSON 报文协议。该协议被 OpenAI（Function Calling / Tool Calling）、DeepSeek、Claude 等广泛采用，用于 描述函数定义、传参、响应、调用结果等内容。

---

## 📦 1. 报文结构总览
一个完整的 Function Calling 报文包含以下几个部分：
```json
{
  "messages": [...],        // 对话历史 + 用户请求
  "tools": [...],           // 函数（工具）定义
  "tool_choice": "auto",    // 是否自动调用某个函数
  "model": "gpt-4-0613",    // 使用的模型名称
  "temperature": 0.7,
  "stream": false
}
```
## 🧩 2. 各字段说明
### ✅ messages：对话历史与请求体
```json
[
  {
    "role": "user",
    "content": "请帮我添加两个数：3 和 5"
  }
]
```

### ✅ tools：函数定义（描述可被调用的函数）
```json
[
  {
    "type": "function",
    "function": {
      "name": "add",
      "description": "对两个整数求和",
      "parameters": {
        "type": "object",
        "properties": {
          "a": {
            "type": "integer",
            "description": "第一个数"
          },
          "b": {
            "type": "integer",
            "description": "第二个数"
          }
        },
        "required": ["a", "b"]
      }
    }
  }
]
```

### ✅ tool_choice（可选）：
```json
"tool_choice": "auto" // 表示让模型自动选择是否调用函数
```
也可以指定调用哪个函数：
```json
"tool_choice": {
  "type": "function",
  "function": { "name": "add" }
}
```

### 🧠 3. 模型调用响应格式
模型识别到需要调用函数时，会返回一条特殊类型的消息：

```json
{
  "role": "assistant",
  "content": null,
  "tool_calls": [
    {
      "id": "call_add_123",
      "type": "function",
      "function": {
        "name": "add",
        "arguments": "{ \"a\": 3, \"b\": 5 }"
      }
    }
  ]
}
```
你需要 根据 tool_calls 字段执行对应函数，并将执行结果继续发送回去。

## 🔁 4. 用户函数执行完成后，返回执行结果
将函数的调用结果作为一个消息（角色为 tool）发回：
```json
{
  "role": "tool",
  "tool_call_id": "call_add_123",
  "content": "结果是 8"
}
```
然后模型才会基于调用结果生成完整的回答。

# ✅ 总结：Function Call 报文结构流转图
```csharp
用户提问 → 模型识别需要调用函数
        → 返回调用请求（tool_calls）
              ↓
      客户端执行函数（add）
              ↓
     客户端把结果作为 `tool` 发回
              ↓
      模型生成最终回答

```

# ✅ 使用场景
- 🔌 Agent 调用插件系统（搜索 / 天气 / 数据库）
- 📱 LLM 接第三方服务（支付 / 数据分析 / 智能家居）
- 🧠 多智能体协作调用
- 🛠️ 像 LangChain / Autogen / Dify 等工具的调用协议都基于此