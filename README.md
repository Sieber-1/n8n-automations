# n8n Automations

Sammlung von n8n-Workflows für Dokumentenverarbeitung und Kommunikationsautomatisierung.

| Workflow | Beschreibung |
|---|---|
| [telegram-voice-to-email](./telegram-voice-to-email/) | Sprachnachricht → Gemini-Transkription → professionelle E-Mail |
| [kap-invoice-extractor](./kap-invoice-extractor/) | KAP-Steuerbescheinigung per Gmail → Felder extrahieren → CSV per Mail |

## Setup (gilt für alle Workflows)

1. n8n öffnen → Workflows → **Import from File** → `workflow.json` im jeweiligen Ordner
2. Credentials in jedem Node zuweisen (Platzhalter `REPLACE_ME`)
3. n8n-Variablen setzen (siehe README des jeweiligen Workflows)
4. Workflow aktivieren und testen

## Tech Stack

n8n · Google Gemini · Gmail API · Telegram Bot API · SMTP
