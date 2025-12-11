# Jira Bug Triage Workflow

## Overview
This n8n workflow automates bug ticket triage by:
1. Detecting new bug tickets via Jira webhook
2. Comparing against recent tickets to identify duplicates
3. Performing root cause analysis using LLM
4. Categorizing priority based on impact
5. Checking webAI documentation to find easy fixes
6. Posting formatted summaries to Slack
7. Auto-labeling high-confidence duplicates
8. Determing priority for easy sorting 

## Workflow Logic

### Node Flow
```
Jira Trigger
    ↓
Get Recent Bug Tickets (last 90 days, open bugs)
    ↓
Prepare Ticket Data (format for LLM)
    ↓
Check for Duplicates (LLM analysis)
    ↓
Analyze & Categorize Ticket (LLM root cause + priority)
    ↓
Format Slack Message (build rich message blocks)
    ↓
    ├→ Send to Slack
    └→ Is High-Confidence Duplicate?
           ├→ YES → Add Duplicate Label + Jira Comment
           └→ NO → End
``` 

### LLM Prompts

**Duplicate Detection**
- Compares summary, description, components
- Returns JSON with confidence level
- Identifies similar tickets by key

**Root Cause Analysis**
- Categorizes by system area (Frontend, Backend, etc.)
- Provides hypothesis with reasoning
- Confidence assessment

**Priority Categorization**
- Evaluates: Critical/High/Medium/Low
- Considers user impact, frequency, workarounds
- Provides reasoning for level