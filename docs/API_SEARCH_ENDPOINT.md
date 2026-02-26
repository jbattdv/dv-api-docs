# DataVertex Search API

## Overview

The DataVertex Search API allows you to discover and filter professional profiles by criteria such as job title, company, location, skills, and more. Search across 700M+ professional profiles to find candidates that match your specific requirements.

**Endpoint:** `POST https://api.data-vertex.com/v1/search`

**Credit Cost:** 1 credit per search (up to 100 profiles per page)

**Billing:** Success-based - you're only charged if results are returned

---

## Authentication

All API requests must include your API key in the request headers:

```
x-api-key: YOUR_API_KEY
```

You can obtain your API key from DataVertex directly.

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
| `search_criteria` | object | Search filters and criteria | Yes |
| `page_size` | integer | Number of profiles per page (1-100) | No (default: 50) |
| `start` | integer | Starting position for pagination | No (default: 1) |

---

## Search Criteria Parameters

The `search_criteria` object supports 80+ parameters organized into logical categories:

### Profile Identification

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `id` | array of strings | DataVertex Profile IDs | `["123456"]` |
| `name` | array of strings | Profile names | `["John Doe"]` |
| `email` | array of strings | Email addresses | `["john.doe@data-vertex.com"]` |
| `phone` | array of strings | Phone numbers | `["+15555555555"]` |
| `handle` | array of strings | Social media handles | `["johndoe"]` |
| `link` | array of strings | Profile URLs | `["https://linkedin.com/in/johndoe"]` |

### Job Title & Role

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `current_title` | array of strings | Current job titles | `["Product Manager", "VP of Product"]` |
| `previous_title` | array of strings | Previous job titles | `["Software Engineer"]` |
| `current_or_previous_title` | array of strings | Current or previous titles | `["VP of Sales", "Director of Sales"]` |
| `department` | array of strings | Company departments | `["Product Management", "Engineering"]` |
| `management_levels` | array of strings | Management levels | `["Director", "VP", "C-Level"]` |

### Current Employer

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `current_employer` | array of strings | Current company names | `["DataVertex"]` |
| `employer` | array of strings | Company names (current) | `["DataVertex"]` |
| `company_name` | array of strings | Company names | `["DataVertex"]` |
| `company_domain` | array of strings | Company domains | `["data-vertex.com"]` |
| `company_email` | array of strings | Company email addresses | `["info@data-vertex.com"]` |
| `company_website_url` | array of strings | Company website URLs | `["data-vertex.com"]` |

### Previous Employer

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `previous_employer` | array of strings | Previous company names | `["Google", "Microsoft"]` |

### Company Attributes

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `company_size` | array of strings | Company size ranges | `["51-200", "201-500"]` |
| `employees` | array of strings | Employee count ranges | `["100-500"]` |
| `company_revenue` | array of strings | Revenue ranges | `["10000000-50000000"]` |
| `revenue` | array of strings | Revenue values | `["10000000-50000000"]` |
| `company_funding_min` | array of strings | Minimum funding amount | `["1000000"]` |
| `company_funding_max` | array of strings | Maximum funding amount | `["50000000"]` |
| `total_funding` | array of strings | Total capital raised | `["10000000"]` |
| `company_publicly_traded` | array of strings | Public trading status | `["true"]` |
| `publicly_traded` | array of strings | Public trading status | `["true"]` |

### Company Location

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `company_country_code` | array of strings | Company country codes | `["US"]` |
| `geo` | array of strings | Geographic regions | `["North America"]` |
| `state` | array of strings | US states | `["MA", "CA"]` |
| `postal_code` | array of strings | Postal codes | `["02110"]` |
| `location` | array of strings | Location with optional radius | `["San Francisco::~50mi"]` |

**Location with Radius:** Add a radius to location searches using the format `"City::~Nmi"` or `"City::~Nkm"`:
- Example: `"location": ["San Francisco::~50mi"]`
- Example: `"location": ["Boston::~25km"]`

