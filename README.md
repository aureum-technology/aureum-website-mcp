# aureum-website-mcp

MCP-Server der [Aureum Technology GmbH](https://aureum-tech.com/), IT-Systemhaus in Rodewisch im Vogtland.

Der Server stellt die öffentlichen Firmendaten von aureum-tech.com für KI-Assistenten bereit: Leistungen, Preise, Kontakt und eine Volltextsuche über die Website samt Ratgeber. Er läuft als Remote-Endpunkt (Streamable HTTP), ist nur lesend und braucht keine Anmeldung.

| | |
|---|---|
| Endpunkt | `https://aureum-tech.com/mcp` |
| Transport | Streamable HTTP (JSON-RPC 2.0 per `POST`) |
| Protokollversion | `2025-06-18` |
| Servername | `aureum-website` 1.0.0 |
| Authentifizierung | keine |
| Sprache der Daten | Deutsch |

Dieses Repository enthält die Beschreibung und Beispielkonfigurationen. Der Quellcode des Servers ist Teil der Website und nicht veröffentlicht.

## Tools

| Tool | Was es liefert | Parameter |
|---|---|---|
| `firma_und_kontakt` | Stammdaten: wer wir sind, Adresse, Telefon, E-Mail, Erreichbarkeit, Registerdaten und Profile | keine |
| `leistungen_auflisten` | Alle Leistungsbereiche mit den einzelnen Leistungen und der jeweiligen Detailseite | keine |
| `preise_abfragen` | Aktuelle Preise: Stundensätze für Firmen- und Privatkunden, Anfahrt, Zuschläge, Notfall-Rufbereitschaft, Datenrettungs-Festpreis. Gleiche Quelle wie die Preisseite der Website | `zielgruppe` (optional): `firma`, `privat` oder `alle` |
| `website_suchen` | Durchsucht alle Seiten von aureum-tech.com (Leistungen, Ratgeber, Preise) und liefert passende Seiten mit URL | `suchbegriff` (Pflicht, 2 bis 120 Zeichen), z. B. `Backup` oder `WLAN Werkhalle` |

## Anbinden

### Claude (Desktop, Web, Mobile)

Einstellungen → Connectors → „Custom connector hinzufügen“ → URL `https://aureum-tech.com/mcp` eintragen. Kein Token nötig.

### Claude Code

```bash
claude mcp add --transport http aureum https://aureum-tech.com/mcp
```

### Cursor

`.cursor/mcp.json` im Projekt oder global:

```json
{
  "mcpServers": {
    "aureum": {
      "url": "https://aureum-tech.com/mcp"
    }
  }
}
```

### Clients ohne Remote-Unterstützung (z. B. ältere Claude-Desktop-Versionen)

Über die Brücke `mcp-remote`:

```json
{
  "mcpServers": {
    "aureum": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://aureum-tech.com/mcp"]
    }
  }
}
```

Fertige Dateien liegen unter `examples/`.

## Direkt per HTTP testen

Initialisieren:

```bash
curl -s https://aureum-tech.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' -i
```

Die Antwort enthält den Header `Mcp-Session-Id`. Mit dieser ID Tools auflisten:

```bash
curl -s https://aureum-tech.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: <ID aus dem Initialize>" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'
```

Und ein Tool aufrufen:

```bash
curl -s https://aureum-tech.com/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: <ID aus dem Initialize>" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"preise_abfragen","arguments":{"zielgruppe":"firma"}}}'
```

Ein `GET` auf den Endpunkt antwortet mit `405`. Das ist so gewollt, der Server spricht nur `POST`.

## Datenschutz und Grenzen

Der Server gibt ausschließlich Daten aus, die auf aureum-tech.com ohnehin öffentlich stehen. Es werden keine Kundendaten, keine Nutzerdaten und keine Inhalte hinter einem Login ausgeliefert. Anfragen werden nicht mit externen Diensten geteilt. Schreibende Operationen gibt es nicht.

Der Dienst wird ohne Verfügbarkeitszusage betrieben. Wer ihn in einem eigenen Produkt einbauen will, meldet sich vorher kurz bei uns.

## Über Aureum Technology

Aureum Technology ist ein IT-Systemhaus aus Rodewisch. Wir betreuen kleine und mittlere Betriebe, Handwerk, Praxen und Kanzleien im Vogtland und um Zwickau: Netzwerk, Server, Datensicherung, Fernwartung und Hardware. Dazu kommt das Digital-Studio mit Webdesign, KI-Automatisierung und MCP-Integrationen für Firmendaten. Diesen Server haben wir zuerst für uns selbst gebaut; das Gleiche richten wir auch für andere Unternehmen ein.

- Website: [aureum-tech.com](https://aureum-tech.com/)
- MCP-Integrationen für Unternehmen: [aureum-tech.com/ki-loesungen/mcp-integrationen](https://aureum-tech.com/ki-loesungen/mcp-integrationen)
- Website-Check für Google- und KI-Sichtbarkeit: [aureum-tech.com/spotlens](https://aureum-tech.com/spotlens)
- Kontakt: info@aureum-tech.com, +49 3744 4399760
- [Impressum](https://aureum-tech.com/impressum) · [Datenschutz](https://aureum-tech.com/datenschutz)

---

## English summary

Remote MCP server of Aureum Technology GmbH, an IT service provider in Rodewisch (Vogtland, Germany). Endpoint `https://aureum-tech.com/mcp`, Streamable HTTP, read-only, no authentication, German-language content. Four tools: `firma_und_kontakt` (company and contact data), `leistungen_auflisten` (service catalogue with page URLs), `preise_abfragen` (current prices, optional `zielgruppe`: `firma` | `privat` | `alle`) and `website_suchen` (full-text search across the website, required `suchbegriff`). Only publicly available data is served; no write operations. Configuration examples for Claude, Claude Code, Cursor and `mcp-remote` are in `examples/`.
