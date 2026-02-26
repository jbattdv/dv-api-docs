# DataVertex Contact Lookup API

## Overview

The DataVertex Contact Lookup API retrieves detailed contact information for specific candidates, including personal emails, phone numbers, and enriched profile data. You can lookup contacts using **Candidate IDs** from search results or **LinkedIn URLs** directly (no search required)

**Endpoint:** `POST https://api.data-vertex.com/v1/contact-lookup`

**Credit Cost:** Variable (3-11 credits per candidate based on enrichments requested)

**Billing:** Success-based - you're only charged for data actually retrieved

**Capacity:** 1 contact per request

**Performance:** For multiple candidates, make concurrent API calls for optimal speed. Each request is processed independently, allowing for parallel execution and faster overall throughput

---

## Authentication

All API requests must include your API key in the request headers:

```
x-api-key: YOUR_API_KEY
```

You can obtain your API key from DataVertex directly or contact dev@data-vertex.com.

---

## Credit Costs

DataVertex uses a flexible, success-based pricing model. You only pay for data that's actually retrieved:

| Enrichment Type | Credit Cost | Parameter |
|-----------------|-------------|-----------|
| **Personal Email** | 3 credits per email | `reveal_personal_email: true` |
| **Mobile Phone** | 6 credits per phone | `reveal_phone: true` |
| **Detailed Profile** | 1 credit per profile | `reveal_detailed_person_enrichment: true` |
| **Healthcare Data** | 1 credit per profile | `reveal_healthcare_enrichment: true` |

### Success-Based Billing Example

If you request email + phone for 5 candidates, but only 3 have emails and 2 have phones:
- **Email charges:** 3 candidates × 3 credits = 9 credits
- **Phone charges:** 2 candidates × 6 credits = 12 credits
- **Total:** 21 credits (not 45 credits)

You're **never charged** for:
- Failed lookups
- Missing data
- Invalid candidate IDs

---

## Request Format

### Headers

| Header | Value | Required |
|--------|-------|----------|
| `x-api-key` | Your API key | Yes |
| `Content-Type` | `application/json` | Yes |

### Body Parameters

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `candidate_ids` | array of strings (1 item) | Single candidate ID from search results | One identifier required |
| `linkedin_urls` | array of strings (1 item) | Single LinkedIn profile URL | One identifier required |
| `reveal_personal_email` | boolean | Retrieve personal email address | No (default: false) |
| `reveal_phone` | boolean | Retrieve mobile phone number | No (default: false) |
| `reveal_detailed_person_enrichment` | boolean | Retrieve skills, experience, education | No (default: false) |
| `reveal_healthcare_enrichment` | boolean | Retrieve NPI, medical license, specialty | No (default: false) |

**Important:** 
- Provide exactly one identifier: either `candidate_ids` with 1 ID OR `linkedin_urls` with 1 URL
- At least one enrichment parameter must be set to `true`
- For multiple candidates, make concurrent API calls

---

## Request Examples

### Option 1: Lookup by Candidate ID (from search results)

Basic Request (Email Only)

```json
{
  "candidate_ids": ["12345"],
  "reveal_personal_email": true,
  "reveal_phone": false,
  "reveal_detailed_person_enrichment": false,
  "reveal_healthcare_enrichment": false
}
```

All Contact Information + Candidate's work history, education, and skills.

```json
{
  "candidate_ids": ["67890"],
  "reveal_personal_email": true,
  "reveal_phone": true,
  "reveal_detailed_person_enrichment": true,
  "reveal_healthcare_enrichment": false
}
```

### Option 2: Lookup by LinkedIn URL

Basic Request (Email Only)

```json
{
  "linkedin_urls": ["https://www.linkedin.com/in/jane-smith-12345"],
  "reveal_personal_email": true,
  "reveal_phone": false,
  "reveal_detailed_person_enrichment": false,
  "reveal_healthcare_enrichment": false
}
```

All Contact Information + Candidate's work history, education, and skills.

```json
{
  "linkedin_urls": ["https://www.linkedin.com/in/john-doe-67890"],
  "reveal_personal_email": true,
  "reveal_phone": true,
  "reveal_detailed_person_enrichment": true,
  "reveal_healthcare_enrichment": false
}
```

---

## Response Format

### Success Response (200 OK)

