# Installation Guide - AI Sales Telegram Bot

Complete deployment guide for the AI-powered Telegram sales assistant.

---

## Prerequisites Checklist

- [ ] N8N instance (cloud or self-hosted)
- [ ] Telegram account
- [ ] Google account (for Sheets)
- [ ] Mistral Cloud account (or API key)
- [ ] Basic understanding of AI agents and LLMs

---

## Part 1: Create Telegram Bot

### Step 1: Access BotFather

1. Open Telegram application
2. Search for **@BotFather** (official bot creation tool)
3. Start conversation by clicking "Start"

### Step 2: Create New Bot

```
Send command: /newbot

@BotFather will ask:
1. Choose a name for your bot (e.g., "My Store Assistant")
2. Choose a username ending with 'bot' (e.g., "mystorebot")

@BotFather responds with:
✅ Done! Congratulations on your new bot.
Token: 1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ
```

**Save this token securely - you'll need it later**

### Step 3: Configure Bot (Optional)

```bash
# Add bot description
/setdescription
# Then send your bot description

# Add bot about text
/setabouttext
# Then send short about text

# Add bot profile picture
/setuserpic
# Then send image file
```

### Step 4: Get Your Chat ID

1. Search for **@userinfobot** on Telegram
2. Send any message to the bot
3. Bot returns your user info including Chat ID
4. Save your Chat ID (numeric value)

---

## Part 2: Setup Google Sheets Inventory

### Step 1: Create Spreadsheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Create new spreadsheet
3. Name it appropriately (e.g., "Store Inventory" or "Product Catalog")

### Step 2: Design Sheet Structure

**Recommended column headers (Row 1):**
```
| Product Name | Size | Price | Quantity | Description | Category |
```

**Example data (Row 2 onward):**
```
| Blue Cotton Shirt | L    | 2500 | 5  | 100% Cotton, comfortable fit        | Shirts   |
| Slim Fit Jeans    | 32   | 3200 | 3  | Original denim, modern cut          | Pants    |
| Running Shoes     | 42   | 4500 | 8  | Lightweight, breathable mesh        | Footwear |
| Leather Wallet    | -    | 1800 | 12 | Genuine leather, multiple pockets   | Access.  |
```

### Step 3: Data Guidelines

**Best Practices:**
- Keep product names clear and consistent
- Use standard size formats (S, M, L, XL or numeric)
- Prices as numbers only (no currency symbols)
- Quantity as integer stock count
- Descriptions detailed but concise (50-100 characters)
- Categories for organization (optional)

**Data Validation:**
```
✅ "Blue Shirt" - Clear and simple
❌ "Blue shirt (NEW!!! 50% OFF)" - Too promotional

✅ Price: 2500 - Numeric only
❌ Price: "$25.00" - Has currency symbol

✅ Description: "100% Cotton, machine washable, slim fit"
❌ Description: "Super amazing best shirt ever!!!" - Too vague
```

### Step 4: Get Spreadsheet ID

1. Copy spreadsheet URL from browser
2. Extract ID from URL:
   ```
   https://docs.google.com/spreadsheets/d/[SPREADSHEET_ID]/edit
   ```
3. Save this ID

---

## Part 3: Get Mistral Cloud API Key

### Step 1: Create Mistral Account

