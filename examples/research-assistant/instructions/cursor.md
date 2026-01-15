# Research Assistant - Cursor Rules

## Trigger Detection

When user mentions:
- "research" + topic
- "look up" + topic
- "search for" + topic
- "/research" command

## Execution Pattern

1. Parse the topic from user input
2. Use FGP browser daemon for web search
3. Extract and summarize results
4. Optionally email findings if requested

## Commands

```bash
# Open search engine
fgp call browser.open -p '{"url": "https://google.com/search?q={encoded_topic}"}'

# Get page content
fgp call browser.snapshot

# Send email (if requested)
fgp call gmail.send -p '{"to": "{email}", "subject": "Research: {topic}", "body": "{content}"}'
```

## Output Format

Present research findings in markdown format with:
- Summary section
- Key findings as bullet points
- Source URLs
