# vLLM API Documentation

## Overview

This document provides comprehensive API documentation for our vLLM deployment. The service provides OpenAI-compatible endpoints for text generation using the `facebook/opt-125m` language model.

**Base URL**: `https://vllm11.byteshop.in/`

## Authentication

All API requests require authentication using a Bearer token in the Authorization header:

```bash
Authorization: Bearer your-vllm-api-key
```

## Available Endpoints

### 1. Health Check

Check if the service is running and healthy.

**Endpoint**: `GET /health`

**Example**:
```bash
curl https://vllm11.byteshop.in/health
```

**Response**: HTTP 200 OK (empty body)

---

### 2. List Available Models

Get information about available models.

**Endpoint**: `GET /v1/models`

**Headers**:
- `Authorization: Bearer your-vllm-api-key`

**Example**:
```bash
curl https://vllm11.byteshop.in/v1/models \
  -H "Authorization: Bearer your-vllm-api-key"
```

**Response**:
```json
{
  "object": "list",
  "data": [
    {
      "id": "facebook/opt-125m",
      "object": "model",
      "created": 1754367982,
      "owned_by": "vllm",
      "root": "facebook/opt-125m",
      "parent": null,
      "max_model_len": 4096,
      "permission": [
        {
          "id": "modelperm-81e27390dbb344919c71db46150da046",
          "object": "model_permission",
          "created": 1754367982,
          "allow_create_engine": false,
          "allow_sampling": true,
          "allow_logprobs": true,
          "allow_search_indices": false,
          "allow_view": true,
          "allow_fine_tuning": false,
          "organization": "*",
          "group": null,
          "is_blocking": false
        }
      ]
    }
  ]
}
```

---

### 3. Text Completions

Generate text completions using the model.

**Endpoint**: `POST /v1/completions`

**Headers**:
- `Content-Type: application/json`
- `Authorization: Bearer your-vllm-api-key`

**Request Body Parameters**:
- `model` (string, required): Model name (`facebook/opt-125m`)
- `prompt` (string, required): The input text to complete
- `max_tokens` (integer, optional): Maximum number of tokens to generate (default: 16)
- `temperature` (float, optional): Sampling temperature (0.0 to 2.0, default: 1.0)
- `top_p` (float, optional): Nucleus sampling parameter (0.0 to 1.0, default: 1.0)
- `n` (integer, optional): Number of completions to generate (default: 1)
- `stop` (string or array, optional): Stop sequences
- `presence_penalty` (float, optional): Presence penalty (-2.0 to 2.0, default: 0.0)
- `frequency_penalty` (float, optional): Frequency penalty (-2.0 to 2.0, default: 0.0)

**Example**:
```bash
curl https://vllm11.byteshop.in/v1/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-vllm-api-key" \
  -d '{
    "model": "facebook/opt-125m",
    "prompt": "The future of artificial intelligence is",
    "max_tokens": 100,
    "temperature": 0.7,
    "top_p": 0.9
  }'
```

**Response**:
```json
{
  "id": "cmpl-dc5232bed50946b8a6787ce8d46ba0bd",
  "object": "text_completion",
  "created": 1754368037,
  "model": "facebook/opt-125m",
  "choices": [
    {
      "index": 0,
      "text": " bright and full of possibilities. We are seeing rapid advancements in machine learning...",
      "logprobs": null,
      "finish_reason": "stop",
      "stop_reason": null,
      "prompt_logprobs": null
    }
  ],
  "service_tier": null,
  "system_fingerprint": null,
  "usage": {
    "prompt_tokens": 7,
    "total_tokens": 57,
    "completion_tokens": 50,
    "prompt_tokens_details": null
  },
  "kv_transfer_params": null
}
```

---

## Programming Language Examples

### Python (OpenAI SDK)

```python
from openai import OpenAI

# Initialize client
client = OpenAI(
    api_key="your-vllm-api-key",
    base_url="https://vllm11.byteshop.in/v1"
)

# Generate completion
response = client.completions.create(
    model="facebook/opt-125m",
    prompt="Write a short story about a robot:",
    max_tokens=150,
    temperature=0.8
)

print(response.choices[0].text)
```

### Python (Requests)

```python
import requests
import json

url = "https://vllm11.byteshop.in/v1/completions"
headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer your-vllm-api-key"
}

data = {
    "model": "facebook/opt-125m",
    "prompt": "Explain quantum computing in simple terms:",
    "max_tokens": 200,
    "temperature": 0.7
}

response = requests.post(url, headers=headers, json=data)
result = response.json()

print(result["choices"][0]["text"])
```

