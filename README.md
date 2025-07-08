# PollenGuard

**Selbstgehostetes Pollen‑Widget inkl. Generator‑Seite und Web‑Push.**

Dieses Projekt liefert Allergiker\:innen eine einfache Ampel‑Anzeige der aktuellen Pollenbelastung und einen Generator, um das Widget als `<iframe>` in beliebige Webseiten einzubetten. Backend und Frontend laufen in Docker‑Containern und lassen sich per Coolify in deinem Homelab deployen.

---

## Features

* **Ampel‑Widget**: gruene/gelbe/orange/rote Anzeige nach maximalem Tagesindex
* **Generator‑Seite**: Standort eingeben → fertiges `<iframe>`‑Snippet kopieren
* **REST API** `/api/pollen?lat=&lon=` sowie slug‑basierte Variante
* **Cron‑Fetch** (00:00) der Tagesprognose, Speicherung in SQLite → nur 1 API‑Call/Tag
* **Web‑Push** 07:00 Europe/Zurich mit persoenlichem Risiko‑Text
* **Komplett selbstgehostet**: kein Tracking, volle Datenkontrolle

---

## Tech‑Stack (Standard‑Preset)

| Ebene           | Technologie                            | Notiz                                       |
| --------------- | -------------------------------------- | ------------------------------------------- |
| Backend         | **FastAPI (Python 3.12)**              | Async, pydantic, leicht zu containerisieren |
| Cron            | APScheduler                            | Laeuft im selben Container                  |
| DB              | SQLite                                 | File‑basiert, reicht fuer Homelab           |
| Frontend Widget | Plain HTML + Tailwind CSS + Vanilla JS | Minimale Bundle‑Groesse                     |
| Generator SPA   | Svelte Kit                             | Schnell & simple                            |
| Web‑Push        | pywebpush                              | VAPID Standard                              |
| Container       | Docker Compose                         | Ein Stack fuer Coolify                      |

*(Express/Node 20 laesst sich alternativ einsetzen; passe Dockerfile & `docker‑compose.yml` entsprechend an.)*

---

## Repository‑Struktur

```text
pollen‑guard/
├── infra/
│   ├── Dockerfile            # Multistage Build
│   ├── docker‑compose.yml    # Services: backend, spa, nginx
│   └── coolify.env           # Beispiel‑Env Variablen
├── backend/
│   ├── app/
│   │   ├── main.py           # FastAPI Entry‑Point
│   │   ├── crud.py
│   │   ├── models.py
│   │   └── scheduler.py      # Cron Job
│   ├── requirements.txt
│   └── tests/
├── frontend/
│   ├── public/               # Widget statische Seite
│   │   └── index.html
│   ├── generator/            # Svelte Kit Code
│   └── package.json
├── db/                       # Bind Mount fuer SQLite
└── README.md
```

---

## Lokale Entwicklung

1. **Voraussetzungen**

   * Docker & Docker Compose
   * (Optional) Python 3.12 & Node 20 falls du ohne Container arbeiten willst
2. **Repo klonen**

   ```bash
   git clone https://github.com/deinuser/pollen‑guard.git
   cd pollen‑guard
   ```
3. **Env‑Datei kopieren & Werte setzen**

   ```bash
   cp infra/coolify.env .env
   # Werte wie VAPID_PUBLIC, VAPID_PRIVATE etc. setzen
   ```
4. **Stack starten**

   ```bash
   docker compose up --build
   ```
5. **Zugang**

   * Backend API: [http://localhost:8000/docs](http://localhost:8000/docs)
   * Widget: [http://localhost:8080/widget](http://localhost:8080/widget)
   * Generator: [http://localhost:8080](http://localhost:8080)

---

## Deployment mit Coolify

1. Verbinde Coolify mit deinem Git‑Repository.
2. Waehle **Dockerfile** Build‑Mode.
3. Weise die Datei `infra/docker‑compose.yml` als Compose‑Override zu (Coolify unterstuetzt das).
4. Lege folgende Env‑Variablen an:

   ```env
   APP_ENV=prod
   TIMEZONE=Europe/Zurich
   OPENMETEO_URL=https://air-quality-api.open-meteo.com/v1/air-quality
   VAPID_PUBLIC=…
   VAPID_PRIVATE=…
   ```
5. Aktiviere **SSL (Let's Encrypt)** in Coolify, setze CORS Header (`*` oder deine Domains) im Backend.
6. Deploy starten – Coolify buildet den Stack, erstellt Container‑Netzwerk & Volumes.

---

## Rate‑Limit & Cache

* Cronjob laeuft einmal taeglich um 00:00 und schreibt die Prognose in Tabelle `forecast`.
* Beim API‑Aufruf wird zuerst `forecast` abgefragt; nur wenn Datum nicht **heute** ist oder Eintrag fehlt, wird die externe API konsultiert.
* Damit sind maximal **1 Request/Tag** an Open‑Meteo noetig, egal wie viele User das Widget nutzen.

---

## Roadmap

| Phase    | Aufgabe                                                     | Status  |
| -------- | ----------------------------------------------------------- | ------- |
| **v0.1** | Grundlegendes FastAPI Backend mit `/api/pollen`             | ToDo    |
|          | Cronjob + SQLite Cache                                      | ToDo    |
|          | Statisches Widget (Ampel)                                   | ToDo    |
| **v0.2** | Generator SPA mit Koordinateneingabe & Iframe‑Snippet       | Planned |
|          | Web‑Push‑Opt‑In & Worker                                    | Planned |
| **v0.3** | User‑Profile (Pollenarten & Reminderzeit)                   | Planned |
|          | Mehrsprachigkeit (DE/EN/FR/IT)                              | Planned |
| **v1.0** | Historische Trend‑Charts, Edge‑Caching, Production Hardened | Future  |

---

## Lizenz

MIT License – siehe `LICENSE` Datei.

---