```json
{
  "success": true,
  "data": {
    "contacts": [
      {
        "lookup_identifier": "12345",
        "lookup_type": "id",
        "success": true,
        "profile": {
          "id": "12345",
          "name": "Jane Smith",
          "current_title": "Software Engineer",
          "current_employer": "TechCorp",
          "personal_email": "jane.smith@gmail.com",
          "phone_number": "+14155551234",
          "skills": ["Python", "JavaScript", "React"],
          "education": [...],
          "experience": [...],
        },
        "contact_data_retrieved": ["email", "phone", "detailed_profile"],
        "retrieved_at": "2025-12-10T12:00:00Z"
      }
    ],
    "summary": {
      "total_requested": 1,
      "successful": 1,
      "failed": 0
    }
  },
  "credits": {
    "used": 10,
    "remaining": 990,
    "billing_note": "Charged for 1 successful lookup only"
  }
}
```

---

## Response Fields

### Contact Object

Each contact in the `contacts` array contains:

Success Case

| Field | Type | Description |
|-------|------|-------------|
| `lookup_identifier` | string | The requested identifier (candidate ID or LinkedIn URL) |
| `lookup_type` | string | Type of lookup: "id" or "linkedin_url" |
| `success` | boolean | Whether lookup succeeded (true) |
| `profile` | object | Enriched profile data (see below) |
| `contact_data_retrieved` | array of strings | Types of data retrieved |
| `retrieved_at` | string | ISO timestamp when data was retrieved |

Failure Case

| Field | Type | Description |
|-------|------|-------------|
| `lookup_identifier` | string | The requested identifier (candidate ID or LinkedIn URL) |
| `lookup_type` | string | Type of lookup: "id" or "linkedin_url" |
| `success` | boolean | Whether lookup succeeded (false) |
| `error` | string | Error description |
| `status_code` | integer | HTTP status code (404, 500, etc.) |

### Profile Object

The profile object contains enriched data based on your requested enrichments:

Basic Fields (Always Included)

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Candidate ID |
| `name` | string | Full name |
| `current_title` | string | Current job title |
| `current_employer` | string | Current company name |

Email Fields (`reveal_personal_email: true`)

| Field | Type | Description |
|-------|------|-------------|
| `personal_email` | string | Personal email address (verified) |

**Multi-Source Data:** DataVertex uses multiple data sources to maximize email retrieval success. If our primary source doesn't have an email, we automatically check secondary sources at no extra cost.

Phone Field (`reveal_phone: true`)

| Field | Type | Description |
|-------|------|-------------|
| `phone_number` | string | Primary/mobile phone number |

Detailed Profile Fields (`reveal_detailed_person_enrichment: true`)

| Field | Type | Description |
|-------|------|-------------|
| `skills` | array | List of skills |
| `experience` | array | Work experience/job history |
| `education` | array | Educational background |
| `job_history` | array | Complete employment history |

Healthcare Fields (`reveal_healthcare_enrichment: true`)

| Field | Type | Description |
|-------|------|-------------|
| `npi_data` | object | National Provider Identifier data |

The `npi_data` object contains:
- `npi_number`: National Provider Identifier
- `license_number`: Medical license number
- `license_state`: State where licensed
- `specialty`: Medical specialty
- `credentials`: Professional credentials (MD, RN, etc.)

### Contact Data Retrieved Array

Indicates what data types were successfully retrieved:

| Value | Description |
|-------|-------------|
| `"email"` | Personal email was found |
| `"phone"` | Phone number was found |
| `"detailed_profile"` | Skills/experience/education data was found |
| `"healthcare_profile"` | Healthcare/NPI data was found |

### Summary Object

| Field | Type | Description |
|-------|------|-------------|
| `total_requested` | integer | Total candidates requested |
| `successful` | integer | Number of successful lookups |
| `failed` | integer | Number of failed lookups |

### Credits Object

| Field | Type | Description |
|-------|------|-------------|
| `used` | integer | Credits charged for this request |
| `remaining` | integer | Your remaining credit balance |
| `billing_note` | string | Explanation of charges |

---

## Code Examples

### cURL

```bash
curl -X POST https://api.data-vertex.com/v1/contact-lookup \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "candidate_ids": ["12345"],
    "reveal_personal_email": true,
    "reveal_phone": true,
    "reveal_detailed_person_enrichment": false,
    "reveal_healthcare_enrichment": false
  }'
```

### Python

