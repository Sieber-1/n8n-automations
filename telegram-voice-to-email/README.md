# Telegram Voice Memo → E-Mail

Sprachnachricht an einen Telegram-Bot schicken, Transkription per Gemini,
daraus formulierte E-Mail, Versand per SMTP, Rueckmeldung im Chat.

![Workflow](docs/architecture.png)

## Benoetigte Credentials

| Credential | Wofuer |
|---|---|
| Telegram Bot Token | Trigger, Datei-Download, Rueckmeldungen |
| Google Gemini API Key | Transkription + Textgenerierung |
| SMTP-Zugang | Mailversand |

Zusaetzlich drei Variablen, siehe `.env.example`: `ALLOWED_CHAT_ID`,
`MAIL_FROM`, `MAIL_TO`.

## Import

1. In n8n: Workflows → Import from File → `workflow.json`
2. Jedem Node die eigenen Credentials zuweisen (im Export stehen Platzhalter)
3. Variablen setzen
4. Mit einer Testnachricht aktivieren

## Ablauf

```
Telegram Trigger
  └─ Absender pruefen ──────────── (fremde Chat-ID: stiller Abbruch)
       └─ Ist Sprachnachricht? ─── nein ─┐
            └─ Voice-Datei laden ────────┤
                 └─ Transkription ───────┤
                      └─ Transkript ok? ─┤
                           └─ E-Mail formulieren ─┤
                                └─ E-Mail senden ─┤
                                     └─ Bestaetigung
                                                  └─ Fehlermeldung an Nutzer
```

## Warum die Zusatz-Nodes

Die naive Variante ist sechs Nodes lang: Trigger → Download → Transkription →
Agent → Senden. Sie funktioniert im Demo-Fall und bricht im Alltag:

- **Absender pruefen**: Ohne diesen Check kann jeder, der die Bot-ID kennt,
  Mails ueber deinen SMTP-Account ausloesen.
- **Ist Sprachnachricht?**: Bei einer Textnachricht ist `message.voice.file_id`
  undefined, der Download schlaegt fehl.
- **Transkript brauchbar?**: Bei leerem Transkript formuliert das Modell sonst
  eine Mail aus dem Nichts.
- **`alwaysOutputData: false`** an der Transkription: Mit `true` (dem
  urspruenglichen Wert) laeuft nach fehlgeschlagenen Retries ein leeres Objekt
  weiter — der Agent halluziniert dann eine vollstaendige E-Mail.
- **Fehlerpfad**: Ohne ihn scheitert der Workflow still, und der Nutzer wartet
  auf eine Mail, die nie kommt.

## Behobene Fehler der ersten Fassung

- Systemprompt begann mit `ou are an email drafting assistant` (fehlendes O).
- Der Prompt verbot `Subject:` im Body, gab aber ein Format vor, das damit
  beginnt — waehrend der Send-Node Zeile 1 als Betreff parst. Bei Befolgung der
  ersten Anweisung landete die Anrede in der Betreffzeile.
- Betreff-Parsing ohne Fallback: fehlte das Praefix, war der Betreff die Anrede.
  Jetzt Regex mit Default.
- Echte Mailadressen im Export. Jetzt Umgebungsvariablen.

## Grenzen

- Ein einzelner erlaubter Absender. Fuer mehrere waere eine Liste noetig.
- Kein Rate Limit: wer autorisiert ist, kann beliebig viele Mails ausloesen.
- Empfaenger ist fest verdrahtet; er wird nicht aus der Sprachnachricht
  abgeleitet.
- Keine Anhaenge, kein Threading, keine Entwurfs-Freigabe vor dem Versand.
- Gemini-Transkription ist bei Dialekt und Hintergrundgeraeusch fehleranfaellig;
  es gibt keine Confidence-Schwelle.
