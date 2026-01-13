# FGP - YC Demo Script (3 Minutes)

**Target:** YC W26 Application Video
**Format:** Live demo with narration
**Duration:** 3:00

---

## Pre-Demo Setup

**Terminal Setup:**
- Clean terminal with large, readable font
- macOS terminal with minimal prompt
- Screen recording at 1080p
- Have Gmail daemon already built but not installed

**Code Editors Open:**
- Claude Code (terminal)
- Cursor (IDE)
- Windsurf (IDE)

---

## Act 1: The Problem (30 seconds)

### Visual: Split Screen Montage

Show 5 different configuration files side-by-side:

1. **Claude Code** - `~/.claude/skills/gmail/SKILL.md`
2. **Cursor** - `.cursor/rules/gmail.mdc`
3. **Windsurf** - `.windsurf/workflows/gmail.md`
4. **Cline** - `cline_mcp_settings.json`
5. **Continue** - `~/.continue/config.yaml`

### Narration

> "AI coding agents are everywhere. Claude Code, Cursor, Windsurf, Cline, Continue. Each one is powerful. But there's a problem."

[Pause 2 seconds - let viewers see the different formats]

> "Every agent has its own format for capabilities. Claude Code uses SKILL.md files. Cursor uses .mdc rules. Windsurf has workflows. Cline uses MCP servers. Continue uses YAML configs."

[Zoom out to show all 5 files]

> "If you want Gmail in all your agents, you configure it five times, five different ways. That's broken."

---

## Act 2: The Solution (45 seconds)

### Visual: Clean Terminal

**Command 1: Detect Agents**

```bash
$ fgp agents
```

**Output:**
```
Detecting installed AI agents...

✓ Claude Code       (~/.claude/skills)
✓ Cursor           (.cursor/rules)
✓ Windsurf         (.windsurf/workflows)

3 agents detected
```

### Narration

> "We built FGP - the universal package manager for AI agents. Watch this."

---

**Command 2: Install Gmail**

```bash
$ fgp install gmail
```

**Output (animated, show each line appearing):**
```
Installing gmail v1.0.0...

[1/5] Downloading package... ✓ (245ms)
[2/5] Installing daemon to ~/.fgp/services/gmail/... ✓
[3/5] Configuring OAuth (Google)... ✓
[4/5] Installing agent skills...
      → Claude Code (SKILL.md) ✓
      → Cursor (.mdc) ✓
      → Windsurf (workflow) ✓
[5/5] Starting daemon... ✓

✨ Gmail is now available in all your AI agents!

Speed: 28ms average (vs 250ms MCP cold start)
Try: Ask any agent to "check my unread emails"
```

### Narration

> "One command. Works everywhere."

[Pause while install runs]

> "Behind the scenes, FGP installs a fast daemon - ten times faster than MCP - handles OAuth once, and automatically creates the right format for each agent."

---

## Act 3: The Experience (45 seconds)

### Visual: Three-Way Split Screen

**Left: Claude Code Terminal**
```
$ claude

User: Check my unread emails

Claude: Checking Gmail... (28ms)

You have 5 unread emails:

1. From: john@startup.com
   Subject: Meeting tomorrow at 10am
   Preview: Hey, can we move our sync to...

2. From: sarah@ycombinator.com
   Subject: Application status
   Preview: We received your application for...

3. [showing 3 more...]
```

**Center: Cursor IDE**
```
User: @gmail What emails did I get today?

Cursor: Fetching today's emails... (25ms)

Today's emails (3):
• 2:34 PM - john@startup.com: Meeting tomorrow
• 11:15 AM - sarah@ycombinator.com: Application status
• 9:42 AM - team@acme.co: Weekly update
```

**Right: Windsurf Terminal**
```
User: /gmail-check

Windsurf: Checking Gmail... (30ms)

📬 5 unread messages
📅 3 received today
⭐ 1 starred

Latest: "Meeting tomorrow" from john@startup.com
```

### Narration

> "Now all three agents have Gmail. Same capability, same speed, different formats. The developer configured it once. FGP handled the rest."

[Let the visuals sit for 3 seconds while responses finish]

> "And it's fast. 25 to 30 milliseconds. MCP takes 250 milliseconds on a cold start."

---

## Act 4: The Marketplace (30 seconds)

### Visual: Terminal

**Command: Search Marketplace**

```bash
$ fgp search calendar
```