```python
import requests

url = "https://api.data-vertex.com/v1/contact-lookup"

headers = {
    "x-api-key": "YOUR_API_KEY",
    "Content-Type": "application/json"
}

payload = {
    "candidate_ids": ["12345"],
    "reveal_personal_email": True,
    "reveal_phone": True,
    "reveal_detailed_person_enrichment": False,
    "reveal_healthcare_enrichment": False
}

response = requests.post(url, headers=headers, json=payload)

if response.status_code == 200:
    data = response.json()
    
    for contact in data['data']['contacts']:
        if contact['success']:
            profile = contact['profile']
            print(f"Name: {profile['name']}")
            print(f"Email: {profile.get('personal_email', 'N/A')}")
            print(f"Phone: {profile.get('phone_number', 'N/A')}")
            print(f"Retrieved: {', '.join(contact['contact_data_retrieved'])}")
            print("---")
    
    print(f"\nCredits used: {data['credits']['used']}")
    print(f"Credits remaining: {data['credits']['remaining']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

### JavaScript (Node.js)

```javascript
const axios = require('axios');

const lookupContact = async (candidateId) => {
  try {
    const response = await axios.post(
      'https://api.data-vertex.com/v1/contact-lookup',
      {
        candidate_ids: [candidateId],
        reveal_personal_email: true,
        reveal_phone: true,
        reveal_detailed_person_enrichment: false,
        reveal_healthcare_enrichment: false
      },
      {
        headers: {
          'x-api-key': 'YOUR_API_KEY',
          'Content-Type': 'application/json'
        }
      }
    );

    const { contacts, summary } = response.data.data;
    
    contacts.forEach(contact => {
      if (contact.success) {
        const { profile, contact_data_retrieved } = contact;
        console.log(`Name: ${profile.name}`);
        console.log(`Email: ${profile.personal_email || 'N/A'}`);
        console.log(`Phone: ${profile.phone_number || 'N/A'}`);
        console.log(`Retrieved: ${contact_data_retrieved.join(', ')}`);
        console.log('---');
      } else {
        console.log(`Failed: ${contact.lookup_identifier} - ${contact.error}`);
      }
    });

    console.log(`Credits used: ${response.data.credits.used}`);
    
    return response.data;
  } catch (error) {
    console.error('Error:', error.response?.data || error.message);
  }
};

// Example usage
lookupContact('12345');
```

---

## Complete Workflow Example

Here's a complete example showing search followed by concurrent contact lookups:

### Python

```python
import requests
from concurrent.futures import ThreadPoolExecutor, as_completed

API_KEY = "YOUR_API_KEY"
BASE_URL = "https://api.data-vertex.com/v1"

headers = {
    "x-api-key": API_KEY,
    "Content-Type": "application/json"
}

# Step 1: Search for candidates
search_response = requests.post(
    f"{BASE_URL}/search",
    headers=headers,
    json={
        "search_criteria": {
            "current_title": ["Software Engineer"],
            "location": ["San Francisco::~50mi"],
            "contact_method": ["personal email"]
        },
        "page_size": 10
    }
)

search_data = search_response.json()

# Step 2: Extract candidate IDs
candidate_ids = [
    profile['id'] 
    for profile in search_data['data']['profiles']
  ][:5]  # Take first 5

print(f"Found {len(candidate_ids)} candidates to lookup")

# Step 3: Lookup contact information concurrently
def lookup_single_contact(candidate_id):
    """Lookup a single candidate"""
    response = requests.post(
        f"{BASE_URL}/contact-lookup",
        headers=headers,
        json={
            "candidate_ids": [candidate_id],
            "reveal_personal_email": True,
            "reveal_phone": True,
            "reveal_detailed_person_enrichment": True,
            "reveal_healthcare_enrichment": False
        }
    )
    return response.json()

# Make concurrent requests (up to 10 at a time)
all_results = []
with ThreadPoolExecutor(max_workers=10) as executor:
    future_to_id = {
        executor.submit(lookup_single_contact, cid): cid 
        for cid in candidate_ids
    }
    
    for future in as_completed(future_to_id):
        try:
            result = future.result()
            all_results.append(result)
        except Exception as e:
            print(f"Error looking up {future_to_id[future]}: {e}")

# Step 4: Process results
print("\nContact Information:")
total_credits = 0
for result in all_results:
    contacts = result['data']['contacts']
    for contact in contacts:
        if contact['success']:
            profile = contact['profile']
            print(f"\n{profile['name']} - {profile['current_title']} at {profile['current_employer']}")
            print(f"  Email: {profile.get('personal_email', 'N/A')}")
            print(f"  Phone: {profile.get('phone_number', 'N/A')}")
            print(f"  Skills: {', '.join(profile.get('skills', [])[:5])}")
        else:
            print(f"\nFailed to lookup {contact['lookup_identifier']}: {contact['error']}")
    
    total_credits += result['credits']['used']

