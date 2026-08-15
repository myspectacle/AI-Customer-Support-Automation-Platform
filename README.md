# AI Customer Support Automation Platform

An event-driven customer support ticket automation system built with **n8n**, using a shared **Google Sheet** as the single source of truth. Each stage of a ticket's lifecycle is handled by an independent workflow, triggered by webhooks or scheduled crons — no workflow calls another directly.

## 🧭 Overview

This platform automates the full lifecycle of a support ticket:

```
Ticket Created → Classified → Assigned → Resolved → Feedback Collected
```

Five decoupled workflows read from and write to one shared Google Sheet, communicating implicitly through sheet state rather than direct workflow-to-workflow calls. This keeps the system loosely coupled, easy to debug, and simple to extend.

## 🏗️ Architecture

- **Trigger layer**: Each workflow has its own independent trigger — either a webhook (for real-time events like ticket creation) or a cron schedule (for periodic checks like reminders or escalations).
- **Data layer**: A single Google Sheet acts as the shared state store for all tickets, avoiding the need for a dedicated database.
- **Notification layer**: Gmail is used for outbound notifications (acknowledgments, assignment alerts, resolution confirmations, feedback requests).

```
                ┌─────────────────────────────┐
                │      Google Sheet (DB)      │
                │  tickets | status | owner …  │
                └───────────▲─────▲───▲───▲────┘
                             │     │   │   │
        ┌──────────┐   ┌────┴┐ ┌──┴──┐│ ┌─┴─────────┐
        │ Workflow │   │ WF  │ │ WF  ││ │ Workflow   │
        │ 01: New  │   │ 02: │ │ 03: ││ │ 05:        │
        │ Ticket   │   │Class│ │Assig││ │ Feedback   │
        │ (webhook)│   │ify  │ │n    ││ │ (cron/     │
        └──────────┘   └─────┘ └─────┘│ │ webhook)   │
                                  ┌────┴┐└────────────┘
                                  │ WF  │
                                  │ 04: │
                                  │Resol│
                                  │ve   │
                                  └─────┘
```

## ⚙️ Workflows

| # | Workflow | Trigger | Purpose |
|---|----------|---------|---------|
| 01 | New Ticket Intake | Webhook | Receives incoming ticket (name, email, subject, message, channel), logs it to the Google Sheet, sends an acknowledgment email |
| 02 | Ticket Classification | Sheet update / cron | Categorizes and prioritizes incoming tickets |
| 03 | Ticket Assignment | Sheet update / cron | Assigns classified tickets to the appropriate owner/queue |
| 04 | Ticket Resolution | Webhook / manual trigger | Marks tickets resolved, notifies the customer |
| 05 | Feedback Collection | Cron / webhook | Sends feedback survey after resolution, logs responses |

> Exact node-level details for each workflow will be documented per-workflow once the exported JSON files are added under `/workflows`.

## 📁 Repository Structure

```
.
├── README.md
├── workflows/           # Exported n8n workflow JSON files (01–05)
├── docs/                 # Per-workflow documentation, sequence diagrams
├── sample-data/          # Example ticket.json payloads for testing
└── .gitignore
```

## 🚀 Setup

1. Import each workflow JSON from `/workflows` into your n8n instance.
2. Connect your Google Sheets and Gmail credentials in n8n.
3. Update the shared Google Sheet ID/URL in each workflow's Sheet node.
4. Activate the webhook-triggered workflows (01, 04) and enable the cron-triggered ones (02, 03, 05).
5. Test end-to-end with a sample ticket:
   ```bash
   curl -X POST http://localhost:5678/webhook/new-support-ticket \
     -H "Content-Type: application/json" \
     -d @sample-data/ticket.json
   ```

## 🧪 Testing

Use the sample payloads in `/sample-data` to trigger each stage manually and verify the ticket's status updates correctly in the Google Sheet at every step.

## 📌 Status

🚧 In progress — workflow exports and per-workflow documentation to be added.
