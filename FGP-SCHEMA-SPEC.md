# FGP Schema Specification

**Version:** 1.0.0
**Status:** Draft
**Extends:** FGP Protocol v1.0
**Last Updated:** 2026-01-15

---

## Abstract

This specification extends the FGP Protocol to support rich JSON Schema-based method signatures. It enables automatic generation of tool definitions for multiple AI agent formats (Claude, OpenAI, Codex, etc.) from a single source of truth.

## Goals

1. **Full JSON Schema Support**: Parameters can use any valid JSON Schema
2. **Multi-Agent Portability**: Single schema → multiple agent formats
3. **Backward Compatibility**: Existing daemons continue to work
4. **Self-Documenting**: Schemas include examples, descriptions, constraints

---

## 1. Extended Method Schema

### 1.1 Current vs Extended Format

**Current (Basic):**
```json
{
  "name": "gmail.send",
  "description": "Send an email",
  "params": [
    {"name": "to", "type": "string", "required": true}
  ]
}
```

**Extended (Full Schema):**
```json
{
  "name": "gmail.send",
  "description": "Send an email",
  "schema": {
    "type": "object",
    "properties": {
      "to": {
        "type": "string",
        "format": "email",
        "description": "Recipient email address"
      },
      "subject": {
        "type": "string",
        "maxLength": 998,
        "description": "Email subject line"
      },
      "body": {
        "type": "string",
        "description": "Email body (plain text or HTML)"
      },
      "cc": {
        "type": "array",
        "items": {"type": "string", "format": "email"},
        "description": "CC recipients"
      },
      "attachments": {
        "type": "array",
        "items": {"$ref": "#/$defs/attachment"},
        "maxItems": 25
      }
    },
    "required": ["to", "subject", "body"],
    "$defs": {
      "attachment": {
        "type": "object",
        "properties": {
          "filename": {"type": "string"},
          "content": {"type": "string", "contentEncoding": "base64"},
          "mimeType": {"type": "string"}
        },
        "required": ["filename", "content"]
      }
    }
  },
  "returns": {
    "type": "object",
    "properties": {
      "messageId": {"type": "string"},
      "threadId": {"type": "string"}
    }
  },
  "examples": [
    {
      "description": "Send a simple email",
      "params": {
        "to": "alice@example.com",
        "subject": "Hello",
        "body": "Hi Alice!"
      }
    }
  ],
  "errors": ["UNAUTHORIZED", "INVALID_RECIPIENT", "QUOTA_EXCEEDED"]
}
```

### 1.2 Schema Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Method name (e.g., "gmail.send") |
| `description` | string | Yes | Human-readable description |
| `schema` | JSON Schema | No* | Full JSON Schema for parameters |
| `params` | array | No* | Legacy parameter list (backward compat) |
| `returns` | JSON Schema | No | Schema for successful response |
| `examples` | array | No | Usage examples |
| `errors` | array | No | Possible error codes |
| `deprecated` | boolean | No | If true, method is deprecated |
| `since` | string | No | Version when method was added |

*Either `schema` or `params` must be present. If both exist, `schema` takes precedence.

---

## 2. New Built-in Method: `schema`

### 2.1 Purpose

Returns full JSON Schema definitions for all methods. Extends `methods` with richer type information.

### 2.2 Request

```json
{
  "id": "1",
  "v": 1,
  "method": "schema",
  "params": {
    "methods": ["gmail.send", "gmail.list"],  // Optional: filter to specific methods
    "format": "json-schema"                   // Optional: output format
  }
}
```

### 2.3 Response

```json
{
  "id": "1",
  "ok": true,
  "result": {
    "service": "gmail",
    "version": "1.0.0",
    "protocol": "fgp@1",
    "methods": [
      {
        "name": "gmail.send",
        "description": "Send an email",
        "schema": { /* full JSON Schema */ },
        "returns": { /* response schema */ },
        "examples": [ /* ... */ ],
        "errors": ["UNAUTHORIZED", "INVALID_RECIPIENT"]
      }
    ]
  }
}
```

### 2.4 Output Formats

| Format | Description |
|--------|-------------|
| `json-schema` | Standard JSON Schema (default) |
| `openai` | OpenAI function calling format |
| `anthropic` | Anthropic tools format |
| `mcp` | MCP tool schema format |

