# Fast Gateway Protocol (FGP) Specification

**Version:** 1.0.0
**Status:** Draft
**License:** MIT
**Last Updated:** 2026-01-12

---

## Abstract

The Fast Gateway Protocol (FGP) defines a standard for building low-latency daemon services that AI agents can call efficiently. Unlike stdio-based protocols (like MCP) that spawn a new process per invocation, FGP uses persistent UNIX sockets with NDJSON framing to achieve 10-30ms response times.

## Design Goals

1. **Low Latency**: 10-30ms warm response times (vs 200-500ms cold starts)
2. **Language Agnostic**: Any language can implement an FGP daemon
3. **Simple Protocol**: NDJSON over UNIX socket - no complex serialization
4. **Stateful Services**: Daemons keep connections, caches, and auth warm
5. **Composable**: Multiple daemons can run independently or behind a hub

---

## 1. Transport Layer

### 1.1 UNIX Domain Sockets (Primary)

FGP daemons MUST listen on a UNIX domain socket.

**Socket Path Convention:**
```
~/.fgp/services/<service-name>/daemon.sock
```

**Examples:**
- `~/.fgp/services/imessage/daemon.sock`
- `~/.fgp/services/gmail/daemon.sock`
- `~/.fgp/services/github/daemon.sock`

**Permissions:**
- Socket file MUST be created with mode `0600` (owner read/write only)
- Socket directory SHOULD be created with mode `0700`

### 1.2 TCP Fallback (Optional)

For non-UNIX systems, daemons MAY additionally listen on localhost TCP:

```
127.0.0.1:<port>
```

Port assignment is implementation-defined. Daemons SHOULD support a `--port` flag.

---

## 2. Framing: NDJSON

FGP uses **Newline-Delimited JSON (NDJSON)** for message framing.

### 2.1 Format

Each message is a single JSON object followed by a newline (`\n`):

```
{"id":"abc123","v":1,"method":"health","params":{}}\n
```

### 2.2 Rules

- Each line MUST be valid JSON
- Lines MUST be terminated with `\n` (LF, 0x0A)
- Lines MUST NOT contain embedded newlines (escape as `\n` in strings)
- Empty lines SHOULD be ignored
- Maximum line length: 10 MB (implementation MAY reject larger)

### 2.3 Encoding

- All messages MUST be UTF-8 encoded
- Implementations MUST handle UTF-8 decoding errors gracefully

---

## 3. Request Format

A request is a JSON object with the following fields:

```json
{
  "id": "unique-request-id",
  "v": 1,
  "method": "service.action",
  "params": { ... }
}
```

### 3.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique request identifier (UUID recommended) |
| `v` | integer | Protocol version (currently `1`) |
| `method` | string | Method name to invoke |
| `params` | object | Method parameters (may be empty `{}`) |

### 3.2 Method Naming

Methods SHOULD follow the pattern: `<namespace>.<action>`

**Examples:**
- `health` (reserved, no namespace)
- `gmail.list`
- `gmail.send`
- `calendar.events`

Reserved methods (no namespace):
- `health`
- `stop`
- `methods`
- `bundle`

### 3.3 Request ID

The `id` field:
- MUST be unique per connection session
- SHOULD be a UUID v4
- Is echoed back in the response for correlation
- MAY be used by clients for timeout/retry tracking

---

## 4. Response Format

A response is a JSON object with the following fields:

```json
{
  "id": "unique-request-id",
  "ok": true,
  "result": { ... },
  "error": null,
  "meta": {
    "server_ms": 12.5,
    "protocol_v": 1
  }
}
```

### 4.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Request ID (echoed from request) |
| `ok` | boolean | `true` if successful, `false` if error |
| `result` | any | Result data (if `ok: true`), or `null` |
| `error` | object\|null | Error info (if `ok: false`), or `null` |
| `meta` | object | Response metadata |

### 4.2 Error Object

When `ok: false`, the `error` field contains:

