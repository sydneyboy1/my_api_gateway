# PHP-Based API Gateway

## Overview

This project implements a simple API Gateway using PHP within a XAMPP environment. The gateway serves as an intermediary between clients and backend microservices, handling essential API management functions including routing, authentication, rate limiting, and logging.

## Project Structure

```
my_api_gateway/
├── gateway.php         # Main gateway script
├── config.php          # Configuration settings
├── services/           # Backend microservices
│   ├── service_users.php
│   └── service_products.php
├── logs/               # Request logs
├── ratelimit_data/     # Rate limiting data storage
├── .htaccess           # URL rewriting rules
└── docs.html           # API documentation
```

## Setup Instructions

### Prerequisites

- XAMPP (or equivalent like WAMP, MAMP) installed and running
- Apache with `mod_rewrite` enabled
- Basic understanding of PHP and HTTP concepts

### Installation Steps

1. Clone or extract this repository into your XAMPP `htdocs` directory:
   ```
   /path/to/xampp/htdocs/my_api_gateway/
   ```

2. Ensure Apache's `mod_rewrite` module is enabled:
   - Open XAMPP Control Panel
   - Click "Config" for Apache
   - Select "httpd.conf"
   - Verify the line `LoadModule rewrite_module modules/mod_rewrite.so` is uncommented
   - Restart Apache if you made changes

3. Make sure the following directories have write permissions for the web server:
   ```
   /my_api_gateway/logs/
   /my_api_gateway/ratelimit_data/
   ```

4. Access the API documentation at:
   ```
   http://localhost/my_api_gateway/docs.html
   ```

## API Authentication

All API requests require authentication using an API key provided in the `X-API-Key` HTTP header.

### Available API Keys

| API Key | Associated User |
|---------|----------------|
| key123  | UserA          |
| key456  | UserB          |
| key789  | UserC          |

## Available Endpoints

| Endpoint | Description |
|----------|-------------|
| GET `/api/users` | Returns a list of all users |
| GET `/api/products` | Returns a list of all products |
| GET `/api/dashboard` | Returns combined data from both services (bonus feature) |

## Rate Limiting

The API Gateway implements rate limiting to protect the backend services:
- 10 requests per minute per API key
- After exceeding this limit, requests will receive a 429 (Too Many Requests) response

## Testing the API

### Using cURL

#### Test Successful Requests

```bash
# Get users data
curl -H "X-API-Key: key123" http://localhost/my_api_gateway/api/users

# Get products data
curl -H "X-API-Key: key123" http://localhost/my_api_gateway/api/products

# Get dashboard data (combined)
curl -H "X-API-Key: key123" http://localhost/my_api_gateway/api/dashboard
```

#### Test Authentication Errors

```bash
# Missing API key
curl http://localhost/my_api_gateway/api/users

# Invalid API key
curl -H "X-API-Key: invalid_key" http://localhost/my_api_gateway/api/users
```

#### Test Rate Limiting

```bash
# Send multiple requests rapidly to trigger rate limiting
for i in {1..15}; do curl -H "X-API-Key: key123" http://localhost/my_api_gateway/api/users; echo; done
```

#### Test Invalid Endpoint

```bash
curl -H "X-API-Key: key123" http://localhost/my_api_gateway/api/nonexistent
```

### Using Postman

1. Create a new request
2. Set the HTTP method to `GET`
3. Enter the URL (e.g., `http://localhost/my_api_gateway/api/users`)
4. Add a header:
   - Key: `X-API-Key`
   - Value: `key123`
5. Click "Send" to execute the request

## Implementation Details

### Features Implemented

1. **Backend Services**
   - Simulated microservices providing user and product data
   - Clean JSON responses with appropriate content type headers

2. **URL Routing**
   - Apache's mod_rewrite directs all `/api/` requests to the gateway
   - Gateway analyzes the request path and forwards to appropriate service
   - Protection against direct access to service scripts and sensitive directories

3. **Authentication**
   - API key validation via custom HTTP header
   - 401 Unauthorized responses for invalid/missing keys

4. **Rate Limiting**
   - File-based tracking of request counts per API key
   - Time-windowed approach (10 requests per minute)
   - 429 Too Many Requests responses when limits are exceeded

5. **Centralized Logging**
   - Detailed logging of all requests with timestamps
   - Records client IP, API key used, requested path, and HTTP status code
   - All logs stored in a central log file

6. **API Documentation**
   - Clear HTML documentation of available endpoints
   - Examples of requests and responses
   - Description of authentication and rate limiting

7. **Bonus: Response Aggregation**
   - Dashboard endpoint combining data from multiple services
   - Single request providing comprehensive data

### Security Considerations

- API keys for authentication
- Prevention of direct access to service scripts
- Rate limiting to prevent abuse
- No exposure of sensitive backend details

## Limitations and Potential Improvements

- **Database Integration**: Replace file-based storage with a database for API keys and rate limiting
- **Caching**: Add response caching to improve performance
- **Load Balancing**: Add support for multiple instances of each microservice
- **Monitoring**: Implement detailed metrics and monitoring
- **HTTPS**: Configure SSL/TLS for secure communication
- **Token-based Auth**: Replace simple API keys with more robust authentication like JWT
- **Service Discovery**: Dynamic discovery of available microservices
- **Circuit Breaking**: Implement circuit breakers for resilience

## Troubleshooting

1. **403 Forbidden Error**
   - Check that mod_rewrite is enabled in Apache
   - Verify .htaccess file is present and correctly formatted

2. **500 Internal Server Error**
   - Check PHP error logs
   - Verify directory permissions for logs and ratelimit_data

3. **API Gateway Not Routing Properly**
   - Confirm URL format is correct
   - Check that the path in $_GET['request_path'] matches expected values

4. **Rate Limiting Issues**
   - Clear the ratelimit_data directory to reset all counters
   - Check file permissions in the ratelimit_data directory
