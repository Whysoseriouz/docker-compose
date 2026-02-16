# 🌍 GeoServer mit Traefik & Let's Encrypt

GeoServer 2.28.0 hinter Traefik als Reverse Proxy mit automatischer SSL-Zertifikatsverwaltung über Let's Encrypt.

## 📋 Voraussetzungen

- Docker & Docker Compose installiert
- Domain zeigt per **A-Record** auf den Server
- Port **80** und **443** sind offen (Let's Encrypt HTTP-Challenge benötigt Port 80)

## ⚡ Schnellstart

### 1. `.env` anpassen

```env
DOMAIN=geoserver.kuntztech.de
ACME_EMAIL=deine-email@example.com
GEOSERVER_ADMIN_PASSWORD=ein-sicheres-passwort
TZ=Europe/Berlin
```

| Variable | Beschreibung |
|---|---|
| `DOMAIN` | Deine Domain für GeoServer |
| `ACME_EMAIL` | E-Mail für Let's Encrypt Registrierung |
| `GEOSERVER_ADMIN_PASSWORD` | Admin-Passwort für GeoServer |
| `TZ` | Zeitzone für alle Container (z.B. `Europe/Berlin`) |

### 2. Starten

```bash
docker compose up -d
```

### 3. Aufrufen

🔗 `https://deine-domain.de/geoserver/`

Login: `admin` / dein gesetztes Passwort

## 🏗️ Architektur

```
Internet
   │
   ▼
┌──────────┐  Port 80/443
│  Traefik │──────────────── Let's Encrypt
│  (Proxy) │
└────┬─────┘
     │ :8080
     ▼
┌──────────┐
│GeoServer │──── /opt/geoserver_data (Volume)
│  2.28.0  │
└──────────┘
```

- **Traefik** terminiert SSL und leitet Requests an GeoServer weiter
- Automatischer **HTTP → HTTPS** Redirect
- Zertifikate werden im `letsencrypt` Volume persistiert
- GeoServer-Daten im `geoserver_data` Volume persistiert

## 🔧 Konfiguration

### 🖥️ GeoServer GUI – Proxy-Einstellungen

Nach dem ersten Login müssen im GeoServer-GUI die Proxy-Einstellungen angepasst werden:

1. Einloggen als `admin`
2. Navigiere zu **Einstellungen** → **Global**
3. **Proxy-Basis-URL** setzen auf:
   ```
   https://geoserver.kuntztech.de/geoserver
   ```
4. ✅ Checkbox **„Anpassung der Proxy-URL über HTTP-Header ermöglichen"** aktivieren
5. Speichern

> 💡 Damit liest GeoServer die `X-Forwarded-Proto` und `X-Forwarded-Host` Header von Traefik aus und generiert korrekte URLs in Capabilities-Dokumenten, GetMap-Requests und der Weboberfläche.

### Proxy-Header

Traefik setzt die Header `X-Forwarded-Proto` und `X-Forwarded-Host` via Middleware, damit GeoServer korrekt hinter dem Proxy arbeitet. Diese Header werden erst wirksam, wenn die obige Checkbox im GUI aktiviert ist.

### CSRF

Der CSRF-Schutz ist per Whitelist konfiguriert (`GEOSERVER_CSRF_WHITELIST`). Die Domain aus der `.env` wird automatisch als erlaubter Origin eingetragen, damit der Login hinter dem Reverse Proxy funktioniert.

## 🛠️ Nützliche Befehle

```bash
# Starten
docker compose up -d

# Stoppen
docker compose down

# Logs anzeigen
docker compose logs -f geoserver

# GeoServer komplett zurücksetzen (⚠️ löscht alle Daten!)
docker compose down
docker volume rm geoserver_geoserver_data
docker compose up -d
```

## ⚠️ Hinweise

- Beim **ersten Start** wird das Admin-Passwort aus der `.env` in das Data-Volume geschrieben. Spätere Änderungen der Umgebungsvariable haben **keinen Effekt** – das Passwort muss dann über die GeoServer-Weboberfläche geändert werden, oder das Volume muss gelöscht werden.
- Die `.env`-Datei enthält Passwörter und sollte **nicht** ins Git-Repository committed werden.
