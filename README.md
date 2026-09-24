# 🔍 SEO Research Agent — n8n Automation Workflow

An end-to-end, AI-powered SEO research and content generation agent built in [n8n](https://n8n.io/). Give it a topic, a website, and one or more target countries — it does the rest: keyword research, competitor analysis, entity extraction, SEO content writing, and an automated SEO checklist.

## ✨ What it does

1. **Input** — Collects a topic, target website URL, and target countries via a simple form.
2. **Website Understanding** — Scrapes the target website itself to understand its actual business/services, so generated content matches the real nature of the site (not generic filler).
3. **Competitor Research** — For each target country, finds the top-ranking competitors via [Serper.dev](https://serper.dev) (Google Search API) and scrapes their pages.
4. **Keyword & Entity Extraction** — Uses Google Gemini to extract 30+ relevant SEO keywords (scored and mapped to countries) and related entities from the competitor data.
5. **Refinement Pass** — Runs a second AI pass to remove irrelevant keywords/entities and clean up the list.
6. **Google Sheets Export** — Saves the best keywords and entities into a Google Sheet, organized by tab.
7. **SEO Content Generation** — Writes a complete, structured SEO article (title, meta description, H1/H2/H3, 900–1300 words) tailored to the target website's actual business.
8. **Automated SEO Checklist** — Verifies title/meta length, heading structure, word count, and confirms every "best" keyword and entity was actually used in the content — flags anything missing.
9. **Output** — Final content is saved to a Google Sheet tab (and optionally exported to a formatted Google Doc with real headings and bold text).

## 🧱 Architecture

This is a **single linear pipeline** (no branching), making it easy to follow and debug:

```
Form Trigger
  → Fetch Target Website → Parse Target Website Content
  → Split Countries
  → Serper (Get Competitors) → Extract Top Competitor URLs
  → Fetch Competitor Page → Parse Page Content
  → Combine All Competitor Data
  → Build LLM Prompt → Gemini (Extract Keywords & Entities) → Parse LLM JSON
  → Build Refine Prompt → Gemini (Refine Keywords & Entities) → Parse Refined JSON
  → Score & Format Keywords → Append Keywords to Sheet
  → Format Entities → Append Entities to Sheet
  → Build Article Prompt → Gemini (Write SEO Article)
  → Run SEO Checklist
  → Append Final Content to Sheet
  → (optional) Build Google Docs Requests → Format & Insert Content (Google Docs)
```

## 🔑 Requirements (all have free tiers)

| Service | Used for | Free tier |
|---|---|---|
| [Serper.dev](https://serper.dev) | Competitor search results (Google SERP data) | 2,500 free queries, no card required |
| [Google AI Studio (Gemini)](https://aistudio.google.com) | Keyword/entity extraction, refinement, content writing | Generous free tier |
| Google Sheets API + Google Drive API | Storing keywords, entities, and final content | Free (Google Cloud project) |
| Google Docs API *(optional)* | Exporting a formatted final document | Free (Google Cloud project) |

## ⚙️ Setup

### 1. Import the workflow
Import the `.json` file into n8n: **Workflows → Import from File**.

### 2. Add your API keys
Replace these placeholders inside the workflow:

- `PUT_SERPER_API_KEY_HERE` → in the **Serper - Get Competitors** node header (`X-API-KEY`)
- `PUT_GEMINI_API_KEY_HERE` → in all three Gemini HTTP Request nodes (URL query param `key=`)

### 3. Connect Google Sheets
1. Create a Google Sheet with 3 tabs: `Keywords`, `Entities`, `Final Content` — each with a header row matching the field names used in the workflow (`keyword | score | countries | is_best`, `entity | type`).
2. In Google Cloud Console, enable **Google Sheets API** and **Google Drive API** for your project.
3. Create an OAuth 2.0 Client ID (Web application) and add n8n's OAuth redirect URL to **Authorized redirect URIs**.
4. In n8n, create a **Google Sheets** credential using that Client ID/Secret, sign in, and select it in all `Append ... to Sheet` nodes.
5. Replace `PUT_YOUR_GOOGLE_SHEET_ID_HERE` with your sheet's ID (or select it via the "From list" resource picker).

### 4. (Optional) Connect Google Docs
1. Enable **Google Docs API** in Google Cloud Console.
2. Create a **generic** `Google OAuth2 API` credential (not the node-specific `Google Docs OAuth2 API`) with scope `https://www.googleapis.com/auth/documents`, so it can be used inside HTTP Request nodes.
3. Sign in and select it in the `Create a document` and `Format & Insert Content` nodes.

### 5. Run it
Trigger the workflow via the form, fill in:
- **Topic** — what the content should be about
- **Website URL** — the site the content is for
- **Target Countries** — comma-separated (e.g. `PK,US,GB`)

## 📌 Notes

- Gemini model names change over time — if you hit a "model not found" error, check [Google's model list](https://ai.google.dev/gemini-api/docs/models) and update the model in the URL.
- The `retryOnFail` setting on the content-writing node handles occasional "503 - model overloaded" errors automatically.
- This project was built for personal/agency use in offering website + SEO services to local businesses — costs stay within free tiers for light/moderate use, but scale-up usage will incur API costs (mainly Gemini and Serper).

## 🛠️ Roadmap
- [ ] Phase 2: Auto-generate an Elementor/WordPress-ready page template (colors, fonts, images, content) from the same data.

## ⚠️ Security

Never commit real API keys to this repository. Placeholder values (`PUT_..._HERE`) are used intentionally — replace them locally or via n8n credentials, not in version control.
