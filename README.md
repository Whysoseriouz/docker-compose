# GeoServer Infrastruktur

Docker-basierte GeoServer-Infrastruktur mit Traefik Reverse Proxy, automatischer SSL-Terminierung via Let's Encrypt und Masterportal als Web-Client.

## Projektstruktur

```
.
├── geoserver/          # GeoServer 2.28.0 + Traefik (Docker Compose)
│   ├── docker-compose.yml
│   ├── .env.example
│   └── README.md       # Detaillierte Setup-Anleitung
└── masterportal/       # Masterportal Web-Client (in Planung)
```

## Komponenten

| Komponente | Beschreibung |
|---|---|
| **GeoServer** | OGC-kompatibler Kartenserver (WMS, WFS, WCS) |
| **Traefik** | Reverse Proxy mit automatischer SSL-Zertifikatsverwaltung |
| **Masterportal** | Web-basierter Geo-Client (in Planung) |

## Voraussetzungen

- Docker & Docker Compose
- Domain mit A-Record auf den Server
- Port 80 und 443 offen

## Schnellstart

```bash
cd geoserver
cp .env.example .env
# .env anpassen (Domain, E-Mail, Passwort)
docker compose up -d
```

Weitere Details siehe [geoserver/README.md](geoserver/README.md).
