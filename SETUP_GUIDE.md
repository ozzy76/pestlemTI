# Threat Intelligence Analysis Workflow - Setup Guide

## Overview

This N8N workflow automates threat intelligence analysis by:
1. Receiving submissions via Google Form webhook
2. Querying multiple RSS feeds for threat intelligence news
3. Verifying claims using Blackbird.ai Context API
4. Applying Words of Estimative Probability (WEP) per Intelligence Community standards
5. Generating and emailing executive reports

## Prerequisites

- N8N instance (self-hosted or cloud)
- Blackbird.ai API credentials
- SMTP server for email delivery
- Google Forms account

## Installation Steps

### 1. Import the Workflow

1. Log into your N8N instance
2. Click "Add Workflow" → "Import from File"
3. Select `threat-intelligence-workflow.json`
4. The workflow will be imported with all nodes and connections

### 2. Configure Blackbird.ai API Credentials

1. In N8N, go to "Credentials" → "New"
2. Select "Header Auth" or create a custom credential type
3. Add the following:
   - **Name**: `Blackbird.ai API`
   - **Authentication Method**: Bearer Token
   - **API Key**: Your Blackbird.ai API key

To obtain Blackbird.ai credentials:
- Visit https://blackbird.ai/
- Sign up for an account
- Navigate to API settings to generate your API key

### 3. Configure Email Credentials

1. Go to "Credentials" → "New"
2. Select "SMTP"
3. Configure your SMTP server:
   - **Host**: Your SMTP server address
   - **Port**: 587 (or 465 for SSL)
   - **Username**: Your email username
   - **Password**: Your email password
   - **Secure**: Enable SSL/TLS

4. Update the "Send Email Report" node:
   - Change `fromEmail` from `noreply@threatintel.local` to your actual sender email

### 4. Activate the Workflow

1. Save the workflow
2. Click "Activate" to enable the webhook trigger
3. Copy the webhook URL from the "Google Form Submission" node
   - It will look like: `https://your-n8n-instance.com/webhook/threat-intelligence-trigger`

### 5. Set Up Google Form

Create a Google Form with the following fields:

#### Required Fields:

1. **Analyst Name** (Short answer)
   - Field name: `analystName`
   - Description: "Your name"

2. **Analysis Scope** (Short answer)
   - Field name: `analysisScope`
   - Description: "What area of threat intelligence to analyze (e.g., Ransomware, Nation-State Actors, APT Groups)"

3. **Keywords** (Paragraph)
   - Field name: `keywords`
   - Description: "Comma-separated keywords to filter intelligence feeds (e.g., ransomware, APT29, zero-day)"

4. **Recipient Email** (Email)
   - Field name: `recipientEmail`
   - Description: "Email address to receive the report"

#### Configure Form Webhook:

Google Forms doesn't natively support webhooks. Use one of these methods:

**Option A: Using Google Apps Script**
1. In your form, click the three dots → "Script editor"
2. Add this script:

```javascript
function onFormSubmit(e) {
  var formResponse = e.response;
  var itemResponses = formResponse.getItemResponses();

  var payload = {};
  for (var i = 0; i < itemResponses.length; i++) {
    var itemResponse = itemResponses[i];
    var title = itemResponse.getItem().getTitle();
    var response = itemResponse.getResponse();

    // Map form fields to expected names
    if (title.includes('Analyst Name')) payload.analystName = response;
    if (title.includes('Analysis Scope')) payload.analysisScope = response;
    if (title.includes('Keywords')) payload.keywords = response;
    if (title.includes('Recipient Email')) payload.recipientEmail = response;
  }

  var options = {
    'method': 'post',
    'contentType': 'application/json',
    'payload': JSON.stringify(payload)
  };

  UrlFetchApp.fetch('YOUR_N8N_WEBHOOK_URL', options);
}
```

3. Replace `YOUR_N8N_WEBHOOK_URL` with your actual webhook URL
4. Click "Triggers" → "Add Trigger"
5. Select: `onFormSubmit`, `From form`, `On form submit`
6. Save and authorize the script

**Option B: Using Zapier/Make.com**
1. Create a Zap/Scenario that triggers on Google Form submission
2. Configure it to send a POST request to your N8N webhook with the form data

