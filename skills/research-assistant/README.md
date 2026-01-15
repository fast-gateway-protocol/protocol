# Research Assistant Skill

A composed FGP skill that demonstrates the skill package format.

## Installation

```bash
fgp skill install research-assistant
```

## Usage

### Via Natural Language (Claude Code / Cursor)

```
research quantum computing basics
look up the latest AI trends
find information about climate change solutions
```

### Via Command

```bash
fgp workflow run research --topic "quantum computing"
```

### With Email Output

```bash
fgp workflow run research --topic "AI trends 2026" --email-to "you@example.com"
```

## Configuration

Edit `~/.fgp/skills/research-assistant/config.yaml`:

```yaml
search_engine: google  # google, duckduckgo, bing
email_results: false
max_results: 10
```

## Requirements

- **browser** daemon (required)
- **gmail** daemon (optional, for email output)

## Workflows

| Workflow | Description |
|----------|-------------|
| `research` | Full research with optional email (default) |
| `quick-search` | Fast search, no email |

## Development

```bash
# Validate the skill manifest
fgp skill validate ./

# Export for Claude Code
fgp skill export claude-code research-assistant

# Run workflow locally
fgp workflow run research --topic "test" --dry-run
```

## License

MIT
