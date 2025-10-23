# Customer Support AI Workflow - n8n

## What the Workflow Does

This n8n workflow provides an automated customer support system that processes incoming questions through a webhook, generates AI-powered responses using Groq's LLM (llama-3.3-70b-versatile), and maintains a complete audit trail. When a customer submits a question via HTTP POST request, the workflow validates the input, generates a professional answer with confidence scoring, logs all interactions to Google Sheets with unique request IDs, and notifies the support team via Gmail. The workflow includes production-grade error handling with automatic retries for transient failures, fallback responses when the AI is unavailable, and alert mechanisms that notify the team if critical components (like Google Sheets logging) fail. Each request is tracked with a unique identifier, status field, and timestamp for complete traceability.

## How to Trigger It

**Test Mode (Development):**
```bash
curl -X POST https://YOUR-N8N-INSTANCE.app.n8n.cloud/webhook-test/customer-question \
  -H "Content-Type: application/json" \
  -d '{
    "question": "How do I reset my password?",
    "customer_name": "John Doe",
    "customer_email": "john@example.com"
  }'
```

**Production Mode:**
Activate the workflow in n8n, then use the production webhook URL (without `-test`). The workflow accepts POST requests with a JSON body containing `question`, `customer_name`/`name`, and `customer_email`/`email` fields. It returns a JSON response with the AI-generated answer, confidence level, follow-up flag, unique request ID, and timestamp. All successful requests are logged to Google Sheets and trigger email notifications to the configured team email address.

## Assumptions and Improvements

**Assumptions:**
- Google Sheets document already exists with appropriate column headers (Timestamp, Request ID, Customer Name, Customer Email, Question, Answer, Confidence, Requires Followup, Status)
- Gmail OAuth and Google Sheets OAuth credentials are properly configured in n8n
- Groq API key is valid and has sufficient quota
- Questions are primarily in English (though the LLM supports multiple languages)
- The team email address is monitored regularly for notifications

**Potential Improvements:**
1. **Advanced validation**: Add spam detection, profanity filters, and rate limiting by IP address or email to prevent abuse
2. **Knowledge base integration**: Connect to a vector database (Pinecone, Weaviate) to retrieve context from documentation before generating answers
3. **Multi-channel support**: Extend webhook to handle Slack, Discord, or live chat integrations beyond just HTTP POST
4. **Analytics dashboard**: Add nodes to track metrics like average response time, confidence scores, question categories, and follow-up rates
5. **Customer response loop**: Implement email sending directly to customers with their answer, and add feedback collection (thumbs up/down)
6. **Escalation workflow**: Automatically create support tickets in systems like Jira or Zendesk when confidence is low or follow-up is required
7. **A/B testing**: Test multiple LLM models or prompt variations and track which produces better answers
8. **Caching layer**: Store common questions and answers in Redis to reduce API calls and improve response time
9. **Internationalization**: Add automatic language detection and route to language-specific models or translate responses
10. **Security enhancements**: Implement webhook authentication with API keys or JWT tokens, and add PII detection/redaction for sensitive data

**Deployment Recommendations:**
- Set up monitoring alerts for workflow execution failures
- Configure backup notification channels (e.g., Slack webhook) if Gmail fails
- Implement daily/weekly summary reports of customer questions and AI performance
- Create a separate Google Sheet for error logs to track system health
- Use n8n's execution history and workflow versioning for rollback capability