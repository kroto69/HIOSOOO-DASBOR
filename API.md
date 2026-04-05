# OLT API Reference

This document describes the HTTP API exposed by the Go backend.

## Base URL

Default local base URL:

```text
http://localhost:3000
```

If you override `SERVER_PORT`, use the matching port in all requests.

When you use the production Docker stack (`docker-compose.prod.yml`), the frontend and API are served from the same public origin. In that setup, use:

```text
http://localhost:8080/api/v1
```

and access health checks via:

```text
http://localhost:8080/health
```

## Authentication flow

1. Call `POST /api/v1/auth/login`
2. Copy the returned access token
3. Send it in the `Authorization` header for protected endpoints

Example header:

```http
Authorization: Bearer <access-token>
```

## Response format

Most endpoints return a JSON envelope similar to:

```json
{
  "success": true,
  "message": "Operation completed",
  "data": {}
}
```

## Health check

### `GET /health`

Returns service health information.

## Authentication endpoints

### `POST /api/v1/auth/login`

Authenticate a user and return an access token.

```json
{
  "username": "admin",
  "password": "admin"
}
```

### `GET /api/v1/auth/me`

Return the current authenticated user.

### `POST /api/v1/auth/change-password`

Change the password of the current authenticated user.

```json
{
  "current_password": "old-password",
  "new_password": "new-password"
}
```

### `GET /api/v1/auth/users` _(admin only)_

List all users.

### `POST /api/v1/auth/users` _(admin only)_

Create a new user.

```json
{
  "username": "operator",
  "password": "strong-password",
  "role": "user"
}
```

### `PUT /api/v1/auth/users/:id/password` _(admin only)_

Reset a user password.

```json
{
  "new_password": "new-password"
}
```

## Audit logs

### `GET /api/v1/audit-logs`

Return activity or audit log entries.

## Device endpoints

### `POST /api/v1/devices`

Create or save a device.

```json
{
  "id": "olt-1",
  "name": "Main OLT",
  "base_url": "192.168.1.1",
  "port": 8080,
  "username": "admin",
  "password": "admin"
}
```

### `GET /api/v1/devices`

List saved devices.

### `GET /api/v1/devices/:id`

Get one device by ID.

### `PUT /api/v1/devices/:id`

Update a saved device.

### `DELETE /api/v1/devices/:id`

Delete one device.

### `DELETE /api/v1/devices`

Delete all devices.

### `GET /api/v1/devices/:id/status`

Check the status of a saved device.

### `POST /api/v1/devices/check-connection`

Test connectivity without saving the device.

```json
{
  "base_url": "192.168.1.1",
  "port": 8080,
  "username": "admin",
  "password": "admin"
}
```

## PON endpoints

### `GET /api/v1/devices/:device_id/pons`

List PON interfaces for a device.

## ONU endpoints

### `GET /api/v1/devices/:device_id/pons/:pon_id/onus`

List ONUs in a specific PON.

### `GET /api/v1/devices/:device_id/onus?pon_id=:pon_id`

Alternative ONU listing route using a query parameter.

### `GET /api/v1/devices/:device_id/onus/:onu_id`

Get ONU details.

### `GET /api/v1/devices/:device_id/onus/:onu_id/traffic`

Get ONU traffic statistics.

### `PUT /api/v1/devices/:device_id/onus/:onu_id`

Update ONU metadata.

```json
{
  "name": "Customer A"
}
```

### `POST /api/v1/devices/:device_id/onus/:onu_id/action`

Run an ONU action.

```json
{
  "action": "reboot"
}
```

### `DELETE /api/v1/devices/:device_id/onus/:onu_id`

Delete an ONU.

### Simplified ONU and PON IDs

The backend accepts simplified identifiers and normalizes them internally.
Examples:

- PON `1` becomes `0/1`
- ONU `1:8` becomes `0/1:8`

## Device maintenance endpoints

### `POST /api/v1/devices/:id/save-config`

Save device configuration.

### `GET /api/v1/devices/:id/logs`

Get device logs.

Optional query parameter:

- `limit` (default: `100`)

## Example login request

Set your base URL first:

```bash
BASE_URL=http://localhost:3000
# or for the production Docker stack:
# BASE_URL=http://localhost:8080
```

```bash
curl -X POST "$BASE_URL/api/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "admin"
  }'
```

## Example authenticated request

```bash
curl "$BASE_URL/api/v1/auth/me" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Common error responses

### Unauthorized

```json
{
  "success": false,
  "message": "Unauthorized"
}
```

### Validation error

```json
{
  "success": false,
  "message": "Invalid request payload"
}
```
