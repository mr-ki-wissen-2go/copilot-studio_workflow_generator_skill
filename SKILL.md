---
name: copilot-studio-it-ticket-workflow-reviewer
display_name: Copilot Studio IT-Ticket-Workflow Reviewer
description: Prüft und optimiert schrittweise JSON-Konfigurationen und UI-Einstellungen eines Microsoft-Copilot-Studio-Workflows für die Klassifizierung, menschliche Freigabe und Beantwortung eingehender IT-Anfragen.
language: de-DE
version: 1.0.0
product: zusammen mit Copilot am 22.09. erarbeitet.
---

# Copilot Studio IT-Ticket-Workflow Reviewer

## Zweck

Dieser Skill begleitet Nutzende beim schrittweisen Aufbau und Review eines IT-Ticket-Workflows in Microsoft Copilot Studio. Er prüft die tatsächlich aus Copilot Studio kopierten JSON-Knoten, übersetzt sie in verständliche Konfigurationen und gibt konkrete Verbesserungsvorschläge für die grafische Oberfläche.

Der Skill ist besonders geeignet, wenn der JSON-Code in Copilot Studio nur angezeigt, aber nicht direkt bearbeitet werden kann.

## Unterstützter Zielworkflow

1. Outlook-Trigger: Bei Eingang einer neuen E-Mail
2. Agent: E-Mail klassifizieren
3. If/Else: Kritische oder unklare Anfrage erkennen
4. Human Review: Menschliche Prüfung in Teams oder Outlook
5. If/Else: Freigabe auswerten
6. Agent: Antwortentwurf erstellen
7. Outlook: Antwort senden oder als Entwurf speichern
8. Optional: Verarbeitung protokollieren

## Arbeitsweise

### 1. Immer den echten Knoten prüfen

Fordere den vollständigen JSON-Code des aktuell ausgewählten Knotens an oder nutze den vom Nutzer bereitgestellten Code. Verlasse dich nicht auf vermutete interne Schemas.

Prüfe für jeden Knoten:

- Knoten-ID, Name, Typ und Version
- Connector, Operation und Verbindung
- Eingaben und dynamische Ausdrücke
- Ausgabe-Schema und tatsächlich verfügbare Felder
- Verzweigungslogik
- Datentypen, insbesondere Boolean, Zahl und Text
- Groß- und Kleinschreibung bei Textvergleichen
- leere oder widersprüchliche Konfigurationen
- Datenschutz- und Sicherheitsrisiken

### 2. Keine direkte JSON-Bearbeitung voraussetzen

Wenn der Nutzer den JSON-Code nicht bearbeiten kann:

- Erkläre exakt, welche Werte in der grafischen Oberfläche gewählt werden müssen.
- Beschreibe die Felder mit den sichtbaren UI-Bezeichnungen.
- Nutze dynamische Inhalte bevorzugt über das Blitzsymbol.
- Weise darauf hin, ob ein Wert als Text, Ausdruck, Zahl oder Boolean einzutragen ist.
- Fordere nach der Änderung den aktualisierten JSON-Code zur Kontrolle an.

### 3. Antwortstruktur pro Knoten

Antworte kompakt in dieser Reihenfolge:

1. **Urteil:** korrekt, verbesserungsbedürftig oder fehlerhaft
2. **Was der Knoten aktuell macht**
3. **Konkrete Probleme**
4. **Einzutragende Werte in der Oberfläche**
5. **Erwartete Verzweigung oder Ausgabe**
6. **Nächster Knoten**

Gib keine erfundenen Connectorfelder oder IDs an. Verwende IDs nur, wenn sie im bereitgestellten JSON stehen.

## Referenzkonfigurationen

### A. Outlook-Trigger

Erwartete Merkmale:

- `triggerType`: `connector`
- Connector: Office 365 Outlook
- Operation: `OnNewEmailV3`
- Ordner ist ausgewählt
- Anlageninhalte standardmäßig nicht einschließen, sofern sie nicht benötigt werden
- Wichtigkeit standardmäßig beliebig
- Nur mit Anlagen standardmäßig deaktiviert

Wichtige Ausgaben:

- `from`
- `subject`
- `body`
- `bodyPreview`
- `importance`
- `hasAttachments`
- `id`
- `conversationId`
- `receivedDateTime`
- `isHtml`
- `sensitivityLabelInfo`

Hinweis: Für Antworten auf die ursprüngliche Nachricht ist die Nachrichten-ID relevant. Für eine neue E-Mail an den Absender ist `from` relevant.

### B. Klassifizierungs-Agent

Empfohlene Einstellungen:

- Modus: Inline
- Websuche: Aus
- Menschliche Eskalation im Agenten: Aus, wenn der Workflow einen eigenen Human-Review-Knoten besitzt
- Ausgabe: Textantwort, falls keine strukturierte Ausgabe verfügbar ist
- Modell: das in der Umgebung verfügbare Reasoning-Modell

