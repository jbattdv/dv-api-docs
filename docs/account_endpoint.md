# DataVertex Account API

## Overview

The DataVertex Account API returns the current credit balance for your account. Use it to check your remaining credits and total credits purchased at any time.

**Endpoint:** `GET https://api.data-vertex.com/v1/account`

**Credit Cost:** 0 credits

---

## Authentication

All API requests must include your API key in the request headers:

```
x-api-key: YOUR_API_KEY
```

[Schedule a quick call](https://calendar.notion.so/meet/claire-liao/datavertex) to get API access and explore how DataVertex can support your recruiting product development.

---

## Request Format

### Headers

| Header | Value | Required |
|--------|-------|----------|
| `x-api-key` | Your API key | Yes |

This endpoint takes no request body.

---

## Response Format

### Success Response (200 OK)

```json
{
  "success": true,
  "credits_remaining": 500,
  "credits_purchased": 1000,
  "timestamp": "2026-06-01T10:00:00.000000"
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether the request succeeded |
| `credits_remaining` | integer | Credits currently available in your account |
| `credits_purchased` | integer | Total credits purchased across your account lifetime |
| `timestamp` | string | ISO timestamp of when the response was generated |

---

## Code Examples

### cURL

```bash
curl -X GET https://api.data-vertex.com/v1/account \
  -H "x-api-key: YOUR_API_KEY"
```

### Python

```python
import requests

url = "https://api.data-vertex.com/v1/account"

headers = {
    "x-api-key": "YOUR_API_KEY"
}

response = requests.get(url, headers=headers)

if response.status_code == 200:
    data = response.json()
    print(f"Credits remaining:  {data['credits_remaining']}")
    print(f"Credits purchased:  {data['credits_purchased']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

### JavaScript (Node.js)

```javascript
const axios = require('axios');

const getAccountInfo = async () => {
  try {
    const response = await axios.get(
      'https://api.data-vertex.com/v1/account',
      {
        headers: {
          'x-api-key': 'YOUR_API_KEY'
        }
      }
    );

    const { credits_remaining, credits_purchased } = response.data;
    console.log(`Credits remaining:  ${credits_remaining}`);
    console.log(`Credits purchased:  ${credits_purchased}`);

    return response.data;
  } catch (error) {
    console.error('Error:', error.response?.data || error.message);
  }
};

getAccountInfo();
```

---

## Error Responses

### 403 Forbidden

Invalid or unrecognized API key:

```json
{
  "success": false,
  "message": "Invalid or inactive API key."
}
```

### 500 Internal Server Error

Server error:

```json
{
  "success": false,
  "message": "Internal server error occurred."
}
```

---

## Error Codes Summary

| Status Code | Meaning | Common Causes |
|-------------|---------|---------------|
| **200** | OK | Request successful, balance returned |
| **403** | Forbidden | Missing or invalid API key |
| **500** | Internal Server Error | Unexpected server error |

---

## Best Practices

### 1. Check Credits Before Expensive Operations

Poll the account endpoint before running large search or lookup jobs to confirm you have sufficient credits:

```python
import requests

def get_credits_remaining(api_key):
    response = requests.get(
        "https://api.data-vertex.com/v1/account",
        headers={"x-api-key": api_key}
    )
    data = response.json()
    return data["credits_remaining"]

credits = get_credits_remaining("YOUR_API_KEY")

if credits < 100:
    print(f"Warning: Low credit balance ({credits} remaining).")
else:
    print(f"Credits available: {credits}. Proceeding with job.")
```

### 2. Monitor Credit Consumption Over Time

Compare `credits_remaining` against `credits_purchased` to track overall consumption:

```python
data = response.json()
credits_used = data["credits_purchased"] - data["credits_remaining"]
pct_used = (credits_used / data["credits_purchased"]) * 100 if data["credits_purchased"] > 0 else 0

print(f"Credits used:      {credits_used}")
print(f"Credits remaining: {data['credits_remaining']}")
print(f"Credits purchased: {data['credits_purchased']}")
print(f"Usage:             {pct_used:.1f}%")
```

---

## Need Help?

- **Support:** dev@data-vertex.com
- **Website:** https://www.data-vertex.com

[Schedule a quick call](https://calendar.notion.so/meet/claire-liao/datavertex) to get API access and explore how DataVertex can support your recruiting product development.

---

*Last Updated: June 1, 2026*