# Pet Finder API Documentation v1.0

## Table of Contents
1. [Introduction](#introduction)
2. [Base URL & Environment](#base-url--environment)
3. [Authentication](#authentication)
4. [Request & Response Format](#request--response-format)
5. [Endpoints](#endpoints)
   - [GET / - API Information](#get---api-information)
   - [GET /health - Health Check](#get-health---health-check)
   - [POST /predict - Pet Detection & Feature Extraction](#post-predict---pet-detection--feature-extraction)
   - [POST /search_similar - Find Similar Pets](#post-search_similar---find-similar-pets)
   - [GET /stats - Database Statistics](#get-stats---database-statistics)
6. [Error Handling](#error-handling)
7. [Status Codes](#status-codes)
8. [Rate Limiting](#rate-limiting)
9. [Code Examples](#code-examples)
10. [Postman Collection](#postman-collection)

---

## Introduction

The Pet Finder API is a machine learning-powered service that identifies cats and dogs in images, extracts their unique features, and enables similarity matching to help reunite lost pets with their owners.

### Key Features
- Automatic pet detection (cats and dogs)
- Feature extraction using deep learning
- Similarity matching against database
- Real-time processing
- RESTful JSON API

---

## Base URL & Environment & API Keys

**Production Base URL:**
```
https://pet-finder-api-581219442968.us-central1.run.app
```

**Environment:** Google Cloud Run (auto-scaling, managed)

**API KEY:** ZLaMxxdSRKxocd5RDTt9cxb9V18JC3EB

---

## Authentication

### Method: API Key

The API uses API key authentication for protected endpoints. Include your API key in the request headers.

**Header Name:** `X-API-Key`

**Format:**
```
X-API-Key: your-api-key-here
```

### Example:
```bash
curl -H "X-API-Key: your-api-key-here" https://pet-finder-api-581219442968.us-central1.run.app/predict
```

### Public vs Protected Endpoints

| Endpoint | Authentication Required |
|----------|------------------------|
| GET / | No |
| GET /health | No |
| POST /predict | Yes |
| POST /search_similar | Yes |
| GET /stats | Yes |

---

## Request & Response Format

### Request Format
- **Content-Type:** `application/json`
- **Accept:** `application/json`
- **Encoding:** UTF-8

### Response Format
All responses are in JSON format with appropriate HTTP status codes.

**Success Response Structure:**
```json
{
    "field1": "value1",
    "field2": "value2"
}
```

**Error Response Structure:**
```json
{
    "error": "Error description message"
}
```

---

## Endpoints

### GET / - API Information

Returns basic information about the API and available endpoints.

**Authentication:** Not required

**Request:**
```http
GET / HTTP/1.1
Host: pet-finder-api-581219442968.us-central1.run.app
```

**Response:**
```json
{
    "name": "Pet Finder API",
    "version": "1.0",
    "status": "running",
    "documentation": "https://github.com/dyeaml/pet-finder-api",
    "endpoints": {
        "public": ["GET /", "GET /health"],
        "authenticated": ["POST /predict", "POST /search_similar", "GET /stats"]
    }
}
```

**Status Codes:**
- `200 OK` - Success

---

### GET /health - Health Check

Checks if the API service is healthy and operational.

**Authentication:** Not required

**Request:**
```http
GET /health HTTP/1.1
Host: pet-finder-api-581219442968.us-central1.run.app
```

**Response:**
```json
{
    "status": "healthy",
    "models_loaded": true,
    "firebase_connected": true,
    "timestamp": "2025-07-11T10:30:45.123456"
}
```

**Response Fields:**
- `status` (string): Service health status ("healthy" or "unhealthy")
- `models_loaded` (boolean): Whether ML models are loaded
- `firebase_connected` (boolean): Database connection status
- `timestamp` (string): Current server time in ISO format

**Status Codes:**
- `200 OK` - Service is healthy
- `503 Service Unavailable` - Service is unhealthy

---

### POST /predict - Pet Detection & Feature Extraction

Analyzes an image to detect cats or dogs, extracts features, and saves to the database.

**Authentication:** Required

**Request:**
```http
POST /predict HTTP/1.1
Host: pet-finder-api-581219442968.us-central1.run.app
Content-Type: application/json
X-API-Key: your-api-key-here

{
    "image": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEASABIAAD..."
}
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| image | string | Yes | Base64 encoded image with data URI prefix |

**Image Format Requirements:**
- Supported formats: JPEG, PNG, GIF
- Maximum size: 10MB
- Must include data URI prefix (e.g., `data:image/jpeg;base64,`)

**Success Response (Pet Detected):**
```json
{
    "prediction": "cat",
    "confidence": "95.67%",
    "message": "Features extracted and saved with ID: 123e4567-e89b",
    "saved": true,
    "image_id": "123e4567-e89b-12d3-a456-426614174000",
    "feature_dim": 2048,
    "bbox": [120, 50, 380, 400]
}
```

**Success Response (No Pet Detected):**
```json
{
    "prediction": "unknown",
    "confidence": "0%",
    "message": "No cat or dog detected in the image",
    "saved": false
}
```

**Response Fields:**
- `prediction` (string): "cat", "dog", or "unknown"
- `confidence` (string): Detection confidence percentage
- `message` (string): Descriptive message about the operation
- `saved` (boolean): Whether features were saved to database
- `image_id` (string): Unique identifier for saved features (UUID v4)
- `feature_dim` (integer): Number of feature dimensions extracted (2048)
- `bbox` (array): Bounding box coordinates [x1, y1, x2, y2] (optional)

**Status Codes:**
- `200 OK` - Success (even if no pet detected)
- `400 Bad Request` - Invalid request (missing image, invalid format)
- `401 Unauthorized` - Missing or invalid API key
- `413 Payload Too Large` - Image exceeds size limit
- `500 Internal Server Error` - Server processing error

**Error Examples:**

Missing API Key:
```json
{
    "error": "Invalid or missing API key",
    "message": "Please provide a valid API key in X-API-Key header"
}
```

No Image Provided:
```json
{
    "error": "No image provided"
}
```

Invalid Image Data:
```json
{
    "error": "Invalid image data"
}
```

---

### POST /search_similar - Find Similar Pets

Searches for similar pets in the database based on an uploaded image.

**Authentication:** Required

**Request:**
```http
POST /search_similar HTTP/1.1
Host: pet-finder-api-581219442968.us-central1.run.app
Content-Type: application/json
X-API-Key: your-api-key-here

{
    "image": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEASABIAAD...",
    "top_k": 10
}
```

**Request Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| image | string | Yes | - | Base64 encoded image with data URI prefix |
| top_k | integer | No | 10 | Number of similar results to return (1-100) |

**Success Response:**
```json
{
    "query_pet_type": "dog",
    "query_confidence": "92.34%",
    "results": [
        {
            "image_id": "abc123-def456-789012",
            "similarity": 0.9534,
            "pet_type": "dog",
            "timestamp": "2025-07-10 15:30:45"
        },
        {
            "image_id": "xyz789-uvw456-123890",
            "similarity": 0.8721,
            "pet_type": "dog",
            "timestamp": "2025-07-09 12:15:30"
        }
    ],
    "total_compared": 150
}
```

**Response Fields:**
- `query_pet_type` (string): Detected pet type in query image
- `query_confidence` (string): Detection confidence for query image
- `results` (array): List of similar pets sorted by similarity
  - `image_id` (string): Unique identifier of the similar pet
  - `similarity` (float): Similarity score (0-1, higher is more similar)
  - `pet_type` (string): Type of the matched pet
  - `timestamp` (string): When the matched pet was added
- `total_compared` (integer): Total number of pets compared

**Status Codes:**
- `200 OK` - Success
- `400 Bad Request` - Invalid request or no pet detected
- `401 Unauthorized` - Missing or invalid API key
- `500 Internal Server Error` - Server processing error

**Error Example (No Pet Detected):**
```json
{
    "error": "No cat or dog detected"
}
```

---

### GET /stats - Database Statistics

Returns statistics about pets stored in the database.

**Authentication:** Required

**Request:**
```http
GET /stats HTTP/1.1
Host: pet-finder-api-581219442968.us-central1.run.app
X-API-Key: your-api-key-here
```

**Response:**
```json
{
    "total_pets": 1250,
    "cats": 650,
    "dogs": 600,
    "last_updated": "2025-07-11T10:30:45.123456"
}
```

**Response Fields:**
- `total_pets` (integer): Total number of pets in database
- `cats` (integer): Number of cats
- `dogs` (integer): Number of dogs
- `last_updated` (string): Timestamp of the statistics

**Status Codes:**
- `200 OK` - Success
- `401 Unauthorized` - Missing or invalid API key
- `500 Internal Server Error` - Database error

---

## Error Handling

### General Error Format

All errors follow a consistent JSON format:

```json
{
    "error": "Brief error description"
}
```

Some errors may include additional details:

```json
{
    "error": "Invalid or missing API key",
    "message": "Please provide a valid API key in X-API-Key header"
}
```

### Common Error Scenarios

1. **Authentication Errors**
   - Missing API key
   - Invalid API key
   - Expired API key

2. **Validation Errors**
   - Missing required parameters
   - Invalid image format
   - Image too large
   - Invalid parameter values

3. **Processing Errors**
   - Model inference failure
   - Database connection issues
   - Timeout errors

4. **Resource Errors**
   - Endpoint not found
   - Method not allowed

---

## Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request successful |
| 400 | Bad Request | Invalid request format or parameters |
| 401 | Unauthorized | Missing or invalid API key |
| 404 | Not Found | Endpoint does not exist |
| 405 | Method Not Allowed | HTTP method not supported for endpoint |
| 413 | Payload Too Large | Request body exceeds size limit |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server-side error |
| 503 | Service Unavailable | Service temporarily unavailable |

---

## Rate Limiting

- **Default Limit:** 1000 requests per minute per API key
- **Burst Limit:** 100 concurrent requests
- **Response Headers:**
  ```
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 999
  X-RateLimit-Reset: 1625832000
  ```

When rate limit is exceeded:
```json
{
    "error": "Rate limit exceeded. Please try again later."
}
```

---

## Code Examples

### JavaScript (Fetch API)

```javascript
const API_BASE = 'https://pet-finder-api-581219442968.us-central1.run.app';
const API_KEY = 'your-api-key-here';

// Health Check
async function checkHealth() {
    const response = await fetch(`${API_BASE}/health`);
    const data = await response.json();
    console.log(data);
}

// Predict Pet
async function predictPet(imageFile) {
    // Convert image to base64
    const reader = new FileReader();
    reader.readAsDataURL(imageFile);
    
    reader.onload = async () => {
        const response = await fetch(`${API_BASE}/predict`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-API-Key': API_KEY
            },
            body: JSON.stringify({
                image: reader.result
            })
        });
        
        const data = await response.json();
        console.log(data);
    };
}

// Search Similar
async function searchSimilar(imageFile, topK = 10) {
    const reader = new FileReader();
    reader.readAsDataURL(imageFile);
    
    reader.onload = async () => {
        const response = await fetch(`${API_BASE}/search_similar`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-API-Key': API_KEY
            },
            body: JSON.stringify({
                image: reader.result,
                top_k: topK
            })
        });
        
        const data = await response.json();
        console.log(data);
    };
}
```

### Python

```python
import requests
import base64

API_BASE = 'https://pet-finder-api-581219442968.us-central1.run.app'
API_KEY = 'your-api-key-here'

# Health Check
def check_health():
    response = requests.get(f'{API_BASE}/health')
    return response.json()

# Predict Pet
def predict_pet(image_path):
    with open(image_path, 'rb') as image_file:
        image_base64 = base64.b64encode(image_file.read()).decode()
    
    headers = {
        'Content-Type': 'application/json',
        'X-API-Key': API_KEY
    }
    
    data = {
        'image': f'data:image/jpeg;base64,{image_base64}'
    }
    
    response = requests.post(
        f'{API_BASE}/predict',
        headers=headers,
        json=data
    )
    
    return response.json()

# Search Similar
def search_similar(image_path, top_k=10):
    with open(image_path, 'rb') as image_file:
        image_base64 = base64.b64encode(image_file.read()).decode()
    
    headers = {
        'Content-Type': 'application/json',
        'X-API-Key': API_KEY
    }
    
    data = {
        'image': f'data:image/jpeg;base64,{image_base64}',
        'top_k': top_k
    }
    
    response = requests.post(
        f'{API_BASE}/search_similar',
        headers=headers,
        json=data
    )
    
    return response.json()

# Get Stats
def get_stats():
    headers = {'X-API-Key': API_KEY}
    response = requests.get(f'{API_BASE}/stats', headers=headers)
    return response.json()
```

### cURL

```bash
# Health Check
curl https://pet-finder-api-581219442968.us-central1.run.app/health

# Get Stats (with authentication)
curl -H "X-API-Key: your-api-key-here" \
  https://pet-finder-api-581219442968.us-central1.run.app/stats

# Predict Pet (with image file)
IMAGE_BASE64=$(base64 -w 0 cat.jpg)  # Linux
# IMAGE_BASE64=$(base64 -i cat.jpg)  # macOS

curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key-here" \
  -d "{\"image\": \"data:image/jpeg;base64,$IMAGE_BASE64\"}" \
  https://pet-finder-api-581219442968.us-central1.run.app/predict

# Search Similar with top_k parameter
curl -X POST \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key-here" \
  -d "{\"image\": \"data:image/jpeg;base64,$IMAGE_BASE64\", \"top_k\": 5}" \
  https://pet-finder-api-581219442968.us-central1.run.app/search_similar
```

---

## Postman Collection

### Import Instructions

1. Open Postman
2. Click "Import" → "Raw text"
3. Paste the JSON below
4. Click "Import"

### Collection JSON

```json
{
    "info": {
        "name": "Pet Finder API",
        "description": "ML-powered pet detection and matching API",
        "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
    },
    "auth": {
        "type": "apikey",
        "apikey": [
            {
                "key": "key",
                "value": "X-API-Key",
                "type": "string"
            },
            {
                "key": "value",
                "value": "{{api_key}}",
                "type": "string"
            },
            {
                "key": "in",
                "value": "header",
                "type": "string"
            }
        ]
    },
    "variable": [
        {
            "key": "base_url",
            "value": "https://pet-finder-api-581219442968.us-central1.run.app",
            "type": "string"
        },
        {
            "key": "api_key",
            "value": "your-api-key-here",
            "type": "string"
        }
    ],
    "item": [
        {
            "name": "Public Endpoints",
            "item": [
                {
                    "name": "Get API Info",
                    "request": {
                        "method": "GET",
                        "header": [],
                        "url": {
                            "raw": "{{base_url}}/",
                            "host": ["{{base_url}}"],
                            "path": [""]
                        }
                    }
                },
                {
                    "name": "Health Check",
                    "request": {
                        "method": "GET",
                        "header": [],
                        "url": {
                            "raw": "{{base_url}}/health",
                            "host": ["{{base_url}}"],
                            "path": ["health"]
                        }
                    }
                }
            ]
        },
        {
            "name": "Protected Endpoints",
            "item": [
                {
                    "name": "Predict Pet",
                    "request": {
                        "method": "POST",
                        "header": [
                            {
                                "key": "Content-Type",
                                "value": "application/json"
                            }
                        ],
                        "body": {
                            "mode": "raw",
                            "raw": "{\n    \"image\": \"data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEASABIAAD...\"\n}"
                        },
                        "url": {
                            "raw": "{{base_url}}/predict",
                            "host": ["{{base_url}}"],
                            "path": ["predict"]
                        }
                    }
                },
                {
                    "name": "Search Similar Pets",
                    "request": {
                        "method": "POST",
                        "header": [
                            {
                                "key": "Content-Type",
                                "value": "application/json"
                            }
                        ],
                        "body": {
                            "mode": "raw",
                            "raw": "{\n    \"image\": \"data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEASABIAAD...\",\n    \"top_k\": 10\n}"
                        },
                        "url": {
                            "raw": "{{base_url}}/search_similar",
                            "host": ["{{base_url}}"],
                            "path": ["search_similar"]
                        }
                    }
                },
                {
                    "name": "Get Statistics",
                    "request": {
                        "method": "GET",
                        "header": [],
                        "url": {
                            "raw": "{{base_url}}/stats",
                            "host": ["{{base_url}}"],
                            "path": ["stats"]
                        }
                    }
                }
            ]
        }
    ]
}
```

### Using the Postman Collection

1. **Set Variables:**
   - Click on the collection name
   - Go to "Variables" tab
   - Update `api_key` with your actual API key

2. **Test Endpoints:**
   - Public endpoints work without configuration
   - Protected endpoints automatically use the API key

3. **Add Test Images:**
   - For predict/search endpoints, replace the sample base64 with your actual image data
   - Use Postman's file upload feature to convert images to base64

---

## Usage Notes

### Best Practices

1. **Image Optimization:**
   - Resize large images before uploading (recommended: 800x800 max)
   - Use JPEG format for photos (smaller file size)
   - Ensure good lighting and clear pet visibility

2. **Error Handling:**
   - Always check response status codes
   - Implement exponential backoff for retries
   - Log errors for debugging

3. **Performance:**
   - Cache results when appropriate
   - Batch requests when possible
   - Monitor rate limits

### Limitations

- Maximum image size: 10MB
- Supported pets: Cats and dogs only
- Feature extraction: 2048-dimensional vectors
- Similarity search: Compares against all stored pets (may be slow with large databases)

### Troubleshooting

**"No pet detected" for valid pet images:**
- Ensure pet is clearly visible
- Check image isn't too dark/blurry
- Pet should occupy at least 20% of image

**401 Unauthorized errors:**
- Verify API key is correct
- Check header name (X-API-Key)
- Ensure no extra spaces in key

**Slow response times:**
- First request may be slower (cold start)
- Large images take longer to process
- Consider reducing image size

---


## Changelog

### v1.0 (Current)
- Initial release
- Pet detection (cats and dogs)
- Feature extraction and storage
- Similarity search
- Basic statistics

---

*Last Updated: July 2025*