**Example: OpenAI format request**
```json
{"method": "schema", "params": {"format": "openai"}}
```

**Response:**
```json
{
  "result": {
    "functions": [
      {
        "name": "gmail_send",
        "description": "Send an email",
        "parameters": {
          "type": "object",
          "properties": {
            "to": {"type": "string", "description": "Recipient"},
            "subject": {"type": "string"},
            "body": {"type": "string"}
          },
          "required": ["to", "subject", "body"]
        }
      }
    ]
  }
}
```

---

## 3. JSON Schema Extensions

### 3.1 Supported JSON Schema Features

FGP schemas MUST support JSON Schema Draft 2020-12. Key features:

| Feature | Support | Notes |
|---------|---------|-------|
| Basic types | ✅ | string, number, integer, boolean, array, object, null |
| `enum` | ✅ | Enumerated values |
| `const` | ✅ | Constant values |
| `format` | ✅ | email, uri, date-time, uuid, etc. |
| `pattern` | ✅ | Regex validation |
| `minLength`/`maxLength` | ✅ | String length constraints |
| `minimum`/`maximum` | ✅ | Number range constraints |
| `minItems`/`maxItems` | ✅ | Array length constraints |
| `items` | ✅ | Array item schema |
| `properties` | ✅ | Object properties |
| `required` | ✅ | Required properties |
| `additionalProperties` | ✅ | Allow/disallow extra properties |
| `$ref` | ✅ | Local references only (`#/$defs/...`) |
| `$defs` | ✅ | Local definitions |
| `oneOf`/`anyOf`/`allOf` | ✅ | Composition |
| `if`/`then`/`else` | ⚠️ | Optional (complex) |
| Remote `$ref` | ❌ | Not supported |

### 3.2 FGP-Specific Extensions

Additional metadata fields (prefixed with `x-fgp-`):

```json
{
  "type": "string",
  "x-fgp-sensitive": true,      // Marks sensitive data (passwords, tokens)
  "x-fgp-large": true,          // Hints this field may be large (base64 files)
  "x-fgp-autocomplete": "email" // UI hint for autocompletion
}
```

---

## 4. Agent Format Conversions

### 4.1 To OpenAI Function Calling

```
FGP schema → OpenAI function
─────────────────────────────
name: "gmail.send"         →  name: "gmail_send" (underscores)
description: "..."         →  description: "..."
schema.properties          →  parameters.properties
schema.required            →  parameters.required
```

**Conversion rules:**
- Replace `.` with `_` in method names
- Remove `$defs` (inline all references)
- Remove unsupported constraints (OpenAI ignores many)
- Truncate descriptions to 1024 chars

### 4.2 To Anthropic Tools

```
FGP schema → Anthropic tool
───────────────────────────
name: "gmail.send"         →  name: "gmail.send" (dots OK)
description: "..."         →  description: "..."
schema                     →  input_schema (JSON Schema)
```

**Conversion rules:**
- Anthropic supports full JSON Schema
- Keep `$defs` and `$ref`
- Add tool_choice hints if method is destructive

### 4.3 To MCP Tools

```
FGP schema → MCP tool
─────────────────────
name: "gmail.send"         →  name: "gmail.send"
description: "..."         →  description: "..."
schema                     →  inputSchema
```

**Conversion rules:**
- MCP uses JSON Schema subset
- Remove `$defs` (inline references)
- Keep required array

---

## 5. Rust SDK Changes

### 5.1 Extended `MethodInfo` Struct

```rust
/// Method information with full schema support.
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct MethodInfo {
    /// Method name (e.g., "gmail.send")
    pub name: String,

    /// Human-readable description
    pub description: String,

    /// Full JSON Schema for parameters (preferred)
    #[serde(skip_serializing_if = "Option::is_none")]
    pub schema: Option<serde_json::Value>,

    /// Legacy parameter list (backward compatibility)
    #[serde(default, skip_serializing_if = "Vec::is_empty")]
    pub params: Vec<ParamInfo>,

    /// Schema for successful response
    #[serde(skip_serializing_if = "Option::is_none")]
    pub returns: Option<serde_json::Value>,

    /// Usage examples
    #[serde(default, skip_serializing_if = "Vec::is_empty")]
    pub examples: Vec<MethodExample>,

    /// Possible error codes
    #[serde(default, skip_serializing_if = "Vec::is_empty")]
    pub errors: Vec<String>,

    /// Deprecation status
    #[serde(default)]
    pub deprecated: bool,
}

/// Usage example for a method.
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct MethodExample {
    /// Description of what this example demonstrates
    pub description: String,

    /// Example parameters
    pub params: serde_json::Value,

    /// Expected result (optional)
    #[serde(skip_serializing_if = "Option::is_none")]
    pub result: Option<serde_json::Value>,
}
```

