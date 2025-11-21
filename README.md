# Kestra Email Automation Workflow

**Self-Hosted Email Processing with AI Summaries using Kestra, Docker, IMAP & Google Gemini**

A complete, production-ready email automation system that fetches emails from your inbox, categorizes them using keywords, generates AI summaries using Google Gemini 2.5 Flash, identifies urgent emails, detects spam, and optionally sends WhatsApp alerts.

---

## What This Project Does

This workflow automates email management by:

- **Fetching Emails** - Pulls emails from any IMAP inbox (Gmail, Outlook, Yahoo, Zoho, etc.)  
- **Categorizing** - Uses keyword-matching to classify emails into 14 categories  
- **AI Summarization** - Generates intelligent summaries using Google Gemini API  
- **Urgent Detection** - Identifies and extracts critical/urgent emails  
- **Spam Detection** - Uses AI to filter out spam automatically  
- **JSON Outputs** - Creates `summary.json` and `urgent.json` files  
- **CSV Reports** - Exports email data in CSV format  
- **WhatsApp Alerts** - Optionally sends WhatsApp notifications for urgent emails  
- **Manual & Event-Based** - Run manually or automatically when new emails arrive  

Everything runs locally on your computer via Docker—no cloud signup required.

---

## What You Need (Requirements)

