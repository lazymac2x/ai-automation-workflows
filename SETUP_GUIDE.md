# Setup Guide — AI Automation Workflows

Step-by-step instructions for importing and configuring each workflow.

---

## General Setup

### n8n Requirements
- n8n version 1.0 or later (self-hosted or n8n Cloud)
- Admin access to create credentials

### How to Import a Workflow
1. Open your n8n dashboard
2. Click **Workflows** in the left sidebar
3. Click the **+** button or **Import from File**
4. Select the `.json` file from this pack
5. The workflow opens in the editor — configure credentials before activating

### Setting Environment Variables (Optional)
Instead of hardcoding values in nodes, you can use n8n environment variables:
- **Self-hosted:** Add to your `.env` file or Docker environment
- **n8n Cloud:** Settings > Environment Variables

---

## Workflow 1: Crypto Signal to Telegram Bot

**File:** `workflows/crypto-signal-telegram.json`

### Credentials Needed
1. **HTTP Header Auth** — Your crypto signal API key
2. **Telegram Bot** — Bot token from @BotFather

### Setup Steps

#### Step 1: Create a Telegram Bot
1. Open Telegram, search for `@BotFather`
2. Send `/newbot` and follow the prompts
3. Copy the bot token (e.g., `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`)
4. Start a chat with your bot (required for the bot to send you messages)
5. Get your Chat ID: send a message to your bot, then visit `https://api.telegram.org/bot<TOKEN>/getUpdates` and find `chat.id`

#### Step 2: Configure n8n Credentials
1. In n8n, go to **Credentials** > **New**
2. Choose **Telegram API** and paste your bot token
3. Choose **HTTP Header Auth** and set:
   - Name: `Authorization`
   - Value: `Bearer YOUR_API_KEY` (or whatever your crypto API requires)

#### Step 3: Update Workflow Nodes
1. Open the workflow
2. Click **Fetch Crypto Signals** node:
   - Update the URL to your crypto signal API endpoint
   - Select the HTTP Header Auth credential you created
3. Click **Send Telegram Alert** node:
   - Select the Telegram credential you created
   - Set `chatId` to your Telegram chat ID
4. Click **Filter BUY Signals** node:
   - Adjust conditions if your API uses different signal field names

#### Step 4: Test & Activate
1. Click **Execute Workflow** to test manually
2. Check the Telegram chat for the alert message
3. If successful, toggle the workflow to **Active**

---

## Workflow 2: RSS Feed to Social Media Auto-Post

**File:** `workflows/rss-to-social.json`

### Credentials Needed
1. **OpenAI API** — API key from platform.openai.com
2. **Twitter/X OAuth2** — Twitter developer app credentials
3. **Slack OAuth2** — Slack app with `chat:write` scope

### Setup Steps