### 5.2 Schema Builder Helper

```rust
use fgp_daemon::schema::SchemaBuilder;

fn method_list(&self) -> Vec<MethodInfo> {
    vec![
        MethodInfo::new("gmail.send", "Send an email")
            .schema(SchemaBuilder::object()
                .property("to", SchemaBuilder::string()
                    .format("email")
                    .description("Recipient email"))
                .property("subject", SchemaBuilder::string()
                    .max_length(998))
                .property("body", SchemaBuilder::string())
                .required(["to", "subject", "body"])
                .build())
            .returns(SchemaBuilder::object()
                .property("messageId", SchemaBuilder::string())
                .build())
            .example("Simple email", json!({
                "to": "alice@example.com",
                "subject": "Hello",
                "body": "Hi!"
            }))
            .errors(["UNAUTHORIZED", "INVALID_RECIPIENT"])
            .build()
    ]
}
```

### 5.3 Automatic Format Conversion

```rust
use fgp_daemon::schema::{to_openai, to_anthropic, to_mcp};

// Get native FGP schema
let methods = service.method_list();

// Convert to different formats
let openai_funcs = to_openai(&methods);
let anthropic_tools = to_anthropic(&methods);
let mcp_tools = to_mcp(&methods);
```

---

## 6. Validation

### 6.1 Server-Side Validation

The SDK SHOULD provide optional parameter validation:

```rust
impl FgpServer {
    /// Enable automatic parameter validation against schema.
    pub fn with_validation(mut self) -> Self {
        self.validate_params = true;
        self
    }
}
```

When enabled:
1. Before `dispatch()`, validate `params` against method's `schema`
2. Return `INVALID_PARAMS` error if validation fails
3. Include validation errors in `error.details`

**Validation error response:**
```json
{
  "ok": false,
  "error": {
    "code": "INVALID_PARAMS",
    "message": "Parameter validation failed",
    "details": {
      "errors": [
        {"path": "/to", "message": "must be a valid email address"},
        {"path": "/attachments/0/content", "message": "must be base64 encoded"}
      ]
    }
  }
}
```

### 6.2 Client-Side Validation

Clients MAY implement pre-flight validation:

```rust
let client = FgpClient::new(socket_path)?
    .with_schema_cache()   // Cache schemas
    .with_validation();     // Validate before sending

// This validates params before sending to daemon
client.call("gmail.send", json!({"to": "invalid"}))?;
// Error: InvalidParams { path: "/to", message: "must be valid email" }
```

---

## 7. Migration Path

### 7.1 Backward Compatibility

Existing daemons using `params: Vec<ParamInfo>` continue to work:
- `methods` response unchanged
- `schema` method synthesizes JSON Schema from `params`
- MCP bridge handles both formats

### 7.2 Migration Steps

1. **No change required**: Existing daemons work as-is
2. **Optional upgrade**: Add `schema` field to `MethodInfo`
3. **Full migration**: Replace `params` with `schema`

### 7.3 Deprecation Timeline

| Phase | Timeline | Action |
|-------|----------|--------|
| v1.0 | Now | Both `params` and `schema` supported |
| v1.1 | +6 months | `params` deprecated, `schema` preferred |
| v2.0 | +12 months | `params` removed, `schema` required |

---

## 8. Examples

### 8.1 Complete Daemon Schema

