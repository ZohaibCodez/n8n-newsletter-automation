# Detailed Setup Guide

## 📋 Prerequisites Checklist

Before you begin, ensure you have:
- [ ] n8n account (cloud at app.n8n.io or self-hosted)
- [ ] Tavily API key
- [ ] Google AI Studio API key (Gemini)
- [ ] Gmail account for drafts
- [ ] ~30 minutes for setup

---

## 🔑 Step 1: Get Your API Keys

### Tavily API (Free Tier)
1. Go to [tavily.com](https://tavily.com)
2. Sign up for free account
3. Navigate to API Keys section
4. Copy your API key
5. **Free tier:** 1,000 searches/month

### Google Gemini API (Free Tier)
1. Visit [Google AI Studio](https://aistudio.google.com)
2. Click "Get API key"
3. Create new project (or use existing)
4. Copy API key
5. **Free tier:** 60 requests/minute

---

## 🔧 Step 2: Configure n8n

### Enable Community Nodes (if needed)
```
n8n Settings → Community Nodes → Enable installation
```

### Add Credentials

#### Tavily
1. In n8n, go to **Credentials** → **Add Credential**
2. Search "Tavily"
3. Paste API key
4. Name it: `Tavily API`
5. Save

#### Google Gemini
1. **Credentials** → **Add Credential**
2. Search "Google AI"
3. Paste API key
4. Name it: `Gemini API`
5. Save

#### Gmail OAuth
1. **Credentials** → **Add Credential**
2. Search "Gmail OAuth2"
3. Click "Connect my account"
4. Authorize with Google
5. Grant permissions: "Send email", "Create drafts"
6. Save

---

## 📥 Step 3: Import Workflow

1. Download `newsletter-workflow.json` from this repo
2. In n8n: **Workflows** → **Add workflow** → **Import from File**
3. Select the JSON file
4. Click "Import"

---

## ⚙️ Step 4: Configure Nodes

### 1. Schedule Trigger
- **Frequency:** Weekly
- **Day:** Sunday (or your preference)
- **Time:** 00:00 (adjust to your timezone)
- **Leave inactive** while testing

### 2. Initial Research (Tavily)
- **Credential:** Select your Tavily credential
- **Query:** Change to your niche
  ```
  Example niches:
  - "sustainable fashion innovations"
  - "remote work productivity tools"
  - "cryptocurrency regulations"
  - "AI tools for small businesses"
  ```
- **Options:**
  - Topic: `news`
  - Time range: `week`
  - Max results: `3`
  - Include raw content: `false`

### 3. Planning Agent
- **Credential:** Select your Gemini credential
- **Model:** `models/gemini-2.0-flash-exp` or `models/gemini-2.5-pro`
- **Verify:**
  - Structured Output Parser is ON
  - JSON schema is loaded (should be pre-configured)

### 4. Topic Research (Tavily)
- **Credential:** Same Tavily credential
- **Query:** Should reference split item: `{{ $json }}`
- **Options:**
  - Time range: `month`
  - Max results: `5`
  - Include raw content: `true` ⚠️ Important!

### 5. Section Writer Agent
- **Credential:** Gemini credential
- **Model:** Use cheaper model like `gemini-2.0-flash-exp`
- **Verify prompts** are loaded

### 6. Editor Agent
- **Credential:** Gemini credential
- **Model:** Use stronger model `models/gemini-2.5-pro`
- **Verify:**
  - Structured Output Parser is ON
  - Both `subject` and `content` fields defined

### 7. Create Draft (Gmail)
- **Credential:** Select your Gmail OAuth
- **To:** Your test email (or leave empty for drafts)
- **Subject:** `={{ $json.output.subject }}`
- **Email Type:** **HTML** ⚠️ Must be HTML!
- **Message:** `={{ $json.output.content }}`

---

## 🧪 Step 5: Test Run

### First Test (Manual)
1. **Disable** Schedule Trigger (toggle off)
2. Click **Execute Workflow** button
3. Watch execution flow:
   - Initial Research → should return 3 articles
   - Planning Agent → should output title + 3 topics
   - Split Out → should show "3 items"
   - Topic Research → should run 3 times
   - Section Writer → should run 3 times
   - Aggregate → should combine to 1 item
   - Editor → should output subject + HTML
   - Gmail → should create draft

4. **Check Gmail:**
   - Open Gmail → Drafts folder
   - Find your newsletter draft
   - Verify HTML formatting looks good

### Pin Data for Faster Testing
After successful run:
1. Click **Pin Data** on nodes that call external APIs
2. Recommended nodes to pin:
   - Initial Research
   - Planning Agent
   - Topic Research
3. This saves API calls during prompt iteration

---

## 🎨 Step 6: Customize

### Change Newsletter Style

Edit **Editor Agent** system prompt:
```
Modify sections:
- Header styling (colors, fonts)
- Layout (single column vs multi-column)
- Footer content (signature, social links)
```

### Adjust Content Length

Edit **Section Writer** system prompt:
```
Current: "350-500 words"
Change to: "200-300 words" (shorter)
         or "500-700 words" (longer)
```

### Add Your Branding

In Editor prompt, add:
```
Include at bottom:
- Logo: <img src="YOUR_LOGO_URL" alt="Logo">
- Tagline: "Your tagline here"
- Social links: [LinkedIn] [Twitter] [Website]
```

---

## 🚀 Step 7: Go Live

1. **Final test run** with real niche query
2. Review generated newsletter quality
3. **Enable Schedule Trigger**
4. Set timezone correctly
5. Monitor first scheduled run

---

## 💰 Cost Estimation

### Free Tier Limits
- **Tavily:** 1,000 searches/month = ~4 newsletters/month with this workflow
- **Gemini:** 60 RPM = sufficient for weekly newsletters
- **Gmail:** Unlimited drafts

### Per Newsletter Cost (if exceeding free tier)
- Tavily searches: ~$0.03 (8 searches × $0.004)
- Gemini API calls: ~$0.02-0.05 (depends on model choice)
- **Total per newsletter:** ~$0.05-0.08

**Annual cost for weekly newsletters:** ~$3-5

---

## ❓ FAQ

**Q: Can I use OpenAI instead of Gemini?**
A: Yes! Replace "Google AI" nodes with "OpenAI" nodes. Use GPT-4 for Editor, GPT-3.5 for others.

**Q: How do I change newsletter frequency?**
A: Modify Schedule Trigger settings (daily, weekly, monthly).

**Q: Can I send directly instead of creating drafts?**
A: Yes, replace "Create Draft" with "Send Email" node. ⚠️ Add approval step first!

**Q: The HTML looks broken in Gmail**
A: Ensure Gmail node "Email Type" is set to "HTML", not "Text".

**Q: How do I add images?**
A: Modify Editor prompt to include `<img>` tags with public URLs. Don't rely on AI to generate images.

---

## 🆘 Getting Help

- **n8n Community:** [community.n8n.io](https://community.n8n.io)
- **Issues:** Open GitHub issue on this repo
- **n8n Docs:** [docs.n8n.io](https://docs.n8n.io)

---

## ✅ Post-Setup Checklist

- [ ] All API keys configured
- [ ] Test run successful
- [ ] Gmail draft received
- [ ] HTML formatting correct
- [ ] Niche query customized
- [ ] Schedule set to your preference
- [ ] Pin data saved for testing
- [ ] Schedule trigger enabled

**You're ready to automate!** 🎉