# Step 5: Check credits
print(f"\n--- Credit Usage ---")
print(f"Search credits: {search_data['credits']['used']}")
print(f"Lookup credits: {total_credits}")
print(f"Total used: {search_data['credits']['used'] + total_credits}")
print(f"Remaining: {all_results[-1]['credits']['remaining']}")
```

### Python Example - Direct LinkedIn URL Lookup

```python
import requests

BASE_URL = "https://api.data-vertex.com/v1"
API_KEY = "your_api_key_here"

headers = {
    "x-api-key": API_KEY,
    "Content-Type": "application/json"
}

# Direct lookup using a LinkedIn URL (no search required)
linkedin_url = "https://www.linkedin.com/in/jane-smith-12345"

# Lookup contact information directly
lookup_response = requests.post(
    f"{BASE_URL}/contact-lookup",
    headers=headers,
    json={
        "linkedin_urls": [linkedin_url],
        "reveal_personal_email": True,
        "reveal_phone": True,
        "reveal_detailed_person_enrichment": False,
        "reveal_healthcare_enrichment": False
    }
)

lookup_data = lookup_response.json()

# Process result
contact = lookup_data['data']['contacts'][0]
if contact['success']:
    profile = contact['profile']
    print(f"{profile['name']} - {profile['current_title']} at {profile['current_employer']}")
    print(f"  LinkedIn: {contact['lookup_identifier']}")
    print(f"  Email: {profile.get('personal_email', 'N/A')}")
    print(f"  Phone: {profile.get('phone_number', 'N/A')}")
else:
    print(f"Failed to lookup {contact['lookup_identifier']}: {contact['error']}")