```json
{
  "code": "NOT_FOUND",
  "message": "Contact not found: John",
  "details": { "search_term": "John" }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | Machine-readable error code (UPPER_SNAKE_CASE) |
| `message` | string | Human-readable error message |
| `details` | object\|null | Additional error context (optional) |

### 4.3 Standard Error Codes

| Code | Description |
|------|-------------|
| `INVALID_REQUEST` | Malformed request (missing fields, bad JSON) |
| `UNKNOWN_METHOD` | Method not found |
| `INVALID_PARAMS` | Invalid or missing parameters |
| `INTERNAL_ERROR` | Server-side error |
| `NOT_FOUND` | Requested resource not found |
| `UNAUTHORIZED` | Authentication required or failed |
| `TIMEOUT` | Operation timed out |
| `SERVICE_UNAVAILABLE` | Dependency not available |

### 4.4 Metadata Object

The `meta` field contains:

| Field | Type | Description |
|-------|------|-------------|
| `server_ms` | number | Server-side execution time in milliseconds |
| `protocol_v` | integer | Protocol version used |

Optional metadata fields:
- `service` (string): Service name
- `daemon_uptime_s` (number): Daemon uptime in seconds
- `request_queue_depth` (integer): Pending requests (if queuing)

---

## 5. Required Methods

Every FGP daemon MUST implement these methods:

### 5.1 `health`

Returns daemon health and status information.

**Request:**
```json
{"id":"1","v":1,"method":"health","params":{}}
```

**Response:**
```json
{
  "id": "1",
  "ok": true,
  "result": {
    "status": "healthy",
    "pid": 12345,
    "started_at": "2026-01-12T10:00:00Z",
    "version": "1.0.0",
    "uptime_seconds": 3600,
    "services": {
      "database": {"ok": true, "latency_ms": 5},
      "api": {"ok": true, "latency_ms": 100}
    }
  },
  "error": null,
  "meta": {"server_ms": 2.1, "protocol_v": 1}
}
```

**Required Result Fields:**
- `status` (string): `"healthy"`, `"degraded"`, or `"unhealthy"`
- `pid` (integer): Process ID
- `version` (string): Daemon version

### 5.2 `stop`

Gracefully shuts down the daemon.

**Request:**
```json
{"id":"2","v":1,"method":"stop","params":{}}
```

**Response:**
```json
{
  "id": "2",
  "ok": true,
  "result": {"message": "Shutting down"},
  "error": null,
  "meta": {"server_ms": 0.5, "protocol_v": 1}
}
```

After sending the response, the daemon MUST:
1. Stop accepting new connections
2. Complete in-flight requests (with reasonable timeout)
3. Close the socket
4. Exit the process

### 5.3 `methods`

Lists available methods and their signatures.

**Request:**
```json
{"id":"3","v":1,"method":"methods","params":{}}
```

**Response:**
```json
{
  "id": "3",
  "ok": true,
  "result": {
    "methods": [
      {
        "name": "gmail.list",
        "description": "List emails from inbox",
        "params": {
          "limit": {"type": "integer", "default": 10, "required": false},
          "unread_only": {"type": "boolean", "default": false, "required": false}
        }
      },
      {
        "name": "gmail.send",
        "description": "Send an email",
        "params": {
          "to": {"type": "string", "required": true},
          "subject": {"type": "string", "required": true},
          "body": {"type": "string", "required": true}
        }
      }
    ]
  },
  "error": null,
  "meta": {"server_ms": 1.0, "protocol_v": 1}
}
```

---

## 6. Optional Standard Methods

### 6.1 `bundle`

Execute multiple methods in a single request.

**Request:**
```json
{
  "id": "4",
  "v": 1,
  "method": "bundle",
  "params": {
    "requests": [
      {"method": "gmail.unread_count", "params": {}},
      {"method": "gmail.list", "params": {"limit": 5}},
      {"method": "calendar.today", "params": {}}
    ]
  }
}
```

**Response:**
```json
{
  "id": "4",
  "ok": true,
  "result": {
    "responses": [
      {"ok": true, "result": {"count": 42}, "error": null},
      {"ok": true, "result": {"emails": [...]}, "error": null},
      {"ok": true, "result": {"events": [...]}, "error": null}
    ]
  },
  "error": null,
  "meta": {"server_ms": 45.2, "protocol_v": 1}
}
```

**Semantics:**
- Requests execute in order (sequential)
- Bundle fails fast: if any request fails, remaining requests are skipped
- `ok: true` on bundle means all requests succeeded

---

## 7. Lifecycle Management

### 7.1 Daemon Startup

Daemons SHOULD support these command-line patterns:

```bash
# Start in foreground (for debugging)
my-daemon start --foreground

# Start in background (daemonize)
my-daemon start

# Check if running
my-daemon status

