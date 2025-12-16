\# Flex API Integration Specification



\## Document Overview



This document provides a complete specification for the API endpoints that Flex must implement to integrate with our webforms platform. The integration enables user authentication, account management, vehicle information retrieval, subscription management, and retention offers.



---



\## Table of Contents



1\. \[Authentication](#authentication)

2\. \[User Lookup Endpoints](#user-lookup-endpoints)

3\. \[Account Management Endpoints](#account-management-endpoints)

4\. \[Vehicle Information Endpoints](#vehicle-information-endpoints)

5\. \[Subscription Management Endpoints](#subscription-management-endpoints)

6\. \[Vehicle Operations Endpoints](#vehicle-operations-endpoints)

7\. \[Error Handling](#error-handling)

8\. \[Response Codes](#response-codes)



---



\## Authentication



\### Overview



Our platform uses a tenant-based authentication model with header-based credentials. Each corporate client receives their own tenant credentials for isolation and security.



\### Authentication Headers



All API requests will include the following headers:



| Header | Description | Required | Example |

|--------|-------------|----------|---------|

| `X-Tenant` | Unique tenant identifier for the corporate client | Always | `acme-corp-123` |

| `X-Tenant-API-Key` | API key associated with the tenant | Always | `sk\_live\_abc123xyz789` |

| `X-User-Id` | User ID for user-authenticated operations | For `/api-user/\*` endpoints | `7h881565xcph6r2e0ukky472bdpawxyg` |

| `X-Account-Id` | Account ID for multi-account users | For `/api-user/\*` endpoints | `acc\_98765` or `USE-DEFAULT-ACCOUNT` |



\### Authentication Flow



1\. \*\*Initial Lookup\*\*: User provides phone number or email on the portal

2\. \*\*User Identification\*\*: We call `/api/get-users-by-phone` or `/api/get-user-by-email` with tenant credentials

3\. \*\*User Context Established\*\*: We extract the `user\_id` from the response

4\. \*\*Authenticated Operations\*\*: We include the `user\_id` in the `X-User-Id` header for subsequent operations



\### Tenant Management



\- Each corporate client is provisioned with a unique `X-Tenant` identifier and `X-Tenant-API-Key`

\- Tenant credentials must be kept secure and rotated periodically

\- All data must be isolated by tenant to ensure privacy and security



\### Account ID Usage



The `X-Account-Id` header is used for users with multiple accounts:

\- If the user has only one account, use `USE-DEFAULT-ACCOUNT`

\- If the user has multiple accounts, we first call `/api-user/get-accounts-by-user` to let them select

\- Once selected, we use the specific account ID in subsequent operations



---



\## User Lookup Endpoints



\### 1. Get Users by Phone Number



Lookup users associated with a phone number.



\*\*Endpoint\*\*: `POST /api/get-users-by-phone`



\*\*Purpose\*\*: This is the primary entry point for users accessing the portal via phone number. We use this to identify the user and retrieve their user ID for subsequent authenticated operations.



\#### Request Headers

```

Content-Type: application/json

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

```



\#### Request Body

```json

{

&nbsp; "phone": "5551234567"

}

```



\*\*Field Descriptions\*\*:

\- `phone` (string, required): Phone number without country code prefix. Format: 10 digits (e.g., "5551234567")



\#### Success Response (200 OK)

```json

{

&nbsp; "data": \[

&nbsp;   {

&nbsp;     "id": "7h881565xcph6r2e0ukky472bdpawxyg",

&nbsp;     "email": "john.doe@example.com",

&nbsp;     "phone": "5551234567",

&nbsp;     "first\_name": "John",

&nbsp;     "last\_name": "Doe",

&nbsp;     "created\_at": "2024-01-15T10:30:00Z"

&nbsp;   }

&nbsp; ]

}

```



\*\*Field Descriptions\*\*:

\- `id` (string): Unique user identifier (critical for authenticated operations)

\- `email` (string): User's email address

\- `phone` (string): User's phone number

\- `first\_name` (string): User's first name

\- `last\_name` (string): User's last name

\- `created\_at` (string): ISO 8601 timestamp of account creation



\#### Error Response (404 Not Found)

```json

{

&nbsp; "error": "No users found for phone number"

}

```



\#### Error Response (400 Bad Request)

```json

{

&nbsp; "error": "Invalid phone number format"

}

```



\#### Notes

\- Phone numbers should be normalized before lookup (remove +1 prefix, spaces, dashes)

\- Multiple users may share the same phone number in some cases (return array)

\- The `id` field is essential and will be used as `X-User-Id` in subsequent requests



---



\### 2. Get User by Email



Lookup a user by email address.



\*\*Endpoint\*\*: `POST /api/get-user-by-email`



\*\*Purpose\*\*: Alternative entry point for users who prefer to identify themselves via email instead of phone number. Used in the same way as phone lookup to establish user identity.



\#### Request Headers

```

Content-Type: application/json

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

```



\#### Request Body

```json

{

&nbsp; "email": "john.doe@example.com"

}

```



\*\*Field Descriptions\*\*:

\- `email` (string, required): User's email address (case-insensitive)



\#### Success Response (200 OK)

```json

{

&nbsp; "id": "7h881565xcph6r2e0ukky472bdpawxyg",

&nbsp; "email": "john.doe@example.com",

&nbsp; "phone": "5551234567",

&nbsp; "first\_name": "John",

&nbsp; "last\_name": "Doe",

&nbsp; "created\_at": "2024-01-15T10:30:00Z"

}

```



\*\*Field Descriptions\*\*: Same as phone lookup response



\#### Error Response (404 Not Found)

```json

{

&nbsp; "error": "No user found for email address"

}

```



\#### Notes

\- Email lookup returns a single user object (not an array)

\- Email addresses should be case-insensitive

\- The `id` field from response becomes the `X-User-Id` for authenticated operations



---



\## Account Management Endpoints



\### 3. Get Accounts by User



Retrieve all accounts associated with a user.



\*\*Endpoint\*\*: `GET /api-user/get-accounts-by-user`



\*\*Purpose\*\*: Some users may have multiple accounts (e.g., personal and business). This endpoint retrieves all accounts so the user can select which one to manage. We present this as an account selection screen before proceeding with vehicle or subscription operations.



\#### Request Headers

```

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

X-User-Id: <user\_id>

X-Account-Id: USE-DEFAULT-ACCOUNT

```



\#### Success Response (200 OK)

```json

{

&nbsp; "accounts": \[

&nbsp;   {

&nbsp;     "id": "acc\_12345",

&nbsp;     "name": "Personal Account",

&nbsp;     "type": "individual",

&nbsp;     "status": "active",

&nbsp;     "created\_at": "2024-01-15T10:30:00Z"

&nbsp;   },

&nbsp;   {

&nbsp;     "id": "acc\_67890",

&nbsp;     "name": "Business Account",

&nbsp;     "type": "business",

&nbsp;     "status": "active",

&nbsp;     "created\_at": "2024-03-20T14:20:00Z"

&nbsp;   }

&nbsp; ]

}

```



\*\*Field Descriptions\*\*:

\- `id` (string): Unique account identifier

\- `name` (string): Display name for the account

\- `type` (string): Account type (individual, business, etc.)

\- `status` (string): Account status (active, suspended, cancelled)

\- `created\_at` (string): ISO 8601 timestamp



\#### Error Response (404 Not Found)

```json

{

&nbsp; "error": "No accounts found for user"

}

```



\#### Notes

\- If only one account is returned, we proceed directly without showing account selection

\- If multiple accounts exist, we present a selection UI and use the chosen account ID in subsequent requests

\- The selected account ID becomes the `X-Account-Id` header value for subsequent operations



---



\## Vehicle Information Endpoints



\### 4. Get Vehicle Data by Account



Retrieve all vehicles and their subscriptions for a user's account.



\*\*Endpoint\*\*: `GET /api-user/get-vehicle-data-by-account`



\*\*Purpose\*\*: This is the primary endpoint for retrieving vehicle information. We use this to display the user's vehicles and their subscription status, especially in the cancellation flow where users need to select which vehicle's subscription to cancel.



\#### Request Headers

```

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

X-User-Id: <user\_id>

X-Account-Id: <account\_id>

```



\*\*Note\*\*: `X-Account-Id` can be `USE-DEFAULT-ACCOUNT` for single-account users or a specific account ID for multi-account users.



\#### Success Response (200 OK)

```json

{

&nbsp; "vehicleData": \[

&nbsp;   {

&nbsp;     "vehicle": {

&nbsp;       "id": 12345,

&nbsp;       "license\_plate\_number": "ABC123",

&nbsp;       "license\_plate\_state": "CA",

&nbsp;       "year": 2022,

&nbsp;       "make": "Toyota",

&nbsp;       "model": "Camry",

&nbsp;       "color": "Silver",

&nbsp;       "vin": "1HGBH41JXMN109186",

&nbsp;       "created\_by\_user\_id": "7h881565xcph6r2e0ukky472bdpawxyg",

&nbsp;       "created\_at": "2024-01-15T10:30:00Z"

&nbsp;     },

&nbsp;     "subscription": {

&nbsp;       "id": 9876,

&nbsp;       "stripe\_subscription\_id": "sub\_1234567890",

&nbsp;       "status": "active",

&nbsp;       "plan\_name": "Premium Wash Plan",

&nbsp;       "plan\_type": "unlimited",

&nbsp;       "price": 29.99,

&nbsp;       "currency": "USD",

&nbsp;       "billing\_cycle": "monthly",

&nbsp;       "current\_period\_start": "2024-11-01T00:00:00Z",

&nbsp;       "current\_period\_end": "2024-12-01T00:00:00Z",

&nbsp;       "cancel\_at\_period\_end": false,

&nbsp;       "location\_id": "loc\_abc123"

&nbsp;     }

&nbsp;   },

&nbsp;   {

&nbsp;     "vehicle": {

&nbsp;       "id": 12346,

&nbsp;       "license\_plate\_number": "XYZ789",

&nbsp;       "license\_plate\_state": "CA",

&nbsp;       "year": 2021,

&nbsp;       "make": "Honda",

&nbsp;       "model": "Accord",

&nbsp;       "color": "Black",

&nbsp;       "vin": "1HGBH41JXMN109187",

&nbsp;       "created\_by\_user\_id": "7h881565xcph6r2e0ukky472bdpawxyg",

&nbsp;       "created\_at": "2024-02-20T15:45:00Z"

&nbsp;     },

&nbsp;     "subscription": null

&nbsp;   }

&nbsp; ]

}

```



\*\*Field Descriptions\*\*:



\*\*Vehicle Object\*\*:

\- `id` (integer): Unique vehicle identifier

\- `license\_plate\_number` (string): License plate number

\- `license\_plate\_state` (string): State/province code

\- `year` (integer): Vehicle year

\- `make` (string): Vehicle manufacturer

\- `model` (string): Vehicle model

\- `color` (string): Vehicle color

\- `vin` (string): Vehicle identification number

\- `created\_by\_user\_id` (string): User ID who added this vehicle

\- `created\_at` (string): ISO 8601 timestamp



\*\*Subscription Object\*\* (null if no active subscription):

\- `id` (integer): Subscription record ID

\- `stripe\_subscription\_id` (string): Payment processor subscription ID (critical for cancellation)

\- `status` (string): Subscription status (active, paused, cancelled, past\_due)

\- `plan\_name` (string): Display name of subscription plan

\- `plan\_type` (string): Plan type (unlimited, per-wash, etc.)

\- `price` (number): Subscription price

\- `currency` (string): Currency code (USD, CAD, etc.)

\- `billing\_cycle` (string): Billing frequency (monthly, annual)

\- `current\_period\_start` (string): Current billing period start date

\- `current\_period\_end` (string): Current billing period end date

\- `cancel\_at\_period\_end` (boolean): Whether subscription will cancel at period end

\- `location\_id` (string): Associated location/facility ID



\#### Error Response (404 Not Found)

```json

{

&nbsp; "error": "No vehicles found for user"

}

```



\#### Empty Response (200 OK)

```json

{

&nbsp; "vehicleData": \[]

}

```



\#### Notes

\- Each vehicle may or may not have an associated subscription (subscription can be null)

\- The `created\_by\_user\_id` field is used to verify vehicle ownership

\- The `stripe\_subscription\_id` is required for cancellation operations

\- The `location\_id` is needed for retention offer operations



---



\### 5. Get Vehicle Data by License Plate



Retrieve vehicle information by license plate number.



\*\*Endpoint\*\*: `POST /api/get-vehicle-data-by-license-plate`



\*\*Purpose\*\*: Alternative vehicle lookup method when the user knows their license plate but may not have logged in yet. Used in simplified cancellation flows where users provide their license plate directly.



\#### Request Headers

```

Content-Type: application/json

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

```



\#### Request Body

```json

{

&nbsp; "license\_plate\_number": "ABC123"

}

```



\*\*Field Descriptions\*\*:

\- `license\_plate\_number` (string, required): License plate number (case-insensitive)



\*\*Optional Fields\*\*:

\- `license\_plate\_state` (string, optional): State code for disambiguation if needed



\#### Success Response (200 OK)

```json

{

&nbsp; "vehicleData": \[

&nbsp;   {

&nbsp;     "vehicle": {

&nbsp;       "id": 12345,

&nbsp;       "license\_plate\_number": "ABC123",

&nbsp;       "license\_plate\_state": "CA",

&nbsp;       "year": 2022,

&nbsp;       "make": "Toyota",

&nbsp;       "model": "Camry",

&nbsp;       "color": "Silver",

&nbsp;       "vin": "1HGBH41JXMN109186",

&nbsp;       "created\_by\_user\_id": "7h881565xcph6r2e0ukky472bdpawxyg",

&nbsp;       "created\_at": "2024-01-15T10:30:00Z"

&nbsp;     },

&nbsp;     "subscription": {

&nbsp;       "id": 9876,

&nbsp;       "stripe\_subscription\_id": "sub\_1234567890",

&nbsp;       "status": "active",

&nbsp;       "plan\_name": "Premium Wash Plan",

&nbsp;       "plan\_type": "unlimited",

&nbsp;       "price": 29.99,

&nbsp;       "currency": "USD",

&nbsp;       "billing\_cycle": "monthly",

&nbsp;       "current\_period\_start": "2024-11-01T00:00:00Z",

&nbsp;       "current\_period\_end": "2024-12-01T00:00:00Z",

&nbsp;       "cancel\_at\_period\_end": false,

&nbsp;       "location\_id": "loc\_abc123"

&nbsp;     }

&nbsp;   }

&nbsp; ]

}

```



\*\*Field Descriptions\*\*: Same as "Get Vehicle Data by Account" endpoint



\#### Error Response (404 Not Found)

```json

{

&nbsp; "error": "No vehicle found for license plate"

}

```



\#### Notes

\- Returns vehicle with subscription data in the same format as the account-based lookup

\- License plate matching should be case-insensitive

\- May return multiple vehicles if license plate is not unique (rare, but possible)



---



\## Subscription Management Endpoints



\### 6. Cancel Subscription



Cancel a user's subscription for a specific vehicle.



\*\*Endpoint\*\*: `POST /api-user/subscription/cancel-subscription`



\*\*Purpose\*\*: This is the primary cancellation endpoint. When a user decides to cancel their membership (after declining retention offers or when no retention offer is available), we call this endpoint to process the cancellation. The cancellation can be immediate or scheduled for the end of the billing period.



\#### Request Headers

```

Content-Type: application/json

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

X-User-Id: <user\_id>

X-Account-Id: <account\_id>

```



\#### Request Body

```json

{

&nbsp; "subscription\_id": "sub\_1234567890",

&nbsp; "cancellation\_reason\_id": "1",

&nbsp; "cancel\_at\_period\_end": true

}

```



\*\*Field Descriptions\*\*:

\- `subscription\_id` (string, required): The stripe subscription ID from the vehicle data

\- `cancellation\_reason\_id` (string, required): Reason code for the cancellation (see notes for mapping)

\- `cancel\_at\_period\_end` (boolean, required): If true, subscription remains active until end of current billing period; if false, cancels immediately



\*\*Cancellation Reason IDs\*\*:

\- `"1"` - Moving/Relocating

\- `"2"` - Too expensive

\- `"3"` - Not using service enough

\- `"4"` - Poor service quality

\- `"5"` - Switching to competitor

\- `"890"` - Other (custom reason)



\#### Success Response (200 OK)

```json

{

&nbsp; "success": true,

&nbsp; "subscription\_id": "sub\_1234567890",

&nbsp; "status": "cancelled",

&nbsp; "cancelled\_at": "2024-12-01T00:00:00Z",

&nbsp; "cancel\_at\_period\_end": true,

&nbsp; "current\_period\_end": "2024-12-01T00:00:00Z"

}

```



\*\*Field Descriptions\*\*:

\- `success` (boolean): Operation success indicator

\- `subscription\_id` (string): The cancelled subscription ID

\- `status` (string): New subscription status

\- `cancelled\_at` (string): Effective cancellation timestamp

\- `cancel\_at\_period\_end` (boolean): Whether cancellation is scheduled

\- `current\_period\_end` (string): When access ends if scheduled cancellation



\#### Error Response (400 Bad Request)

```json

{

&nbsp; "error": "Subscription not found or already cancelled"

}

```



\#### Error Response (401 Unauthorized)

```json

{

&nbsp; "error": "User not authorized to cancel this subscription"

}

```



\#### Notes

\- We typically set `cancel\_at\_period\_end: true` to allow users to use the service until they've paid for

\- The subscription ID must match a valid, active subscription for the authenticated user

\- The cancellation reason is used for analytics and retention analysis

\- After successful cancellation, we display a confirmation message to the user



---



\### 7. Pause Vehicle Subscription



Temporarily pause a subscription for a specified number of billing cycles.



\*\*Endpoint\*\*: `POST /api-user/subscription/pause-vehicle-subscription`



\*\*Purpose\*\*: Allows users to temporarily pause their subscription rather than cancelling (e.g., going on vacation, vehicle in repair, seasonal usage). The subscription automatically resumes after the specified number of cycles. This is a retention tool that prevents full cancellations.



\#### Request Headers

```

Content-Type: application/json

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

X-User-Id: <user\_id>

X-Account-Id: <account\_id>

```



\#### Request Body

```json

{

&nbsp; "vehicle\_id": 12345,

&nbsp; "number\_of\_cycles": 2,

&nbsp; "comment": "Going on vacation for 2 months"

}

```



\*\*Field Descriptions\*\*:

\- `vehicle\_id` (integer, required): Vehicle ID whose subscription to pause

\- `number\_of\_cycles` (integer, required): Number of billing cycles to pause (1-6 typically)

\- `comment` (string, required): User's reason for pausing (free text)



\#### Success Response (200 OK)

```json

{

&nbsp; "success": true,

&nbsp; "subscription\_id": "sub\_1234567890",

&nbsp; "status": "paused",

&nbsp; "paused\_until": "2025-02-01T00:00:00Z",

&nbsp; "number\_of\_cycles": 2,

&nbsp; "resume\_date": "2025-02-01T00:00:00Z"

}

```



\*\*Field Descriptions\*\*:

\- `success` (boolean): Operation success indicator

\- `subscription\_id` (string): The subscription ID

\- `status` (string): New subscription status (paused)

\- `paused\_until` (string): Date when subscription will resume

\- `number\_of\_cycles` (integer): Number of cycles paused

\- `resume\_date` (string): Auto-resume date



\#### Error Response (400 Bad Request)

```json

{

&nbsp; "error": "Invalid number of cycles (must be between 1 and 6)"

}

```



\#### Notes

\- Pausing prevents billing but maintains the subscription relationship

\- The subscription automatically resumes on the resume\_date

\- Users typically choose 1-3 cycles for vacation, 4-6 for seasonal closures

\- This is offered as an alternative to cancellation in retention flows



---



\### 8. Get Retention Offer



Retrieve a retention offer when a user attempts to cancel their subscription.



\*\*Endpoint\*\*: `POST /api-user/subscription/get-retention-offer`



\*\*Purpose\*\*: When a user initiates cancellation, we call this endpoint to check if there's a retention offer available (e.g., discounted rate, free month, pause option). If an offer exists, we present it to the user before proceeding with cancellation. This is a critical retention tool.



\#### Request Headers

```

Content-Type: application/json

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

X-User-Id: <user\_id>

X-Account-Id: <account\_id>

```



\#### Request Body

```json

{

&nbsp; "subscription\_id": "sub\_1234567890",

&nbsp; "cancellation\_reason\_id": 890,

&nbsp; "location\_id": "loc\_abc123"

}

```



\*\*Field Descriptions\*\*:

\- `subscription\_id` (string, required): The subscription ID being cancelled

\- `cancellation\_reason\_id` (integer, required): Numeric reason code (890 is default/other)

\- `location\_id` (string, required): Location ID from the vehicle's subscription data



\#### Success Response with Offer (200 OK)

```json

{

&nbsp; "id": 5678,

&nbsp; "subscription\_id": "sub\_1234567890",

&nbsp; "offer\_type": "discount",

&nbsp; "description": "We'd like to offer you 50% off for the next 3 months",

&nbsp; "discount\_percentage": 50,

&nbsp; "duration\_months": 3,

&nbsp; "new\_price": 14.99,

&nbsp; "original\_price": 29.99,

&nbsp; "currency": "USD",

&nbsp; "expires\_at": "2024-12-20T23:59:59Z"

}

```



\*\*Field Descriptions\*\*:

\- `id` (integer): Unique offer identifier (needed for accepting/declining)

\- `subscription\_id` (string): Associated subscription ID

\- `offer\_type` (string): Type of offer (discount, pause, free\_month, downgrade)

\- `description` (string): Human-readable offer description

\- `discount\_percentage` (number): Percentage discount (if applicable)

\- `duration\_months` (integer): How long the offer lasts

\- `new\_price` (number): Price with offer applied

\- `original\_price` (number): Current price

\- `currency` (string): Currency code

\- `expires\_at` (string): When the offer expires



\#### Response with No Offer (200 OK)

```json

{

&nbsp; "no\_offer": true

}

```



\#### Error Response (404 Not Found)

```json

{

&nbsp; "error": "Subscription not found"

}

```



\#### Notes

\- Not all subscriptions will have retention offers available

\- If `no\_offer: true` or empty response, we proceed directly to cancellation

\- If offer exists, we display it prominently with accept/decline buttons

\- Offers are typically time-limited and expire if not acted upon

\- The offer `id` is required for the respond endpoint



---



\### 9. Respond to Retention Offer



Accept or decline a retention offer.



\*\*Endpoint\*\*: `POST /api-user/subscription/respond-retention-offer`



\*\*Purpose\*\*: After presenting a retention offer to the user, they can either accept it (apply the offer and keep subscription active) or decline it (proceed with cancellation). This endpoint processes their decision.



\#### Request Headers

```

Content-Type: application/json

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

X-User-Id: <user\_id>

X-Account-Id: <account\_id>

```



\#### Request Body

```json

{

&nbsp; "subscription\_id": "sub\_1234567890",

&nbsp; "cancellation\_reason\_id": 890,

&nbsp; "location\_id": "loc\_abc123",

&nbsp; "retention\_offer\_id": 5678,

&nbsp; "accepted": true

}

```



\*\*Field Descriptions\*\*:

\- `subscription\_id` (string, required): The subscription ID

\- `cancellation\_reason\_id` (integer, required): Original cancellation reason code

\- `location\_id` (string, required): Location ID

\- `retention\_offer\_id` (integer, required): Offer ID from get-retention-offer response

\- `accepted` (boolean, required): true to accept offer, false to decline and proceed with cancellation



\#### Success Response - Accepted (200 OK)

```json

{

&nbsp; "success": true,

&nbsp; "action": "offer\_accepted",

&nbsp; "subscription\_id": "sub\_1234567890",

&nbsp; "status": "active",

&nbsp; "new\_price": 14.99,

&nbsp; "discount\_ends": "2025-03-01T00:00:00Z",

&nbsp; "message": "Your offer has been applied! You'll now pay $14.99/month for the next 3 months."

}

```



\*\*Field Descriptions\*\*:

\- `success` (boolean): Operation success indicator

\- `action` (string): What action was taken (offer\_accepted or offer\_declined)

\- `subscription\_id` (string): The subscription ID

\- `status` (string): Subscription status after action

\- `new\_price` (number): New subscription price (if accepted)

\- `discount\_ends` (string): When discount expires (if accepted)

\- `message` (string): User-friendly confirmation message



\#### Success Response - Declined (200 OK)

```json

{

&nbsp; "success": true,

&nbsp; "action": "offer\_declined",

&nbsp; "subscription\_id": "sub\_1234567890",

&nbsp; "status": "cancelled",

&nbsp; "cancelled\_at": "2024-12-01T00:00:00Z",

&nbsp; "message": "Your subscription has been cancelled."

}

```



\#### Error Response (400 Bad Request)

```json

{

&nbsp; "error": "Invalid or expired retention offer"

}

```



\#### Notes

\- If `accepted: true`, the offer is applied and subscription continues with new terms

\- If `accepted: false`, the subscription is cancelled (same effect as calling cancel endpoint)

\- Offer IDs are single-use and expire after response or timeout

\- We track acceptance rates for retention optimization



---



\## Vehicle Operations Endpoints



\### 10. Create Vehicle



Add a new vehicle to a user's account.



\*\*Endpoint\*\*: `POST /api-user/create-vehicle`



\*\*Purpose\*\*: Allows users to add additional vehicles to their account. This is used when users purchase new vehicles or want to add a second vehicle to their membership. After creation, the vehicle is available for subscription enrollment.



\#### Request Headers

```

Content-Type: application/json

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

X-User-Id: <user\_id>

X-Account-Id: <account\_id>

```



\#### Request Body

```json

{

&nbsp; "vif\_id": 123456,

&nbsp; "color\_id": 5,

&nbsp; "license\_plate\_number": "XYZ789",

&nbsp; "license\_plate\_state": "CA",

&nbsp; "vin": "1HGBH41JXMN109187",

&nbsp; "model\_label\_override": "Accord Sport"

}

```



\*\*Field Descriptions\*\*:

\- `vif\_id` (integer, required): Vehicle Identification File ID - references a pre-defined vehicle make/model/year combination

\- `color\_id` (integer, required): Color ID from a predefined color list

\- `license\_plate\_number` (string, required): License plate number

\- `license\_plate\_state` (string, required): State/province code

\- `vin` (string, optional): Vehicle identification number (17 characters)

\- `model\_label\_override` (string, optional): Custom model label if user wants to specify trim level



\*\*Note\*\*: The `vif\_id` and `color\_id` are typically obtained from a separate vehicle search/lookup API that's not covered in this integration spec. For Flex implementation, these can be simple lookup tables or IDs that reference your vehicle database.



\#### Success Response (200 OK)

```json

{

&nbsp; "id": 12347,

&nbsp; "license\_plate\_number": "XYZ789",

&nbsp; "license\_plate\_state": "CA",

&nbsp; "year": 2021,

&nbsp; "make": "Honda",

&nbsp; "model": "Accord Sport",

&nbsp; "color": "Blue",

&nbsp; "vin": "1HGBH41JXMN109187",

&nbsp; "created\_by\_user\_id": "7h881565xcph6r2e0ukky472bdpawxyg",

&nbsp; "created\_at": "2024-12-16T10:00:00Z",

&nbsp; "status": "active"

}

```



\*\*Field Descriptions\*\*:

\- `id` (integer): Newly created vehicle ID

\- Other fields: Same as vehicle object in vehicle data endpoints

\- `status` (string): Vehicle status (active, inactive)



\#### Error Response (400 Bad Request)

```json

{

&nbsp; "error": "Vehicle with this license plate already exists"

}

```



\#### Error Response (422 Unprocessable Entity)

```json

{

&nbsp; "error": "Invalid VIN format"

}

```



\#### Notes

\- License plate must be unique within the account

\- The vehicle is created but does not automatically have a subscription - that's a separate enrollment process

\- The `created\_by\_user\_id` is set to the authenticated user's ID



---



\### 11. Remove Vehicle



Remove a vehicle from a user's account.



\*\*Endpoint\*\*: `POST /api-user/remove-vehicle`



\*\*Purpose\*\*: Allows users to remove vehicles they no longer own or want in their account. If the vehicle has an active subscription, it should be cancelled first (or this endpoint should handle cancellation automatically, depending on your business logic).



\#### Request Headers

```

Content-Type: application/json

X-Tenant: <tenant\_id>

X-Tenant-API-Key: <tenant\_api\_key>

X-User-Id: <user\_id>

X-Account-Id: <account\_id>

```



\#### Request Body

```json

{

&nbsp; "vehicle\_id": 12345

}

```



\*\*Field Descriptions\*\*:

\- `vehicle\_id` (integer, required): The vehicle ID to remove



\#### Success Response (200 OK)

```json

{

&nbsp; "success": true,

&nbsp; "id": 12345,

&nbsp; "license\_plate\_number": "ABC123",

&nbsp; "status": "removed",

&nbsp; "removed\_at": "2024-12-16T10:30:00Z"

}

```



\*\*Field Descriptions\*\*:

\- `success` (boolean): Operation success indicator

\- `id` (integer): The removed vehicle ID

\- `license\_plate\_number` (string): License plate of removed vehicle

\- `status` (string): New status (removed, deleted)

\- `removed\_at` (string): Timestamp of removal



\#### Error Response (400 Bad Request)

```json

{

&nbsp; "error": "Cannot remove vehicle with active subscription"

}

```



\#### Error Response (401 Unauthorized)

```json

{

&nbsp; "error": "User not authorized to remove this vehicle"

}

```



\#### Error Response (404 Not Found)

```json

{

&nbsp; "error": "Vehicle not found"

}

```



\#### Notes

\- Business logic decision: Should this automatically cancel subscriptions, or require cancellation first?

\- Recommended: Require cancellation first to ensure users understand billing implications

\- The vehicle should be soft-deleted (marked as removed) rather than hard-deleted for record keeping

\- Users can only remove vehicles they created or have ownership of



---



\## Error Handling



\### Standard Error Response Format



All error responses should follow this consistent format:



```json

{

&nbsp; "error": "Human-readable error message",

&nbsp; "error\_code": "SPECIFIC\_ERROR\_CODE",

&nbsp; "details": {

&nbsp;   "field": "Additional context if applicable"

&nbsp; }

}

```



\### Common Error Scenarios



\#### Authentication Errors

\- \*\*Invalid Tenant\*\*: Return 401 with message "Invalid tenant credentials"

\- \*\*Invalid API Key\*\*: Return 401 with message "Invalid API key"

\- \*\*Missing Headers\*\*: Return 400 with message "Required authentication header missing: X-Tenant"



\#### Authorization Errors

\- \*\*Wrong User\*\*: Return 401 with message "User not authorized to access this resource"

\- \*\*Wrong Account\*\*: Return 401 with message "Account ID does not match user's accounts"



\#### Validation Errors

\- \*\*Missing Required Field\*\*: Return 400 with message "Required field missing: {field\_name}"

\- \*\*Invalid Format\*\*: Return 400 with message "Invalid format for field: {field\_name}"



\#### Resource Not Found Errors

\- Return 404 with specific message about what wasn't found

\- Examples: "User not found", "Vehicle not found", "Subscription not found"



\### Error Codes



Recommended error codes for programmatic handling:



\- `TENANT\_NOT\_FOUND` - Invalid tenant

\- `USER\_NOT\_FOUND` - User doesn't exist

\- `VEHICLE\_NOT\_FOUND` - Vehicle doesn't exist

\- `SUBSCRIPTION\_NOT\_FOUND` - Subscription doesn't exist

\- `UNAUTHORIZED` - User not authorized for operation

\- `VALIDATION\_ERROR` - Input validation failed

\- `ALREADY\_EXISTS` - Resource already exists (e.g., duplicate vehicle)

\- `ACTIVE\_SUBSCRIPTION` - Cannot perform operation with active subscription

\- `OFFER\_EXPIRED` - Retention offer has expired

\- `INVALID\_STATE` - Resource in invalid state for operation



---



\## Response Codes



\### HTTP Status Codes



| Code | Meaning | Usage |

|------|---------|-------|

| 200 | OK | Successful GET request or POST operation completed |

| 201 | Created | Resource successfully created (alternative to 200 for POST) |

| 400 | Bad Request | Invalid input, validation error, missing required fields |

| 401 | Unauthorized | Invalid credentials or user not authorized |

| 404 | Not Found | Resource doesn't exist |

| 422 | Unprocessable Entity | Request format is valid but data is invalid (e.g., invalid VIN) |

| 500 | Internal Server Error | Server-side error occurred |



\### Success Response Patterns



\#### GET Requests

\- Return 200 with the requested data

\- Return 200 with empty array/object if no results (not 404)



\#### POST Requests

\- Return 200 or 201 with the created/modified resource

\- Include relevant identifiers and timestamps



\### Rate Limiting (Recommended)



Consider implementing rate limiting with these headers:



```

X-RateLimit-Limit: 1000

X-RateLimit-Remaining: 999

X-RateLimit-Reset: 1640000000

```



Return 429 (Too Many Requests) when rate limit is exceeded.



---



\## Implementation Notes



\### Data Consistency

\- All timestamps should be in ISO 8601 format with UTC timezone

\- Currency codes should follow ISO 4217 (USD, CAD, EUR, etc.)

\- State codes should be standardized (use 2-letter codes for US/Canada)

\- Phone numbers should be stored normalized (10 digits, no formatting)



\### Security Considerations

\- All endpoints must use HTTPS in production

\- Tenant API keys must be kept secret and rotated periodically

\- Validate that user\_id in headers matches the actual user for the requested resource

\- Implement request logging for audit trails

\- Consider IP whitelisting for additional security



\### Performance Expectations

\- API response time should be < 500ms for 95th percentile

\- Support concurrent requests with proper connection pooling

\- Implement caching where appropriate (especially for vehicle lookups)



\### Testing \& Staging

\- Provide a staging/sandbox environment with test tenant credentials

\- Support a test mode flag for testing cancellations without actually cancelling

\- Provide test user accounts with known data



\### Versioning

\- Consider API versioning strategy (e.g., /api/v1/)

\- Maintain backward compatibility or provide migration path for breaking changes



\### Support Contact

For implementation questions or issues with this specification, please contact:

\- Technical Support: \[Your contact details]

\- Documentation: \[Link to additional documentation if available]



---



\*\*Document Version\*\*: 1.0  

\*\*Last Updated\*\*: December 15, 2024  

