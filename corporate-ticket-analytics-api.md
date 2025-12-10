# Corporate Ticket Analytics API Documentation

## Overview

The Corporate Ticket Analytics API provides secure endpoints for third-party systems to access ticket data, statistics, and analytics. This API is designed for automated systems, email integrations, and external reporting tools.

## Authentication

The Public API uses API key authentication with automatic corporate isolation.

**Header Required:**

```text
x-hellowash-api-key: <your-hellowash-api-key>
```

**Important Notes:**

- API keys are corporate-specific and automatically determine data access permissions
- Corporate filtering is handled automatically - no `corporateId` parameter needed
- API keys have specific permissions that control which endpoints can be accessed
- Each API key is tied to a specific corporate and can only access that corporate's data

## Endpoints

### GET /api/public/locations

Retrieves all locations accessible to the API key's corporate.

#### Example Request

```bash
curl -H "x-hellowash-api-key: your-api-key-here" \
  https://restacall.com/api/public/locations
```

#### Response Format

```json
{
  "status": "success",
  "data": {
    "corporateId": "corp_example_abc123",
    "locations": [
      {
        "id": "loc_example_001",
        "name": "Downtown Branch"
      },
      {
        "id": "loc_example_002",
        "name": "Uptown Branch"
      }
    ],
    "metadata": {
      "totalLocations": 2,
      "retrievedAt": "2024-01-15T10:30:00.000Z"
    }
  }
}
```

### GET /api/public/tickets/stats

Retrieves ticket statistics for the API key's corporate. This endpoint is optimized for email integration and automated reporting.

#### Query Parameters

| Parameter       | Type   | Required | Description                                                             |
| --------------- | ------ | -------- | ----------------------------------------------------------------------- |
| `period`        | string | No       | Time period: "day", "week", "month", "year", "custom" (default: "week") |
| `startDate`     | string | No\*     | Start date in ISO 8601 format (required if period="custom")             |
| `endDate`       | string | No\*     | End date in ISO 8601 format (required if period="custom")               |
| `locationIds`   | string | No       | Comma-separated location IDs or single location ID (optional)           |
| `ticketChannels`| string | No       | Comma-separated ticket channels: "call", "email", "internal", "webform" (optional) |

**Notes:**

- If `locationIds` are provided, they must belong to the API key's corporate
- If no `locationIds` are provided, all locations for the corporate will be included
- Location IDs can be passed as: single ID or comma-separated string

#### Example Requests

**Basic weekly stats:**

```bash
curl -H "x-hellowash-api-key: your-api-key-here" \
  https://restacall.com/api/public/tickets/stats?period=week
```

**Custom date range with specific locations:**

```bash
curl -H "x-hellowash-api-key: your-api-key-here" \
  "https://restacall.com/api/public/tickets/stats?period=custom&startDate=2024-01-01&endDate=2024-01-07&locationIds=loc_example_001,loc_example_002"
```

**Monthly stats for all corporate locations:**

```bash
curl -H "x-hellowash-api-key: your-api-key-here" \
  https://restacall.com/api/public/tickets/stats?period=month
```

#### Response Format

```json
{
  "status": "success",
  "data": {
    "total": 40,
    "open": 20,
    "resolved": 20,
    "inProgress": 12,
    "pending": 8,
    "byChannel": {
      "call": {
        "total": 15,
        "open": 8,
        "resolved": 7,
        "inProgress": 5,
        "pending": 3
      },
      "email": {
        "total": 15,
        "open": 7,
        "resolved": 8,
        "inProgress": 4,
        "pending": 3
      },
      "internal": {
        "total": 5,
        "open": 3,
        "resolved": 2,
        "inProgress": 2,
        "pending": 1
      },
      "webform": {
        "total": 5,
        "open": 2,
        "resolved": 3,
        "inProgress": 1,
        "pending": 1
      }
    },
    "metadata": {
      "period": "week",
      "query": {
        "startDate": null,
        "endDate": null,
        "locationCount": 2
      },
      "retrievedAt": "2024-01-15T10:30:00.000Z"
    }
  }
}
```

### GET /api/public/tickets/details

Retrieves detailed ticket information with pagination for the API key's corporate.

#### Permissions Required

- `read_tickets`

#### Query Parameters

| Parameter       | Type   | Required | Description                                                                            |
| --------------- | ------ | -------- | -------------------------------------------------------------------------------------- |
| `period`        | string | No       | Time period: "day", "week", "month", "year", "custom" (default: "week")                |
| `startDate`     | string | No\*     | Start date in ISO 8601 format (required if period="custom")                            |
| `endDate`       | string | No\*     | End date in ISO 8601 format (required if period="custom")                              |
| `status`        | string | No       | Filter by status: "open", "in_progress", "pending", "resolved", "all" (default: "open") |
| `locationIds`   | string | No       | Comma-separated location IDs or single location ID (optional)                          |
| `ticketChannels`| string | No       | Comma-separated ticket channels: "call", "email", "internal", "webform" (optional)     |
| `page`          | number | No       | Page number for pagination (default: 1)                                                |
| `limit`         | number | No       | Number of tickets per page, max 500 (default: 100)                                     |

