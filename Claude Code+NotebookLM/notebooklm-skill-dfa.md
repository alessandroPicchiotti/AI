---
name: notebooklm
description: Query, manage, and automate Google NotebookLM from your AI agent. Add sources, generate branded slide decks, run research, and keep auth alive automatically. Configured for DFA Course Academy.
---

# NotebookLM Skill - DFA Course Academy

Connect your AI agent to Google NotebookLM. Add sources automatically, query your notebooks, generate branded slide decks, and keep your session alive without manual re-login.

Built for Krystian Wojtarowicz / DFA Course Academy - AI tools education, YouTube content, and Skool community management.

Built on top of [notebooklm-py](https://github.com/teng-lin/notebooklm-py) by Teng Lin.

---

## Prerequisites

### 1. Install notebooklm-py

```bash
pip install "notebooklm-py[browser]"
```

The `[browser]` extra installs Playwright, needed for the one-time login. All queries run without a browser after that.

**If pip is unavailable or the package is outdated**, install from a specific GitHub release tag:

```bash
# Get the latest release tag
LATEST_TAG=$(curl -s https://api.github.com/repos/teng-lin/notebooklm-py/releases/latest | grep '"tag_name"' | cut -d'"' -f4)
pip install "git+https://github.com/teng-lin/notebooklm-py@${LATEST_TAG}"
```

> Do NOT install from `git+https://github.com/teng-lin/notebooklm-py` without a tag - the main branch may contain unstable code.

### 2. Install Playwright's Chromium browser (one-time)

```bash
playwright install chromium
```

---

## Auth Setup (One-Time)

```bash
python scripts/nlm.py login
```

This opens a browser window. Sign into your Google account (the one connected to your DFA Course Academy workspace), wait until you see the NotebookLM homepage, then press ENTER in the terminal. Your session is saved to `~/.notebooklm/storage_state.json`.

**After this, your agent can access NotebookLM without any manual login.**

### Verify it worked

```bash
python scripts/nlm.py list
```

You should see your notebooks listed.

---

## Core Commands

All commands run through `python scripts/nlm.py`.

### Notebooks

```bash
# List all notebooks
python scripts/nlm.py list

# Create a new notebook
python scripts/nlm.py create "My Research"

# Describe a notebook (AI summary of contents)
python scripts/nlm.py describe NOTEBOOK_ID
```

### Sources

```bash
# Add a URL (YouTube videos, articles, course pages, etc.)
python scripts/nlm.py add-source --notebook-id NOTEBOOK_ID --url "https://example.com/article"

# Add a local file (PDF, markdown, txt, docx)
python scripts/nlm.py add-source --notebook-id NOTEBOOK_ID --file "/path/to/doc.pdf"

# Add raw text (e.g. course scripts, lesson outlines)
python scripts/nlm.py add-source --notebook-id NOTEBOOK_ID --title "My Notes" --text "Content here..."

# List sources in a notebook
python scripts/nlm.py sources --notebook-id NOTEBOOK_ID
```

### Querying

```bash
# Ask a question (set active notebook first, or pass --notebook-id)
python scripts/nlm.py ask "What are the main concepts?" --notebook-id NOTEBOOK_ID
```

### Local Library (save notebook metadata)

```bash
# Add to local library with a name and description
python scripts/nlm.py library-add --notebook-id NOTEBOOK_ID --name "ChatGPT Course Research" --description "Sources for the ChatGPT Image Generation course"

# Set as the active (default) notebook
python scripts/nlm.py library-activate chatgpt-course-research

# List library
python scripts/nlm.py library-list
```

**Suggested notebook names for DFA Course Academy:**
- `zapier-course` - Zapier automation course content
- `notebooklm-course` - NotebookLM course research and scripts
- `chatgpt-image-gen` - ChatGPT Image Generation course
- `n8n-ai-agents` - n8n and AI agents course
- `youtube-research` - YouTube video research and competitor analysis
- `ai-masters-content` - AI Masters paid community content
- `ai-enthusiasts-content` - AI Enthusiasts free community content

### Audio & Reports

```bash
# Generate deep-dive audio (podcast style)
python scripts/nlm.py generate-audio --notebook-id NOTEBOOK_ID

# Generate a briefing doc
python scripts/nlm.py generate-report --notebook-id NOTEBOOK_ID

# Generate a study guide
python scripts/nlm.py generate-report --notebook-id NOTEBOOK_ID --format study_guide

# Generate a blog post
python scripts/nlm.py generate-report --notebook-id NOTEBOOK_ID --format blog_post
```

### Artifacts

```bash
# List all generated artifacts (audio, reports, etc.)
python scripts/nlm.py artifacts --notebook-id NOTEBOOK_ID
```

---

## Slide Generation

### DFA Course Academy Brand Colors (pre-filled)

DFA Course Academy uses the following brand colors. These are already filled into the slide prompt template below - no need to ask Krystian every time.

| Role | Color | Hex |
|------|-------|-----|
| Primary | Gold / Yellow | `#F5C518` |
| Secondary | White | `#FFFFFF` |
| Background | Dark Navy | `#0D1B2A` |
| Text | Off-White | `#F0F0F0` |
| Corner Icon | Lightning bolt or "DFA" monogram | - |

> If Krystian specifies different colors for a specific project or campaign, override these defaults for that task only.

---

### Slide Prompt Template (DFA Blackboard Style)

Use this as a **Focus Prompt** inside NotebookLM (Studio -> Focus). The agent fills in the content placeholders before sending.

```
Create a [NUMBER]-slide presenter deck for "[VIDEO TITLE]."

Design: Dark blackboard on #0D1B2A. TITLES: bold slab serif (Rockwell or Roboto Slab), stamped/typeset, color #F5C518. Body text: handwritten chalk-style, color #F0F0F0. Each slide features one key glossy 3D object that floats above the chalk background. Chalk dashed connector lines link 3D elements to surrounding sketch labels. Every slide: dashed border in #F5C518 + lightning bolt icon top-right corner. #F5C518 PRIMARY, #FFFFFF SECONDARY, #F0F0F0 text, #0D1B2A background.

Slide 1: [SLIDE_1_DESCRIPTION]
Slide 2: [SLIDE_2_DESCRIPTION]
...

CRITICAL: Every slide - slab serif TITLE in #F5C518 at top, dashed #F5C518 border, lightning bolt icon top-right. Chalk body text. #0D1B2A background. One 3D object per slide. #F5C518 PRIMARY.
```

**Placeholders to fill (content only - colors are pre-set):**
| Placeholder | What to fill |
|-------------|-------------|
| `[NUMBER]` | How many slides |
| `[VIDEO TITLE]` | The topic/title - e.g. "How to Use NotebookLM for Course Creation" |
| `[SLIDE_N_DESCRIPTION]` | Per-slide: what the slide covers, what 3D object appears, what labels surround it |

### Per-slide description format

Each slide description should be around 150 characters. Be specific:

```
Slide 2: [TOPIC]. 3D [OBJECT] floating [POSITION]. Chalk labels: "[LABEL 1]," "[LABEL 2]," "[LABEL 3]." #F5C518 dashed arrow pointing to [KEY ELEMENT].
```

**3D object suggestions for DFA Course Academy content:**
- AI tools / ChatGPT -> glowing brain model or chat bubble
- Automation / Zapier / n8n -> terminal monitor or gear network
- Knowledge / NotebookLM -> glowing books or filing cabinet
- YouTube / video -> camera or play button
- Community / Skool -> group of people icons or speech bubbles
- Course creation -> laptop with course interface
- Time / scheduling -> clock or calendar
- Cloud / remote tools -> glowing cloud

### How to submit the focus prompt

Once filled in, run:

```bash
python scripts/nlm.py generate-report --notebook-id NOTEBOOK_ID --format custom --prompt "YOUR FILLED PROMPT HERE"
```

Or paste it directly into NotebookLM -> Studio -> Focus prompt.

> **Note:** NotebookLM has a ~5,000 character limit on focus prompts. If your prompt is too long, shorten the per-slide descriptions.

---

## DFA Course Academy Use Cases

### Course Research

Add YouTube videos, articles, and documentation as sources, then ask NotebookLM to summarize key concepts for a course lesson:

```bash
# Add a YouTube video on Zapier
python scripts/nlm.py add-source --notebook-id NOTEBOOK_ID --url "https://youtube.com/watch?v=EXAMPLE"

# Ask for a lesson structure
python scripts/nlm.py ask "Summarize the 5 most important Zapier concepts a beginner needs to know" --notebook-id NOTEBOOK_ID
```

### YouTube Video Research

Use a notebook per video topic. Add competitor videos, articles, and your own past scripts, then query for angles and talking points.

### AI Masters Community Content

Keep a notebook with AI Masters lesson ideas, member questions, and course updates. Query it to generate new content ideas or Community Expert briefing notes.

### Skool Post Ideas

Add trending AI news articles as sources, then ask for post ideas tailored to the AI Enthusiasts (free) or AI Masters (paid) community.

---

## Keeping Auth Alive (Optional)

Google session cookies expire after 7-30 days. There are two approaches:

### Option A - Simple (re-login when it breaks)

When you get an auth error, just run:

```bash
python scripts/nlm.py login
```

Takes 30 seconds. Fine for occasional use.

### Option B - Headless Auto-Refresh (recommended for agent setups)

`scripts/refresh_auth.py` silently refreshes your cookies using the persistent browser profile - no browser window, no manual steps. Run it every few days before cookies expire.

```bash
python scripts/refresh_auth.py          # Refresh cookies
python scripts/refresh_auth.py --check  # Just check if they're valid
```

**To automate this**, add a cron job:

```
# Refresh every 3 days at 5:30am
30 5 */3 * * python /path/to/scripts/refresh_auth.py
```

**For Claude Code agent setups**, add this to your agent's `cron-registry.json`:

```json
{
  "id": "notebooklm-auth-refresh",
  "name": "NotebookLM cookie refresh (every 3 days)",
  "cron": "30 5 */3 * *",
  "prompt": "Run: python /path/to/scripts/refresh_auth.py - if it fails, alert Krystian to run notebooklm login manually.",
  "enabled": true
}
```

> **How it works:** Your first login saves a full browser profile to `~/.notebooklm/browser_profile/`. Google keeps this session alive much longer than raw cookies because the browser handles token rotation automatically. The refresh script uses this profile headlessly to export fresh cookies - no human interaction needed.

---

## Troubleshooting

**Auth error / "Authentication expired"**
Run `python scripts/refresh_auth.py` first. If that fails, run `python scripts/nlm.py login` to do a full re-login.

**`nlm` command not found in terminal**
Use `python scripts/nlm.py` instead. This is a Windows PATH issue with certain Python installations (common with Windows Store Python). The scripts work fine when called with `python` directly.

**"No notebook specified"**
Either pass `--notebook-id YOUR_ID` or set an active notebook first:
```bash
python scripts/nlm.py library-activate your-notebook-name
```

**Slide generation times out**
NotebookLM can take 3-6 minutes to generate custom reports. If the command times out, check your artifacts list - the generation likely completed on NotebookLM's side:
```bash
python scripts/nlm.py artifacts --notebook-id NOTEBOOK_ID
```

**Playwright / Chromium not found**
Run `playwright install chromium` to install the browser.

**Windows: "UnicodeEncodeError" on output**
Set `PYTHONUTF8=1` in your environment, or run `$env:PYTHONUTF8="1"` in PowerShell before running the script.

---

## Credits

- **notebooklm-py** by [Teng Lin](https://github.com/teng-lin/notebooklm-py) - the library that makes this possible
- **This skill** customized for [DFA Course Academy](https://dfacourseacademy.com) by Krystian Wojtarowicz
