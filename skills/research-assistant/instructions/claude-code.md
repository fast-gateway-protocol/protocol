# Research Assistant - Claude Code Instructions

## When to Use This Skill

Invoke this skill when the user:
- Asks to research a topic ("research quantum computing")
- Wants to look something up ("look up the latest AI news")
- Requests information gathering ("find information about...")
- Uses the /research command

## How to Execute

### Step 1: Start Browser Daemon
```bash
fgp call browser.open -p '{"url": "https://google.com"}'
```

### Step 2: Perform Search
```bash
fgp call browser.fill -p '{"selector": "input[name=q]", "value": "{topic}"}'
fgp call browser.click -p '{"selector": "input[type=submit]"}'
```

### Step 3: Extract Results
```bash
fgp call browser.snapshot
```

### Step 4: Email Results (Optional)
If the user requested email output:
```bash
fgp call gmail.send -p '{"to": "{email}", "subject": "Research: {topic}", "body": "{summary}"}'
```

## Response Format

After gathering information, present results as:
1. **Summary** - Brief overview of findings
2. **Key Points** - Bullet list of important information
3. **Sources** - Links to relevant pages

## Error Handling

- If browser daemon is not running, start it with `fgp start browser`
- If Gmail is not configured, inform user and skip email step
- If search returns no results, suggest alternative search terms

## Performance

- Browser operations: ~10ms each
- Full research workflow: ~500ms-2s depending on page load
