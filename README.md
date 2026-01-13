# Fast Gateway Protocol (FGP)

**10-30ms response times for AI agent tool calls.**

FGP is an open protocol for building low-latency daemon services that AI agents can call efficiently. Unlike stdio-based protocols that spawn a new process per invocation, FGP uses persistent UNIX sockets with NDJSON framing.

## Why FGP?

| Aspect | MCP (stdio) | FGP (socket) |
|--------|-------------|--------------|
| Latency | 200-500ms | 10-30ms |
| State | Stateless | Stateful |
| Auth | Per-call | Shared store |
| Batching | None | `bundle` method |

## Quick Example

**Request:**
```json
{"id":"1","v":1,"method":"gmail.list","params":{"limit":5}}
```

**Response:**
```json
{"id":"1","ok":true,"result":{"emails":[...]},"meta":{"server_ms":12}}
```

## Specification

See [FGP-PROTOCOL.md](./FGP-PROTOCOL.md) for the complete specification.

## SDKs

| Language | Package | Status |
|----------|---------|--------|
| Rust | `fgp-daemon` | Coming soon |
| Python | `fgp-daemon-py` | Coming soon |
| Node.js | `fgp-daemon-node` | Planned |

## Directory Structure

FGP services install to `~/.fgp/`:

```
~/.fgp/
├── config.json              # Global configuration
├── services/                # Installed services
│   ├── gmail/
│   │   ├── manifest.json
│   │   └── daemon.sock
│   └── github/
│       ├── manifest.json
│       └── daemon.sock
├── auth/                    # Shared authentication
│   ├── google/
│   └── github/
└── logs/                    # Service logs
```

## Required Methods

Every FGP daemon must implement:

- `health` - Returns status, pid, version, uptime
- `stop` - Graceful shutdown
- `methods` - Lists available methods

## License

MIT
