# 🤖 AI Social Media Automation — n8n Workflow

> **Input a topic → Get platform-specific captions, hashtags, a generated image, and posts to Facebook + Instagram — all logged to Google Sheets. Automatically.**

![Workflow Status](https://img.shields.io/badge/status-active-brightgreen)
![n8n](https://img.shields.io/badge/n8n-1.0+-orange)
![Gemini](https://img.shields.io/badge/AI-Gemini%201.5%20Pro-blue)
![Leonardo](https://img.shields.io/badge/Image-Leonardo%20AI-purple)
![Platforms](https://img.shields.io/badge/platforms-FB%20%7C%20IG%20%7C%20LinkedIn%20%7C%20YouTube-red)

---

## 📌 What This Does

Send one API call with a topic. Get back:

- ✅ Platform-specific captions for **Facebook, Instagram, LinkedIn, YouTube**
- ✅ Relevant **hashtags + CTAs** for each platform
- ✅ **AI-generated image** via Leonardo AI
- ✅ **Auto-posted** to Facebook + Instagram
- ✅ Everything **logged to Google Sheets** with status, post IDs, errors

---

## 🏗️ Architecture

```
POST /webhook/social-media-trigger
        │
        ▼
 ┌─────────────┐     ┌──────────────┐     ┌───────────────┐
 │  Validator  │────▶│  Gemini API  │────▶│  Leonardo AI  │
 │ (Code node) │     │ (4 platform  │     │ (image gen)   │
 └─────────────┘     │  captions)   │     └───────┬───────┘
                     └──────────────┘             │
                                                  ▼
                              ┌──────────────────────────────────┐
                              │         Post to Platforms        │
                              │  Facebook │ Instagram │ (+ more) │
                              └──────────────────┬───────────────┘
                                                 │
                                                 ▼
                                      ┌──────────────────┐
                                      │  Google Sheets   │
                                      │  (audit log)     │
                                      └──────────────────┘
```

---

## 📁 Project Structure

```
n8n-social-media/
├── workflow/
│   └── n8n_social_media_workflow.json   # Import this into n8n
├── docs/
│   ├── API_Setup_Documentation.md       # Step-by-step API setup
│   ├── Test_Cases.md                    # 3 full test cases with payloads
│   └── Final_Notes_Issues_Limitations.md
├── test/
│   └── test_payloads.json               # Ready-to-use curl payloads
├── screenshots/                         # Demo screenshots
├── .env.example                         # Copy → .env and fill in keys
├── .gitignore
└── README.md
```

---

## ⚡ Quick Start

### 1. Prerequisites

- [Node.js 18+](https://nodejs.org/)
- n8n installed globally
- API keys (see [API Setup](#-api-setup) below)

```bash
npm install -g n8n
```

### 2. Clone & configure

```bash
git clone https://github.com/yourusername/n8n-social-media-automation
cd n8n-social-media-automation
cp .env.example .env
# Fill in your API keys in .env
```

### 3. Start n8n

```bash
n8n start
# Opens at http://localhost:5678
```

### 4. Import the workflow

1. Open `http://localhost:5678`
2. Click **Workflows** → **+ New** → **Import from File**
3. Select `workflow/n8n_social_media_workflow.json`
4. Add credentials (see below)
5. Toggle workflow to **Active**

### 5. Fire your first test

```bash
curl -X POST http://localhost:5678/webhook/social-media-trigger \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "AI tools transforming small businesses in 2025",
    "brand": "TechLearn",
    "tone": "motivational",
    "platforms": ["facebook", "instagram"]
  }'
```

---

## 🔑 API Setup

| Service | Where to get key | Time |
|---|---|---|
| **Google Gemini** | [aistudio.google.com](https://aistudio.google.com) → Get API Key | 2 min |
| **Leonardo AI** | [app.leonardo.ai](https://app.leonardo.ai) → Account → API Access | 2 min |
| **Facebook Page Token** | [developers.facebook.com](https://developers.facebook.com) → Graph API Explorer | 15 min |
| **Instagram User ID** | Same app as Facebook — linked account | 5 min |
| **Google Sheets OAuth2** | [console.cloud.google.com](https://console.cloud.google.com) → Sheets API → OAuth2 | 10 min |

> **Full step-by-step instructions** → see [`docs/API_Setup_Documentation.md`](docs/API_Setup_Documentation.md)

### n8n Credential Setup Summary

| Credential Name | Type | Key Fields |
|---|---|---|
| Gemini API Key | HTTP Query Auth | name=`key`, value=your key |
| Leonardo AI API Key | HTTP Header Auth | `Authorization: Bearer YOUR_KEY` |
| Facebook Page API | HTTP Query Auth | name=`access_token` |
| Instagram Graph API | HTTP Query Auth | name=`access_token` (same token) |
| Google Sheets OAuth2 | Google Sheets OAuth2 API | Client ID + Secret → Sign in |

---

## 📥 Input Schema

`POST /webhook/social-media-trigger`

```json
{
  "topic": "string (required) — what to post about",
  "brand": "string (optional, default: 'Your Brand')",
  "tone": "string (optional) — professional | casual | motivational | witty",
  "platforms": ["facebook", "instagram", "linkedin", "youtube"],
  "schedule_time": "ISO 8601 (optional) — for scheduled posting"
}
```

### Example inputs

```json
// Tech product launch
{
  "topic": "Launching our AI-powered productivity app for remote teams",
  "brand": "TeamSync AI",
  "tone": "professional",
  "platforms": ["facebook", "instagram", "linkedin"]
}

// Lifestyle brand
{
  "topic": "5 morning habits that changed my productivity",
  "brand": "MindfulMornings",
  "tone": "casual",
  "platforms": ["facebook", "instagram"]
}

// E-commerce sale
{
  "topic": "Summer sale — 40% off all products this weekend only",
  "brand": "StyleHub",
  "tone": "witty",
  "platforms": ["facebook", "instagram"]
}
```

---

## 📤 Output Schema

```json
{
  "success": true,
  "run_id": "run_1718294827364",
  "topic": "your input topic",
  "image_url": "https://cdn.leonardo.ai/...",
  "facebook_status": "SUCCESS",
  "facebook_post_id": "123456789_987654321",
  "instagram_status": "SUCCESS",
  "instagram_media_id": "18023456789012345",
  "overall_status": "PARTIAL_SUCCESS",
  "logged_to_sheets": true,
  "message": "Workflow completed successfully"
}
```

---

## 🧪 Test Cases

See [`docs/Test_Cases.md`](docs/Test_Cases.md) for full test cases. Summary:

| # | Scenario | Platforms | Expected Result |
|---|---|---|---|
| 1 | Tech product launch | FB + IG + LinkedIn | Full success, image generated |
| 2 | Invalid input (missing topic) | — | Validator throws error, logged to Error Logs |
| 3 | Lifestyle / casual tone | FB + IG | Success with emoji-heavy captions |

---

## 📊 Google Sheets Structure

**Tab 1: Social Media Logs**

| Column | Description |
|---|---|
| Run ID | Unique run identifier |
| Timestamp | When the workflow ran |
| Topic | Input topic |
| Brand | Brand name |
| Image URL | Leonardo AI generated image URL |
| FB Caption / Hashtags / CTA / Post ID / Status | Facebook post details |
| IG Caption / Hashtags / CTA / Media ID / Status | Instagram post details |
| LinkedIn Caption | Generated (not auto-posted) |
| YouTube Title + Description | Generated (not auto-posted) |
| Overall Status | SUCCESS / PARTIAL_SUCCESS / FAILED |

**Tab 2: Error Logs**

| Column | Description |
|---|---|
| Run ID | Links to main log |
| Error Message | Full error text |
| Error Node | Which node failed |
| Timestamp | When it failed |

---

## ⚠️ Known Limitations

| Issue | Workaround |
|---|---|
| Facebook token expires in ~60 days | Exchange for long-lived token; refresh every 50 days |
| Instagram requires Business account | Convert account in-app (free, takes 1 min) |
| Leonardo AI takes 15–45s to generate | Wait node set to 30s; use polling for reliability |
| LinkedIn API requires partner access | Content generated but must be posted manually |
| YouTube requires actual video file | Description/title generated; upload is manual |
| n8n Cloud free tier: 5 runs/month | Use self-hosted n8n via Docker (unlimited) |

> **Full details** → [`docs/Final_Notes_Issues_Limitations.md`](docs/Final_Notes_Issues_Limitations.md)

---

## 🐳 Docker (Optional — Unlimited Runs)

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

---

## 🔐 Security Notes

- All API keys are stored in **n8n's encrypted credentials store** — never hardcoded in nodes
- `.env` file is git-ignored — never commit it
- For production: enable n8n basic auth (`N8N_BASIC_AUTH_ACTIVE=true`)
- Facebook/Instagram tokens should be long-lived and stored as n8n credentials only

---

## 💰 Cost Per Run

| Service | Free Tier | Paid |
|---|---|---|
| Gemini 1.5 Pro | 60 req/min free | ~$0.002 / 1K tokens |
| Leonardo AI | 150 credits/day | ~$0.02–$0.05/image |
| Facebook API | Free forever | Free |
| Instagram API | Free forever | Free |
| Google Sheets API | Free (500 req/100s) | Free |
| **Total** | **Free within limits** | **< $0.10/run** |

---

## 🛠️ Tech Stack

- **n8n** — workflow automation engine
- **Google Gemini 1.5 Pro** — content generation
- **Leonardo AI** — image generation (Diffusion XL)
- **Meta Graph API** — Facebook + Instagram posting
- **Google Sheets API** — logging and audit trail

---

## 📄 License

MIT — do whatever you want with it.

---

*Built as part of the n8n Intern Final Selection Round Assignment.*