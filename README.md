# HelloWash API Documentation

Official HelloWash API Integration Documentation repository for integrating with HelloWash's public APIs.

## Table of Contents

- [Getting Started](#getting-started)
- [API Documentation](#api-documentation)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [Support](#support)

## Getting Started

### Authentication

All API requests require authentication using an API key in the header:

```text
x-hellowash-api-key: <your-hellowash-api-key>
```

API keys are corporate-specific and automatically handle data access permissions. Corporate filtering is handled automatically - no `corporateId` parameter needed. Each API key is tied to a specific corporate and can only access that corporate's data. Contact your HelloWash account manager to request an API key.

### Base URL

```text
https://restacall.com/api/public
```

### Quick Start

```bash
curl -H "x-hellowash-api-key: your-api-key-here" \
  https://restacall.com/api/public/locations
```

## API Documentation

### Corporate Ticket Analytics API

[Corporate Ticket Analytics API Documentation](./corporate-ticket-analytics-api.md)

**Available Endpoints:**

- `GET /api/public/locations` - Retrieves all locations accessible to the API key's corporate
- `GET /api/public/tickets/stats` - Retrieves ticket statistics with filtering options
- `GET /api/public/tickets/details` - Retrieves detailed ticket information with pagination (requires `read_tickets` permission)

## Error Handling

All endpoints return errors in the following format:

```json
{
  "status": "error",
  "message": "Error description"
}
```

**Common HTTP Status Codes:**

- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `429` - Too Many Requests
- `500` - Internal Server Error

## Rate Limiting

API requests are subject to rate limiting based on your API key configuration. If you exceed the rate limit, you'll receive a `429 Too Many Requests` response.

## Support

- **API Key Requests:** Contact your HelloWash account manager
- **Technical Support:** Contact your HelloWash support representative
- **Documentation:** [Corporate Ticket Analytics API Documentation](./corporate-ticket-analytics-api.md)
