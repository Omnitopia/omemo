# 🤖 Omni Memory × AstrBot 快速接入指南

> 让你的 AstrBot 机器人拥有长期记忆能力

---

## 前提条件

- AstrBot 已正常运行（本地或 Docker 部署均可）
- Omni Memory 已启动并配置好上游模型端点（参见 QUICKSTART.md）
- 你的 LLM 已在 LM Studio / Ollama / 云端 API 中正常运行

## 架构说明

接入后的请求链路：

```
QQ 用户发消息 → AstrBot → Omni Memory → LLM（如 LM Studio）
                              ↕
                        memories.json（自动记忆）
```

AstrBot 不再直连 LLM，而是通过 Omni Memory 中转，记忆的注入和提取全部自动完成。

---

## 第一步：确认 Omni Memory 地址

Omni Memory 默认运行在 `http://0.0.0.0:8080`。

- 如果 AstrBot 和 Omni Memory 在**同一台机器**上直接运行：地址是 `http://localhost:8080/v1`
- 如果 AstrBot 运行在 **Docker** 中：不能用 localhost，需要用宿主机的局域网 IP，例如 `http://192.168.x.x:8080/v1`
- 如果你修改了端口（比如 .env 里 PORT=8082）：地址改为对应端口

## 第二步：修改 AstrBot 的 LLM 配置

进入 AstrBot 管理后台，找到你的 LLM 提供商配置（通常在"模型提供商"或"Provider"设置中）：

1. 将 **API Base URL** 改为 Omni Memory 的地址，例如：`http://192.168.x.x:8082/v1`
2. **API Key** 保持不变（Omni Memory 会原样转发给上游）
3. **模型名称** 必须与 Omni Memory 端点配置中的模型名一致

> ⚠️ 注意：URL 末尾不要有多余空格，否则会导致 404 错误

## 第三步：清空对话历史

首次接入时，建议在 AstrBot 中**清空该模型的对话历史**再开始测试。

原因：旧的对话历史中可能包含图片数据（如 Playwright 截图等），这些数据会被一并发送到 LLM，如果数据量过大或格式异常，可能导致上游模型报错。

## 第四步：测试

在 QQ 上给机器人发一条包含个人信息的消息，例如：

```
你好，我叫小明，我是一名程序员
```

然后打开 Omni Memory 的 Web UI（如 http://localhost:8082），进入「记忆管理」页面，检查是否自动生成了类似这样的记忆：

```
用户名叫小明
用户是一名程序员
```

如果看到了，说明接入成功 🎉

---

## 已知限制

**单用户记忆**：当前版本的 Omni Memory 所有对话共享一个 memories.json，这意味着不同 QQ 用户与机器人的对话记忆会混在一起。如果你的机器人会和多个用户聊天，需要注意这一点。

**多模态兼容性**：AstrBot 默认使用 OpenAI 多模态消息格式（content 为列表而非字符串），需要确保 Omni Memory 已应用相关兼容性补丁（参见项目 README 或联系维护者）。

---

## 常见问题

**Q: 报 404 Not Found**
A: 检查 AstrBot 填写的 API URL 末尾是否有多余空格，正确格式是 `http://x.x.x.x:8082/v1`

**Q: 报 400 validation errors**
A: 说明 Omni Memory 不支持 AstrBot 发送的消息格式，需要应用多模态兼容补丁

**Q: 报 "failed to process image"**
A: 对话历史中有旧的图片数据导致上游模型报错，清空对话历史即可

**Q: Docker 中 AstrBot 连不上 Omni Memory**
A: Docker 容器内的 localhost 指向容器自身，需要使用宿主机的局域网 IP 地址
