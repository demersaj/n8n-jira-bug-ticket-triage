# Jira Bug Ticket Triage Workflow

## Overview
This n8n workflow automatically triages new bug tickets in Jira. When a bug is created, it analyzes the ticket, checks for duplicates, determines priority, searches for relevant documentation, and posts a summary to Slack.

## What It Does

1. **Triggers on new bug tickets** - Only runs for bug tickets (not other issue types) and ignores tickets from the IT project
2. **Prevents duplicate processing** - Checks if a ticket has already been triaged to avoid running multiple times
3. **Fetches recent tickets** - Gets the last 90 days of open bugs for duplicate detection
4. **Analyzes log files** - If log files are attached, downloads and analyzes them for error messages
5. **Checks for duplicates** - Uses LLM to compare against recent tickets and identify potential duplicates
6. **Root cause analysis** - LLM analyzes the bug and provides root cause hypothesis, category, and technical area
7. **Priority recommendation** - LLM determines priority (Lowest, Low, Medium, High, Urgent) based on impact and urgency
8. **Documentation search** - Searches for relevant help documentation that might resolve the issue
9. **Updates Jira** - Sets priority, adds "Duplicate" label if needed, and posts a triage summary comment
10. **Notifies Slack** - Sends a formatted message with all the analysis details

## Workflow Flow

```
Jira Trigger (new issue created)
    ↓
Check if Bug Ticket (must be Bug type, not IT project)
    ↓
Check if Already Triaged (stops if auto-triaged label exists)
    ↓
Mark as Processing (adds auto-triaged label to prevent race conditions)
    ↓
Get Recent Bug Tickets (last 90 days, open bugs only)
    ↓
Extract New Ticket Data (standardizes ticket fields)
    ↓
Prepare Data for Analysis (combines ticket + existing tickets, identifies log files)
    ↓
Download Log File (if log files attached)
    ↓
Prepare Log File Info (combines log content with ticket data)
    ↓
    ├→ Check for Duplicates (LLM)
    ├→ Analyze & Categorize Ticket (LLM)
    └→ Search Help Docs (LLM)
    ↓
Merge Analysis Results (combines all LLM outputs)
    ↓
    ├→ Map Priority to Jira Format (converts to Jira priority IDs: 1-5)
    │   └→ Update Priority (sets priority in Jira)
    ├→ Format Slack Message
    │   └→ Send to Slack
    ├→ Add Jira Comment (posts triage summary)
    │   └→ Mark as Triaged
    └→ Is High-Confidence Duplicate?
           └→ Add Duplicate Label
```

## Key Features

### Duplicate Detection
- Compares new ticket against recent tickets from the last 90 days
- Returns confidence level (high/medium/low) and similarity score
- Lists similar ticket keys with clickable links in both Jira comments and Slack messages
- Automatically adds "Duplicate" label for high-confidence duplicates

### Priority Management
- LLM recommends priority based on user impact, urgency, and severity
- Maps LLM priorities to Jira priority IDs:
  - 1 = Lowest
  - 2 = Low
  - 3 = Medium
  - 4 = High
  - 5 = Urgent
- Automatically updates the priority field in Jira

### Log File Analysis
- Automatically detects log files attached to tickets (`.log`, `.txt`, `.out`, `.err`, etc.)
- Downloads and analyzes log content for error messages and stack traces
- Only analyzes log files if they're actually provided (won't hallucinate errors)
- Limits log content to 30,000 characters to avoid token limits

### Documentation Linking
- Searches for relevant help documentation from support.webai.com
- Only links documentation that actually exists (prevents hallucination)
- Includes documentation in both Jira comments and Slack messages
- Reduces priority to "Low" if it's determined to be a user education issue

### Jira Comment Format
The workflow posts a comprehensive triage summary comment that includes:
- Duplicate detection results (if applicable) with links to similar tickets
- Relevant documentation links (if found)
- Root cause analysis (category, hypothesis, confidence, reasoning)
- Priority recommendation with reasoning
- Recommended actions
- Estimated complexity

### Slack Notification
Sends a rich formatted message with:
- Ticket summary and description
- Priority recommendation with emoji indicators
- Root cause analysis
- AI-generated summary
- Help documentation (if found)
- Duplicate warnings (if applicable)
- Recommended actions
- Clickable "View in Jira" button

## Configuration

### Jira Projects
The workflow only processes tickets from these projects:
- PRODBUGS
- DEVBUGS
- "Andrew Bug Test"

### Priority Mapping
The workflow maps LLM priority recommendations to these Jira priority names:
- Lowest (ID: 1)
- Low (ID: 2)
- Medium (ID: 3)
- High (ID: 4)
- Urgent (ID: 5)

## Requirements

- n8n instance with Jira and Slack integrations configured
- OpenAI API access for LLM analysis
- Jira webhook configured to trigger on issue creation
- Slack webhook or bot token for notifications
