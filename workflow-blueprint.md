# Workflow-Blueprint

```text
Outlook: Neue E-Mail
  ↓
Agent: Klassifizieren
  ↓
If/Else: Sicherheitsvorfall, Berechtigung, Unklar oder leer?
  ├─ Ja → Human Review
  │          ↓
  │       If/Else: Freigegeben?
  │          ├─ Ja → Antwort-Agent
  │          └─ Nein → Ende oder Eskalation
  └─ Nein → Antwort-Agent
             ↓
        Outlook: Antworten, senden oder Entwurf speichern
             ↓
        Optional: Metadaten protokollieren
```

## Empfohlene Knotennamen

- Neue IT-Anfrage eingegangen
- IT-Anfrage klassifizieren
- Menschliche Prüfung erforderlich?
- IT-Anfrage fachlich prüfen
- Freigabe erhalten?
- Antwort erstellen
- Antwort senden oder als Entwurf speichern
- Verarbeitung dokumentieren