# Stop gracefully
my-daemon stop
```

### 7.2 PID File

Daemons SHOULD write a PID file:

```
~/.fgp/services/<service-name>/daemon.pid
```

### 7.3 Socket Cleanup

On startup, if the socket file exists but no process is listening:
- Remove the stale socket file
- Create a new socket

On shutdown:
- Close the socket
- Remove the socket file
- Remove the PID file

---

## 8. Authentication

### 8.1 Auth Store Location

FGP daemons SHOULD use a shared authentication store:

```
~/.fgp/auth/<service-name>/
```

**Examples:**
- `~/.fgp/auth/google/oauth-token.json`
- `~/.fgp/auth/github/pat.json`
- `~/.fgp/auth/twitter/` (API key in macOS Keychain)

### 8.2 Auth Resolution Order

Daemons SHOULD check credentials in this order:
1. `~/.fgp/auth/<service>/` (FGP unified store)
2. Service-specific legacy locations (for migration)
3. Environment variables (e.g., `GITHUB_TOKEN`)
4. macOS Keychain (for secrets)

### 8.3 Auth Errors

If authentication fails, return error code `UNAUTHORIZED`:

```json
{
  "ok": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "OAuth token expired. Re-authenticate with: fgp auth google"
  }
}
```

---

## 9. Manifest Format

Each installable FGP service includes a `manifest.json`:

```json
{
  "name": "gmail-daemon",
  "version": "1.0.0",
  "description": "Gmail integration for FGP",
  "protocol": "fgp@1",
  "entrypoint": "./daemon",
  "socket": "gmail/daemon.sock",
  "auth": {
    "required": ["google-oauth"],
    "store": "~/.fgp/auth/google/"
  },
  "methods": [
    "gmail.list",
    "gmail.send",
    "gmail.search"
  ],
  "platforms": ["darwin", "linux"],
  "repository": "https://github.com/fgp/gmail-daemon",
  "license": "MIT"
}
```

### 9.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Package name (lowercase, hyphens) |
| `version` | string | Semantic version |
| `protocol` | string | Protocol version (`fgp@1`) |
| `entrypoint` | string | Executable path relative to package |
| `socket` | string | Socket path relative to `~/.fgp/services/` |
| `methods` | array | List of method names |
| `platforms` | array | Supported platforms (`darwin`, `linux`, `windows`) |

---

## 10. Error Handling

### 10.1 Connection Errors

If the client cannot connect to the socket:
- The daemon may not be running
- The socket file may be stale
- Permissions may be incorrect

Clients SHOULD provide helpful error messages:

```
Error: Cannot connect to gmail daemon
  Socket: ~/.fgp/services/gmail/daemon.sock

  Try: fgp start gmail
```

### 10.2 Timeout Handling

Clients SHOULD implement request timeouts:
- Default: 30 seconds
- Configurable per-request via client API

If a request times out, the client SHOULD:
1. Close the connection
2. NOT retry automatically (let caller decide)
3. Return a timeout error to the caller

### 10.3 Malformed Requests

If a daemon receives an invalid request:

```json
{
  "id": null,
  "ok": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Missing required field: method"
  },
  "meta": {"server_ms": 0.1, "protocol_v": 1}
}
```

Note: `id` is `null` if the request didn't include one.

---

## 11. Implementation Guidelines

### 11.1 Concurrency

Daemons MAY implement different concurrency models:
- **Single-threaded sequential**: Simple, no race conditions
- **Thread pool**: Higher throughput
- **Async**: Maximum concurrency

The protocol does not require any specific model.

### 11.2 Logging

Daemons SHOULD log to:
```
~/.fgp/logs/<service-name>.log
```

Log format recommendation (JSON Lines):
```json
{"ts":"2026-01-12T10:00:00Z","level":"info","msg":"Started","pid":12345}
```

### 11.3 Graceful Degradation

If a daemon's backend service is unavailable:
- Return `SERVICE_UNAVAILABLE` for affected methods
- `health` method should report `status: "degraded"`
- Continue serving methods that don't require the backend

---

## 12. SDK Requirements

Official FGP SDKs (Rust, Python, Node) MUST provide:

### 12.1 Server-side

- Socket server with NDJSON framing
- Request/response type definitions
- Health/stop/methods method scaffolding
- Lifecycle management (start/stop/status)
- Auth store integration helpers

### 12.2 Client-side

- Socket connection with timeout
- Request/response serialization
- Connection pooling (optional)
- Error handling and retries

---

## Appendix A: Example Session

**Client connects to `~/.fgp/services/gmail/daemon.sock`**

**Request 1: Health check**
```
{"id":"req-001","v":1,"method":"health","params":{}}\n
```

**Response 1:**
```
{"id":"req-001","ok":true,"result":{"status":"healthy","pid":12345,"version":"1.0.0","uptime_seconds":3600},"error":null,"meta":{"server_ms":1.2,"protocol_v":1}}\n
```

**Request 2: List emails**
```
{"id":"req-002","v":1,"method":"gmail.list","params":{"limit":5,"unread_only":true}}\n
```

**Response 2:**
```
{"id":"req-002","ok":true,"result":{"emails":[{"id":"abc","subject":"Hello","from":"john@example.com"}]},"error":null,"meta":{"server_ms":45.3,"protocol_v":1}}\n
```

**Request 3: Stop daemon**
```
{"id":"req-003","v":1,"method":"stop","params":{}}\n
```

**Response 3:**
```
{"id":"req-003","ok":true,"result":{"message":"Shutting down"},"error":null,"meta":{"server_ms":0.5,"protocol_v":1}}\n
```

**Connection closed by server**

---

## Appendix B: Comparison with MCP

| Aspect | MCP | FGP |
|--------|-----|-----|
| Transport | stdio (process spawn) | UNIX socket (persistent) |
| Latency | 200-500ms cold start | 10-30ms warm |
| State | Stateless | Stateful |
| Framing | JSON-RPC 2.0 | NDJSON |
| Discovery | Manifest in package | `methods` call + manifest |
| Auth | Per-tool | Shared store |
| Batching | None | `bundle` method |

---

## Appendix C: Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-12 | Initial specification |

---

*Copyright 2026. Licensed under MIT.*