# Check credits
print(f"\n--- Credit Usage ---")
print(f"Lookup credits: {lookup_data['credits']['used']}")
print(f"Remaining: {lookup_data['credits']['remaining']}")
```

---

## Error Responses

### 400 Bad Request

Missing or invalid parameters:

```json
{
  "success": false,
  "message": "Either candidate_ids or linkedin_urls array is required."
}
```

Or invalid enrichment parameters:

```json
{
  "success": false,
  "message": "At least one enrichment option must be enabled (set to true)."
}
```

Or too many identifiers:

```json
{
  "success": false,
  "message": "Must provide exactly 1 identifier (candidate_id or linkedin_url) per request. You provided 5. For multiple candidates, make concurrent API calls."
}
```

### 403 Forbidden

Authentication error:

```json
{
  "success": false,
  "message": "Invalid or inactive API key."
}
```

Or insufficient credits:

```json
{
  "success": false,
  "message": "Insufficient credits. Required: 27, Available: 10",
  "credits_info": {
    "required": 27,
    "available": 10,
    "breakdown": "3 contacts × 9 credits each"
  }
}
```

**Note:** The `required` credits shown is the **maximum** if all data is found. Actual charges will be lower based on success-based billing.

### 404 Not Found

Individual candidate not found (per-contact error within 200 response):

```json
{
  "candidate_id": "99999",
  "success": false,
  "error": "Profile not found",
  "status_code": 404
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
| **200** | OK | Request successful (check individual contact success) |
| **400** | Bad Request | Missing parameters, invalid enrichment options, too many IDs |
| **403** | Forbidden | Missing/invalid API key, insufficient credits |
| **404** | Not Found | Candidate not found (per-contact, within 200 response) |
| **500** | Internal Server Error | Unexpected server error |

**Important:** A 200 response doesn't mean all lookups succeeded. Check each contact's `success` field individually.

---

## Best Practices

### 1. Request Only What You Need

Only enable enrichments you'll actually use to minimize credit costs:

```python
# Just need emails for outreach
payload = {
    "candidate_ids": ["12345"],
    "reveal_personal_email": True,
    "reveal_phone": False,
    "reveal_detailed_person_enrichment": False,
    "reveal_healthcare_enrichment": False
}
```

### 2. Make Concurrent Requests for Multiple Candidates

For optimal performance when looking up multiple candidates, make concurrent API calls:

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def lookup_contact(candidate_id):
    """Lookup a single candidate"""
    response = requests.post(
        "https://api.data-vertex.com/v1/contact-lookup",
        headers=headers,
        json={
            "candidate_ids": [candidate_id],
            "reveal_personal_email": True,
            "reveal_phone": True
        }
    )
    return response.json()

# Process candidates concurrently (up to 15 at a time recommended)
with ThreadPoolExecutor(max_workers=15) as executor:
    futures = {
        executor.submit(lookup_contact, cid): cid 
        for cid in all_candidate_ids
    }
    
    for future in as_completed(futures):
        try:
            result = future.result()
            # Process result
            print(f"Completed: {futures[future]}")
        except Exception as e:
            print(f"Error: {e}")
```

**Performance Benefits:**
- Results available progressively (don't wait for all to finish)
- Better user experience with streaming results
- Efficient parallel processing

### 3. Handle Partial Failures Gracefully

Not all lookups will succeed. Handle failures appropriately:

```python
for result in results:
    contacts = result['data']['contacts']
    for contact in contacts:
        if contact['success']:
            # Process successful lookup
            print(f"Success: {contact['profile']['name']}")
        else:
            # Log failure
            print(f"Failed: {contact['lookup_identifier']} - {contact['error']}")
```

### 4. Validate Identifiers

Ensure identifiers are valid:

```python
# Good: IDs from DataVertex search
candidate_ids = [profile['id'] for profile in search_results['profiles']]

# Good: Valid LinkedIn URLs
linkedin_urls = [
    "https://www.linkedin.com/in/john-doe-12345",
    "https://www.linkedin.com/in/jane-smith-67890"
]

# Bad: Random or malformed identifiers
candidate_ids = ["invalid_id", "expired_id"]
linkedin_urls = ["not-a-linkedin-url", "malformed"]
```

### 5. Monitor Success-Based Billing

Track what data you're actually getting:

```python
email_count = 0
phone_count = 0
total_credits = 0

for result in results:
    contacts = result['data']['contacts']
    for contact in contacts:
        if contact['success']:
            if 'email' in contact['contact_data_retrieved']:
                email_count += 1
            if 'phone' in contact['contact_data_retrieved']:
                phone_count += 1
    
    total_credits += result['credits']['used']

print(f"Retrieved {email_count} emails and {phone_count} phones")
print(f"Total credits used: {total_credits}")
```

---

## Common Use Cases

### Example 1: Email-Only Lookup for Cold Outreach

```json
{
  "candidate_ids": ["12345"],
  "reveal_personal_email": true,
  "reveal_phone": false,
  "reveal_detailed_person_enrichment": false,
  "reveal_healthcare_enrichment": false
}
```

**Cost:** 3 credits per email found (success-based)

### Example 2: Full Contact + Profile Data

```json
{
  "candidate_ids": ["12345"],
  "reveal_personal_email": true,
  "reveal_phone": true,
  "reveal_detailed_person_enrichment": true,
  "reveal_healthcare_enrichment": false
}
```

**Cost:** Up to 10 credits per contact (3 email + 6 phone + 1 profile)

### Example 3: Healthcare Professional Lookup

```json
{
  "candidate_ids": ["98765"],
  "reveal_personal_email": true,
  "reveal_phone": true,
  "reveal_detailed_person_enrichment": false,
  "reveal_healthcare_enrichment": true
}
```

**Cost:** Up to 10 credits per contact (3 email + 6 phone + 1 healthcare)

### Example 4: Phone-Only Lookup for SMS Campaigns

```json
{
  "candidate_ids": ["12345"],
  "reveal_personal_email": false,
  "reveal_phone": true,
  "reveal_detailed_person_enrichment": false,
  "reveal_healthcare_enrichment": false
}
```

**Cost:** 6 credits per phone found (success-based)

### Example 5: Concurrent Lookups for Multiple Candidates

For multiple candidates, make concurrent API calls:

```python
from concurrent.futures import ThreadPoolExecutor

candidate_ids = ["12345", "67890", "11111", "22222"]

def lookup_contact(cid):
    return requests.post(
        "https://api.data-vertex.com/v1/contact-lookup",
        headers={"x-api-key": API_KEY, "Content-Type": "application/json"},
        json={
            "candidate_ids": [cid],
            "reveal_personal_email": True,
            "reveal_phone": True
        }
    ).json()

# Make concurrent requests
with ThreadPoolExecutor(max_workers=15) as executor:
    results = list(executor.map(lookup_contact, candidate_ids))

# Total cost: Up to 9 credits per contact (3 email + 6 phone) × 4 contacts = up to 36 credits
```

**Use Case:** Efficiently lookup multiple candidates from search results or external lists.

---

## Need Help?

- **Support:** dev@data-vertex.com
- **Website:** https://www.data-vertex.com

---

*Last Updated: February 20, 2026*