### Industry & Sector

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `company_industry` | array of strings | Company industries | `["Software Engineering"]` |
| `industry` | array of strings | Industries | `["Software"]` |
| `primary_industry` | array of strings | Primary industries | `["Software"]` |
| `company_industry_keywords` | array of strings | Industry keywords | `["SaaS", "B2B"]` |
| `industry_keywords` | array of strings | Industry keywords | `["AI", "Machine Learning"]` |
| `company_naics_code` | array of strings | NAICS codes | `["541330", "541512"]` |
| `naics_code` | array of strings | NAICS codes | `["541330"]` |
| `naics_codes` | array of strings | NAICS codes | `["541330", "541340"]` |
| `company_sic_code` | array of strings | SIC codes | `["7372"]` |
| `sic_code` | array of strings | SIC codes | `["7372"]` |
| `sic_codes` | array of strings | SIC codes | `["7372", "7373"]` |

### Company Intelligence

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `company_competitors` | array of strings | Competitor domains | `["competitor.com"]` |
| `competitors` | array of strings | Competitor domains | `["competitor.com"]` |
| `company_tag` | array of strings | Company tags | `["unicorn", "startup"]` |
| `company_intent` | array of strings | Company intent signals | `["hiring"]` |
| `company_news_timestamp` | array of strings | News event filters | `["mergers & acquisitions::one_month"]` |
| `company_list` | array of strings | Company list names | `["Fortune 500"]` |
| `company_list_id` | array of strings | Company list IDs | `["12345"]` |

**Company News Timestamp Format:** Use `"category::time_period"`:
- Categories: `mergers & acquisitions`, `executive changes`, `funding`
- Time periods: `one_week`, `one_month`, `three_months`, `six_months`, `all_time`
- Example: `["funding::three_months", "executive changes::one_month"]`

### Education

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `school` | array of strings | Schools attended | `["Stanford University", "MIT"]` |
| `degree` | array of strings | Degree types | `["Bachelors", "Masters", "PhD"]` |
| `major` | array of strings | Academic majors | `["Computer Science", "Biology"]` |

### Skills & Experience

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `skills` | array of strings | Listed skills (any match) | `["Python", "SQL", "Machine Learning"]` |
| `all_skills` | array of strings | Required skills (must match all) | `["python", "sql", "machine learning"]` |
| `years_experience` | array of strings | Years of experience | `["10", "15"]` |
| `description` | array of strings | Profile descriptions | `["Experienced software engineer..."]` |

### Social & Connections

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `connections` | array of strings | LinkedIn connection counts | `["500+"]` |

### Healthcare (for healthcare professionals)

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `health_credentials` | array of strings | Healthcare credentials | `["MD", "RN", "PhD"]` |
| `health_license` | array of strings | Healthcare licenses | `["MA12345"]` |
| `health_npi` | array of strings | National Provider Identifiers | `["1234567890"]` |
| `health_specialization` | array of strings | Medical specializations | `["Cardiology Technician", "Clinical Psychologist"]` |

### Metadata & Filters

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `keyword` | string or null | General keyword search | `"data enrichment"` |
| `keywords` | string or null | Multiple keywords (comma-separated) | `"sales prospecting, data enrichment"` |
| `insight` | array of strings | Insight filters | `["funding"]` |
| `news_timestamp` | array of strings | News timestamp filters | `["one_month"]` |
| `update_time` | array of strings | Profile update time | `["2025-01-01"]` |
| `is_primary` | array of strings | Primary profile flag | `["true"]` |
| `veteran_status` | array of strings | Veteran status (only 'True' accepted) | `["True"]` |

---

## Advanced Search Features

### Exact Match

Add quotes around search terms to specify exact matches:

```json
{
  "search_criteria": {
    "name": ["\"Marc Benioff\""],
    "current_employer": ["\"IBM\""]
  }
}
```

Without quotes, `"Marc Benioff"` matches "Marc Benioff", "Benioff Marc", "Marc Anthony Benioff", and ignores typos.
With quotes, only exact matches are returned.

### Exclude Terms

Prepend `-` to filter values to exclude results matching those terms:

