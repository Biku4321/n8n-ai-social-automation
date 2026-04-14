# API Setup Documentation
## n8n AI Social Media Automation Workflow

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Google Gemini API Setup](#1-google-gemini-api-setup)
3. [Leonardo AI API Setup](#2-leonardo-ai-api-setup)
4. [Facebook Page API Setup](#3-facebook-page-api-setup)
5. [Instagram Graph API Setup](#4-instagram-graph-api-setup)
6. [Google Sheets API Setup](#5-google-sheets-api-setup)
7. [n8n Credential Configuration](#6-n8n-credential-configuration)
8. [Workflow Import & Activation](#7-workflow-import--activation)
9. [Testing the Workflow](#8-testing-the-workflow)

---

## Prerequisites

- n8n instance running (self-hosted or cloud) — version 1.0+
- Node.js 18+ (for self-hosted)
- Active accounts on: Google, Meta (Facebook/Instagram), Leonardo AI

---

## 1. Google Gemini API Setup

### Step 1: Enable Gemini API
1. Go to [https://aistudio.google.com/](https://aistudio.google.com/)
2. Sign in with your Google account
3. Click **"Get API Key"** → **"Create API Key"**
4. Select a Google Cloud project (or create new)
5. Copy the generated API Key — save it securely

### Step 2: Check Quotas
- Free tier: 60 requests/minute for Gemini 1.5 Pro
- If you need more, upgrade in Google Cloud Console → APIs & Services → Gemini API

### Environment Variable
```
GEMINI_API_KEY=AIza_your_key_here
```

### n8n Credential Setup
- Credential Type: **HTTP Query Auth**
- Name: `Gemini API Key`
- Parameter Name: `key`
- Parameter Value: `your_gemini_api_key`

---

## 2. Leonardo AI API Setup

### Step 1: Create Account
1. Go to [https://app.leonardo.ai/](https://app.leonardo.ai/)
2. Create account / Sign in
3. Navigate to **Account Settings** → **API Access**
4. Click **"Create New API Key"**
5. Copy and save your API key

### Step 2: Get Model ID (Optional — defaults are fine)
- Default model used: `b24e16ff-06e3-43eb-8d33-4416c2d75876` (Leonardo Diffusion XL)
- To get other model IDs:
  ```
  GET https://cloud.leonardo.ai/api/rest/v1/platformModels
  Authorization: Bearer YOUR_API_KEY
  ```

### Rate Limits
- Free tier: 150 API credits/day
- Each image generation costs ~10–25 credits depending on resolution

### n8n Credential Setup
- Credential Type: **HTTP Header Auth**
- Name: `Leonardo AI API Key`
- Header Name: `Authorization`
- Header Value: `Bearer your_leonardo_api_key`

---

## 3. Facebook Page API Setup

### Step 1: Create Meta Developer App
1. Go to [https://developers.facebook.com/](https://developers.facebook.com/)
2. Click **"My Apps"** → **"Create App"**
3. Choose **"Business"** type
4. Fill in app name and email → **Create App**

### Step 2: Get Page Access Token
1. In App Dashboard → **Add Product** → **Facebook Login**
2. Go to **Tools** → **Graph API Explorer**
3. Select your app from dropdown
4. Click **"Generate Access Token"**
5. Grant permissions: `pages_manage_posts`, `pages_read_engagement`, `publish_to_groups`
6. Select your Page from the dropdown
7. Copy the **Page Access Token** (this is long-lived after exchange)

### Step 3: Get Long-Lived Token (Recommended)
```bash
curl -X GET "https://graph.facebook.com/oauth/access_token
  ?grant_type=fb_exchange_token
  &client_id=YOUR_APP_ID
  &client_secret=YOUR_APP_SECRET
  &fb_exchange_token=SHORT_LIVED_TOKEN"
```

### Step 4: Get Page ID
```bash
curl "https://graph.facebook.com/me/accounts?access_token=YOUR_PAGE_TOKEN"
```
Copy the `id` field from the response.

### n8n Credential Setup
- Credential Type: **HTTP Query Auth**
- Name: `Facebook Page API`
- Parameter Name: `access_token`
- Parameter Value: `your_page_access_token`
- Also note your **Page ID** to put in the node URL

---

## 4. Instagram Graph API Setup

### Prerequisites
- Instagram account must be a **Professional/Business account**
- Must be linked to a Facebook Page

### Step 1: Link Instagram to Facebook Page
1. On Facebook Page → **Settings** → **Linked Accounts** → **Instagram**
2. Log in with Instagram credentials
3. Complete linking

### Step 2: Get Instagram User ID
1. Go to Graph API Explorer
2. Make request: `GET /me/accounts`
3. Find your page, then: `GET /{page-id}?fields=instagram_business_account`
4. Copy the `id` from `instagram_business_account`

### Step 3: Permissions Needed
- `instagram_basic`
- `instagram_content_publish`
- `pages_read_engagement`

### n8n Credential Setup
- Credential Type: **HTTP Query Auth**
- Name: `Instagram Graph API`
- Parameter Name: `access_token`
- Parameter Value: `same_page_access_token_as_facebook`
- Note your **Instagram User ID** for node URLs

---

## 5. Google Sheets API Setup

### Step 1: Create Google Cloud Project (if not done)
1. Go to [https://console.cloud.google.com/](https://console.cloud.google.com/)
2. **New Project** → Name it → **Create**

### Step 2: Enable Google Sheets API
1. Go to **APIs & Services** → **Library**
2. Search "Google Sheets API" → **Enable**
3. Also enable **Google Drive API**

### Step 3: Create OAuth2 Credentials
1. **APIs & Services** → **Credentials** → **Create Credentials** → **OAuth Client ID**
2. Application type: **Web Application**
3. Add Authorized redirect URIs:
   - For n8n cloud: `https://app.n8n.cloud/rest/oauth2-credential/callback`
   - For self-hosted: `https://YOUR_N8N_DOMAIN/rest/oauth2-credential/callback`
4. Download the JSON or note **Client ID** and **Client Secret**

### Step 4: Prepare Google Sheet
Create a new Google Sheet with two tabs:

**Tab 1: "Social Media Logs"** — Column headers:
```
Run ID | Timestamp | Topic | Brand | Image URL | Image Generated |
FB Caption | FB Hashtags | FB CTA | FB Post ID | FB Status | FB Error |
IG Caption | IG Hashtags | IG CTA | IG Post ID | IG Status | IG Error |
LinkedIn Caption | LinkedIn Hashtags | YouTube Title | YouTube Description |
Overall Status | Posted At
```

**Tab 2: "Error Logs"** — Column headers:
```
Run ID | Timestamp | Topic | Error Message | Error Node | Overall Status
```

### Step 5: Get Sheet ID
From the URL: `https://docs.google.com/spreadsheets/d/SHEET_ID_HERE/edit`
Copy the ID between `/d/` and `/edit`.

### n8n Credential Setup
- Credential Type: **Google Sheets OAuth2 API**
- Name: `Google Sheets OAuth2`
- Client ID: from Step 3
- Client Secret: from Step 3
- Click **Sign in with Google** and authorize

---

## 6. n8n Credential Configuration

After setting up all external APIs, configure credentials in n8n:

1. Open n8n → **Credentials** (left sidebar)
2. Click **"+ Add Credential"** for each:

| Credential Name | Type | Key Fields |
|---|---|---|
| Gemini API Key | HTTP Query Auth | name=`key`, value=API key |
| Leonardo AI API Key | HTTP Header Auth | name=`Authorization`, value=`Bearer KEY` |
| Facebook Page API | HTTP Query Auth | name=`access_token`, value=Page Token |
| Instagram Graph API | HTTP Query Auth | name=`access_token`, value=Page Token |
| Google Sheets OAuth2 | Google Sheets OAuth2 API | Client ID + Secret |

---

## 7. Workflow Import & Activation

### Import the Workflow
1. Open n8n dashboard
2. Click **"+ New Workflow"** or **"Import from file"**
3. Select **"Import from JSON"**
4. Upload `n8n_social_media_workflow.json`
5. Click **Import**

### Update Placeholders
After importing, update these values in the workflow:

| Node | Field to Update | Value |
|---|---|---|
| Post to Facebook | URL | Replace `{{ $credentials.pageId }}` with your actual Page ID |
| Instagram - Create Container | URL | Replace `{{ $credentials.igUserId }}` with your Instagram User ID |
| Instagram - Publish | URL | Same as above |
| Log to Google Sheets | documentId | Replace `YOUR_GOOGLE_SHEET_ID_HERE` |
| Log Error to Sheets | documentId | Same Sheet ID |

### Assign Credentials
Click each HTTP node → select the appropriate credential from dropdown.

### Activate
Toggle the workflow to **Active** (top right switch).

---

## 8. Testing the Workflow

### Webhook URL
After activation, your webhook URL will be:
```
https://YOUR_N8N_DOMAIN/webhook/social-media-trigger
```

### Test Request (cURL)
```bash
curl -X POST https://YOUR_N8N_DOMAIN/webhook/social-media-trigger \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "Summer fitness tips for busy professionals",
    "brand": "FitLife Co.",
    "tone": "motivational",
    "platforms": ["facebook", "instagram"]
  }'
```

### Expected Response
```json
{
  "success": true,
  "run_id": "run_1234567890",
  "topic": "Summer fitness tips for busy professionals",
  "image_url": "https://cdn.leonardo.ai/...",
  "facebook_status": "SUCCESS",
  "instagram_status": "SUCCESS",
  "overall_status": "PARTIAL_SUCCESS",
  "logged_to_sheets": true,
  "message": "Workflow completed successfully"
}
```

---

## Quick Troubleshooting

| Error | Likely Cause | Fix |
|---|---|---|
| 401 Unauthorized (Gemini) | Wrong/expired API key | Regenerate in Google AI Studio |
| 400 Bad Request (Leonardo) | Invalid model ID | Use default model ID in docs |
| 190 Error (Facebook/IG) | Expired access token | Generate new long-lived token |
| Gemini JSON parse error | Response not pure JSON | Check system prompt in Gemini node |
| Google Sheets 403 | Insufficient OAuth scopes | Re-authorize with correct scopes |