Empfohlene Anweisung:

```text
Analysiere die eingehende E-Mail als IT-Service-Anfrage.

Bewerte ausschließlich den Inhalt der übergebenen E-Mail.

Bestimme:
- category
- priority
- confidence
- humanReview

Zulässige Werte für category:
- Störung
- Serviceanfrage
- Berechtigung
- Sicherheitsvorfall
- Allgemeine Frage
- Unklar

Zulässige Werte für priority:
- Kritisch
- Hoch
- Mittel
- Niedrig

Klassifizierungsregeln:
- Verwende "Störung" bei technischen Fehlern, Ausfällen oder Fehlfunktionen.
- Verwende "Serviceanfrage" bei Anforderungen, Bestellungen, Änderungen oder neuen Leistungen.
- Verwende "Berechtigung" bei Rollen-, Zugriffs-, Konto- oder Berechtigungsänderungen.
- Verwende "Sicherheitsvorfall" bei Phishing, Malware, verdächtigen Aktivitäten, Datenschutzverletzungen, Datenverlust oder möglichen Angriffen.
- Verwende "Allgemeine Frage" bei Informations- oder Supportanfragen ohne konkrete Störung oder Änderung.
- Verwende "Unklar", wenn die Anfrage nicht eindeutig klassifiziert werden kann.

Regeln für confidence:
- confidence muss eine ganze Zahl zwischen 0 und 100 sein.
- confidence beschreibt die Sicherheit der Klassifizierung.
- Verwende niedrige Werte bei mehrdeutigen oder unvollständigen Informationen.

Regeln für humanReview:
Setze humanReview auf true, wenn mindestens eine der folgenden Bedingungen erfüllt ist:
- confidence < 80
- category = Sicherheitsvorfall
- category = Berechtigung
- category = Unklar

Andernfalls setze humanReview auf false.

WICHTIG:
- Gib ausschließlich gültiges JSON zurück.
- Gib keine Erklärungen oder Begründungen zurück.
- Gib keinen Markdown-Codeblock zurück.
- Gib keinen zusätzlichen Text vor oder nach dem JSON zurück.
- Gib das JSON in genau einer Zeile zurück.
- Verwende exakt die folgenden Feldnamen und Kategorien.

Antwortformat:
{"category":"","priority":"","confidence":0,"humanReview":false}
```

Wichtig: Die Anweisung muss im selben Agentenknoten auch die E-Mail als Eingabe referenzieren. Prüfe im echten JSON, ob die Triggerwerte tatsächlich in der Anweisung enthalten sind. Ohne Betreff und Nachrichtentext kann der Agent die eingehende E-Mail nicht zuverlässig analysieren.

Empfohlene Eingabeblöcke:

```text
Betreff:
@{triggerOutputs()?['body/subject']}

Absender:
@{triggerOutputs()?['body/from']}

E-Mail-Inhalt:
@{triggerOutputs()?['body/body']}
```

Wenn der Body HTML enthält, empfiehlt sich eine vorgelagerte HTML-zu-Text-Konvertierung oder eine geeignete Klartextquelle. `bodyPreview` kann abgeschnitten sein und sollte nicht ohne Prüfung als vollständiger Ersatz verwendet werden.

### C. Erste If/Else-Verzweigung

Wenn der Agent nur Text zurückgibt, prüfe nicht `message <= 80`. Das wäre ein Text-Zahl-Vergleich.

Robuster Prototyp:

- Agent-Antwort enthält `Sicherheitsvorfall`
- ODER enthält `Berechtigung`
- ODER enthält `Unklar`
- ODER ist leer

Die Vergleichswerte werden als reiner Text eingegeben:

```text
Sicherheitsvorfall
Berechtigung
Unklar
```

Nicht verwenden:

```text
@{"Sicherheitsvorfall"}
@{"Unklar"}
```

Diese Schreibweise ist kein geeigneter Literalwert für den Textvergleich in der UI.

Wenn strukturierte Ausgaben verfügbar sind, bevorzuge eine direkte Boolean-Prüfung von `humanReview = true`.

### D. Human Review

Empfohlene Einstellungen:

- Kanal: Teams oder Outlook
- Titel: dynamisch mit Betreff
- Zugewiesen an: interne Person oder Funktionskonto im eigenen Mandanten
- Eingaben:
  - `Kommentar`: Text
  - `Freigeben`: Ja/Nein
- Beide Antworten nur dann als Pflichtfeld markieren, wenn ein Kommentar zwingend benötigt wird. Für eine reine Freigabe sollte nur `Freigeben` verpflichtend sein.

Empfohlene Nachricht:

```text
Eine IT-Anfrage wurde automatisch als kritisch, berechtigungsbezogen oder unklar eingestuft.

Bitte prüfen Sie die Anfrage und entscheiden Sie, ob eine Antwort erstellt werden darf.

KI-Einstufung:
@{body('<ID-DES-KLASSIFIZIERUNGS-AGENTEN>')?['message']}

Absender:
@{triggerOutputs()?['body/from']}

Betreff:
@{triggerOutputs()?['body/subject']}

Inhalt:
@{triggerOutputs()?['body/body']}
```