```json
{
  "search_criteria": {
    "current_title": ["Software Engineer", "Software Developer", "-Senior", "-Sr"]
  }
}
```

This matches Software Engineers and Software Developers that don't have "Senior" or "Sr" in their title.

### Numeric Operators

Some fields support mathematical operators:

```json
{
  "search_criteria": {
    "company_funding_min": ["1000000+"],
    "company_funding_max": ["<90000000"],
    "company_revenue": ["1000000-90000000"]
  }
}
```

Valid operators: `+`, `<`, `>=`, `<=`, `-` (range)

---

## Response Format

### Success Response (200 OK)

```json
{
  "success": true,
  "data": {
    "profiles": [
      {
        "id": "12345",
        "name": "Jane Smith",
        "current_title": "Software Engineer",
        "current_employer": "TechCorp",
        "location": "San Francisco, CA",
        "linkedin_url": "https://linkedin.com/in/janesmith"
      }
    ],
    "pagination": {
      "current_page": 1,
      "page_size": 50,
      "start": 1,
      "total": 1500,
      "has_next": true,
      "next_start": 51
    }
  },
  "credits": {
    "used": 1,
    "remaining": 999
  }
}
```

### Response Fields

Profile Object

Each profile in the `profiles` array contains:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique candidate ID (use for contact lookup) |
| `name` | string | Full name |
| `current_title` | string | Current job title |
| `current_employer` | string | Current company name |
| `location` | string | Current location |
| `linkedin_url` | string | LinkedIn profile URL |

Note: Search results do **not** include contact information (email/phone). Use the [Contact Lookup API](./API_CONTACT_LOOKUP_ENDPOINT.md) with the candidate `id` to retrieve contact details.

Pagination Object

| Field | Type | Description |
|-------|------|-------------|
| `current_page` | integer | Current page number |
| `page_size` | integer | Profiles per page |
| `start` | integer | Starting position |
| `total` | integer | Total matching profiles |
| `has_next` | boolean | Whether more results exist |
| `next_start` | integer or null | Starting position for next page |

Credits Object

| Field | Type | Description |
|-------|------|-------------|
| `used` | integer | Credits charged for this request |
| `remaining` | integer | Your remaining credit balance |

---

## Code Examples

### cURL

```bash
curl -X POST https://api.data-vertex.com/v1/search \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search_criteria": {
      "current_title": ["Software Engineer", "Senior Software Engineer"],
      "location": ["San Francisco::~50mi"],
      "skills": ["Python", "React"]
    },
    "page_size": 50,
    "start": 1
  }'
```

### Python

```python
import requests
import json

url = "https://api.data-vertex.com/v1/search"

headers = {
    "x-api-key": "YOUR_API_KEY",
    "Content-Type": "application/json"
}

payload = {
    "search_criteria": {
        "current_title": ["Software Engineer", "Senior Software Engineer"],
        "location": ["San Francisco::~50mi"],
        "skills": ["Python", "React"]
    },
    "page_size": 50,
    "start": 1
}

response = requests.post(url, headers=headers, json=payload)

if response.status_code == 200:
    data = response.json()
    print(f"Found {len(data['data']['profiles'])} profiles")
    print(f"Credits used: {data['credits']['used']}")
    print(f"Credits remaining: {data['credits']['remaining']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

### JavaScript (Node.js)

```javascript
const axios = require('axios');

const searchCandidates = async () => {
  try {
    const response = await axios.post(
      'https://api.data-vertex.com/v1/search',
      {
        search_criteria: {
          current_title: ['Software Engineer', 'Senior Software Engineer'],
          location: ['San Francisco::~50mi'],
          skills: ['Python', 'React']
        },
        page_size: 50,
        start: 1
      },
      {
        headers: {
          'x-api-key': 'YOUR_API_KEY',
          'Content-Type': 'application/json'
        }
      }
    );

    console.log(`Found ${response.data.data.profiles.length} profiles`);
    console.log(`Credits used: ${response.data.credits.used}`);
    console.log(`Credits remaining: ${response.data.credits.remaining}`);
    
    return response.data;
  } catch (error) {
    console.error('Error:', error.response?.data || error.message);
  }
};