#### Step 1: OpenAI API Key
1. Go to [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Create a new API key
3. In n8n: Credentials > New > OpenAI API > paste the key

#### Step 2: Twitter/X App
1. Go to [developer.twitter.com](https://developer.twitter.com)
2. Create a new project and app
3. Enable OAuth 2.0 with `tweet.read` and `tweet.write` scopes
4. Set callback URL to your n8n OAuth callback URL
5. In n8n: Credentials > New > Twitter OAuth2 API > enter Client ID and Secret

#### Step 3: Slack App
1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Create a new app from scratch
3. Add OAuth scopes: `chat:write`, `channels:read`
4. Install to workspace and copy the Bot Token
5. In n8n: Credentials > New > Slack OAuth2 API > configure

#### Step 4: Configure Workflow
1. Open the workflow
2. Click **RSS Feed Trigger**: set the RSS URL you want to monitor
3. Click **AI Generate Social Posts**: select your OpenAI credential
4. Click **Post to Twitter/X**: select your Twitter credential
5. Click **Notify Slack**: select your Slack credential, set the channel name

#### Step 5: Test & Activate
1. Execute manually to test with the latest RSS item
2. Check Twitter for the posted tweet
3. Check Slack for the log message
4. Activate when ready

---

## Workflow 3: AI Email Auto-Responder

**File:** `workflows/email-ai-responder.json`

### Credentials Needed
1. **IMAP** — Email account to monitor (Gmail, Outlook, etc.)
2. **OpenAI API** — For classification and response drafting
3. **SMTP** — Email account for sending replies
4. **Google Sheets OAuth2** — For logging

### Setup Steps

#### Step 1: IMAP Configuration
For **Gmail**:
- Host: `imap.gmail.com`, Port: `993`, SSL: `true`
- Use an App Password (not your regular password): Google Account > Security > 2-Step Verification > App Passwords

For **Outlook**:
- Host: `outlook.office365.com`, Port: `993`, SSL: `true`

#### Step 2: SMTP Configuration
For **Gmail**:
- Host: `smtp.gmail.com`, Port: `465`, SSL: `true`
- Same App Password as IMAP

For **Outlook**:
- Host: `smtp.office365.com`, Port: `587`, TLS: `true`

#### Step 3: Google Sheets Setup
1. Create a new Google Sheet
2. Add headers in Row 1: `Date`, `From`, `Subject`, `Category`, `Sentiment`, `Confidence`, `Requires Human`, `Response Sent`, `Response Tone`
3. In n8n, create Google Sheets OAuth2 credentials
4. Update the **Log to Google Sheets** node with your Sheet URL

#### Step 4: Configure Workflow
1. Select IMAP credential in the trigger node
2. Select OpenAI credential in both AI nodes
3. Select SMTP credential in the Send Email node
4. Update the from email address in the Send Email node
5. Update the Google Sheet URL in the logging node

#### Step 5: Important Considerations
- Start with the workflow **inactive** and test with a few emails
- Review AI-drafted responses before enabling auto-send
- Set `requires_human: true` threshold in the classification to catch sensitive emails
- Consider adding a manual approval step for high-stakes emails

---

## Workflow 4: Website Change Monitor & Alert

**File:** `workflows/web-monitor-alert.json`

### Credentials Needed
1. **Telegram Bot** — Same as Workflow 1
2. **Slack OAuth2** — Same as Workflow 2
3. **SMTP** — For email alerts

### Setup Steps

#### Step 1: Configure the Target URL
1. Open the workflow
2. Click **Fetch Target Page** node
3. Set the URL you want to monitor (e.g., a competitor's pricing page)

#### Step 2: Configure Alert Channels
You can use any combination of the three alert channels:
- **Telegram:** Set your bot credential and chat ID
- **Slack:** Set your Slack credential and channel
- **Email:** Set your SMTP credential, from address, and recipient

To disable a channel, simply disconnect its node from "Format Alert Messages."

#### Step 3: Understanding the Hash Comparison
- The workflow uses n8n's **static data** to store the previous page hash
- First run establishes the baseline (no alert sent)
- Subsequent runs compare against the stored hash
- HTML tags, scripts, and styles are stripped before comparison
- Only meaningful text content changes trigger alerts

#### Step 4: Customize Detection
Edit the **Compare with Previous** Code node to:
- Monitor specific CSS selectors instead of the full page
- Ignore certain dynamic elements (ads, timestamps)
- Adjust the content truncation length

---

## Workflow 5: Daily Business Report Generator

**File:** `workflows/daily-report-generator.json`

### Credentials Needed
1. **HTTP Header Auth** (x3) — For analytics, sales, and social APIs
2. **OpenAI API** — For report generation
3. **SMTP** — For email delivery
4. **Slack OAuth2** — For Slack delivery

### Setup Steps

#### Step 1: Connect Your Data Sources
This workflow is designed to work with any REST API. Update the three HTTP Request nodes:

**Analytics API** — Examples:
- Google Analytics Data API
- Plausible Analytics API
- Mixpanel Export API
- Custom analytics endpoint

**Sales API** — Examples:
- Stripe API (`/v1/charges?created[gte]=...`)
- Shopify Admin API
- Gumroad API
- Custom sales dashboard API

**Social Media API** — Examples:
- Twitter Analytics API
- Instagram Graph API
- LinkedIn Marketing API
- Buffer/Hootsuite API

#### Step 2: Adapt the Data Aggregation
Edit the **Aggregate Report Data** Code node to match your actual API response structures. The default handles common field names, but you may need to map different field names.

#### Step 3: Customize the Report
- Edit the AI system prompt to match your business context
- Add your company name, specific KPIs, or comparison targets
- Adjust the HTML email template colors/branding in the Format node

#### Step 4: Set Recipients
1. Update the email recipients in the **Email Report** node
2. Set the Slack channel in the **Slack Report** node
3. Adjust the schedule if 9 AM doesn't work for your timezone

#### Step 5: Test
1. Run manually first to check the report format
2. If APIs aren't connected yet, the workflow generates a report noting that data sources need configuration
3. Refine the AI prompt based on the output quality

---

## Troubleshooting

### Common Issues

**"Credential not found" error**
- Ensure all credentials are created in n8n before activating
- Check that credential names match what the workflow expects

**Workflow triggers but nothing happens**
- Check the execution log in n8n for error details
- Verify API endpoints are accessible from your n8n server
- Ensure API keys have the correct permissions

**AI responses are malformed**
- The Code nodes include fallback parsing — check the fallback output
- Ensure `responseFormat: json_object` is supported by your chosen model
- Try switching to `gpt-4o` if `gpt-4o-mini` gives poor results

**Telegram bot doesn't send messages**
- Verify you've started a chat with the bot first
- Double-check the chat ID (use the getUpdates API method)
- Ensure the bot token is correct

**Rate limiting**
- Add a Wait node between iterations for API-heavy workflows
- Reduce polling frequency in Schedule Trigger nodes
- Use n8n's built-in retry mechanism

---

## Getting Help

- n8n Documentation: [docs.n8n.io](https://docs.n8n.io)
- n8n Community Forum: [community.n8n.io](https://community.n8n.io)
- GitHub Issues: [github.com/lazymac2x/ai-automation-workflows/issues](https://github.com/lazymac2x/ai-automation-workflows/issues)