#### Example Request

```bash
curl -H "x-hellowash-api-key: your-api-key-here" \
  "https://restacall.com/api/public/tickets/details?period=week&status=open&page=1&limit=50"
```

#### Response Format

```json
{
  "status": "success",
  "data": {
    "tickets": [
      {
        "title": "Customer reported issue",
        "callerNumber": "+15551234567",
        "priority": "medium",
        "status": "open",
        "assignedTo": {
          "username": "user.example"
        },
        "locationId": {
          "name": "Downtown Branch"
        },
        "timestamps": {
          "created_at": "2024-01-15T09:00:00.000Z",
          "updated_at": "2024-01-15T10:00:00.000Z"
        }
      }
    ],
    "totalTickets": 150,
    "totalPages": 3,
    "currentPage": 1,
    "limit": 50,
    "breakdown": {
      "call": 60,
      "email": 50,
      "internal": 25,
      "webform": 15
    },
    "metadata": {
      "period": "week",
      "query": {
        "startDate": null,
        "endDate": null,
        "status": "open",
        "locationCount": 5
      },
      "retrievedAt": "2024-01-15T10:30:00.000Z"
    }
  }
}
```

## Security Features

1. **Corporate Isolation:**
   - Each API key is tied to a specific corporate
   - No cross-corporate data access possible
   - Corporate filtering is automatic and enforced at the API level

2. **Location Validation:**
   - Provided location IDs are automatically validated against the corporate
   - Invalid location IDs are filtered out with warnings
   - If no valid location IDs remain, an error is returned

3. **Permission-Based Access:**
   - API keys have specific permissions that control endpoint access
   - Each endpoint validates required permissions before processing

4. **Automatic Data Filtering:**
   - All queries are automatically filtered by the API key's corporate
   - No manual corporate ID management required

## Usage Examples

### Email Integration Example

```javascript
// Example email template integration
const API_BASE_URL = "https://restacall.com/api/public";
const API_KEY = "your-api-key-here";

const response = await fetch(`${API_BASE_URL}/tickets/stats?period=week`, {
  headers: {
    "x-hellowash-api-key": API_KEY,
  },
});

const data = await response.json();

if (data.status === "success") {
  const { total, open, resolved, inProgress, pending } = data.data;

  const emailBody = `
Hi Team,

This week we have ${open} out of ${total} HelloWash tickets open, please make sure you are working on them.

Breakdown:
- In Progress: ${inProgress} tickets
- Pending: ${pending} tickets
- Resolved: ${resolved} tickets

Overall completion rate: ${Math.round((resolved / total) * 100)}%

Best regards,
HelloWash System
  `;
}
```

### Automated Reporting Example

```javascript
// Example automated reporting script
const API_BASE_URL = "https://restacall.com/api/public";
const API_KEY = "your-api-key-here";

async function generateWeeklyReport() {
  try {
    // Get locations
    const locationsResponse = await fetch(`${API_BASE_URL}/locations`, {
      headers: { "x-hellowash-api-key": API_KEY },
    });
    const locationsData = await locationsResponse.json();

    // Get stats for each location
    const statsResponse = await fetch(`${API_BASE_URL}/tickets/stats?period=week`, {
      headers: { "x-hellowash-api-key": API_KEY },
    });
    const statsData = await statsResponse.json();

    // Generate report
    console.log("Weekly Report:");
    console.log(`Total Tickets: ${statsData.data.total}`);
    console.log(`Open Tickets: ${statsData.data.open}`);
    console.log(`Resolved Tickets: ${statsData.data.resolved}`);
  } catch (error) {
    console.error("Error generating report:", error);
  }
}
```

## Error Handling

All endpoints return errors in the following format:

```json
{
  "status": "error",
  "message": "Error description"
}
```

Common HTTP status codes:

- `400` - Bad Request (invalid parameters)
- `401` - Unauthorized (invalid or missing API key)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found (resource not found)
- `429` - Too Many Requests (rate limit exceeded)
- `500` - Internal Server Error

## Rate Limiting

API requests are subject to rate limiting based on your API key configuration. If you exceed the rate limit, you'll receive a `429 Too Many Requests` response.

## Support

- **API Key Requests:** Contact your HelloWash account manager
- **Technical Support:** Contact your HelloWash support representative
