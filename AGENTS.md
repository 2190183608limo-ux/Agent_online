# AI Compare Server - 开发规范

## 项目概述

这是一个基于 Express + Puppeteer 的多平台 AI 对比服务器，使用 CommonJS 模块系统。

## 代码规范

- **模块系统**：CommonJS（`require` / `module.exports`），不要使用 ESM
- **命名**：文件名使用 kebab-case，变量/函数使用 camelShot，类使用 PascalCase
- **注释**：中文注释即可，关键逻辑处必须加注释
- **错误处理**：所有异步操作必须有 try/catch，不允许未捕获的 Promise 拒绝
- **日志**：统一使用 `./logger.js`，不要直接 `console.log`

## 文件职责

| 文件 | 职责 |
|------|------|
| `server.js` | Express 路由定义、启动流程、钩子注册 |
| `browser.js` | Puppeteer 浏览器生命周期、页面操作、回答检测 |
| `database.js` | 对话/回答的 JSON 持久化存储 |
| `knowledge.js` | 知识库 CRUD、相似度计算、缓存 |
| `answer-service.js` | 编排层：知识库 → 网页 AI 的查询流程 |
| `index-service.js` | Worker Thread 知识搜索（带超时） |
| `security.js` | API 认证中间件、速率限制 |
| `hooks.js` | 事件钩子系统（发布/订阅模式） |
| `validator.js` | 请求参数校验中间件 |
| `platforms.json` | AI 平台 CSS 选择器配置 |

## 添加新 AI 平台

1. 在 `platforms.json` 中添加平台配置（需要正确的 CSS 选择器）
2. 如果平台有特殊交互逻辑（如验证码处理），在 `browser.js` 中添加
3. 重启服务器即可生效

## 浏览器自动化注意事项

- 使用 `puppeteer-extra-plugin-stealth` 降低被检测风险
- 输入使用 `nativeInputValueSetter` 兼容 React 框架
- 回答检测依赖 DOM 轮询（`STABLE_POLLS` + `isGrowing` 判断）
- 所有页面操作加超时保护，避免永久等待

## 数据存储

- **本地模式**：数据保存在 `data/` 和 `knowledge/` 目录
- **Google Drive 模式**：自动检测桌面端挂载路径，或通过环境变量指定
- 使用写入临时文件 + rename 的方式保证数据完整性

## 测试

```bash
# 语法检查
npm run check

# 启动服务器
npm start
```

## 提交规范

- 提交信息用中文或英文均可，但要清晰描述改动
- 不要提交 `.env`、浏览器数据、对话记录等敏感内容
- 新增 API 时同步更新 README
