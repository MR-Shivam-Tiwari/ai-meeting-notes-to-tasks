# 03 — AI Meeting Notes to Tasks & Summary

**Problem:** Meeting decisions and action items live in a transcript nobody re-reads. Tasks get forgotten and the same points are re-discussed next week.

**Solution:** Send a call transcript to the workflow and get back a clean summary, the decisions made, and one task row per action item — with owner and due date extracted — saved to a tracker sheet and emailed to the team.

## Flow
`Transcript Webhook → AI Summarise & Extract Tasks (OpenAI) → Build Task Rows (one item per task) → Save Tasks to Sheet → Email Meeting Summary (Gmail)`

The AI returns strict JSON: `{title, summary, decisions[], tasks[{task, owner, due}]}`. The prompt forbids inventing facts: unstated owners become `Unassigned`, unstated dates become `No date`.

## Setup
1. Import `workflow.json` into n8n.
2. Create a Header Auth credential for OpenAI (`Authorization: Bearer <your key>`) and select it in the *AI Summarise* node.
3. Connect Google Sheets (columns: meeting, summary, decisions, task, owner, due, created_at) and Gmail credentials; set the recipient in the Gmail node.
4. POST a transcript — in production this pairs with a recorder (Fireflies/Otter) webhook or a Drive folder trigger for transcript files.

## Sample input
```json
POST /webhook/meeting-transcript
{ "title": "Sprint Planning - 14 Oct",
  "transcript": "Asha: The checkout bug is priority one. Rohan, can you fix it by Thursday? Rohan: Yes, Thursday EOD. Dev: I'll update the onboarding screens next week. We decided to postpone the referral feature to November." }
```
Expected rows: "Fix checkout bug — Rohan — Thursday", "Update onboarding screens — Dev — next week", decision: referral feature postponed to November.

## Resume bullet
Built an AI meeting-notes automation in n8n + OpenAI that converts transcripts into summaries, decisions and owner-assigned task rows in a tracker, with the summary emailed to attendees automatically.

## Interview talking points
- Extraction vs invention: the key prompt rule is 'only facts stated in the transcript' — hallucinated owners/dates destroy trust in the tool.
- Data shape: one n8n item per task (fan-out in the Code node) is what makes the Sheets append and per-task follow-ups clean.
- Extension ideas: create the tasks directly in Jira/Asana/ClickUp, add a reminder workflow for overdue items.
