<div align="center" width="100%">
    <img src="./docs/assets/upstat.png" width="128" alt="" />
</div>

# Upstat

> simple and easy-to-use self-hosted status monitoring tool

![](./docs/assets/dashboard.png)

## 💻 Live Demo

Try it.

Demo Server (Location: Singapore): [https://demo.upstat.com](https://upstat.chamanbudhathoki.com.np/)

Username: `demo`
Password: `demodemo`

## ⭐ Features

It needs more features but for now...

- Monitoring uptime for HTTP(s)
- Status and Latency Chart
- Notifications via Discord
- 60-second intervals
- Fancy, Reactive, Fast UI/UX
- Multiple status pages
- Map status pages to specific domains
- Ping chart
- Certificate info
- PWA
- Sqlite & Postgres database support

And dozens of smaller features to be added.

## 🔧 How to Install

### 🐳 Docker

For Sqlite

```bash
curl https://raw.githubusercontent.com/chamanbravo/upstat/main/docker-compose-sqlite.yml -o docker-compose.yml
docker compose up
```

For Postgres

```bash
curl -O https://raw.githubusercontent.com/chamanbravo/upstat/main/docker-compose.yml
docker compose up
```

Upstat is now running on http://localhost:9866

> [!IMPORTANT]
> Make sure to change the enviornment values before deploying.

### 💪🏻 Non-Docker

Requirements:

- Node.js 14 / 16 / 18 / 20.4
- npm 9
- Golang 1.21+
- Postgres (Optional)

```shell
cp .sample.env .env
```

```shell
air
cd web && npm run dev
```

## Tech stack

- React
- Shadcn
- Golang
- Postgres/Sqlite

## 🔌 API Documentation

The Upstat API allows you to programmatically manage monitors and status pages. All API requests require authentication via JWT token.

### Base URL

```
http://your-upstat-instance:8000/api
```

For local development: `http://127.0.0.1:8000/api`

### Authentication

All API endpoints (except auth endpoints) require authentication using a Bearer token in the Authorization header.

#### 1. Get Access Token

**Endpoint:** `POST /api/auth/signin`

**Request Body:**
```json
{
  "username": "your_username",
  "password": "your_password"
}
```

**Response:**
```json
{
  "message": "success",
  "user": {
    "username": "your_username",
    "email": "user@example.com",
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

**Token Details:**
- **Access Token**: Valid for 3 hours
- **Refresh Token**: Valid for 7 days
- Use the `access_token` in the Authorization header for all subsequent requests

#### 2. Using the Token

Include the token in the Authorization header of your requests:

```bash
Authorization: Bearer YOUR_ACCESS_TOKEN
```

### API Endpoints

#### Create a Monitor

**Endpoint:** `POST /api/monitors`

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "My Website",
  "url": "https://example.com",
  "type": "http",
  "frequency": 60,
  "method": "get",
  "notificationChannels": [],
  "statusPages": []
}
```

**Parameters:**
- `name` (required): Name of the monitor
- `url` (required): URL to monitor
- `type` (required): Monitor type (currently supports "http")
- `frequency` (required): Check interval in seconds (e.g., 60 for 1 minute)
- `method` (required): HTTP method ("get", "post", "put", "delete", etc.)
- `notificationChannels` (optional): Array of notification channel IDs to alert
- `statusPages` (optional): Array of status page IDs to display this monitor on

**Response:**
```json
{
  "message": "success"
}
```

**cURL Example:**
```bash
curl -X POST http://your-upstat-instance:8000/api/monitors \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Website",
    "url": "https://example.com",
    "type": "http",
    "frequency": 60,
    "method": "get",
    "notificationChannels": [],
    "statusPages": []
  }'
```

#### List All Monitors

**Endpoint:** `GET /api/monitors`

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

**Response:**
```json
{
  "message": "success",
  "monitors": [
    {
      "id": "1",
      "name": "My Website",
      "url": "https://example.com",
      "frequency": 60,
      "status": "green",
      "heartbeat": []
    }
  ]
}
```

#### Get Monitor Details

**Endpoint:** `GET /api/monitors/{id}`

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

**Response:**
```json
{
  "message": "success",
  "monitor": {
    "id": 1,
    "name": "My Website",
    "url": "https://example.com",
    "type": "http",
    "method": "get",
    "frequency": 60,
    "status": "green"
  }
}
```

#### Update a Monitor

**Endpoint:** `PATCH /api/monitors/{id}`

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request Body:** Same as Create Monitor

#### Delete a Monitor

**Endpoint:** `DELETE /api/monitors/{id}`

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

#### Create a Status Page

**Endpoint:** `POST /api/status-pages`

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "My Status Page",
  "slug": "my-status-page"
}
```

**Parameters:**
- `name` (required): Display name of the status page
- `slug` (required): URL-friendly identifier (will be used in the URL: `/status/my-status-page`)

**Response:**
```json
{
  "message": "Success"
}
```

**cURL Example:**
```bash
curl -X POST http://your-upstat-instance:8000/api/status-pages \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Status Page",
    "slug": "my-status-page"
  }'
