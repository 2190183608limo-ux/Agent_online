# AI Compare Server

多平台 AI 对比服务器 —— 自动化向多个 AI 平台提问并抓取回答，支持知识库缓存、对话历史和 Google Drive 云存储。

## 功能特性

- **多平台支持**：DeepSeek、ChatGPT、Claude、Gemini、通义千问、豆包、元宝、Mimo 共 8 个 AI 平台
- **自动化操作**：通过 Puppeteer 自动打开浏览器、发送问题、等待回答
- **知识库缓存**：已问过的问题优先从本地知识库返回，避免重复请求
- **相似问题匹配**：使用 Jaccard + 余弦相似度算法，匹配近似问题
- **对话管理**：支持多轮对话、历史记录查看、对话删除
- **Google Drive 存储**：自动检测 Google Drive 桌面端，支持云端存储对话和知识库
- **钩子系统**：可扩展的事件钩子机制，支持自定义回调
- **安全机制**：API 密钥认证、速率限制、请求校验

## 快速开始

### 前置条件

- Node.js >= 16
- Google Chrome 浏览器（用于 Puppeteer 自动化）
- npm 或 yarn

### 安装

```bash
git clone https://github.com/your-username/ai-compare-server.git
cd ai-compare-server
npm install
```

### 配置

```bash
# 复制配置模板
cp .env.example .env

# 编辑 .env，按需修改配置
```

### 启动

```bash
npm start
```

服务启动后访问 http://localhost:3000

## 项目结构

```
ai-compare-server/
├── server.js              # Express 主服务器，所有 API 路由
├── browser.js             # Puppeteer 浏览器自动化
├── database.js            # JSON 文件存储（对话/回答）
├── knowledge.js           # 知识库管理（相似度匹配）
├── answer-service.js      # 回答服务（协调知识库与网页 AI）
├── index-service.js       # Worker Thread 知识搜索
├── knowledge-index-worker.js  # 知识库索引 Worker
├── security.js            # API 认证、速率限制
├── google-auth.js         # Google Drive 认证
├── hooks.js               # 事件钩子系统
├── logger.js              # 日志工具
├── utils.js               # 通用工具函数
├── validator.js           # 请求参数校验
├── platforms.json         # AI 平台配置（选择器、URL 等）
├── public/
│   └── index.html         # 前端单页面应用
├── .env.example           # 环境变量模板
└── package.json
```

## API 接口

### 提问

```
POST /api/ask
Content-Type: application/json

{
  "question": "你的问题",
  "platform": "deepseek"
}
```

### 查询知识库（索引匹配）

```
POST /api/index
Content-Type: application/json

{
  "question": "你的问题",
  "platform": "deepseek"
}
```

### 采纳索引结果

```
POST /api/accept
Content-Type: application/json

{
  "question_id": 123,
  "platform": "deepseek"
}
```

### 重新提问（跳过知识库）

```
POST /api/ask-retry
Content-Type: application/json

{
  "question": "你的问题",
  "platform": "deepseek"
}
```

### 获取对话列表

```
GET /api/conversations?limit=20
```

### 删除对话

```
DELETE /api/conversations/:id
```

### 获取统计信息

```
GET /api/stats
```

### 存储模式管理

```
GET  /api/storage/mode          # 查看当前存储模式
POST /api/storage/mode          # 切换存储模式

{
  "target": "all",              # conversations | knowledge | all
  "mode": "google_drive",       # local | google_drive
  "googlePath": "G:\\My Drive\\ai-compare-data"
}
```

### 平台管理

```
GET    /api/platforms                     # 获取所有平台
POST   /api/platforms                     # 添加平台
DELETE /api/platforms/:id                 # 删除平台
POST   /api/platforms/:id/test-open       # 测试打开平台
```

## 平台配置

平台配置保存在 `platforms.json` 中，每个平台需要以下选择器：

| 字段 | 说明 |
|------|------|
| `inputSelector` | 输入框的 CSS 选择器 |
| `responseSelector` | 回答内容的 CSS 选择器 |
| `loadingSelector` | 加载状态的选择器（可选） |
| `stopSelector` | 停止生成按钮的选择器（可选） |
| `submitSelector` | 发送按钮的选择器（可选） |
| `newConversationSelector` | 新建对话按钮的选择器（可选） |

### 添加新平台

通过 API 动态添加：

```bash
curl -X POST http://localhost:3000/api/platforms \
  -H "Content-Type: application/json" \
  -d '{
    "id": "my-platform",
    "name": "My AI",
    "url": "https://example.com/chat",
    "inputSelector": "textarea",
    "responseSelector": ".assistant-message"
  }'
```

或直接编辑 `platforms.json` 后重启服务器。

## 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `PORT` | `3000` | 服务器端口 |
| `API_KEYS` | 空 | API 密钥（逗号分隔） |
| `DISABLE_AUTH` | `true` | 禁用认证 |
| `KB_THRESHOLD` | `0.65` | 知识库匹配阈值 |
| `KB_STALE_DAYS` | `45` | 知识库过期天数 |
| `BROWSER_RESPONSE_TIMEOUT_MS` | `180000` | 浏览器响应超时 |
| `BROWSER_STABLE_POLLS` | `8` | 稳定轮询次数 |
| `BROWSER_POLL_INTERVAL_MS` | `1200` | 轮询间隔（毫秒） |
| `GOOGLE_DRIVE_DATA_DIR` | 自动检测 | 对话存储路径 |
| `GOOGLE_DRIVE_KB_DIR` | 自动检测 | 知识库存储路径 |

完整配置请参考 `.env.example`。

## Google Drive 存储

1. 安装 [Google Drive 桌面客户端](https://www.google.com/drive/download/)
2. 在 `.env` 中设置 `GOOGLE_DRIVE_DATA_DIR` 和 `GOOGLE_DRIVE_KB_DIR` 指向 Google Drive 挂载路径
3. 重启服务器，数据将自动存储到 Google Drive

## Agent 配置指南

本项目可作为 AI Agent 的工具服务使用：

### 作为 MCP 工具

在 Agent 的 MCP 配置中添加本服务：

```json
{
  "mcpServers": {
    "ai-compare": {
      "url": "http://localhost:3000",
      "description": "多平台 AI 对比服务"
    }
  }
}
```

### 典型 Agent 调用流程

```
1. POST /api/index     → 先查知识库是否有相似问题
2. 如果命中 → 直接返回缓存答案
3. 如果未命中 → POST /api/ask → 自动向网页 AI 提问
4. POST /api/accept    → 用户确认采纳后存入知识库
```

### Agent 集成示例

```javascript
// 查询知识库
const indexRes = await fetch('http://localhost:3000/api/index', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ question: '什么是机器学习？', platform: 'deepseek' })
});
const { found, result } = await indexRes.json();

if (found && result.usable) {
  // 知识库命中，直接使用
  console.log(result.bestMatch.answer);
} else {
  // 知识库未命中，发送到网页 AI
  const askRes = await fetch('http://localhost:3000/api/ask', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ question: '什么是机器学习？', platform: 'deepseek' })
  });
  const data = await askRes.json();
  console.log(data.response);
}
```

## 安全说明

- 请勿将 `.env`、`credentials.json`、`service-account.json` 等文件提交到版本控制
- 生产环境建议启用 API 密钥认证（设置 `DISABLE_AUTH=false`）
- 浏览器数据目录 `browser-data/` 包含登录状态，请妥善保管

## License

MIT