1. Visit [Mistral AI Console](https://console.mistral.ai/)
2. Sign up for free account
3. Verify email address
4. Complete profile setup

### Step 2: Generate API Key

1. Navigate to "API Keys" section
2. Click "Create new secret key"
3. Name your key (e.g., "N8N Telegram Bot")
4. Copy the generated key immediately
5. Store securely (can't view again after closing)

### Step 3: Note Pricing

- Free tier includes credits for testing
- Production usage: ~$0.0002 per 1K tokens
- Average conversation: ~150 tokens
- Cost per 1000 messages: ~$0.20

---

## Part 4: Setup N8N

### Option A: N8N Cloud (Easiest)

1. Visit [n8n.cloud](https://n8n.cloud)
2. Create free account
3. Verify email
4. Access workspace

### Option B: Self-Hosted (Docker)

```bash
# Pull N8N image
docker pull n8nio/n8n

# Run container
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n

# Access at http://localhost:5678
```

### Option C: Self-Hosted (npm)

```bash
# Install globally
npm install n8n -g

# Start N8N
n8n start

# Access at http://localhost:5678
```

---

## Part 5: Configure N8N Credentials

### 1. Telegram API Credential

**Steps:**
1. N8N → Settings → Credentials
2. Click "Add Credential"
3. Select "Telegram API"
4. Name: "Telegram Bot"
5. Access Token: [Paste your bot token]
6. Click "Save"

### 2. Google Sheets OAuth2

**Steps:**
1. N8N → Settings → Credentials
2. Add → "Google Sheets OAuth2"
3. Name: "Google Sheets"
4. Client ID: [From Google Cloud Console]
5. Client Secret: [From Google Cloud Console]
6. Click "Connect my account"
7. Authorize in popup window
8. Save credential

**Google Cloud Console Setup:**
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create new project or select existing
3. Enable Google Sheets API
4. Create OAuth 2.0 Client ID
5. Add authorized redirect URI:
   ```
   https://your-n8n-instance.com/rest/oauth2-credential/callback
   ```
6. Copy Client ID and Secret

### 3. Mistral Cloud API

**Steps:**
1. N8N → Settings → Credentials
2. Add → "Mistral Cloud API"
3. Name: "Mistral AI"
4. API Key: [Paste your Mistral API key]
5. Save credential

---

## Part 6: Import Workflow

### Step 1: Download Workflow

```bash
git clone https://github.com/Benbrika8/ai-sales-telegram-bot.git
cd ai-sales-telegram-bot
```

### Step 2: Import to N8N

1. Open N8N dashboard
2. Click "Import from File" (top right)
3. Select `sl.json` from downloaded folder
4. Workflow appears in canvas

### Step 3: Assign Credentials

**For each node:**

1. **Telegram Trigger** (`recept`):
   - Credential: Select "Telegram Bot"
   - Updates: Leave as ["message", "*"]

2. **Google Sheets** (`Pull out the dressing table`):
   - Credential: Select "Google Sheets"
   - Document: Click dropdown → Select your inventory spreadsheet
   - Sheet: Select the sheet tab name

3. **AI Agent**:
   - No credential needed
   - Review prompt (Arabic by default)
   - Customize if needed

4. **Mistral Cloud Chat Model**:
   - Credential: Select "Mistral AI"
   - Model: Default (mistral-tiny or mistral-small)

5. **Telegram Send** (`²`):
   - Credential: Select "Telegram Bot"
   - Chat ID: Uses dynamic value from trigger

---

## Part 7: Customize AI Prompt

### Understanding the Prompt

The AI Agent node contains a prompt that defines bot behavior:

```javascript
Current structure:
- Role definition (professional store assistant)
- Goal (help customers, drive sales)
- Rules (inventory-only, friendly tone, alternatives)
- Inventory injection (JSON data from sheets)
- Customer message context
```

### Customization Examples

**For Electronics Store:**
```javascript
"You are an expert electronics advisor.
Help customers find the perfect device based on their needs.
Only recommend products from this inventory:
{ JSON.stringify($items(...)) }

Customer question: { $('recept').first().json.message.text }"
```

**For Food Delivery:**
```javascript
"You are a friendly restaurant assistant.
Help customers explore our menu and place orders.
Available items today:
{ JSON.stringify($items(...)) }

Customer message: { $('recept').first().json.message.text }"
```

**For Multi-language (English):**
```javascript
"You are a professional store assistant.
Your goal is to help customers and drive sales.
Rules:
1. Only suggest products from inventory below
2. Apologize and suggest alternatives if unavailable
3. Use friendly tone with emojis
4. Show accurate pricing

Inventory:
{ JSON.stringify($items(...)) }

Customer: { $('recept').first().json.message.text }"
```

---

## Part 8: Testing

### Step 1: Activate Workflow

1. In N8N, find the workflow toggle (top right)
2. Switch to "Active" (green indicator)

### Step 2: Test Bot

1. Open Telegram
2. Search for your bot username (e.g., @mystorebot)
3. Start conversation: Send `/start`
4. Send test message: "What products do you have?"

### Step 3: Verify Execution

**Check in N8N:**
1. Go to "Executions" tab
2. View recent workflow runs
3. Click on execution to see details
4. Inspect each node's input/output

**Expected Flow:**
```
Trigger receives message
  ↓
Google Sheets fetches inventory
  ↓
AI Agent processes with inventory context
  ↓
Mistral generates response
  ↓
Telegram sends response to customer
```

### Step 4: Test Scenarios

**Test Case 1: Available Product**
```
You: "Do you have blue shirts?"
Bot: Should list available blue shirts with prices
```

**Test Case 2: Unavailable Product**
```
You: "I need red shoes size 45"
Bot: Should apologize and suggest alternatives
```

**Test Case 3: Price Inquiry**
```
You: "How much are the jeans?"
Bot: Should provide accurate price from inventory
```

**Test Case 4: General Question**
```
You: "What categories do you have?"
Bot: Should list product categories from inventory
```

---

## Troubleshooting

### Bot Doesn't Respond

**Problem:** Send message to bot but no response

**Solutions:**
1. Check workflow is active (green toggle)
2. Verify Telegram bot token is correct
3. Ensure N8N instance is running
4. Check Telegram trigger node is configured
5. Review execution log for errors

### Wrong Products Suggested

**Problem:** Bot suggests products not in inventory

**Solutions:**
1. Verify Google Sheets data is loading:
   - Check execution log
   - Inspect "Pull out the dressing table" node output
2. Update AI prompt to be more strict:
   ```javascript
   "ONLY suggest products explicitly listed in the inventory JSON below.
   NEVER recommend products not in this list."
   ```
3. Check sheet name matches workflow configuration

### Slow Response (>5 seconds)

**Problem:** Bot takes too long to respond

**Solutions:**
1. **Optimize Google Sheets:**
   - Reduce number of rows
   - Remove unnecessary columns
   - Use filters instead of all data

2. **Upgrade Mistral Model:**
   ```javascript
   // In Mistral node, change model:
   mistral-tiny → mistral-small (faster)
   ```

3. **Check N8N Resources:**
   - Monitor CPU/memory usage
   - Upgrade instance if needed

4. **Reduce Prompt Length:**
   - Shorten instructions
   - Remove verbose examples

### Generic Responses

**Problem:** Bot gives vague answers instead of specific products

**Solutions:**
1. **Enhance Product Descriptions:**
   ```
   ❌ "Shirt"
   ✅ "Blue Cotton Shirt - Size L - Comfortable casual fit"
   ```

2. **Improve AI Prompt:**
   ```javascript
   "When recommending products, ALWAYS include:
   - Exact product name
   - Size
   - Price
   - Brief description"
   ```

3. **Verify Inventory Loading:**
   - Check execution log
   - Ensure JSON.stringify() is working
   - Confirm data format is correct

### Error: "Failed to fetch inventory"

**Problem:** Google Sheets node fails

**Solutions:**
1. Re-authorize Google Sheets OAuth2:
   - Delete existing credential
   - Create new and re-authorize

2. Check spreadsheet permissions:
   - Share sheet with N8N service account
   - Grant edit access

3. Verify spreadsheet ID is correct:
   - Extract ID from URL again
   - Update in workflow

---

## Security Best Practices

### API Key Management

```bash
# Never commit credentials to Git
# Use environment variables
# Store in N8N secure vault
# Rotate keys regularly
```

### Access Control

```bash
# Limit bot access to authorized users
# Implement user authentication
# Monitor for abuse
# Rate limiting for excessive requests
```

### Data Privacy

```bash
# Don't log sensitive customer data
# Comply with data protection regulations
# Implement data retention policies
# Secure customer information
```

---

## Optimization Tips

### For High Traffic (>1000 messages/day)

**1. Caching Strategy:**
```javascript
// Cache inventory in memory
// Refresh every 5 minutes instead of every message
```

**2. Queue Management:**
```javascript
// Enable N8N queue mode
executions.mode: "queue"
```

**3. Load Balancing:**
```javascript
// Run multiple N8N instances
// Distribute load across workers
```

### Cost Optimization

**1. Use Cheaper Mistral Model:**
```javascript
mistral-tiny: Lowest cost, fastest
mistral-small: Balanced
mistral-medium: Highest quality, slowest
```

**2. Optimize Token Usage:**
```javascript
// Shorter prompts
// Concise product descriptions
// Limit conversation context
```

**3. Implement Caching:**
```javascript
// Cache common responses
// Reduce repeated API calls
```

---

## Next Steps

1. ✅ Monitor initial conversations
2. ✅ Collect customer feedback
3. ✅ Iterate on AI prompt based on real interactions
4. ✅ Add more products to inventory
5. ✅ Implement order placement workflow
6. ✅ Integrate payment gateway
7. ✅ Add analytics tracking

---

## Need Help?

- 📧 Email: salahbenbrika2@gmail.com
- 💬 GitHub Issues: [https://github.com/Benbrika8/ai-sales-telegram-bot/issues](https://github.com/Benbrika8/ai-sales-telegram-bot/issues)
- 📖 N8N Docs: [docs.n8n.io](https://docs.n8n.io/)
- 🤖 LangChain Docs: [python.langchain.com](https://python.langchain.com/)
- 🧠 Mistral Docs: [docs.mistral.ai](https://docs.mistral.ai/)

---

**Installation Guide Version 1.0**  
Last Updated: March 2026  
Author: Benbrika Cherif Salah Eddine