**Output:**
```
🔍 Searching curated marketplace...

Found 3 packages matching "calendar":

╔══════════════════════════════════════════════════════╗
║ google-calendar                        [Official]    ║
║ ⭐ 1,247 installs                                    ║
║                                                      ║
║ Gmail, Drive, and Calendar in one daemon             ║
║ OAuth integration with Google Workspace              ║
║                                                      ║
║ $ fgp install google-calendar                        ║
╚══════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════╗
║ notion-calendar                                      ║
║ ⭐ 342 installs                                      ║
║                                                      ║
║ Sync Notion database tasks with calendar events      ║
║ Automatic time blocking and reminders                 ║
║                                                      ║
║ $ fgp install notion-calendar                        ║
╚══════════════════════════════════════════════════════╝

╔══════════════════════════════════════════════════════╗
║ apple-calendar                        [Native]       ║
║ ⭐ 892 installs                                      ║
║                                                      ║
║ macOS iCal integration (no cloud required)           ║
║ Local calendar access via EventKit                   ║
║                                                      ║
║ $ fgp install apple-calendar                         ║
╚══════════════════════════════════════════════════════╝
```

### Narration

> "We're building a curated marketplace. Every package is reviewed. One install, works everywhere."

[Scroll through results slowly]

> "Developers love it because it just works. We win because we own the distribution layer."

---

## Act 5: The Pitch (30 seconds)

### Visual: Clean Slides

**Slide 1: Logo + Tagline**
```
╔═══════════════════════════════════════════════╗
║                                               ║
║              F  G  P                          ║
║                                               ║
║   The Universal Package Manager               ║
║        for AI Agents                          ║
║                                               ║
╚═══════════════════════════════════════════════╝
```

**Slide 2: Key Features (Checklist)**
```
✓ One install, works everywhere
  (Claude Code, Cursor, Windsurf, Cline, Continue)

✓ 10x faster than MCP
  (28ms warm vs 250ms cold start)

✓ Curated marketplace
  (Quality-controlled, reviewed packages)

✓ Open Core business model
  (Protocol = MIT, Hub + Marketplace = Proprietary)
```

**Slide 3: The Moat**
```
┌─────────────────────────────────────────────┐
│  OPEN SOURCE (MIT)                          │
│  • FGP Protocol                             │
│  • Rust & Python SDKs                       │
│  • Reference implementations                │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│  PROPRIETARY                                │
│  • Translation Layer (agent adapters)       │
│  • Hub (unified lifecycle)                  │
│  • Marketplace (distribution)               │
└─────────────────────────────────────────────┘
```

### Narration

> "MCP was supposed to solve this, but every agent went their own way. We're not competing with any of them - we're making all of them better."

[Slide transition]

> "The protocol is open source. The translation layer and marketplace are our moat."

[Final slide]

> "Think Homebrew, but for AI agents. That's FGP."

---

**Final Screen:**
```
╔═══════════════════════════════════════════════╗
║                                               ║
║              fgp.dev                          ║
║                                               ║
║         Launching February 2026               ║
║                                               ║
║    github.com/fgp-protocol/fgp               ║
║                                               ║
╚═══════════════════════════════════════════════╝
```

[Fade to black, 2 seconds]

---

## Post-Production Notes

**Audio:**
- Professional mic (no echo)
- Background music: Subtle, modern, tech startup vibe
- Mix: Narration 80%, music 20%

**Pacing:**
- Act 1: Fast (build urgency)
- Act 2: Medium (let commands breathe)
- Act 3: Slow (show working agents)
- Act 4: Medium (marketplace browse)
- Act 5: Slow (drive home message)

**Editing:**
- Cut dead air
- Speed up install animations slightly (1.2x)
- Add subtle transitions between acts
- Use text overlays for key stats (10x faster, etc.)

**Length:**
- Target: 2:45 - 3:00
- If over: Cut Act 4 or shorten Act 3
- If under: Add more marketplace browsing

---

## Backup: If Live Demo Fails

Have a pre-recorded screencast ready. Narrate over it live. Say: "Let me show you a quick demo" and play the video. Then continue with slides.

---

## YC Application Questions (Quick Reference)

**What is your company going to make?**
> Universal package manager for AI coding agents. One command installs capabilities across Claude Code, Cursor, Windsurf, and others.

**Why is this important?**
> AI agents exploded but fragmented. Developers waste time configuring each agent separately. We're the distribution layer for AI capabilities.

**What's your traction?**
> Built working Gmail and iMessage daemons proving 10x speed advantage. 1 design partner secured. Located in SF, actively recruiting co-founder.

**What makes this defensible?**
> The translation layer. Anyone can build on our protocol, but maintaining compatibility across 5+ agents is ongoing work that creates lock-in. Plus we own the marketplace.

**Business model?**
> Open Core. Protocol + SDKs are MIT. Hub and marketplace are proprietary. Revenue from enterprise licenses ($1K-10K/year) and eventual marketplace fees (30% on paid packages).

---

*Last Updated: 01/12/2026 03:15 AM PST (via pst-timestamp)*