### JavaScript (Node.js)

```javascript
const OpenAI = require('openai');

const client = new OpenAI({
  apiKey: 'your-vllm-api-key',
  baseURL: 'https://vllm11.byteshop.in/v1'
});

async function generateText() {
  try {
    const completion = await client.completions.create({
      model: 'facebook/opt-125m',
      prompt: 'The benefits of renewable energy include:',
      max_tokens: 100,
      temperature: 0.6
    });
    
    console.log(completion.choices[0].text);
  } catch (error) {
    console.error('Error:', error);
  }
}

generateText();
```

### JavaScript (Fetch)

```javascript
async function generateCompletion(prompt) {
  const response = await fetch('https://vllm11.byteshop.in/v1/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer your-vllm-api-key'
    },
    body: JSON.stringify({
      model: 'facebook/opt-125m',
      prompt: prompt,
      max_tokens: 150,
      temperature: 0.7
    })
  });
  
  const data = await response.json();
  return data.choices[0].text;
}

// Usage
generateCompletion("Write a haiku about technology:")
  .then(text => console.log(text))
  .catch(error => console.error('Error:', error));
```

### PHP

```php
<?php
$url = 'https://vllm11.byteshop.in/v1/completions';
$data = [
    'model' => 'facebook/opt-125m',
    'prompt' => 'List three advantages of cloud computing:',
    'max_tokens' => 120,
    'temperature' => 0.7
];

$options = [
    'http' => [
        'header' => [
            'Content-Type: application/json',
            'Authorization: Bearer your-vllm-api-key'
        ],
        'method' => 'POST',
        'content' => json_encode($data)
    ]
];

$context = stream_context_create($options);
$response = file_get_contents($url, false, $context);
$result = json_decode($response, true);

echo $result['choices'][0]['text'];
?>
```

---

## Response Fields Explanation

### Completion Response

- `id`: Unique identifier for the completion
- `object`: Type of object returned (`text_completion`)
- `created`: Unix timestamp of creation
- `model`: Model used for generation
- `choices`: Array of generated completions
  - `index`: Choice index (0-based)
  - `text`: Generated text
  - `finish_reason`: Reason completion finished (`stop`, `length`, etc.)
  - `logprobs`: Log probabilities (if requested)
- `usage`: Token usage statistics
  - `prompt_tokens`: Tokens in the input prompt
  - `completion_tokens`: Tokens in the generated completion
  - `total_tokens`: Total tokens used

---

## Best Practices

### 1. Temperature Settings
- **Low temperature (0.1-0.3)**: More focused, deterministic outputs
- **Medium temperature (0.5-0.8)**: Balanced creativity and coherence
- **High temperature (0.9-1.5)**: More creative but potentially less coherent

### 2. Token Management
- Set appropriate `max_tokens` to control response length
- Monitor token usage for cost optimization
- Use `stop` sequences to terminate generation at specific points

### 3. Prompt Engineering
- Be specific and clear in your prompts
- Provide examples when needed
- Use consistent formatting for better results

### 4. Error Handling
Always implement proper error handling:

```python
try:
    response = client.completions.create(...)
    return response.choices[0].text
except Exception as e:
    print(f"API Error: {e}")
    return None
```

---

## Error Codes

| Status Code | Description |
|-------------|-------------|
| 200 | Success |
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized - Invalid or missing API key |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error |
| 503 | Service Unavailable - Model loading or overloaded |

---

## Rate Limits

- Current rate limits depend on server capacity
- Implement exponential backoff for 429 responses
- Consider batching requests when possible

---

## Support and Troubleshooting

### Common Issues

1. **401 Unauthorized**: Check your API key
2. **Model not found**: Ensure you're using `facebook/opt-125m`
3. **Slow responses**: Model generation takes time, especially for longer outputs
4. **Empty responses**: Check your prompt and parameters

### Getting Help

- Check server logs for detailed error information
- Verify network connectivity to the endpoint
- Test with simple prompts first before complex use cases

---

## Model Information

**Current Model**: `facebook/opt-125m`
- **Type**: Causal language model
- **Parameters**: 125 million
- **Context Length**: 4096 tokens
- **Best Use Cases**: Text completion, creative writing, code generation
- **Limitations**: Smaller model, may produce less coherent long-form text

---

*Last Updated: August 2025*