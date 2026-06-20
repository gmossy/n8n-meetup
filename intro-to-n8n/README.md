# Intro to n8n: Daily Top AI Headline

**Kickoff package:** beginner n8n onboarding for engineers  
**Developer:** Glenn Mossy  
**Contributor:** Martin  
**Goal:** start n8n, import or build one workflow, and run it successfully in about 15 minutes.

This guide assumes you have never used n8n and may be new to Docker. The workflow is intentionally key-free: it uses Google News RSS instead of a news API that requires credentials.

## What Is n8n?

n8n is a fair-code workflow automation tool. It gives you a visual canvas where you connect nodes together. Each node does one job: start a workflow, fetch data, transform data, branch, send a message, call an API, or return output.

Most beginner workflows follow the same shape:

- **Trigger:** how the workflow starts. Example: click **Test workflow**, run every morning, receive a webhook.
- **Fetch:** get data from somewhere. Example: read RSS, call an HTTP API, query a database.
- **Transform:** clean or reshape the data. Example: sort items, limit to one result, rename fields.
- **Output:** show, store, or send the result. Example: output panel, email, Slack, Discord, webhook response.

In this first workflow, you will fetch recent AI headlines from RSS, sort them newest first, keep the top item, and output a clean object with `title`, `link`, and `publicationDate`.

## Setup: Docker + n8n

### What Docker Is

Docker runs apps in containers. A container is a packaged runtime with the app and its dependencies bundled together. We use Docker for n8n because it avoids a long local install: no manual Node.js setup, no global packages, and no cleanup mystery.

For this demo, Docker gives you:

- one command to start n8n
- a named volume so your workflows survive restarts
- the same n8n version for everyone
- a clean reset path if something goes sideways

### Install Docker

**Windows/macOS**

1. Install Docker Desktop: <https://www.docker.com/products/docker-desktop/>
2. Start Docker Desktop.
3. Wait until it says Docker is running.

**Linux**

Install Docker Engine for your distro: <https://docs.docker.com/engine/install/>

Then verify Docker:

```bash
docker --version
docker compose version
```

Expected result: both commands print a version. If either command fails, fix Docker before continuing.

## Start n8n

From this folder:

```bash
cd intro-to-n8n
docker compose up -d
```

Open n8n:

```text
http://localhost:5678
```

On first run, n8n asks you to create the owner account. Use an email/password you will remember for this local instance.

### Useful Docker Commands

Start n8n:

```bash
docker compose up -d
```

View logs:

```bash
docker compose logs -f n8n
```

Stop n8n without deleting your workflows:

```bash
docker compose down
```

Reset everything, including workflows and owner account:

```bash
docker compose down -v
```

Be careful with `-v`: it deletes the named Docker volume where n8n stores data.

### Troubleshooting

**Port 5678 is already in use**

Something else is already listening on port 5678. Either stop that app or edit `docker-compose.yml`:

```yaml
ports:
  - "5679:5678"
```

Then open:

```text
http://localhost:5679
```

**Container will not start**

Check logs:

```bash
docker compose logs n8n
```

Most first-timer issues are Docker Desktop not running, port conflicts, or a copy/paste indentation mistake in YAML.

**Where data lives**

n8n data is stored in the named Docker volume:

```text
intro_n8n_data
```

That is why workflows survive `docker compose down`. They do not survive `docker compose down -v`.

## Workflow: Daily Top AI Headline

This workflow reads an AI news RSS feed and outputs the newest headline.

RSS source:

```text
https://news.google.com/rss/search?q=artificial+intelligence&hl=en-US&gl=US&ceid=US:en
```

Google News RSS is unofficial. It is convenient for a key-free demo, but it can occasionally rate-limit or vary by region. Before presenting, test the feed on the demo network. If it is flaky, use a direct publisher feed instead.

Curated alternatives you can paste into the RSS node:

- VentureBeat AI: `https://venturebeat.com/category/ai/feed/`
- MIT Technology Review AI: `https://www.technologyreview.com/topic/artificial-intelligence/feed/`
- Hacker News front page RSS: `https://news.ycombinator.com/rss`