```

#### List All Status Pages

**Endpoint:** `GET /api/status-pages`

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

**Response:**
```json
{
  "message": "Status pages list",
  "statusPages": [
    {
      "id": 1,
      "name": "My Status Page",
      "slug": "my-status-page"
    }
  ]
}
```

#### Get Status Page Details

**Endpoint:** `GET /api/status-pages/{id}`

**Headers:**
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

#### Add Monitor to Status Page

To add a monitor to a status page, include the status page ID in the `statusPages` array when creating or updating a monitor:

**Example:**
```bash
curl -X POST http://your-upstat-instance:8000/api/monitors \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production Server",
    "url": "https://example.com",
    "type": "http",
    "frequency": 60,
    "method": "get",
    "notificationChannels": [],
    "statusPages": ["1"]
  }'
```

Or update an existing monitor:

```bash
curl -X PATCH http://your-upstat-instance:8000/api/monitors/1 \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production Server",
    "url": "https://example.com",
    "type": "http",
    "frequency": 60,
    "method": "get",
    "notificationChannels": [],
    "statusPages": ["1", "2"]
  }'
```

### Complete Automation Example

Here's a complete example of automating server monitoring setup:

```bash
#!/bin/bash

# Configuration
UPSTAT_URL="http://your-upstat-instance:8000"
USERNAME="your_username"
PASSWORD="your_password"

# 1. Get access token
TOKEN_RESPONSE=$(curl -s -X POST "$UPSTAT_URL/api/auth/signin" \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"$USERNAME\",\"password\":\"$PASSWORD\"}")

ACCESS_TOKEN=$(echo $TOKEN_RESPONSE | jq -r '.user.access_token')

# 2. Create a status page
STATUS_PAGE_RESPONSE=$(curl -s -X POST "$UPSTAT_URL/api/status-pages" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production Servers",
    "slug": "production"
  }')

# 3. Get status page ID
STATUS_PAGES=$(curl -s -X GET "$UPSTAT_URL/api/status-pages" \
  -H "Authorization: Bearer $ACCESS_TOKEN")

STATUS_PAGE_ID=$(echo $STATUS_PAGES | jq -r '.statusPages[0].id')

# 4. Add multiple servers to monitor
declare -a SERVERS=(
  "https://server1.example.com"
  "https://server2.example.com"
  "https://server3.example.com"
)

for SERVER_URL in "${SERVERS[@]}"
do
  SERVER_NAME=$(echo $SERVER_URL | sed 's|https://||' | sed 's|http://||')
  
  curl -X POST "$UPSTAT_URL/api/monitors" \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    -H "Content-Type: application/json" \
    -d "{
      \"name\": \"$SERVER_NAME\",
      \"url\": \"$SERVER_URL\",
      \"type\": \"http\",
      \"frequency\": 60,
      \"method\": \"get\",
      \"notificationChannels\": [],
      \"statusPages\": [\"$STATUS_PAGE_ID\"]
    }"
  
  echo "Added monitor for $SERVER_NAME"
done

echo "Setup complete! View your status page at: $UPSTAT_URL/status/production"
```

### Error Responses

All endpoints may return error responses in the following format:

**401 Unauthorized:**
```json
{
  "message": "Unauthorized",
  "error": "Invalid authorization header format"
}
```

**400 Bad Request:**
```json
{
  "message": "Validation error message"
}
```

**500 Internal Server Error:**
```json
{
  "message": "Error description"
}
```

### Rate Limiting

Currently, there are no rate limits enforced on the API. However, it's recommended to implement reasonable delays between requests when automating tasks.

### Public Status Page Access

Status pages are publicly accessible without authentication:

**Endpoint:** `GET /api/status-pages/{slug}/summary`

This endpoint does not require authentication and returns the current status of all monitors on the status page. You can use this to embed status information on your own website or check status programmatically.

**Example:**
```bash
curl http://your-upstat-instance:8000/api/status-pages/production/summary
```

## 🙌 Contributing

I welcome contributions! Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo, make changes and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## Contributors

<a href="https://github.com/chamanbravo/upstat/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=chamanbravo/upstat" />
</a>

## 📄 License

This project is licensed under the [MIT License](https://opensource.org/license/mit/).

## 🖼 More Screenshots

Create a Monitor

<img src="./docs/assets/create.png" width="512" alt="" />

Monitor Page

<img src="./docs/assets/chart.png" width="512" alt="" />

Settings Page

<img src="./docs/assets/settings.png" width="512" alt="" />

Notifications

<img src="./docs/assets/notifications.png" width="512" alt="" />

<img src="./docs/assets/discord_notification.png" width="512" alt="" />
