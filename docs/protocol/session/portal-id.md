# GetPortalId

Requests the game host's portal identifier. This is used by the controller to associate the host with a specific portal or application instance.

## Request

| Field | Value |
|-------|-------|
| **Method** | `GetPortalId` |
| **Return Method** | `onPortalId` |
| **Arguments** | 0 |

This call is sent by the controller to the game host over a direct connection (not through the registry).

## Response: onPortalId

The game host responds with its portal ID string:

| # | Type | Description |
|---|------|-------------|
| 1 | `string` | The host's portal identifier. |