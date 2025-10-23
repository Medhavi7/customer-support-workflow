# Setup Guide - Customer Support AI Workflow

## Prerequisites

1. **n8n Instance** - Cloud or self-hosted (v1.0.0+)
2. **Groq API Key** - Sign up at https://console.groq.com
3. **Google Account** - For Sheets and Gmail OAuth
4. **Google Sheet** - Pre-created with proper headers

## Step 1: Create Google Sheet

Create a new Google Sheet with these column headers in Row 1:

```
Timestamp | Request ID | Customer Name | Customer Email | Question | Answer | Confidence | Requires Followup | Status
```

Copy the Sheet ID from the URL:
```
https://docs.google.com/spreadsheets/d/[YOUR_SHEET_ID_HERE]/edit
```

## Step 2: Configure n8n Credentials

### A. Groq API
1. In n8n, go to **Credentials** → **Add Credential**
2. Search for "Groq"
3. Add your API key from https://console.groq.com/keys
4. Save as "Groq API"

### B. Google Sheets OAuth2
1. Go to **Credentials** → **Add Credential**
2. Search for "Google Sheets OAuth2 API"
3. Click **Connect my account**
4. Authorize with your Google account
5. Save as "Google Sheets OAuth2"

### C. Gmail OAuth2
1. Go to **Credentials** → **Add Credential**
2. Search for "Gmail OAuth2"
3. Click **Connect my account**
4. Authorize with your Google account (must have Gmail)
5. Save as "Gmail OAuth2"

## Step 3: Import Workflow

1. In n8n, click **Workflows** → **Add Workflow**
2. Click the **⋮** menu → **Import from File**
3. Paste the contents of `customer-support-workflow.json`
4. Click **Import**

## Step 4: Update Configuration

1. Click on the **"Workflow Configuration"** node
2. Update these values:
   - `sheetId`: Your Google Sheet ID from Step 1
   - `teamEmail`: Your support team email address

## Step 5: Reconnect Credentials

For each of these nodes, click on them and reconnect your credentials:

1. **Groq Chat Model** → Select "Groq API"
2. **Log to Google Sheets** → Select "Google Sheets OAuth2"
3. **Notify Team via Gmail** → Select "Gmail OAuth2"
4. **Alert - Sheets Failed** → Select "Gmail OAuth2"

## Step 6: Test the Workflow

1. Click **Execute Workflow** at the bottom
2. You should see "Waiting for webhook call"
3. Copy the test webhook URL (it will show in the webhook node)

4. Test with curl:
```bash
curl -X POST https://YOUR-N8N-URL/webhook-test/customer-question \
  -H "Content-Type: application/json" \
  -d '{
    "question": "How do I reset my password?",
    "customer_name": "Test User",
    "customer_email": "test@example.com"
  }'
```

5. You should receive:
   - ✅ JSON response in terminal
   - ✅ New row in Google Sheet
   - ✅ Email notification

## Step 7: Activate for Production

1. Toggle the workflow to **Active** (top right)
2. Use the production webhook URL (without `-test`)
3. Monitor executions in the **Executions** tab

## Troubleshooting

### "Invalid input" error
- Ensure the `question` field is not empty
- Check that email format is valid

### No email received
- Check spam folder
- Verify Gmail OAuth is properly connected
- Confirm team email address is correct

### Google Sheets not updating
- Verify Sheet ID is correct
- Ensure Sheet has the correct column headers
- Check Google Sheets OAuth permissions

### AI generation fails
- Verify Groq API key is valid
- Check Groq API quota at https://console.groq.com
- The workflow will still work with fallback response

## API Endpoints

### Request Format
```json
{
  "question": "Your customer question here",
  "customer_name": "John Doe",  // or "name"
  "customer_email": "john@example.com"  // or "email"
}
```

### Success Response
```json
{
  "success": true,
  "requestId": "12345-1698765432000",
  "answer": "To reset your password...",
  "confidence": "high",
  "requiresFollowup": false,
  "timestamp": "2025-10-22T22:53:21.961-04:00",
  "status": "success"
}
```

### Error Response
```json
{
  "success": false,
  "error": "Invalid input",
  "message": "Please provide a valid question and email address"
}
```

## Monitoring

- **Executions Tab**: View all workflow runs
- **Google Sheets**: Complete audit trail
- **Gmail**: Real-time notifications
- **Status Field**: Track success vs ai_error states

## Support

For issues or questions:
1. Check the n8n execution logs for error details
2. Verify all credentials are properly connected
3. Ensure external services (Groq, Google) are operational
4. Review the README.md for assumptions and limitations