searchCandidates();
```

---

## Pagination Example

To retrieve the next page of results:

```python
import requests

url = "https://api.data-vertex.com/v1/search"
headers = {
    "x-api-key": "YOUR_API_KEY",
    "Content-Type": "application/json"
}

# First page
payload = {
    "search_criteria": {
        "current_title": ["Software Engineer"]
    },
    "page_size": 100,
    "start": 1
}

response = requests.post(url, headers=headers, json=payload)
data = response.json()

# Check if there's a next page
if data['data']['pagination']['has_next']:
    next_start = data['data']['pagination']['next_start']
    
    # Get next page
    payload['start'] = next_start
    next_response = requests.post(url, headers=headers, json=payload)
    next_data = next_response.json()
```

---

## Error Responses

### 400 Bad Request

Missing or invalid parameters:

```json
{
  "success": false,
  "message": "search_criteria is required."
}
```

### 403 Forbidden

Authentication or authorization error:

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
  "message": "Insufficient credits. Required: 1, Available: 0",
  "credits_info": {
    "required": 1,
    "available": 0
  }
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
| **200** | OK | Request successful, data returned |
| **400** | Bad Request | Malformed request, missing required parameters, invalid page_size |
| **403** | Forbidden | Missing/invalid API key, insufficient credits |
| **500** | Internal Server Error | Unexpected server error |

---

## Best Practices

### 1. Use Specific Search Criteria

The more specific your search criteria, the better your results:

```json
{
  "search_criteria": {
    "current_title": ["Software Engineer"],
    "location": ["San Francisco::~25mi"],
    "skills": ["Python", "Django"],
    "years_experience": ["5"]
  }
}
```

### 2. Optimize Page Size

- Use `page_size: 100` for maximum profiles per request
- Smaller page sizes (10-20) for testing or incremental processing
- Balance between API calls and processing time

### 3. Handle Pagination Efficiently

```python
all_profiles = []
start = 1

while True:
    response = search_candidates(start=start)
    profiles = response['data']['profiles']
    all_profiles.extend(profiles)
    
    if not response['data']['pagination']['has_next']:
        break
    
    start = response['data']['pagination']['next_start']
```

### 4. Store Candidate IDs

Search results don't include contact information. Store the `id` field from each profile to use with the [Contact Lookup API](./API_CONTACT_LOOKUP_ENDPOINT.md):

```python
candidate_ids = [profile['id'] for profile in response['data']['profiles']]
```

### 5. Monitor Your Credits

Check your remaining credits in each response to avoid interruptions:

```python
if response['credits']['remaining'] < 100:
    print("Warning: Low credit balance!")
```

---

## Common Use Cases

### Example 1: Find Software Engineers in Bay Area

```json
{
  "search_criteria": {
    "current_title": ["Software Engineer", "Senior Software Engineer", "Staff Engineer"],
    "location": ["San Francisco::~50mi"],
    "skills": ["Python", "JavaScript", "React"]
  },
  "page_size": 100
}
```

### Example 2: Find Healthcare Professionals

```json
{
  "search_criteria": {
    "current_title": ["Registered Nurse", "RN"],
    "health_credentials": ["RN"],
    "location": ["Boston::~30mi"],
    "years_experience": ["3"]
  },
  "page_size": 50
}
```

### Example 3: Find Sales Leaders at Tech Companies

```json
{
  "search_criteria": {
    "current_title": ["VP of Sales", "Director of Sales", "Sales Manager"],
    "company_industry": ["Software", "SaaS"],
    "company_size": ["51-200", "201-500"],
    "management_levels": ["Director", "VP"]
  },
  "page_size": 100
}
```

### Example 4: Find Recent Job Changers

```json
{
  "search_criteria": {
    "current_title": ["Product Manager"],
    "location": ["New York::~40mi"],
    "job_change_range_days": ["30"]
  },
  "page_size": 50
}
```

---

## Need Help?

- **Support:** dev@data-vertex.com
- **Website:** https://www.data-vertex.com

---

*Last Updated: February 2, 2026*

