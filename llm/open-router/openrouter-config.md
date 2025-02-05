# Open Router config

Open router api

```bash
curl --location 'https://openrouter.ai/api/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer ****************' \
--data '{
    "model": "deepseek/deepseek-r1:free",
    "messages": [
        {
            "role": "user",
            "content": "What is the meaning of life?"
        }
    ]
}'
```