```json
{
  "service": "github",
  "version": "1.0.0",
  "protocol": "fgp@1",
  "methods": [
    {
      "name": "github.create_issue",
      "description": "Create a new issue in a repository",
      "schema": {
        "type": "object",
        "properties": {
          "owner": {
            "type": "string",
            "description": "Repository owner (user or org)",
            "pattern": "^[a-zA-Z0-9_-]+$"
          },
          "repo": {
            "type": "string",
            "description": "Repository name"
          },
          "title": {
            "type": "string",
            "description": "Issue title",
            "minLength": 1,
            "maxLength": 256
          },
          "body": {
            "type": "string",
            "description": "Issue body (Markdown)"
          },
          "labels": {
            "type": "array",
            "items": {"type": "string"},
            "description": "Labels to apply"
          },
          "assignees": {
            "type": "array",
            "items": {"type": "string"},
            "description": "Users to assign"
          }
        },
        "required": ["owner", "repo", "title"]
      },
      "returns": {
        "type": "object",
        "properties": {
          "number": {"type": "integer", "description": "Issue number"},
          "url": {"type": "string", "format": "uri"},
          "id": {"type": "integer"}
        }
      },
      "examples": [
        {
          "description": "Create a bug report",
          "params": {
            "owner": "fast-gateway-protocol",
            "repo": "daemon",
            "title": "Fix socket timeout",
            "body": "The socket times out after 30s...",
            "labels": ["bug"]
          }
        }
      ],
      "errors": ["UNAUTHORIZED", "NOT_FOUND", "VALIDATION_FAILED"]
    }
  ]
}
```

### 8.2 Generated OpenAI Format

```json
{
  "functions": [
    {
      "name": "github_create_issue",
      "description": "Create a new issue in a repository",
      "parameters": {
        "type": "object",
        "properties": {
          "owner": {"type": "string", "description": "Repository owner (user or org)"},
          "repo": {"type": "string", "description": "Repository name"},
          "title": {"type": "string", "description": "Issue title"},
          "body": {"type": "string", "description": "Issue body (Markdown)"},
          "labels": {"type": "array", "items": {"type": "string"}},
          "assignees": {"type": "array", "items": {"type": "string"}}
        },
        "required": ["owner", "repo", "title"]
      }
    }
  ]
}
```

### 8.3 Generated Anthropic Format

```json
{
  "tools": [
    {
      "name": "github.create_issue",
      "description": "Create a new issue in a repository",
      "input_schema": {
        "type": "object",
        "properties": {
          "owner": {
            "type": "string",
            "description": "Repository owner (user or org)",
            "pattern": "^[a-zA-Z0-9_-]+$"
          },
          "repo": {"type": "string", "description": "Repository name"},
          "title": {
            "type": "string",
            "description": "Issue title",
            "minLength": 1,
            "maxLength": 256
          },
          "body": {"type": "string", "description": "Issue body (Markdown)"},
          "labels": {"type": "array", "items": {"type": "string"}},
          "assignees": {"type": "array", "items": {"type": "string"}}
        },
        "required": ["owner", "repo", "title"]
      }
    }
  ]
}
```

---

## Appendix A: Format Conversion Reference

| FGP Feature | OpenAI | Anthropic | MCP |
|-------------|--------|-----------|-----|
| Method name dots | Convert to `_` | Keep as-is | Keep as-is |
| `description` | ✅ (1024 char limit) | ✅ | ✅ |
| `schema` | → `parameters` | → `input_schema` | → `inputSchema` |
| `$defs` / `$ref` | Inline | ✅ | Inline |
| `format` | ⚠️ Ignored | ✅ | ⚠️ Partial |
| `pattern` | ⚠️ Ignored | ✅ | ✅ |
| `minLength`/`maxLength` | ⚠️ Ignored | ✅ | ✅ |
| `examples` | ❌ Not supported | ❌ | ❌ |
| `returns` | ❌ Not supported | ❌ | ❌ |
| `errors` | ❌ Not supported | ❌ | ❌ |

---

## Appendix B: Implementation Checklist

- [ ] Extend `MethodInfo` struct with `schema`, `returns`, `examples`, `errors`
- [ ] Implement `schema` built-in method in `FgpServer`
- [ ] Add `SchemaBuilder` helper in SDK
- [ ] Add format converters: `to_openai()`, `to_anthropic()`, `to_mcp()`
- [ ] Update MCP bridge to use `schema` when available
- [ ] Add optional validation layer
- [ ] Update existing daemons to use rich schemas
- [ ] Document migration path

---

*Copyright 2026. Licensed under MIT.*