### 1. Software & Tools
| Item | Link | Notes |
|------|------|-------|
| **Docker Desktop** | [Download](https://www.docker.com/products/docker-desktop) | Required to run Kestra locally |
| **Git** | [Download](https://git-scm.com/) | To clone the repository |
| **Text Editor** | VS Code, Notepad, etc. | To edit `.env` file |

### 2. Online Accounts & API Keys

#### A. Email Account (with IMAP enabled)
You need an email account that supports IMAP protocol:

| Email Provider | Setup Instructions |
|---|---|
| **Gmail** | 1. Go to [myaccount.google.com/security](https://myaccount.google.com/security)<br/>2. Enable "2-Step Verification"<br/>3. Create "App Password" for Mail<br/>4. Copy the 16-character password |
| **Outlook/Hotmail** | 1. Go to [account.microsoft.com/account/security](https://account.microsoft.com/account/security)<br/>2. Ensure IMAP is enabled<br/>3. Use your email password |
| **Yahoo Mail** | 1. Go to [account.yahoo.com/account/security](https://account.yahoo.com/account/security)<br/>2. Create "App Password"<br/>3. Copy the password |
| **Zoho Mail** | 1. Go to [mail.zoho.com/cpanel/home](https://mail.zoho.com/cpanel/home)<br/>2. Enable IMAP in Settings<br/>3. Create "App Password" |

#### B. Google Gemini API Key (for AI summaries)
1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Click "Create API Key"
3. Copy the key (looks like: `AIzaSyD...`)

#### C. (Optional) Twilio WhatsApp
Get Account SID and Auth Token from [Twilio Console](https://console.twilio.com/)

---

## Step-by-Step Setup Guide

Follow these steps exactly, in order. Don't skip any!

### Step 1: Clone the Repository

Open your terminal/PowerShell and run:

```bash
git clone https://github.com/Nsk364/Kestra-Assignment-by-DV.git
cd Kestra-Assignment-by-DV
```

This creates a folder and downloads the 3 project files:
- `docker-compose.yml` - Docker configuration
- `Flow.yaml` - The Kestra workflow definition
- `secret keys .env` - Environment variables template

### Step 2: Create and Configure the `.env` File

**Windows PowerShell:**
```powershell
Rename-Item "secret keys .env" ".env"
```

**macOS/Linux:**
```bash
mv "secret keys .env" .env
```

Open `.env` and fill in:

```env
EMAIL_HOST=imap.gmail.com              # Gmail, Outlook, Yahoo, Zoho
EMAIL_PORT=993
EMAIL_USER=your_email@gmail.com
EMAIL_PASSWORD=your_app_password
AI_API_KEY=your_gemini_api_key
TWILIO_ACCOUNT_SID=optional
TWILIO_AUTH_TOKEN=optional
TWILIO_FROM_WHATSAPP=whatsapp:+1234567890
TWILIO_TO_WHATSAPP=whatsapp:+919876543210
```

**Common Email Hosts:**

| Provider | HOST |
|----------|------|
| Gmail | `imap.gmail.com` |
| Outlook | `outlook.office365.com` |
| Yahoo Mail | `imap.mail.yahoo.com` |
| Zoho Mail | `imap.zoho.com` |

**SECURITY WARNING**: Never commit the `.env` file to GitHub. It contains sensitive credentials. The `.gitignore` should already protect it, but double-check!

### Step 3: Start Kestra Using Docker

In the same folder (project root), run:

**Windows PowerShell:**
```powershell
docker compose up
```

**macOS/Linux:**
```bash
docker-compose up
```

**What happens:**
- Docker downloads the Kestra image (~500 MB, first time only)
- Kestra starts running locally
- You'll see logs printed in the terminal

**Leave this terminal running!** Do not close it.

You should see output like:
```
kestra  | 2025-01-10 10:23:45.123 INFO  Server started in 2.5 seconds
kestra  | Server listening on [0.0.0.0:8080]
```

### Step 4: Open Kestra UI

Open your web browser and go to:

```
http://localhost:8080
```

You should see the Kestra interface with a "Flows" section.

If you get an error about "cloud.kestra.io", make sure Docker is running (Step 3).

---

## 📂 Import the Workflow

### Step 4.1: Copy the Workflow YAML

1. Open `Flow.yaml` from the project folder in a text editor
2. **Select all** (`Ctrl+A` or `Cmd+A`)
3. **Copy** (`Ctrl+C` or `Cmd+C`)

### Step 4.2: Create New Flow in Kestra

1. In Kestra UI (at http://localhost:8080), click **"Flows"** in the sidebar
2. Click **"Create Flow"** or **"Create New"**
3. Paste the YAML content into the editor
4. Click **"Save"**

### Step 4.3: Verify the Flow

You should see a flow with:
- **Namespace**: `nipun.email`
- **ID**: `email_flow`
- **Status**: Active

---

## Running the Workflow

### Option 1: Manual Trigger (Recommended for Testing)

1. Go to **Flows** → **nipun.email** → **email_flow**
2. Click the **"Execute"** or **"Run"** button
3. Fill in the input parameters:

| Parameter | Default | What It Does |
|-----------|---------|--------------|
| `mode` | `manual` | How the workflow runs. Keep as `manual` |
| `days` | `1` | Scan emails from last N days. Try `1`, `7`, or `30` |
| `summary_word_limit` | `20` | Max words in each AI summary |

4. Click **"Execute"** or **"Run Now"**
5. Wait for execution (takes 30 seconds - 2 minutes depending on email count)

### Option 2: Automatic Event-Based Trigger

The workflow has an **IMAP event trigger** configured in `Flow.yaml`:

```yaml
triggers:
  - id: gmail_new_email
    type: io.kestra.plugin.notifications.mail.MailReceivedTrigger
    protocol: IMAP
    host: "imap.gmail.com"
    port: 993
    username: # mail id
    password: # app password
    folder: "INBOX"
    ssl: true
    interval: "PT1M"
    disabled: false
```

This means:
- Automatically triggers when new emails arrive in INBOX
- Checks for new emails every 1 minute (PT1M)
- Useful for staying updated on incoming emails
- Note: Gmail credentials in this trigger may need to be filled manually

To disable automatic triggers, set `disabled: true` in the YAML and redeploy.

---

## Understanding the Outputs

After the workflow executes, find the outputs:

### Step 1: View Execution Results

1. Click on **"Executions"** in Kestra sidebar
2. Find the latest run (top of the list)
3. Click on it to open the execution details
4. Look for the **"Outputs"** tab

### Step 2: What You'll See

**summary.json** - Contains:
```json
{
  "mode": "manual",
  "emails_scanned": 15,
  "counts_by_category": {
    "urgent": 3,
    "promotions": 5,
    "personal": 4,
    "other": 3
  },
  "counts_by_sender": {
    "boss@company.com": 2,
    "newsletter@site.com": 1,
    ...
  },
  "emails": [
    {
      "subject": "Security Alert",
      "from": "google@accounts.google.com",
      "date": "Thu, 10 Jan 2025 10:20:00 +0000",
      "categories": ["urgent"],
      "is_spam_rule": false,
      "is_spam_ai": false,
      "summary": "Google detected suspicious activity. Verify your account."
    },
    ...
  ]
}
```

**urgent.json** - Contains:
```json
{
  "urgent_count": 3,
  "urgent_emails": [
    {
      "subject": "URGENT: Contract needs approval",
      "from": "manager@company.com",
      "summary": "Your contract awaits approval by EOD.",
      ...
    }
  ]
}
```

**email_report.csv** - Spreadsheet with all emails:
```
subject,from,date,categories,summary
"Urgent: Review Meeting","boss@company.com","2025-01-10",...
"Sale: 50% Off","promo@shop.com","2025-01-09",...
```

### Step 3: Download the JSON Files

To save the outputs:

1. In the Executions tab, look for tasks **"write_summary"** and **"write_urgent"**
2. Click on each task
3. Look for **"Download"** button next to the output

Or check Kestra's internal storage folder (usually in Docker volume `kestra-data`).

---

## WhatsApp Alerts Feature

One of the most powerful features is automatic WhatsApp notifications for urgent emails!

### How It Works

When the workflow detects **urgent emails**, it can send a formatted WhatsApp message to your phone:

```
🚨 New Urgent: Deadline complete the deadline (abhinavteja1290@gmail.com)

Other new urgents:
2. CS#331: The ingredient my own career was missing (team@cs.resumeworded.com)
3. Deadline Today (nipunsaikokonda@gmail.com)
...and 7 more!

Open your inbox to review.
```

**Features:**
- Shows the most recent urgent email with a notification
- Lists up to 3 other recent urgent emails
- Shows count if there are more than 3
- Includes sender email address
- Only sends on new urgent emails (no duplicate alerts)
- Triggers automatically when new emails arrive (checks every 1 minute)

### Setup WhatsApp Alerts (Optional)

WhatsApp alerts are **optional**. If you don't set them up, the workflow still works perfectly - just without WhatsApp notifications.

#### Step 1: Create a Twilio Account

1. Go to [https://www.twilio.com/](https://www.twilio.com/)
2. Click **"Sign Up"** → Fill in your details
3. Verify your email
4. Confirm your phone number
5. Dashboard opens automatically

#### Step 2: Enable WhatsApp Integration

1. In Twilio Console, go to **Messaging** → **Try it out** → **Send an SMS**
2. Look for **WhatsApp** section
3. Click **"Request WhatsApp Sandbox"**
4. Confirm your phone number
5. Send the test message to activate

#### Step 3: Get Your Credentials

In Twilio Console:

1. Go to **Account** → **API Keys & Tokens**
2. Copy your **Account SID** (starts with `AC...`)
3. Copy your **Auth Token** (long random string)
4. Go to **Messaging** → **Services** → Find WhatsApp service
5. Note your WhatsApp **From Number** (e.g., `whatsapp:+1234567890`)
6. Note your **To Number** (your personal WhatsApp number, e.g., `whatsapp:+919876543210`)

#### Step 4: Update `.env` File

Open your `.env` file and fill:

```env
# ===== TWILIO WHATSAPP CONFIGURATION =====
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_very_long_auth_token_here
TWILIO_FROM_WHATSAPP=whatsapp:+1234567890
TWILIO_TO_WHATSAPP=whatsapp:+919876543210
```

⚠️ **Important**: 
- Include the `whatsapp:` prefix before phone numbers
- Phone numbers must include country code (e.g., `+1` for USA, `+91` for India)
- These are credentials - keep them secret!

#### Step 5: Verify It Works

1. Run the workflow manually with some urgent emails
2. Check your WhatsApp for the alert message
3. If no alert, check the execution logs in Kestra

### Message Format Explained

```
🚨 New Urgent: [Subject] ([Sender Email])
   ↑ Emoji          ↑ Email subject    ↑ Who sent it

Other new urgents:
2. [Subject] ([Sender Email])    ← Second most recent
3. [Subject] ([Sender Email])    ← Third most recent
...and N more!                   ← If there are 4+ urgent emails

Open your inbox to review.       ← Call to action
```

### What Counts as "Urgent"?

An email is marked as urgent if it contains keywords like:
- `urgent`, `asap`, `immediately`, `deadline`, `reminder`, `critical`, `overdue`

And it's NOT spam (both rule-based and AI-detected spam are filtered out).

### Alert Deduplication

The workflow is smart about alerts:
- ✅ Won't send duplicate alerts for the same email
- ✅ Tracks which emails have been alerted (stored temporarily)
- ✅ Only alerts on NEW urgent emails

### Troubleshooting WhatsApp

#### ❌ Not receiving WhatsApp messages?

1. **Check Twilio Account is Active**
   - Go to [Twilio Console](https://console.twilio.com/)
   - Verify account status (not suspended)

2. **Verify Phone Numbers in `.env`**
   ```env
   # Should look like:
   TWILIO_FROM_WHATSAPP=whatsapp:+1234567890
   TWILIO_TO_WHATSAPP=whatsapp:+919876543210
   ```
   - Include `whatsapp:` prefix
   - Use correct country codes
   - No spaces or special characters

3. **Check Execution Logs**
   - Go to Kestra UI → Executions → Click run
   - Look for task `send_whatsapp_alert`
   - Check logs for errors

4. **Verify WhatsApp Sandbox**
   - Make sure you completed the Twilio WhatsApp sandbox setup
   - Send test message from Twilio console to confirm

#### ❌ Getting "Connection refused" error?

- Your Twilio credentials are incorrect
- Copy them exactly from Twilio console (no extra spaces)
- Re-verify Account SID and Auth Token

#### ❌ Receiving alerts but they're empty?

- No new urgent emails detected
- Check your email has words like "urgent", "deadline", "asap"
- Verify `days` parameter is set correctly

### Disabling WhatsApp Alerts

If you want to turn off WhatsApp:

**Option 1**: Delete the 4 Twilio variables from `.env`
```env
# Leave these empty:
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_FROM_WHATSAPP=
TWILIO_TO_WHATSAPP=
```

**Option 2**: Comment out the task in `Flow.yaml`
```yaml
# - id: send_whatsapp_alert
#   type: io.kestra.plugin.scripts.python.Script
#   ...
```

The workflow will skip WhatsApp and continue with other tasks.

---

## Email Categories (14 Types)

Your emails are automatically classified into these 14 categories:

| Category | Keywords Detected |
|----------|-------------------|
| **urgent** | urgent, asap, immediately, deadline, reminder, critical |
| **promotions** | offer, discount, sale, deal, coupon, cashback |
| **news** | news, update, headline, breaking, announcement, newsletter |
| **finance** | payment, invoice, bank, statement, transaction, salary |
| **jobs** | interview, opening, hiring, job, career, placement |
| **sports** | cricket, football, tennis, match, tournament, league |
| **events_meetings** | event, meeting, webinar, conference, workshop, invitation |
| **education_academics** | exam, assignment, project, result, class, lecture |
| **travel** | flight, train, bus, ticket, hotel, reservation, trip |
| **shopping_ecommerce** | order, shipment, delivery, package, amazon, flipkart |
| **bills_utilities** | bill, utility, electricity, water, phone, recharge |
| **health_medical** | appointment, doctor, hospital, medicine, lab, clinic |
| **personal** | family, friend, birthday, anniversary, wishes |
| **spam** | lottery, crypto, click here, win money, 100% free |
| **other** | Everything else that doesn't match above |

---

## Troubleshooting

### Problem: "Cannot connect to Docker daemon"

**Solution:**
- Docker Desktop is not running
- Open Docker Desktop application
- Wait 30 seconds for it to fully start
- Run `docker compose up` again

### Problem: Kestra redirects to cloud.kestra.io

**Solution:**
- The Docker container is not running
- Check if `docker compose up` terminal is still open
- If closed, run it again
- Wait for "Server started" message

### Problem: IMAP Login Failed

**Solution:**
1. **For Gmail**: Did you use App Password (16 chars), not regular password?
2. **Check EMAIL_HOST**: Make sure it's the correct IMAP server
3. **Check EMAIL_USER**: Verify the email address is correct (case-sensitive)
4. **Check EMAIL_PASSWORD**: Ensure no extra spaces at beginning/end
5. Test manually:
   ```powershell
   # Windows: Test IMAP connection
   $credential = New-Object System.Net.NetworkCredential("your_email@gmail.com", "your_app_password")
   ```

### Problem: "AI summaries are failing" or "API Key error"

**Solution:**
1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Verify the API key is active
3. Check in `.env` that `AI_API_KEY` matches exactly
4. Ensure there are no extra spaces

### Problem: WhatsApp alerts not working

**Solution:**
1. Twilio credentials are optional - the workflow will skip WhatsApp if not configured
2. If you want alerts, fill in all 4 Twilio variables in `.env`
3. Make sure Twilio account has WhatsApp enabled
4. Verify phone numbers include country code (e.g., `+1234567890`)

### Problem: "Port 8080 already in use"

**Solution:**
```powershell
# Find what's using port 8080
Get-NetTCPConnection -LocalPort 8080

# Kill the process (replace PID with the number from above)
Stop-Process -Id <PID> -Force

# Or change port in docker-compose.yml:
# "8081:8080"  (instead of 8080:8080)
```

### Problem: "ModuleNotFoundError: No module named requests"

**Solution:**
This should be installed automatically. If not:
```powershell
# Inside Kestra execution, the dependencies are:
docker compose exec kestra pip install requests
```

---

## File Structure

```
Kestra-Assignment-by-DV/
├── docker-compose.yml      # Docker configuration (don't edit)
├── Flow.yaml              # Kestra workflow definition
├── .env                   # Your credentials (KEEP SECRET!)
└── README.md              # This file
```

---

## Security Best Practices

1. **Never commit `.env` to GitHub**
   - The file contains passwords and API keys
   - It should be in `.gitignore`

2. **Use App Passwords, not regular passwords**
   - Gmail, Yahoo, and others require special "App Passwords"
   - Regular passwords won't work with IMAP

3. **Rotate API keys regularly**
   - Delete old Google Gemini API keys
   - Create new ones every 3-6 months

4. **Don't share execution logs**
   - They may contain email headers with sensitive info

---

## 📊 Workflow Architecture

```
┌─────────────────────────────────────┐
│      Docker Desktop on Computer     │
│  (Running Kestra Container)         │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   Kestra Web UI (localhost:8080)    │
│  - View Flows                       │
│  - Execute Workflows                │
│  - View Execution Logs              │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   Kestra Email Processing Workflow  │
│  1. Fetch Emails (IMAP)             │
│  2. Categorize (Keywords)           │
│  3. Summarize (Gemini API)          │
│  4. Detect Spam (AI)                │
│  5. Extract Urgent                  │
│  6. Generate CSV Report             │
│  7. Send WhatsApp (Optional)        │
└──────────────┬──────────────────────┘
               │
        ┌──────┴──────┬──────────┐
        ▼             ▼          ▼
   summary.json  urgent.json  email_report.csv
```

---

## Example: Running a Complete Workflow

### Scenario: You want to scan Gmail for urgent emails

**Step 1:** Update `.env`
```env
EMAIL_HOST=imap.gmail.com
EMAIL_USER=yourname@gmail.com
EMAIL_PASSWORD=xyzz abcd efgh ijkl    # App Password (16 chars)
AI_API_KEY=AIzaSyD...                 # Gemini API Key
```

**Step 2:** Start Docker
```powershell
docker compose up
```

**Step 3:** Open http://localhost:8080 → Click your flow → Click Execute

**Step 4:** Set parameters:
```
mode: manual
days: 7
summary_word_limit: 50
```

**Step 5:** Click Execute

**Step 6:** Wait 1-2 minutes

**Step 7:** Go to Executions → Click latest run → View Outputs

**Step 8:** Download `urgent.json` to see only critical emails

---

## Tips and Tricks

### Tip 1: Test with a small number of emails first
- Set `days: 1` to scan only today's emails
- This is faster and helps you test

### Tip 2: Increase summary word limit for detailed summaries
- Default is 20 words (brief)
- Try 50-100 for more detail
- More words = slower execution

### Tip 3: Monitor execution logs
- Click on a running execution
- Open the **Logs** tab
- Watch real-time progress

### Tip 4: Save outputs regularly
- Download JSON files after important runs
- Useful for reports

### Tip 5: Control automatic triggers
- Event-based trigger runs when new emails arrive (checks every 1 minute)
- To disable: Set `disabled: true` in the `triggers` section of `Flow.yaml`
- For manual runs only: Disable the trigger and use the Execute button

---

## Getting Help

If something doesn't work:

1. **Check the Execution Logs**
   - Kestra UI → Executions → Click run → Logs tab
   - Errors are shown here

2. **Verify your `.env` file**
   - Check for typos or missing values
   - Ensure no extra spaces

3. **Check Docker is running**
   - Windows: Look for Docker icon in system tray
   - macOS: Look for whale icon in menu bar

4. **Check Kestra Documentation**
   - Visit [docs.kestra.io](https://docs.kestra.io)

---

## Next Steps & Enhancements

Once you have the basic setup working, consider:

- Configure WhatsApp alerts for urgent emails
- Add Slack notifications
- Store results in a database (PostgreSQL)
- Create daily email summaries
- Build a dashboard to visualize email trends
- Add sentiment analysis to summaries
- Integrate with calendar apps

---

## License

This project is for the DiligenceVault Technical Assignment.

---

## Author

**Nipun Sai Kokonda**

Email Automation Workflow using Kestra, IMAP, and Google Gemini AI

---

## Completion Checklist

Use this checklist to ensure you haven't missed anything:

- [ ] Downloaded and installed Docker Desktop
- [ ] Installed Git
- [ ] Cloned the repository
- [ ] Created `.env` file from `secret keys .env`
- [ ] Filled in EMAIL_HOST, EMAIL_USER, EMAIL_PASSWORD
- [ ] Filled in AI_API_KEY (Google Gemini)
- [ ] (Optional) Filled in Twilio WhatsApp credentials
- [ ] Ran `docker compose up` and confirmed Docker is running
- [ ] Opened http://localhost:8080 and saw Kestra UI
- [ ] Imported Flow.yaml into Kestra
- [ ] Executed the workflow manually
- [ ] Downloaded and reviewed output JSON files
- [ ] Read the troubleshooting section

**Once all checkboxes are ticked, your system is fully operational!**

---

## Learning Resources

- **Kestra Documentation**: https://docs.kestra.io
- **Google Gemini API**: https://ai.google.dev/
- **Docker Documentation**: https://docs.docker.com/
- **Python Email Handling**: https://docs.python.org/3/library/email.html

---

**Last Updated**: November 2025  
**Status**: Production Ready
