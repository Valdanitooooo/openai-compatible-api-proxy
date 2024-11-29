# Openai compatible API proxy

## Introduce

Some OpenAI clients or third-party applications are unable to change the `base_url` or `model` parameters, thus unable to use local LLMs. 
I used the OpenResty proxy model's OpenAI-compatible RESTful API to solve this problem.

## Preconditions

You have deployed the local model and provided an OpenAI compatible RESTful API, Perhaps you will use tools such as vllm, Xinference, Ollama, SGLang, etc.

## Usage

### 1. Provide necessary environment variables

Create an `.env` file and configure it like the `.env_example` file

### 2. Run

```
docker compose up -d
```

### 3. Call your model like calling the OpenAI API

#### Chat

API docs: https://platform.openai.com/docs/api-reference/chat/create

```
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

#### Vision

API docs: https://platform.openai.com/docs/guides/vision

```
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


#### Embeddings

API docs: https://platform.openai.com/docs/api-reference/embeddings/create

```
curl http://localhost:9999/v1/embeddings \
  -H "Authorization: Bearer xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "input": "The food was delicious and the waiter...",
    "model": "text-embedding-ada-002",
    "encoding_format": "float"
  }'
```