Prüfe bei dynamischen Feldtiteln, dass nur der sichtbare Titel eingetragen wurde:

```text
Kommentar
Freigeben
```

Nicht:

```text
"title": "Kommentar"
"title": "Freigeben"
```

### E. Freigabe-If/Else

Wenn das Human-Review-Ausgabefeld `boolean` heißt:

- Eigenschaft: dynamischer Inhalt `boolean` aus dem Human-Review-Knoten
- Operator: Gleich
- Wert: Boolean `true`

Erwartetes Ausdrucksmuster:

```json
{
  "and": [
    {
      "equals": [
        "@body('<ID-DES-HUMAN-REVIEW-KNOTENS>')?['boolean']",
        true
      ]
    }
  ]
}
```

`true` darf nicht als Text in Anführungszeichen und nicht als `@{true}` eingetragen werden.

### F. Antwort-Agent

Empfohlene Einstellungen:

- Websuche: Aus
- Menschliche Unterstützung: Aus, da die Freigabe bereits erfolgt ist
- Ausgabe: Textantwort

Empfohlene Anweisung:

```text
Erstelle eine professionelle Antwort auf die eingehende IT-Anfrage.

Verwende folgende Informationen:

E-Mail:
@{triggerOutputs()?['body/body']}

Betreff:
@{triggerOutputs()?['body/subject']}

Absender:
@{triggerOutputs()?['body/from']}

Klassifizierung:
@{body('<ID-DES-KLASSIFIZIERUNGS-AGENTEN>')?['message']}

Kommentar der Prüfung:
@{body('<ID-DES-HUMAN-REVIEW-KNOTENS>')?['text']}

Regeln:
- Antworte in derselben Sprache wie die Anfrage.
- Bestätige den Eingang der Anfrage.
- Fasse das Anliegen kurz zusammen.
- Berücksichtige den Kommentar der prüfenden Person.
- Erfinde keine Ticketnummern, Bearbeitungszeiten, Ursachen oder Lösungen.
- Stelle bei fehlenden Informationen gezielte Rückfragen.
- Wiederhole keine Kennwörter oder andere Geheimnisse.
- Gib ausschließlich den fertigen E-Mail-Text zurück.
```

### G. E-Mail-Aktion

Unterscheide klar:

1. **Neue E-Mail senden**
   - An: ursprünglicher Absender
   - Betreff: `RE: ` plus ursprünglicher Betreff
   - Text: `message` des Antwort-Agenten

2. **Auf E-Mail antworten**
   - Nachrichten-ID: ursprüngliche Nachrichten-ID
   - Antworttext: `message` des Antwort-Agenten
   - Bevorzugt, wenn der Thread erhalten bleiben soll

3. **Entwurf speichern**
   - Für Pilotbetrieb empfohlen, wenn keine abschließende Versandfreigabe unmittelbar vorliegt

Prüfe die tatsächlich gewählte Outlook-Operation im JSON. Behaupte nicht, dass „E-Mail senden“ automatisch im ursprünglichen Thread antwortet.

## Sicherheits- und Governance-Prüfung

Achte besonders auf:

- keine automatische Antwort bei Sicherheits- oder Datenschutzvorfällen ohne Freigabe
- keine geheimen Zugangsdaten in Prompts, Protokollen oder Antworten
- keine vollständigen E-Mail-Inhalte in Protokollen, wenn Metadaten genügen
- Prüfung vorhandener Sensitivity Labels
- keine automatische Verarbeitung verschlüsselter Inhalte ohne getestete Lesbarkeit
- dokumentierte Zuständigkeit für Human Review
- Fehlerpfad bei leerer Agent-Antwort oder fehlgeschlagenem Connector
- Pilotbetrieb mit Entwürfen vor automatischem Versand

## Testfälle

Fordere mindestens diese Tests:

1. Normale technische Störung
2. Serviceanfrage
3. Berechtigungsänderung
4. Phishing- oder Sicherheitsmeldung
5. Unklare Anfrage
6. Leere Nachricht
7. Automatische Abwesenheitsantwort
8. Human Review: Ja
9. Human Review: Nein
10. Agent liefert ungültiges JSON
11. Nachricht enthält HTML
12. Nachricht trägt ein Sensitivity Label

## Grenzen

- Der Skill bearbeitet den Copilot-Studio-Canvas nicht direkt.
- Der Skill erzeugt keine unbekannten Connector-IDs oder Verbindungsschlüssel.
- Der Skill prüft immer die tatsächlich bereitgestellten JSON-Knoten.
- Der Skill darf keine sichere Funktionsfähigkeit behaupten, bevor die Testläufe erfolgreich waren.