**Option C: Using Google Sheets + N8N Google Sheets Trigger**
1. Link your Google Form to a Google Sheet
2. Replace the webhook trigger in the workflow with "Google Sheets Trigger"
3. Configure it to trigger on new rows

## Workflow Configuration

### Customizing RSS Feeds

Edit the "Prepare RSS Feed URLs" node to add/remove RSS feeds:

```javascript
const rssFeeds = [
  { name: 'Your Feed Name', url: 'https://example.com/feed' },
  // Add more feeds here
];
```

The workflow includes 10 threat intelligence feeds by default from your OPML file.

### Adjusting Analysis Timeframe

The workflow analyzes items from the last 7 days. To change this, edit the "Filter RSS Items" node:

```javascript
// Change this line:
sevenDaysAgo.setDate(sevenDaysAgo.getDate() - 7);
// To:
sevenDaysAgo.setDate(sevenDaysAgo.getDate() - 14); // For 14 days
```

### Customizing Words of Estimative Probability

The WEP scale in the "Assign Words of Estimative Probability" node follows Intelligence Community Directive 203 standards:

| Term | Probability Range | Confidence Level |
|------|------------------|------------------|
| Almost Certain | 95-100% | Very High |
| Highly Likely | 80-95% | High |
| Likely | 55-80% | Moderate to High |
| Roughly Even Chance | 45-55% | Moderate |
| Unlikely | 20-45% | Low to Moderate |
| Highly Unlikely | 5-20% | Low |
| Remote | 0-5% | Very Low |

You can adjust these ranges in the `WEP_SCALE` object if needed.

## Testing the Workflow

### Manual Test

1. In N8N, click "Execute Workflow" on the webhook node
2. Provide test data:

```json
{
  "analystName": "Test Analyst",
  "analysisScope": "Ransomware Threats",
  "keywords": "ransomware,malware,cybercrime",
  "recipientEmail": "your-email@example.com"
}
```

3. Monitor each node's execution
4. Check your email for the report

### Live Test

1. Submit your Google Form
2. Monitor the workflow execution in N8N
3. Verify the email report is received

## Troubleshooting

### Webhook Not Triggering
- Verify the webhook is activated
- Check the webhook URL is correct in Google Apps Script
- Review N8N logs for incoming requests

### Blackbird.ai API Errors
- Verify your API credentials are correct
- Check you haven't exceeded API rate limits
- Ensure the claim text is within API limits (usually 500-1000 characters)

### No Email Received
- Check SMTP credentials are correct
- Verify the recipient email is valid
- Check spam folder
- Review N8N execution logs for email node errors

### RSS Feeds Not Loading
- Verify RSS feed URLs are accessible
- Some feeds may require authentication
- Check for rate limiting on RSS sources

## Security Considerations

1. **API Keys**: Store all credentials securely in N8N's credential manager
2. **Webhook Security**: Consider adding authentication to your webhook endpoint
3. **Email Security**: Use TLS/SSL for SMTP connections
4. **Data Handling**: This workflow processes potentially sensitive threat intelligence
5. **Access Control**: Restrict access to N8N instance and Google Form

## Monitoring and Maintenance

### Regular Checks
- Monitor Blackbird.ai API usage and costs
- Review RSS feed availability
- Check email delivery success rates
- Update RSS feed list as sources change

### Performance Optimization
- Adjust the "Wait for Analysis" duration based on Blackbird.ai response times
- Implement error handling for failed API calls
- Consider rate limiting for high-volume usage

## Support and Resources

- N8N Documentation: https://docs.n8n.io
- Blackbird.ai Documentation: https://docs.blackbird.ai/context
- Intelligence Community Directive 203: https://www.dni.gov/files/documents/ICD/ICD%20203%20Analytic%20Standards.pdf
- Words of Estimative Probability: https://www.cia.gov/resources/csi/studies-in-intelligence/

## Version History

- v1.0 (2026-01-11): Initial release
  - Google Form webhook integration
  - 10 RSS feed sources
  - Blackbird.ai Context verification
  - WEP assessment per ICD 203
  - HTML email report generation
