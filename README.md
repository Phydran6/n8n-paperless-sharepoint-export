# Paperless-NGX → SharePoint Export (n8n Workflow)

Automatischer Export von Dokumenten aus Paperless-NGX nach Microsoft SharePoint, ausgelöst durch ein Tag. Jedes Dokument wird nach erfolgreichem Upload aus Paperless gelöscht und der Ablauf per Telegram gemeldet.

---

## Ablauf

```
[Trigger] → Dokument mit "Fertig"-Tag holen → Tags → Ordnerpfad bauen
         → PDF downloaden → SharePoint Ordner anlegen (falls nötig)
         → PDF Binary wiederherstellen → SharePoint Upload
              ├─ Erfolg → Paperless löschen → Zusammenfassung → Telegram ✅
              └─ Fehler → in Paperless behalten → Zusammenfassung → Telegram ❌
```

---

## Features

- Läuft alle 3 Sekunden, verarbeitet je einen Durchlauf ein Dokument
- Ordnerstruktur in SharePoint wird aus den Paperless-Tags gebaut (z.B. Tag `Rechnungen` → `Archiv/Paperless/Rechnungen/`)
- Ordner wird nur angelegt wenn er noch nicht existiert
- Dateinamen werden für SharePoint bereinigt (Sonderzeichen → `_`)
- Dokument wird **nur gelöscht** wenn der Upload erfolgreich war → kein Datenverlust
- Abschlussmeldung per Telegram mit Dokument, Zielordner und Status

---

## Voraussetzungen

| Was | Details |
|---|---|
| n8n | Self-hosted, Zugriff auf Paperless-API und SharePoint |
| Paperless-NGX | API-Token vorhanden, Tag "Fertig" angelegt |
| Microsoft SharePoint | OAuth2-App in Entra ID registriert |
| Telegram Bot | Bot-Token und Chat-ID bekannt |

---

## Einrichtung

### 1. Credentials in n8n anlegen

**Paperless API** (Typ: `Header Auth`)
- Header Name: `Authorization`
- Header Value: `Token DEIN_PAPERLESS_API_TOKEN`

**Microsoft SharePoint** (Typ: `Microsoft SharePoint OAuth2 API`)
- Entra App mit `Sites.ReadWrite.All` Berechtigung
- Tenant-ID, Client-ID und Client-Secret eintragen

**Telegram** (Typ: `Telegram API`)
- Bot-Token vom BotFather

---

### 2. Workflow importieren

In n8n: `Workflows → Import → JSON-Datei hochladen`

---

### 3. Platzhalter anpassen

Nach dem Import folgende Stellen im Workflow anpassen:

| Platzhalter | Wo | Beschreibung |
|---|---|---|
| `YOUR_PAPERLESS_URL` | Nodes: *Get 1 Document*, *Get All Tags*, *Download PDF*, *Delete from Paperless* | URL deiner Paperless-Instanz, z.B. `https://dms.example.com` |
| `YOUR_FERTIG_TAG_ID` | Node: *Get 1 Document with Fertig Tag* | Numerische ID des "Fertig"-Tags (in Paperless unter Einstellungen → Tags) |
| `YOUR_TENANT.sharepoint.com/sites/YOUR_SITE` | Node: *Build SharePoint Path* (Code) | SharePoint Site-URL |
| `/sites/YOUR_SITE/YOUR_SHAREPOINT_BASE_PATH/` | Node: *Build SharePoint Path* (Code) | Basispfad in SharePoint, z.B. `/sites/MeineSite/DMS/Archiv/` |
| `YOUR_TELEGRAM_CHAT_ID` | Node: *Telegram Benachrichtigung* | Chat-ID für Benachrichtigungen |
| `YOUR_PAPERLESS_CREDENTIAL_ID` | Alle Paperless-Nodes | Wird automatisch durch deine angelegte Credential ersetzt |
| `YOUR_SHAREPOINT_CREDENTIAL_ID` | SharePoint-Nodes | Wird automatisch durch deine angelegte Credential ersetzt |
| `YOUR_TELEGRAM_CREDENTIAL_ID` | Telegram-Node | Wird automatisch durch deine angelegte Credential ersetzt |

> **Tipp:** Credential-IDs müssen nicht manuell eingetragen werden — einfach in jedem Node die passende Credential aus dem Dropdown auswählen, dann überschreibt n8n die Platzhalter automatisch.

---

### 4. Fertig-Tag ID ermitteln

```
GET https://DEINE_PAPERLESS_URL/api/tags/
Authorization: Token DEIN_TOKEN
```

In der Antwort die `id` des Tags mit `name: "Fertig"` raussuchen und im Node *Get 1 Document with Fertig Tag* unter `tags__id__in` eintragen.

---

## SharePoint Ordnerstruktur

Die Ordner werden automatisch aus den Paperless-Tags des Dokuments gebildet:

```
Paperless Tags: ["Rechnungen", "Telekom"]
→ SharePoint: /Archiv/Paperless/Rechnungen/Telekom/Telekom Rechnung Feb 2026.pdf
```

Dokumente ohne verwertbare Tags landen in `/Archiv/Paperless/Sonstige/`.

---

## Verwandte Workflows

Dieses Projekt ist Teil eines größeren Paperless-NGX + Microsoft 365 Setups:
- **Mail Anhänge → SharePoint** — liest Outlook-Anhänge, OCR via Stirling-PDF, KI-Kategorisierung, Upload nach SharePoint

---

## Lizenz

MIT
