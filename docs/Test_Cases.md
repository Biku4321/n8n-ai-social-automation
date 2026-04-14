# Test Cases — n8n AI Social Media Automation Workflow

---

## Test Case 1: Standard Success Flow — Tech Product Launch

### Input Payload
```json
{
  "topic": "Launching our new AI-powered productivity app that helps remote teams collaborate better",
  "brand": "TeamSync AI",
  "tone": "professional",
  "platforms": ["facebook", "instagram", "linkedin"]
}
```

### Expected Behaviour
| Step | Expected Result |
|---|---|
| Input Validator | Passes validation; run_id generated |
| Gemini API Call | Returns platform-specific content JSON |
| Content Parsing | Extracts captions, hashtags, CTAs for all 4 platforms |
| Leonardo AI | Generates image of futuristic team collaboration workspace |
| Facebook Post | Creates photo post; returns `post_id` |
| Instagram | Creates container → publishes; returns `media_id` |
| Google Sheets | New row added to "Social Media Logs" tab |
| Webhook Response | `overall_status: "PARTIAL_SUCCESS"`, both platform IDs present |

### Expected Gemini Output (Sample)
```json
{
  "facebook": {
    "caption": "🚀 Exciting news! TeamSync AI is here to transform how your remote team works. Our new productivity app brings intelligent collaboration tools that adapt to your team's unique workflow. Say goodbye to endless email chains and missed deadlines — TeamSync AI keeps everyone aligned, no matter the timezone. Try it free today!",
    "hashtags": ["RemoteWork", "Productivity", "AITools", "TeamSync", "FutureOfWork"],
    "cta": "👉 Click the link in bio to start your free 14-day trial!"
  },
  "instagram": {
    "caption": "Working remote doesn't mean working alone 🤝✨ TeamSync AI just dropped and it's changing the game for distributed teams. Real-time collaboration, smart task management, and AI insights that actually get your workflow. We built this for YOU 💪",
    "hashtags": ["RemoteWork", "ProductivityHacks", "AIApp", "TeamSync", "WorkFromHome", "StartupLife", "TechLaunch", "CollaborationTools", "FutureOfWork", "DigitalNomad"],
    "cta": "🔗 Link in bio — grab your free trial before spots fill up!"
  },
  "linkedin": {
    "caption": "I'm thrilled to announce the launch of TeamSync AI — a productivity platform designed specifically for remote and hybrid teams.\n\nAfter 18 months of development and feedback from 500+ teams, we've built something that truly understands how modern teams work:\n\n✅ AI-powered task prioritization\n✅ Real-time collaboration spaces\n✅ Automated progress tracking\n✅ Cross-timezone scheduling intelligence\n\nRemote work shouldn't mean reduced productivity. TeamSync AI is here to prove it.\n\nWould love to hear from fellow founders and team leads — what's your biggest remote collaboration challenge?",
    "hashtags": ["ProductLaunch", "RemoteWork", "AI"],
    "cta": "Comment below or DM me to get early access for your team."
  },
  "youtube": {
    "title": "TeamSync AI: The Future of Remote Team Collaboration | Full Demo",
    "description": "In this video, we walk you through TeamSync AI — the AI-powered productivity platform built for remote teams. See how smart task management, real-time collaboration, and intelligent scheduling can transform your team's output. Watch the full demo, see real use cases, and find out how to get started with your free 14-day trial today.",
    "hashtags": ["TeamSyncAI", "RemoteWork", "ProductivityApp", "AITools", "TeamManagement"],
    "cta": "Subscribe for weekly productivity tips. Start your free trial — link in description."
  },
  "image_prompt": "Futuristic digital workspace with diverse remote team members connected through glowing holographic screens, collaborative AI interface floating in center, warm blue and purple tones, professional tech aesthetic, high quality 3D render",
  "video_prompt": "Time-lapse of a remote team dashboard coming alive with AI-powered tasks auto-organizing, notifications flowing, team avatars connecting across a world map visualization"
}
```

