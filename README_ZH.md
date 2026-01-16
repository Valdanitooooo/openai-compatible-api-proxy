[English](README.md) | [中文简体](README.zh-CN.md)

# OpenAI 兼容 API 代理（OpenResty）

## 介绍

部分 OpenAI 客户端或第三方应用无法自定义 `base_url` 或 `model` 参数，因此难以接入本地部署的大模型。

本项目基于 **OpenResty** 实现了一个 OpenAI 兼容的 RESTful API 代理层，使本地模型可以“像 OpenAI 一样被调用”，无需修改客户端代码即可适配主流工具和应用。

---

## 前置条件

你已经部署了本地大模型服务，并提供了 **OpenAI 兼容的 RESTful API**，例如基于：

* vLLM
* Xinference
* Ollama
* SGLang
* 其他兼容 OpenAI 接口的推理框架

---

## 使用方法

### 1. 修改配置

按实际情况修改nginx.conf


---

### 2. 启动服务

```bash
docker compose up -d
```

启动后，OpenResty 代理服务默认监听：

```
http://localhost:9999
```

---

### 3. 像调用 OpenAI 一样调用你的本地模型

#### Chat（对话）

API 文档：[https://platform.openai.com/docs/api-reference/chat/create](https://platform.openai.com/docs/api-reference/chat/create)

```bash
curl http://localhost:9999/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer xxx" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello!"
      }
    ]
  }'
```

---

#### Vision（视觉）

API 文档：[https://platform.openai.com/docs/guides/vision](https://platform.openai.com/docs/guides/vision)

```bash
curl http://localhost:9999/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer xxx" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [
      {
        "role": "user",
        "content": [
          {
            "type": "text",
            "text": "What’s in this image?"
          },
          {
            "type": "image_url",
            "image_url": {
              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
            }
          }
        ]
      }
    ],
    "max_tokens": 300
  }'
```

---

#### Embeddings（向量）

API 文档：[https://platform.openai.com/docs/api-reference/embeddings/create](https://platform.openai.com/docs/api-reference/embeddings/create)

```bash
curl http://localhost:9999/v1/embeddings \
  -H "Authorization: Bearer xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "input": "The food was delicious and the waiter...",
    "model": "text-embedding-ada-002",
    "encoding_format": "float"
  }'
```

---

#### 认证方式

使用 `nginx_auth.conf` 进行鉴权配置。

API 文档：[https://platform.openai.com/docs/api-reference/authentication](https://platform.openai.com/docs/api-reference/authentication)

```bash
curl http://localhost:9999/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"
```

---

## 4. 生产环境修改 Nginx 配置（热重载）

在生产环境中修改 OpenResty / Nginx 配置时：

**无需重启容器，也不会中断正在处理的 API 请求。**

修改配置文件后，执行：

```bash
docker compose exec openresty nginx -s reload
```

该命令会执行**优雅重载（graceful reload）**，效果如下：

* 已建立的 API 请求**继续正常执行**
* 新请求将使用**更新后的配置**
* **零停机、零抖动、零中断**

这是推荐的生产运维方式。

