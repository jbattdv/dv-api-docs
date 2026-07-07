# DataVertex Candidate Search API

## Overview

The DataVertex Candidate Search API allows you to discover and filter professional profiles by criteria such as job title, company, location, skills, and more. Search across 800M+ professional profiles to find candidates that match your specific requirements.

[Schedule a quick call](https://calendar.notion.so/meet/claire-liao/datavertex) to get API access and explore how DataVertex can support your recruiting product development.

**Endpoint:** `POST https://api.data-vertex.com/v1/search`

**Credit Cost:** 1 credit per standard search (up to 100 profiles per page). Job description searches using `jd_search` cost **2 credits** per search.

**Billing:** Success-based — you are only charged when results are returned

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

<table>
  <thead>
    <tr>
      <th>Parameter</th>
      <th>Type</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>search_criteria</code></td>
      <td>object</td>
      <td>Search filters and criteria. Required unless <code>free_text_search</code> or <code>jd_search</code> is provided.</td>
      <td>Conditional</td>
    </tr>
    <tr>
      <td><code>free_text_search</code></td>
      <td>string</td>
      <td>Natural language search request (max 300 characters). The API parses this into structured search criteria automatically. Mutually exclusive with <code>jd_search</code>.</td>
      <td>Conditional</td>
    </tr>
    <tr>
      <td><code>jd_search</code></td>
      <td>string</td>
      <td>Full job description (max 8,500 characters). The API parses this into structured search criteria automatically. Costs 2 credits when results are returned. Mutually exclusive with <code>free_text_search</code>.</td>
      <td>Conditional</td>
    </tr>
    <tr>
      <td><code>page_size</code></td>
      <td>integer</td>
      <td>Number of profiles per page (1-100)</td>
      <td>No (default: 50)</td>
    </tr>
    <tr>
      <td><code>start</code></td>
      <td>integer</td>
      <td>Starting position for pagination</td>
      <td>No (default: 1)</td>
    </tr>
    <tr>
      <td><code>include_similar_titles</code></td>
      <td>boolean</td>
      <td>Automatically expand <code>current_title</code> with similar titles</td>
      <td>No (default: <code>false</code>; default <code>true</code> when <code>jd_search</code> is used)</td>
    </tr>
  </tbody>
</table>

> **Note:** At least one of `search_criteria`, `free_text_search`, or `jd_search` must be provided. `free_text_search` and `jd_search` cannot be used in the same request. `search_criteria` can be combined with either AI input — see [free text search](#semantic-candidate-search---free_text_search) and [job description search](#job-description-search---jd_search) below for merge behavior.

#### Extended Title Search - include_similar_titles

When set to `true` at the top level of the request, the API automatically expands your `current_title` array with similar, titles before executing the search. This broadens your candidate pool without requiring you to manually list every relevant title variation.

- Only applies when `current_title` is provided with fewer than 10 titles
- Titles are added up to a maximum of 10 total
- If `current_title` is not present in your request, this parameter has no effect
- Default: `false` for standard and free-text searches; default `true` when `jd_search` is used (you may set it to `false` to disable)

```json
{
  "search_criteria": {
    "current_title": ["software engineer"],
    "location": ["Austin::~30mi"]
  },
  "page_size": 50,
  "start": 1,
  "include_similar_titles": true
}
```

#### Semantic Candidate Search - free_text_search

Write a natural language description of the candidates you are looking for — up to 300 characters. The API will parse your request into structured search criteria.

**Supported fields parsed from free text:**

| Free Text Concept | Mapped To |
|-------------------|-----------|
| Job titles | `current_title` |
| Skills | `skills` |
| Location | `location` |
| Years of experience | `years_experience` |
| Current or previous employer | `current_employer` / `previous_employer` |
| School, degree, major | `school` / `degree` / `major` |
| Industry / sector / domain / market | `company_industry` |

**Behavior when combined with `search_criteria`:**
- Fields parsed from `free_text_search` are **merged** into any existing `search_criteria` — array values are combined
- Exception: `years_experience` always **overrides** any value already in `search_criteria` rather than merging
- `free_text_search` can be used as the sole input — `search_criteria` is not required when it is provided

**Compatibility with `include_similar_titles`:** Fully supported. The `current_title` values parsed from `free_text_search` are populated first, and then `include_similar_titles` expands them as normal.

**Limit:** 300 characters. Requests exceeding this return a 400 error.

```json
{
  "free_text_search": "Software engineers with 5+ years of Python experience in Chicago",
  "page_size": 50,
  "start": 1
}
```

The response includes a `free_text_searched` field showing exactly what is mapped:

```json
{
  "free_text_searched": {
    "current_title": ["Software Engineer"],
    "skills": ["Python"],
    "location": ["Chicago"],
    "years_experience": ["5+"]
  }
}
```

#### Job Description Search - jd_search

Submit a full job description — up to 8,500 characters — and the API parses it into structured search criteria to find candidates who fit the role. This is designed for recruiting workflows where you already have a job posting and want to translate it into a profile search without building filters manually.

**Credit cost:** 2 credits per search when results are returned (standard searches cost 1 credit).

**Supported fields parsed from a job description:**

| JD Concept | Mapped To |
|------------|-----------|
| Job titles | `current_title` |
| Skills | `skills` |
| Location | `location` |
| Years of experience | `years_experience` |
| Major | `major` |
| Industry | `company_industry` |

**Behavior when combined with `search_criteria`:**
- Fields parsed from `jd_search` are **merged** into any existing `search_criteria` — array values are combined
- Exception: `years_experience` always **overrides** any value already in `search_criteria` rather than merging
- `jd_search` can be used as the sole input — `search_criteria` is not required when it is provided

**Compatibility with `include_similar_titles`:** Enabled by default for `jd_search`. After the job description is parsed, `current_title` values are expanded with similar titles (up to 10 total) before the search runs. Set `"include_similar_titles": false` to search only the parsed titles.

**Mutual exclusivity:** `jd_search` and `free_text_search` cannot be sent in the same request.

**Limit:** 8,500 characters. Requests exceeding this return a 400 error.

```json
{
  "jd_search": "Acme Corp is hiring a Senior Software Engineer in Chicago. 5+ years of experience with Python and AWS required. Bachelor's in Computer Science preferred.",
  "page_size": 50,
  "start": 1
}
```

The response includes a `jd_searched` field showing exactly what was mapped:

```json
{
  "jd_searched": {
    "current_title": ["Senior Software Engineer"],
    "skills": ["Python", "AWS"],
    "location": ["Chicago"],
    "years_experience": ["5+"],
    "current_employer": ["-Acme Corp"],
    "major": ["Computer Science"],
    "company_industry": ["IT & Software"]
  }
}
```

---

## Search Criteria Parameters

The `search_criteria` object supports 60+ parameters organized into logical categories:

### Profile Identification

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `link` | array of strings | Profile URLs | `["https://linkedin.com/in/johndoe"]` |
| `name` | array of strings | Profile names | `["John Doe"]` |
| `email` | array of strings | Email addresses | `["john.doe@data-vertex.com"]` |
| `phone` | array of strings | Phone numbers | `["+15555555555"]` |
| `handle` | array of strings | Social media handles | `["johndoe"]` |
| `id` | array of strings | DataVertex Profile IDs | `["123456"]` |

### Job Title & Role

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `current_title` | array of strings | Current job titles | `["Product Manager", "VP of Product"]` |
| `previous_title` | array of strings | Previous job titles | `["Software Engineer", "Software Developer"]` |
| `current_or_previous_title` | array of strings | Current or previous titles | `["VP of Sales", "Director of Sales"]` |
| `department` | array of strings | Company departments | `["Product Management", "Engineering"]` |
| `management_levels` | array of strings | Management levels | `["Director", "VP", "C-Level"]` |

### Employer

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `current_employer` | array of strings | Current company names | `["DataVertex"]` |
| `previous_employer` | array of strings | Previous company names | `["Google", "Microsoft"]` |
| `company_name` | array of strings | Company names | `["DataVertex"]` |
| `company_domain` | array of strings | Company domains | `["data-vertex.com"]` |
| `company_email` | array of strings | Company email addresses | `["info@data-vertex.com"]` |
| `company_website_url` | array of strings | Company website URLs | `["data-vertex.com"]` |

### Company Attributes

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `company_size` | array of strings | Company size ranges | `["51-200", "201-500"]` |
| `employees` | array of strings | Employee count ranges | `["100-500"]` |
| `company_revenue` | array of strings | Revenue ranges | `["10000000-50000000"]` |
| `company_funding_min` | array of strings | Minimum funding amount | `["1000000"]` |
| `company_funding_max` | array of strings | Maximum funding amount | `["50000000"]` |
| `total_funding` | array of strings | Total capital raised | `["10000000"]` |
| `company_publicly_traded` | array of strings | Public trading status | `["true"]` |

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
| `company_industry_keywords` | array of strings | Industry keywords | `["SaaS", "B2B"]` |
| `company_naics_code` | array of strings | NAICS codes | `["541330", "541512"]` |
| `company_sic_code` | array of strings | SIC codes | `["7372"]` |

### Company Intelligence

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `company_competitors` | array of strings | Competitor domains | `["competitor.com"]` |

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
| `years_experience` | array of strings | Years of experience | `["1","2,","3"]` or ["1-3"]|

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
| `keywords` | array of strings | Multiple keywords (comma-separated) | `["Consulting", "Staffing"]` |

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

company_funding_min, company_funding_max, employees, and company_size support mathematical operators:

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
        "linkedin_url": "https://linkedin.com/in/janesmith",
        "name": "Jane Smith",
        "current_title": "Software Engineer",
        "current_employer": "TechCorp",
        "location": "San Francisco, CA",
        "id": "12345",
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

When `free_text_search` is used, the response also includes a `free_text_searched` field:

```json
{
  "success": true,
  "data": { "..." },
  "credits": {
    "used": 1,
    "remaining": 999
  },
  "free_text_searched": {
    "current_title": ["Software Engineer"],
    "skills": ["Python"],
    "location": ["Chicago"],
    "years_experience": ["5+"]
  }
}
```

When `jd_search` is used, the response includes `jd_searched` and charges 2 credits when profiles are returned:

```json
{
  "success": true,
  "data": { "..." },
  "credits": {
    "used": 2,
    "remaining": 998
  },
  "jd_searched": {
    "current_title": ["Senior Software Engineer"],
    "skills": ["Python", "AWS"],
    "location": ["Chicago::~50mi"],
    "years_experience": ["5+"],
    "current_employer": ["-Acme Corp"]
  },
  "similar_titles": [
    "senior software engineer",
    "software engineer",
    "staff software engineer"
  ]
}
```

### Response Fields

Profile Object

Each profile in the `profiles` array contains:

| Field | Type | Description |
|-------|------|-------------|
| `linkedin_url` | string | LinkedIn profile URL |
| `name` | string | Full name |
| `current_title` | string | Current job title |
| `current_employer` | string | Current company name |
| `location` | string | Current location |
| `id` | string | Only useful for profiles without a linkedin_url |

Note: Search results do **not** include contact information (email/phone). Use the [Lookup API](./candidate_lookup_endpoint.md) with the `linkedin_url` to retrieve contact details.

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

Top-level Fields

| Field | Type | Description |
|-------|------|-------------|
| `similar_titles` | array or null | Expanded titles used for the search (only present when `include_similar_titles: true` and `current_title` was provided) |
| `free_text_searched` | object | The structured criteria parsed from your `free_text_search` input (only present when `free_text_search` was used) |
| `jd_searched` | object | The structured criteria parsed from your `jd_search` input (only present when `jd_search` was used) |

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
    "start": 1,
    "include_similar_titles": false
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
    "start": 1,
    "include_similar_titles": False
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
        start: 1,
        include_similar_titles: false
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
    "start": 1,
    "include_similar_titles": False
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
  "message": "search criteria was not provided or not formed correctly."
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

For `jd_search` requests, the required credit amount is 2:

```json
{
  "success": false,
  "message": "Insufficient credits. Required: 2, Available: 1",
  "credits_info": {
    "required": 2,
    "available": 1
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
  },
  "include_similar_titles": false
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

### 4. Store LinkedIn URL

Search results don't include contact information. Store the `linkedin_url` field from each profile to use with the [Lookup API](./candidate_lookup_endpoint.md):

```python
linkedin_url = [profile['linkedin_url'] for profile in response['data']['profiles']]
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
  "page_size": 100,
  "include_similar_titles": false
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
  "page_size": 50,
  "include_similar_titles": false
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
  "page_size": 100,
  "include_similar_titles": false
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
  "page_size": 50,
  "include_similar_titles": false
}
```

### Example 5: Free Text Search

Use `free_text_search` to describe candidates in plain English instead of building structured criteria manually. The API maps your input to the appropriate fields and executes the search.

```json
{
  "free_text_search": "Nurses with at least 3 years of experience in Boston",
  "page_size": 50,
  "start": 1
}
```

You can also combine `free_text_search` with explicit `search_criteria` — the parsed fields are merged in. The exception is `years_experience`, which is always overridden by the free text value rather than merged:

```json
{
  "free_text_search": "Python engineers with 5+ years experience",
  "search_criteria": {
    "location": ["Austin::~30mi"],
    "company_size": ["51-200"]
  },
  "page_size": 50,
  "start": 1
}
```

Use `free_text_search` with `include_similar_titles` to both describe your candidates naturally and broaden the title match:

```json
{
  "free_text_search": "Data engineers with Spark experience in Seattle",
  "include_similar_titles": true,
  "page_size": 100,
  "start": 1
}
```

### Example 6: Job Description Search

Paste a job posting and let the API extract search criteria. No `search_criteria` object is required. `include_similar_titles` defaults to `true`, so related titles are expanded automatically.

```json
{
  "jd_search": "DataVertex is seeking a Product Manager in Austin, TX. 3+ years in B2B SaaS. Experience with roadmap planning and SQL required. MBA a plus.",
  "page_size": 50,
  "start": 1
}
```

Review `jd_searched` in the response to see how the job description was interpreted before using results in your workflow. This request costs 2 credits when profiles are returned.

You can combine `jd_search` with explicit `search_criteria` to layer additional filters on top of the parsed fields:

```json
{
  "jd_search": "Senior Data Engineer role at TechCorp in Seattle. Python and Spark required.",
  "search_criteria": {
    "company_size": ["201-500"]
  },
  "page_size": 50,
  "start": 1
}
```

---

### Example 7: Expand a Single Title with include_similar_titles

When you only have one or a few titles in mind, use `include_similar_titles` to automatically broaden your search to related roles. The API will add up to 10 total titles before executing the search.

**Page 1 — send your title and let the API expand it:**

```json
{
  "search_criteria": {
    "current_title": ["data engineer"],
    "location": ["Chicago::~40mi"]
  },
  "page_size": 100,
  "start": 1,
  "include_similar_titles": true
}
```

The response includes a `similar_titles` field containing the exact titles that were searched. Save this array — you will need it for all subsequent pages:

```json
{
  "success": true,
  "data": {
    "profiles": [...],
    "pagination": {
      "current_page": 1,
      "page_size": 100,
      "start": 1,
      "total": 4800,
      "has_next": true,
      "next_start": 101
    }
  },
  "credits": {
    "used": 1,
    "remaining": 997
  },
  "similar_titles": [
    "data engineer",
    "analytics engineer",
    "data architect",
    "etl developer",
    "data pipeline engineer",
    "big data engineer",
    "data infrastructure engineer"
  ]
}
```

**Page 2 onward — pass `similar_titles` back as `current_title` and set `include_similar_titles` to `false`:**

```json
{
  "search_criteria": {
    "current_title": [
      "data engineer",
      "analytics engineer",
      "data architect",
      "etl developer",
      "data pipeline engineer",
      "big data engineer",
      "data infrastructure engineer"
    ],
    "location": ["Chicago::~40mi"]
  },
  "page_size": 100,
  "start": 101,
  "include_similar_titles": false
}
```

This ensures every page is searched against the same set of titles, giving you consistent and complete results across your entire pagination sequence.

### Example 8: Job Description Search with Pagination

`jd_search` enables `include_similar_titles` by default. Use the same pagination pattern as structured searches: capture `similar_titles` from page 1, then pass that array as `current_title` on subsequent pages.

**Page 1 — send the job description:**

```json
{
  "jd_search": "GrowthCo is hiring a Marketing Manager in Denver. 4+ years of digital marketing experience. HubSpot and Google Analytics required.",
  "page_size": 100,
  "start": 1
}
```

**Page 2 onward — use `similar_titles` from the page 1 response as `current_title`, omit `jd_search`, and set `include_similar_titles` to `false`:**

```json
{
  "search_criteria": {
    "current_title": [
      "marketing manager",
      "digital marketing manager",
      "growth marketing manager"
    ],
    "skills": ["HubSpot", "Google Analytics"],
    "location": ["Denver::~50mi"],
    "years_experience": ["4+"],
    "current_employer": ["-GrowthCo"]
  },
  "page_size": 100,
  "start": 101,
  "include_similar_titles": false
}
```

Use the full parsed criteria from `jd_searched` (not only titles) so every page applies the same filters. Re-running `jd_search` on later pages would parse the text again and may produce slightly different criteria.

---

## Need Help?

- **Support:** dev@data-vertex.com
- **Website:** https://www.data-vertex.com

[Schedule a quick call](https://calendar.notion.so/meet/claire-liao/datavertex) to get API access and explore how DataVertex can support your recruiting product development.

---

*Last Updated: July 7, 2026*