## Option A: Import The Workflow

1. Open n8n at `http://localhost:5678`.
2. Go to **Workflows**.
3. Click **Import from File**.
4. Select `daily_top_ai_headline_workflow.json`.
5. Open the imported workflow.
6. Click **Test workflow**.
7. Inspect the output of the final **Edit Fields** node.

Expected output shape:

```json
{
  "title": "Example AI headline",
  "link": "https://...",
  "publicationDate": "2026-06-20T..."
}
```

## Option B: Build It From Scratch

### 1. Create A New Workflow

1. In n8n, click **Create Workflow**.
2. Rename it to `Daily Top AI Headline`.
3. Click the first node placeholder.
4. Choose **Manual Trigger**.

### 2. Add RSS Read

1. Click the plus button after **Manual Trigger**.
2. Search for **RSS Read** or **RSS Feed Read**.
3. Select the RSS node.
4. Set **URL** to:

```text
https://news.google.com/rss/search?q=artificial+intelligence&hl=en-US&gl=US&ceid=US:en
```

Gotcha: if the node returns no items during a live demo, paste in one of the curated fallback feeds above.

### 3. Add Sort

1. Click the plus button after **RSS Read**.
2. Search for **Sort**.
3. Add the **Sort** node.
4. Configure it to sort by the RSS date field.
5. Field name:

```text
isoDate
```

6. Order:

```text
Descending
```

Gotcha: some RSS feeds use `pubDate` instead of `isoDate`. If sorting by `isoDate` does not work, use `pubDate`.

### 4. Add Limit

1. Click the plus button after **Sort**.
2. Search for **Limit**.
3. Add the **Limit** node.
4. Set **Max Items** to:

```text
1
```

This keeps only the newest headline.

### 5. Add Edit Fields

1. Click the plus button after **Limit**.
2. Search for **Edit Fields**.
3. Add the node.
4. Add three fields:

| Field name | Value |
| --- | --- |
| `title` | `={{ $json.title }}` |
| `link` | `={{ $json.link }}` |
| `publicationDate` | `={{ $json.isoDate || $json.pubDate }}` |

5. Enable the option that keeps only the fields you set, if available in your n8n version.

Gotcha: n8n expressions start with `={{ ... }}`. If you paste without the equals sign, n8n treats it as plain text.

## Run And Read The Output

1. Click **Test workflow**.
2. n8n runs each node from left to right.
3. Click **RSS Read** to see all fetched feed items.
4. Click **Sort** to see the same items ordered newest first.
5. Click **Limit** to see one item.
6. Click **Edit Fields** to see the clean final object.

The output panel is your first debugging tool. If a downstream node looks wrong, click the node before it and inspect what changed.

## Stretch Goals

### Run Every Morning

Swap **Manual Trigger** for **Schedule Trigger**:

1. Delete or disable **Manual Trigger**.
2. Add **Schedule Trigger**.
3. Set it to run every day at:

```text
8:00 AM
```

4. Reconnect it to **RSS Read**.
5. Activate the workflow when you are ready for automatic runs.

Gotcha: test runs are manual. Scheduled runs only happen when the workflow is active.

### Add One Output

Pick one:

- **Email:** add an Email node and store SMTP or Gmail credentials in n8n credentials.
- **Slack:** add a Slack node and connect Slack credentials in n8n.
- **Discord:** add an HTTP Request node that posts to a Discord webhook URL.

Credentials belong in n8n credentials or environment variables, not in exported workflow JSON.

### Try A Proper News API Later

NewsAPI.org and similar services provide richer filtering, but they require API keys. That is a good next step after the first demo. Keep this first workflow key-free so everyone can complete it quickly.

## Concepts Learned

You used:

- a **Manual Trigger** to start a workflow on demand
- **RSS Read** to fetch external data without credentials
- **Sort** to order feed items by date
- **Limit** to keep one result
- **Edit Fields** to transform a noisy RSS item into a clean object
- the **output panel** to inspect each node

Next, a more advanced workflow can add real triggers, authentication, branching, error handling, and production output channels. That is where n8n starts feeling less like a demo and more like a small backend you can keep improving.