### Pass/Fail Criteria
- ✅ PASS: Both `facebook_status` and `instagram_status` = "SUCCESS" in Sheets log
- ✅ PASS: Image URL is a valid Leonardo CDN URL
- ✅ PASS: All 4 platform captions present in Sheets row
- ❌ FAIL: Any API returns 4xx/5xx without being caught by error handler

---

## Test Case 2: Error Handling — Missing Topic Field

### Input Payload
```json
{
  "brand": "TestBrand",
  "tone": "casual"
}
```
*(Note: `topic` field is intentionally missing)*

### Expected Behaviour
| Step | Expected Result |
|---|---|
| Input Validator | Throws error: "Missing required field: topic" |
| Error Handler | Catches error, logs to "Error Logs" sheet |
| Gemini API | NOT called |
| Posting nodes | NOT called |
| Webhook Response | HTTP 500 with error message |

### Expected Error Log Row in Sheets
| Run ID | Timestamp | Topic | Error Message | Error Node | Overall Status |
|---|---|---|---|---|---|
| ERROR_1234567 | 2025-01-15T10:30:00Z | Unknown | Missing required field: topic | Input Validator | ERROR |

### Pass/Fail Criteria
- ✅ PASS: Error is caught; does NOT crash the workflow
- ✅ PASS: Row appears in "Error Logs" tab (NOT "Social Media Logs")
- ✅ PASS: No API calls made to Gemini, Leonardo, or social platforms
- ❌ FAIL: Workflow crashes entirely without logging

---

## Test Case 3: Lifestyle/Consumer Brand — Instagram + Facebook

### Input Payload
```json
{
  "topic": "Handmade organic skincare products using Himalayan herbs, perfect for winter skin care routine",
  "brand": "Himalaya Glow",
  "tone": "warm and personal",
  "platforms": ["facebook", "instagram"]
}
```

### Expected Behaviour
| Step | Expected Result |
|---|---|
| Gemini | Generates warm, story-driven captions with skincare hashtags |
| Image Prompt | Nature-inspired: Himalayan herbs, clay pots, natural light aesthetic |
| Leonardo AI | Generates organic/natural aesthetic product image |
| Facebook | Long-form story post with image |
| Instagram | Emoji-rich, 10 hashtags, conversational CTA |
| Sheets | Full row logged with correct brand name |

### Expected Gemini Highlights (Sample)
- **Instagram Caption**: Warm, emoji-filled, talks about ingredients, personal story hook
- **Instagram Hashtags**: `OrganicSkincare`, `HimalayanHerbs`, `NaturalBeauty`, `WinterSkin`, `CleanBeauty`, `SkincareRoutine`, `HimalayanGlow`, `HandmadeSkincare`, `GlowUp`, `SkincareCommunity`
- **Image Prompt**: `"Flat lay of organic skincare products surrounded by dried Himalayan herbs, saffron, and rose petals on a wooden surface, warm golden hour lighting, earthy tones, natural minimalist aesthetic, professional product photography"`

### Performance Benchmarks
| Metric | Expected |
|---|---|
| Gemini API response time | < 5 seconds |
| Leonardo image generation | 15–30 seconds |
| Facebook post API | < 3 seconds |
| Instagram create + publish | < 10 seconds (with 5s wait) |
| Total end-to-end | < 60 seconds |

### Pass/Fail Criteria
- ✅ PASS: Brand name "Himalaya Glow" appears in all captions
- ✅ PASS: Instagram has 10 hashtags (platform-specific count)
- ✅ PASS: Image reflects Himalayan/organic aesthetic (visual check)
- ✅ PASS: Google Sheets row has "Himalaya Glow" in Brand column
- ❌ FAIL: Generic/non-brand-specific captions generated

---

## Summary Table

| Test | Scenario | Primary Check | Status |
|---|---|---|---|
| TC-1 | Standard success flow | All APIs succeed, Sheets logged | Expected PASS |
| TC-2 | Missing required input | Error caught, error log created | Expected PASS |
| TC-3 | Consumer/lifestyle brand | Brand voice applied, correct hashtag counts | Expected PASS |
