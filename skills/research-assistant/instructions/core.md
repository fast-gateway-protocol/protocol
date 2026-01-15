# Research Assistant Skill

This skill helps you research topics using web search and optionally email the findings.

## Capabilities

- **Web Search**: Search the web for information on any topic
- **Content Extraction**: Extract and summarize relevant content from search results
- **Email Output**: Optionally email the research findings (requires Gmail daemon)

## Usage

### Basic Research
```
research {topic}
```

Example: "research quantum computing basics"

### Research with Email
```
research {topic} and email results to {email}
```

Example: "research AI trends 2026 and email results to john@example.com"

## Available Workflows

1. **research** (default) - Full research workflow with optional email
2. **quick-search** - Fast search without email output

## Configuration

- `search_engine`: google (default), duckduckgo, or bing
- `email_results`: true/false (default: false)
- `max_results`: number of results to return (default: 10)

## Dependencies

- **browser** daemon (required) - For web navigation and content extraction
- **gmail** daemon (optional) - For emailing results
