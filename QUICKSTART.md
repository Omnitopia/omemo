# 🧠 Omni Memory 极简启动指南

> 5 步上手，给你的 LLM 加上长期记忆！

---

## 这是什么？

Omni Memory 是一个 **API 代理服务**，架在你的应用和 LLM 之间，让模型拥有长期记忆能力。兼容 OpenAI API 格式，换个地址就能用。

```
你的应用 → omemo（自动注入/提取记忆）→ 你的 LLM
```

---

## 第一步：克隆项目

```bash
git clone https://github.com/OmniDimen/omemo.git
cd omemo
```

## 第二步：安装依赖

```bash
pip install -r requirements.txt
```

> 如果遇到 SSL 证书报错，加上：`--trusted-host pypi.org --trusted-host files.pythonhosted.org`

## 第三步：启动服务

```bash
python main.py
```

看到以下输出说明启动成功：

```
🚀 Omni Memory 启动成功
📁 数据目录: ./data
🧠 记忆模式: builtin
💉 注入模式: full
INFO:     Uvicorn running on http://0.0.0.0:8080
```

> 💡 想修改端口？在项目根目录创建 `.env` 文件，写入 `PORT=8082`（或你想要的端口号）

## 第四步：打开 Web UI 配置

浏览器访问 **http://localhost:8080**（如果改了端口则访问对应端口）

### 4.1 添加 API 端点

点击 **「添加端点」**，填写你的 LLM 信息：

| 字段 | 示例 |
|---|---|
| 名称 | `my-llm` |
| 提供商 | `openai`（大多数本地模型选这个） |
| API URL | `http://localhost:1234/v1`（你的模型地址） |
| API Key | `any-key`（本地模型随便填） |
| 模型列表 | `qwen3-8b`（你的模型 ID，用逗号分隔多个） |

### 4.2 确认记忆设置

切换到 **「记忆设置」** 页面，默认配置即可：

- ✅ 记忆方式：**内置记忆**（模型自动记忆）
- ✅ 注入方式：**全量注入**（所有记忆都注入）

点 **「保存设置」**。

## 第五步：开始使用

将你的应用的 API 地址改为 omemo 的地址即可：

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8080/v1",  # 指向 omemo
    api_key="any-key"
)

response = client.chat.completions.create(
    model="qwen3-8b",  # 你配置的模型名
    messages=[{"role": "user", "content": "你好，我叫小明！"}]
)

print(response.choices[0].message.content)
```

模型会记住你说过的话，下一次对话自动带上之前的记忆 🎉

---

## 📋 常用操作

| 操作 | 方法 |
|---|---|
| 查看记忆 | Web UI → 记忆管理 |
| 手动添加记忆 | Web UI → 添加记忆 |
| 搜索记忆 | Web UI → 搜索框 |
| 删除记忆 | Web UI → 点击删除按钮 |

---

## 📂 项目结构一览

```
omemo/
├── main.py              # 主程序入口
├── config.py            # 配置管理
├── models.py            # 数据模型定义
├── requirements.txt     # Python 依赖
├── .env                 # 环境变量（自行创建）
├── config/              # 配置文件（启动后自动生成）
│   ├── endpoints.json   # API 端点配置
│   └── memory_settings.json  # 记忆设置
├── data/                # 数据存储（自动生成）
│   └── memories.json    # 记忆数据
├── memory/              # 记忆核心模块
├── api/                 # API 适配器
├── static/              # Web UI 静态资源
└── templates/           # Web UI 模板
```

---

*Apache License 2.0 · [OmniDimen/omemo](https://github.com/OmniDimen/omemo)*
