# Intro to n8n AI: AI News Briefing Digest

**Developer:** Glenn Mossy  
**Contributor:** Martin  
**Level:** slightly advanced follow-up to `intro-to-n8n`  
**Goal:** turn a key-free RSS feed into an AI-ranked technical news briefing using OpenAI `gpt-4o-mini`.

This package builds directly on the beginner **Daily Top AI Headline** workflow. The first version taught trigger, fetch, sort, limit, and field transformation. This version adds one production-shaped idea: use a secret API key safely and ask an LLM for structured JSON output.

## What This Workflow Does

The workflow reads recent AI headlines from RSS, keeps the newest five items, sends them to OpenAI, and returns one clean AI briefing.

Final output shape:

```json
{
  "topHeadline": "Example AI headline",
  "sourceLink": "https://...",
  "publicationDate": "2026-06-20T12:00:00.000Z",
  "summary": "A short technical summary.",
  "whyItMatters": "Why engineers should care.",
  "tags": ["agents", "model release"],
  "relevanceScore": 8,
  "candidateCount": 5,
  "generatedAt": "2026-06-20T12:05:00.000Z"
}
```

## Concepts Added

- **Environment secrets:** `OPENAI_API_KEY` is read from the n8n environment, not stored in workflow JSON.
- **Multi-item prompt prep:** the workflow combines five RSS items into one compact prompt.
- **Structured model output:** the OpenAI request asks for a JSON object.
- **Parsing and fallback:** the final Code node parses JSON and gives a debug-friendly fallback if parsing fails.

## Files

- `ai_news_briefing_workflow.json` - importable n8n workflow
- `docker-compose.yml` - pinned local n8n Docker setup
- `.env.example` - template for your OpenAI key
- `index.html` - GitHub Pages-friendly landing page
- `workflow-diagram.svg` - visual workflow diagram

## Setup

From this folder:

```bash
cp .env.example .env
```

Edit `.env` and add your own OpenAI key:

```bash
OPENAI_API_KEY=your_openai_key_here
```

Do **not** commit `.env`. It is intentionally ignored by Git in normal projects, and the real key should never appear in README files, screenshots, exported workflows, or frontend JavaScript.

Start n8n:

```bash
docker compose up -d
```

Open:

```text
http://localhost:5678
```

First run only: create the local n8n owner account.

Useful commands:

```bash
docker compose logs -f n8n
docker compose down
docker compose down -v
```

`docker compose down -v` resets the local n8n volume and deletes workflows/credentials for this demo.

## Import The Workflow

1. Open n8n at `http://localhost:5678`.
2. Go to **Workflows**.
3. Choose **Import from File**.
4. Select `ai_news_briefing_workflow.json`.
5. Open the imported workflow.
6. Click **Test workflow**.
7. Click the final **Parse AI Briefing** node and inspect the output panel.

Gotcha: if OpenAI returns `401`, your `OPENAI_API_KEY` is missing, invalid, or n8n was started before you edited `.env`. Run:

```bash
docker compose down
docker compose up -d
```

## Workflow Nodes

```text
Manual Trigger
  -> RSS Read
  -> Sort Newest First
  -> Limit To Top 5
  -> Prepare AI Prompt
  -> OpenAI Briefing
  -> Parse AI Briefing
```

### Manual Trigger

Starts the workflow when you click **Test workflow**. This keeps the advanced demo controlled and avoids accidental token use.

### RSS Read

Reads a keyless Google News RSS search feed:

```text
https://news.google.com/rss/search?q=artificial+intelligence&hl=en-US&gl=US&ceid=US:en
```

Google News RSS is unofficial. Test it from the meeting network before presenting. If it is flaky, paste in one of these:

- VentureBeat AI: `https://venturebeat.com/category/ai/feed/`
- MIT Technology Review AI: `https://www.technologyreview.com/topic/artificial-intelligence/feed/`
- Hacker News RSS: `https://news.ycombinator.com/rss`

### Sort Newest First

Sorts RSS items by `isoDate` descending. Some feeds use `pubDate`; if you swap feeds and sorting looks wrong, inspect the RSS Read output and adjust the field.

### Limit To Top 5

Keeps only five items before calling OpenAI. This reduces token use and gives the model enough choice to rank the most relevant item.

### Prepare AI Prompt

Combines the five RSS items into one JSON list and asks the model to pick the most important item for engineers.

### OpenAI Briefing

Calls:

```text
https://api.openai.com/v1/chat/completions
```

Model:

```text
gpt-4o-mini
```

Authorization header:

```text
Bearer {{ $env.OPENAI_API_KEY }}
```

The request uses JSON mode:

```json
{
  "response_format": { "type": "json_object" }
}
```

That makes the next node easier because it can parse the model response.

### Parse AI Briefing

Parses `choices[0].message.content`, returns the final clean object, and includes a fallback object if the model response was not valid JSON.

## Build It From Scratch

If you want to teach the construction instead of importing:

1. Build the beginner workflow first.
2. Change **Limit** from `1` to `5`.
3. Add a **Code** node called `Prepare AI Prompt`.
4. Use `$input.all()` to combine the five RSS items.
5. Add an **HTTP Request** node for OpenAI.
6. Use `Bearer {{ $env.OPENAI_API_KEY }}` in the Authorization header.
7. Add a final **Code** node to parse the AI JSON response.

The key teaching moment: workflow JSON is safe to share because it contains the expression `{{ $env.OPENAI_API_KEY }}`, not the secret itself.

## Stretch Goals

- Swap **Manual Trigger** for **Schedule Trigger** and run it every morning at 8:00.
- Add a Discord webhook output for the final briefing.
- Store the daily briefing in a Google Sheet, Airtable, or Notion database.
- Add a second RSS feed and merge sources before ranking.
- Add an IF node that only sends alerts when `relevanceScore >= 8`.

## Token Use

This workflow is intentionally small:

- five RSS items
- one OpenAI request
- compact JSON output
- manual trigger by default

For demos, avoid rapid repeated runs. When you switch to a Schedule Trigger, start with once per day.

