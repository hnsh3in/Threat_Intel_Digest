# Setup Guide

Step-by-step deployment and configuration for the Threat Intel Digest pipeline.

---

## 1. Infrastructure

### AWS Lightsail (recommended)

- Instance: Ubuntu 24.04 LTS, AWS region of your choice
- Plan: $7/month - 1GB RAM, 2 vCPUs, 40GB SSD
- Add a static IP and attach it to the instance
- Add 2GB swap to prevent OOM during large RSS fetch runs:

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Firewall

Keep port 5678 off the public firewall. n8n should bind to `127.0.0.1` only. Access it via SSH tunnel:

```bash
ssh -i "your-key.pem" -L 5678:localhost:5678 ubuntu@YOUR_STATIC_IP
```

Then open `http://localhost:5678` in your browser.

---

## 2. Docker

```bash
docker run -d --name n8n --restart unless-stopped \
  -p 127.0.0.1:5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e GENERIC_TIMEZONE="UTC" \
  -e TZ="UTC" \
  -e N8N_SECURE_COOKIE=false \
  n8nio/n8n
```

**Log rotation** - add to `/etc/docker/daemon.json` before starting the container:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

---

## 3. Credentials

Create these four credentials in n8n before importing the workflow.

### Claude API Key

- Type: **Header Auth**
- Name: `Claude API Key` (must match this exactly - the workflow references it by name)
- Header Name: `x-api-key`
- Header Value: your Anthropic API key from [console.anthropic.com](https://console.anthropic.com)

> **Set a monthly spend limit.**
> In the Anthropic Console go to **Settings > Limits** and set a monthly credit limit before activating the workflow. If your API key is ever compromised, an uncapped key can drain your entire credit balance. A limit of $10–$20/month is well above the normal running cost (~$5/month) while capping worst-case exposure.

### Gmail OAuth2

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a project, enable the Gmail API
3. OAuth consent screen: External, add your email as a test user
4. **Publish the app to production** - on the OAuth consent screen page click **Publish App** and confirm. Apps left in test mode issue refresh tokens that expire after 7 days, which will break the workflow silently. Publishing removes that limit. No Google review is required for personal use since you are only authorising your own account.
5. Create OAuth 2.0 credentials (Web application)
6. Add redirect URI: `http://localhost:5678/rest/oauth2-credential/callback`
7. In n8n: create a **Gmail OAuth2 API** credential, paste Client ID and Secret, then click Connect

### Google Sheets OAuth2

Same Google Cloud project. Enable the Google Sheets API. Create a second OAuth2 credential in n8n for Sheets using the same client credentials.

### Discord Webhook

In your Discord server: Server Settings > Integrations > Webhooks > New Webhook. Copy the webhook URL - you will paste it into the workflow after import (see step 5 below).

---

## 4. Import the Workflow

1. Open n8n at `http://localhost:5678`
2. Go to **Workflows > Import from File**
3. Select `workflow/Threat_Intel_Digest_Published_v1.0.json`
4. The workflow imports as inactive

---

## 5. Post-Import Configuration

These values are sanitised in the export and must be updated manually after import.

### Nodes to update

| Node | Field | Value |
|---|---|---|
| `Format Discord (CJS)` | `[YOUR_LOCALE]` | e.g. `en-US` |
| `Format Discord (CJS)` | `[YOUR_TIMEZONE]` | e.g. `UTC` |
| `Format Gmail (CJS)` | `[YOUR_LOCALE]` | e.g. `en-US` |
| `Format Gmail (CJS)` | `[YOUR_TIMEZONE]` | e.g. `UTC` |
| `Format Error Notification (CJS)` | `[YOUR_LOCALE]` | e.g. `en-US` |
| `Format Error Notification (CJS)` | `[YOUR_TIMEZONE]` | e.g. `UTC` |
| `Build Claude Summary Payload (CJS)` | `[INSERT YOUR LOCALE]` | e.g. `en-US` |
| `Build Claude Summary Payload (CJS)` | `[INSERT YOUR TIMEZONE]` | e.g. `UTC` |
| `Build Claude Summary Payload (CJS)` | `[INSERT YOUR TIMEZONE ABBREVIATION]` | e.g. `UTC` |
| `Discord Webhook (HTTP)` | Webhook URL | your Discord webhook URL |
| `Discord Error Webhook (HTTP)` | Webhook URL | your Discord webhook URL |
| `Send a message` | `sendTo` | your Gmail address |
| `Gmail Error Notify (Gmail)` | `sendTo` | your Gmail address |
| `Append row in sheet` | Spreadsheet URL | your Google Sheets URL |

### Triage prompt customisation

Open `Build Triage Payload (CJS)`. The system prompt contains placeholder brackets for:

- `[INSERT YOUR COMPANY INFO]` - brief description of your organisation and sector
- `[INSERT YOUR ANALYST ENVIRONMENT]` - OS, AD, cloud platform, EDR/XDR, SIEM
- `[INSERT YOUR MONITORED REGIONS]` - e.g. APAC, EU, US, CN
- `[INSERT YOUR COMPANY NAME]` and `[INSERT YOUR PRODUCT/PLATFORM NAME]` - auto-elevates to score 10
- `[INSERT YOUR MONITORED VENDOR STACK]` - e.g. Fortinet, Ivanti, Palo Alto, VMware
- `[INSERT YOUR RELEVANT SECTORS]` - e.g. energy, utilities, ports, manufacturing, banking
- `[INSERT YOUR SIEM/XDR/SOAR PLATFORMS]` - e.g. Microsoft Sentinel, Defender XDR
- `[INSERT YOUR CLOUD/PRODUCTIVITY STACK]` - e.g. M365, Azure, Entra ID
- `[INSERT YOUR CLOUD PLATFORM]` - e.g. Azure, AWS, GoogleCloud

The more precisely these match your actual stack and region, the more accurate the triage scores will be.

---

## 6. Google Sheets Audit Log

Create a sheet named `Triage Audit` with these column headers in row 1:

```
runTimestamp | feedName | title | verdict | triageScore | triageReason | triageCategory | link | pubDate
```

The `Append row in sheet` node maps to these column names exactly.

---

## 7. Activate

Once all credentials and placeholders are configured:

1. Open the workflow
2. Toggle **Active** in the top-right corner
3. The workflow will run at the times configured in the Schedule Trigger node (default: 00:00 and 12:00 UTC)

To test immediately without waiting for the schedule: click the **Execute Workflow** button (play icon, top right).

---

## Known Issues and Workarounds

| Issue | Cause | Workaround |
|---|---|---|
| IF node boolean condition fails | n8n v2.14.2 bug | Use String `is not empty` check - already applied in this workflow |
| HTTP Request rejects expressions in JSON body | n8n re-parses JSON body mode | Raw mode with pre-stringified payload from Code node - already applied |
| Discord messages out of order or 429 errors | Simultaneous webhook calls | SplitInBatches (batch: 1) + Wait 2s - already configured |
| All items blocked on repeated test runs | Dedup SQLite has seen all links | Temporarily bypass the `Deduplicate by Link` node during testing |
| n8n instability during large RSS runs | CPU burst exhaustion on 1GB RAM | 2s Wait between feeds + 2GB swap |
| Some feeds return zero items silently | n8n RSS node swallows errors | `Unpack Feed Fallback (CJS)` surfaces these before the IF check |
| Bot-blocked feeds (BankInfoSecurity, Fortiguard) | Default User-Agent rejected | HTTP fallback with browser User-Agent - already configured |
| Claude Triage returns JSON wrapped in markdown fences | Haiku occasional format non-compliance | `Decode Claude Response (CJS)` strips fences before `JSON.parse` |

---

## Adjusting the Triage Threshold

The threshold is set in `Parse + Filter (CJS)`:

```javascript
const THRESHOLD = 4;
```

- Raise to `5` to reduce noise if too many marginal items are passing
- Lower to `3` to widen coverage if relevant items are being missed
- Review the `verdict` column in the audit log after 2 weeks to calibrate

---

## Credential IDs

After importing the workflow and creating credentials, n8n assigns new internal IDs automatically. If the workflow shows credential errors on first import, open each affected node, re-select the correct credential from the dropdown, and save. This is expected behaviour on first import.
