# KAP Invoice Extractor

n8n-Workflow: Steuerliche Kapitalertragsbescheinigungen (KAP-Dokumente) per
Gmail empfangen → PDF-Text extrahieren → Felder per Gemini auslesen →
strukturiert als CSV per Mail zurücksenden.

![Workflow Canvas](docs/canvas.png)

## Was es tut

Banken verschicken Jahressteuerbescheinigungen als PDF per Mail. Dieser
Workflow erkennt neue Mails mit PDF-Anhang, extrahiert automatisch alle
steuerrelevanten Felder und liefert sie als CSV — bereit für die
Steuererklärung.

## Extrahierte Felder

| Feld | Beschreibung |
|---|---|
| Hoehe_der_Kapitalertraege | Gesamthöhe der Kapitalerträge |
| Gewinn_Aktienveraeusserungen | Gewinn aus Aktienverkäufen |
| Gewinn_AltAnteile | Gewinn aus Altanteilen |
| Ersatzbemessungsgrundlage | Ersatzbemessungsgrundlage |
| Sparer_Pauschbetrages | Angerechneter Sparer-Pauschbetrag |
| Kapitalertragsteuer | Einbehaltene Kapitalertragsteuer |
| Solidaritaetszuschlag | Einbehaltener Solidaritätszuschlag |
| Kirchensteuer_zur_Kapitalsteuer_Evangelisch | Kirchensteuer (ev.) |
| Kirchensteuer_zur_Kapitalsteuer | Kirchensteuer gesamt |
| Summe_angerechnete_auslaendische_Steuer | Angerechnete ausländische Steuer |
| Summe_anrechenbare_nicht_angerechnete_auslaendische_Steuer | Noch nicht angerechnete ausländische Steuer |

## Ablauf

```
Gmail Trigger (pollt jede Minute)
  └─ Get a message (lädt PDF-Anhang)
       └─ Extract from File (PDF → Text)
            └─ AI Agent (Gemini extrahiert Felder als JSON)
                 └─ Edit Fields (JSON-Felder einzeln mappen)
                      └─ Convert to File (→ CSV)
                           └─ Send email (CSV als Anhang)
```

## Benötigte Credentials

| Credential | Wo eintragen |
|---|---|
| Gmail OAuth2 | Nodes: Gmail Trigger, Get a message |
| Google Gemini API Key | Node: Google Gemini Chat Model |
| SMTP-Zugangsdaten | Node: Send email |

n8n-Variablen (Settings → Variables):
```
MAIL_FROM    Absenderadresse
MAIL_TO      Empfängeradresse
```

## Import

1. n8n → Workflows → Import from File → `workflow.json`
2. Credentials zuweisen
3. Variablen setzen
4. Workflow aktivieren

## Bekannte Grenzen

- Erwartet den PDF-Anhang als erstes Attachment (`attachment_0`).
  Mails mit mehreren Anhängen oder ohne Anhang schlagen fehl.
- Kein Fehler-Ausgang implementiert — bei Extraktionsfehler läuft
  ein leeres JSON weiter. Produktiv sollte ein Fehlerbenachrichtigungs-Node
  ergänzt werden.
- Die Extraktion ist auf das Format deutscher Jahressteuerbescheinigungen
  ausgelegt. Andere PDF-Formate liefern möglicherweise null-Werte.
