# CREDENZIALI E ACCESSI - DMS / MIO-HUB

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Ultimo Aggiornamento:** 21 Novembre 2025 - 18:20 GMT+1  
**Stato:** Documento Ufficiale  
**⚠️ ATTENZIONE:** Questo documento contiene informazioni sensibili. Proteggere adeguatamente.

---

## 1. SERVER HETZNER (Produzione Backend)

### Informazioni Server

```
Provider: Hetzner Cloud
IP: 157.90.29.66
Hostname: orchestratore.mio-hub.me
OS: Ubuntu 22.04 LTS
```

### Accesso SSH

```bash
# Metodo 1: SSH Key (RACCOMANDATO)
ssh -i /home/ubuntu/.ssh/hetzner_server_key root@157.90.29.66

# Metodo 2: Password (Backup)
ssh root@157.90.29.66
# Password: tkFxkgnT4ieh
```

**SSH Key Location:**
- Path locale: `/home/ubuntu/.ssh/hetzner_server_key`
- Tipo: ed25519
- Fingerprint: (verificare con `ssh-keygen -lf /home/ubuntu/.ssh/hetzner_server_key`)

### Directory Principali

```
/root/mihub/                    # Repository principale (tRPC backend)
/root/mihub-backend-rest/       # Backend REST attualmente in produzione
/root/MIO-hub/                  # Repository legacy/configurazioni
/root/.pm2/                     # PM2 process manager
```

### Processi Attivi

```bash
# Verificare processi PM2
pm2 list

# Backend attualmente in esecuzione
# Nome: mihub-backend
# Script: /root/mihub-backend-rest/index.js
# Port: 3001 (interno)
# Uptime: Verificare con pm2 list
```

### Nginx Configuration

```
Reverse Proxy: Nginx
Domain: orchestratore.mio-hub.me
Port esterno: 443 (HTTPS)
Port interno: 3001 (backend)
Config: /etc/nginx/sites-available/orchestratore.mio-hub.me
```

---

## 2. DATABASE POSTGRESQL (Neon)

### Informazioni Database

```
Provider: Neon.tech
Account Email: chcndr@gmail.com
Project: neondb
Region: us-east-1 (AWS)
```

### Connection String

```
Host: ep-bold-silence-adftsojg-pooler.c-2.us-east-1.aws.neon.tech
Database: neondb
User: neondb_owner
Password: npg_lYG6JQ5Krtsi
SSL Mode: require
Channel Binding: require
Port: 5432
```

**Full Connection String:**
```
postgresql://neondb_owner:npg_lYG6JQ5Krtsi@ep-bold-silence-adftsojg-pooler.c-2.us-east-1.aws.neon.tech/neondb?sslmode=require&channel_binding=require
```

### Accesso Web Console

```
URL: https://console.neon.tech
Email: chcndr@gmail.com
Password: (verificare con utente)
```

### Schema Tabelle Principali

```sql
-- Mercati
markets (id, name, city, address, lat, lng, active, created_at, updated_at)

-- Geometrie GIS
market_geometry (id, market_id, container_geojson, center_lat, center_lng, gcp, png_url, created_at, updated_at)

-- Piazzole
stalls (id, market_id, number, lat, lng, area_mq, category, gis_feature_id, created_at, updated_at)

-- Marker personalizzati
custom_markers (id, market_id, lat, lng, icon, label, created_at, updated_at)

-- Aree personalizzate
custom_areas (id, market_id, geometry_geojson, label, color, created_at, updated_at)

-- Ambulanti
vendors (id, name, business_name, vat_number, email, phone, created_at, updated_at)

-- Concessioni
concessions (id, stall_id, vendor_id, start_date, end_date, active, created_at, updated_at)

-- Presenze
attendances (id, stall_id, vendor_id, date, status, created_at, updated_at)
```

---

## 3. FRONTEND VERCEL (Dashboard)

### Informazioni Deployment

```
Provider: Vercel
Account Email: chcndr@gmail.com
Project: dms-hub-app-new
Domain Produzione: https://dms-hub-app-new.vercel.app
Framework: Next.js + React + Vite
```

### Repository GitHub

```
Repository: https://github.com/Chcndr/dms-hub-app-new
Branch Produzione: master
Branch Sviluppo: feature/gis-market-map-v1
```

### Variabili Ambiente (Vercel)

```bash
# Produzione
VITE_API_URL=https://orchestratore.mio-hub.me

# Locale (.env.local)
VITE_API_URL=http://localhost:3333
```

### Accesso Vercel Dashboard

```
URL: https://vercel.com/dashboard
Email: chcndr@gmail.com
Password: (verificare con utente)
```

### Deploy

```bash
# Auto-deploy da GitHub
# Ogni push su master → deploy automatico

# Deploy manuale
vercel --prod

# Preview deploy (da branch)
vercel
```

---

## 4. REPOSITORY GITHUB

### Account

```
Username: Chcndr
Email: chcndr@gmail.com
```

### Repository Principali

#### 4.1 Backend (mihub)

```
URL: https://github.com/Chcndr/mihub
Branch principale: master
Branch GIS: feature/gis-market-map-v1
Tecnologia: Node.js + Express + tRPC + Drizzle ORM
```

**Directory locale:**
```
/home/ubuntu/mihub/
├── apps/
│   ├── api/              # Backend API
│   │   ├── src/
│   │   │   └── server.ts
│   │   ├── server/
│   │   │   ├── gisRouter.ts      # ✅ NUOVO (GIS endpoint)
│   │   │   ├── dmsHubRouter.ts
│   │   │   └── routers.ts
│   │   ├── data/
│   │   │   └── editor-v3-sample.json  # ✅ NUOVO
│   │   └── drizzle/
│   │       └── schema.ts
│   └── frontend/         # Frontend (non usato, preferire dms-hub-app-new)
└── package.json
```

#### 4.2 Frontend (dms-hub-app-new)

```
URL: https://github.com/Chcndr/dms-hub-app-new
Branch principale: master
Branch GIS: feature/gis-market-map-v1
Tecnologia: React + Vite + Tailwind CSS
```

**Directory locale:**
```
/home/ubuntu/dms-hub-app-new/
├── client/
│   └── src/
│       ├── pages/
│       │   ├── MarketGISPage.tsx     # ✅ NUOVO (Mappa GIS)
│       │   ├── DashboardPA.tsx
│       │   └── ...
│       └── App.tsx                    # ✅ MODIFICATO (route /market-gis)
├── server/
├── dist/                              # Build output
└── package.json
```

#### 4.3 Blueprint (dms-system-blueprint)

```
URL: https://github.com/Chcndr/dms-system-blueprint
Branch principale: main
Tecnologia: Markdown documentation
```

**Directory locale:**
```
/home/ubuntu/dms-system-blueprint/
├── MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md          # ✅ AGGIORNATO
├── CREDENZIALI_E_ACCESSI.md                   # ✅ NUOVO (questo file)
├── REPORT_STATO_MAPPA_GIS_v2_21Nov2025.md    # ✅ NUOVO
├── docs/
│   ├── AS-IS/
│   └── TO-BE/
└── references/
```

### Clonare Repository

```bash
# Backend
gh repo clone Chcndr/mihub

# Frontend
gh repo clone Chcndr/dms-hub-app-new

# Blueprint
gh repo clone Chcndr/dms-system-blueprint
```

---

## 5. SERVIZI ESTERNI

### 5.1 Heroku (DMS Legacy)

```
Account Email: chcndr@gmail.com
App Name: (verificare)
Database: PostgreSQL (legacy)
Stato: Sistema legacy da migrare
```

### 5.2 OpenStreetMap (Mappe)

```
Provider: OpenStreetMap
Tile Server: https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png
Licenza: Open Database License
Uso: Mappa base per visualizzazione GIS
```

### 5.3 Leaflet (Libreria Mappe)

```
Libreria: react-leaflet + leaflet
Versione: (verificare package.json)
Documentazione: https://react-leaflet.js.org/
```

---

## 6. ENDPOINT API IMPLEMENTATI

### 6.1 Backend REST (Produzione Attuale)

**Base URL:** `https://orchestratore.mio-hub.me`

```
GET  /health                              # Health check
GET  /api/dmsHub/markets/list            # Lista mercati (mock)
GET  /api/dmsHub/markets/getById?id=1    # Dettaglio mercato (mock)
GET  /api/dmsHub/stalls/listByMarket     # Lista piazzole (mock)
```

### 6.2 Backend GIS (Nuovo - Non ancora in produzione)

**Base URL:** `http://localhost:3333` (locale) / `https://orchestratore.mio-hub.me` (produzione)

```
GET  /api/gis/market-map                 # ✅ NUOVO - Dati mappa GIS Editor v3
GET  /api/gis/health                     # ✅ NUOVO - Health check modulo GIS
```

**Esempio Response `/api/gis/market-map`:**

```json
{
  "success": true,
  "data": {
    "container": [[42.759, 11.111], ...],
    "center": { "lat": 42.758, "lng": 11.114 },
    "stalls_geojson": {
      "type": "FeatureCollection",
      "features": [...]
    },
    "markers_geojson": { "type": "FeatureCollection", "features": [] },
    "areas_geojson": { "type": "FeatureCollection", "features": [] }
  },
  "meta": {
    "endpoint": "gis.marketMap",
    "timestamp": "2025-11-21T17:43:40.123Z",
    "source": "editor-v3-sample.json",
    "stalls_count": 3
  }
}
```

---

## 7. ROUTE FRONTEND

### Dashboard PA

**Base URL:** `https://dms-hub-app-new.vercel.app`

```
/                           # Homepage
/dashboard-pa               # Dashboard Pubblica Amministrazione
/market-gis                 # ✅ NUOVO - Mappa GIS Mercato
/mappa                      # Mappa negozi (diversa da GIS)
/wallet                     # Wallet digitale
/civic                      # Servizi civici
/hub-operatore              # Hub operatore
/mihub                      # MIO-HUB
/guardian/endpoints         # Guardian API endpoints
/guardian/logs              # Guardian logs
/guardian/debug             # Guardian debug
```

---

## 8. FILE DATI IMPORTANTI

### 8.1 JSON Editor v3

```
# File di esempio (3 piazzole)
/home/ubuntu/mihub/apps/api/data/editor-v3-sample.json

# File completo Grosseto (160 piazzole)
/home/ubuntu/dms-hub-app-new/grosseto_160_posteggi_PRONTO.json
/home/ubuntu/dms-hub-app-new/client/public/data/grosseto_complete.json
/home/ubuntu/dms-hub-app-new/client/public/data/grosseto_full.geojson
```

### 8.2 Credenziali Originali

```
# PDF con credenziali complete
/home/ubuntu/upload/mihub_credentials(3).pdf
```

---

## 9. COMANDI UTILI

### 9.1 Backend (Hetzner)

```bash
# Connessione SSH
ssh -i /home/ubuntu/.ssh/hetzner_server_key root@157.90.29.66

# Verificare processi
pm2 list
pm2 logs mihub-backend
pm2 restart mihub-backend

# Aggiornare codice
cd /root/mihub
git fetch origin
git checkout feature/gis-market-map-v1
git pull origin feature/gis-market-map-v1

# Build e deploy
cd apps/api
npm install
npm run build
pm2 restart mihub-backend

# Verificare endpoint
curl https://orchestratore.mio-hub.me/health
curl https://orchestratore.mio-hub.me/api/gis/market-map
```

### 9.2 Frontend (Locale)

```bash
# Installare dipendenze
cd /home/ubuntu/dms-hub-app-new
npm install

# Build
npm run build

# Dev server
npm run dev

# Test endpoint
curl http://localhost:3333/api/gis/market-map
```

### 9.3 Database (Neon)

```bash
# Connessione psql
psql "postgresql://neondb_owner:npg_lYG6JQ5Krtsi@ep-bold-silence-adftsojg-pooler.c-2.us-east-1.aws.neon.tech/neondb?sslmode=require"

# Query test
SELECT * FROM markets LIMIT 5;
SELECT * FROM stalls LIMIT 5;
SELECT COUNT(*) FROM stalls;
```

### 9.4 Git (Push Branch)

```bash
# Backend
cd /home/ubuntu/mihub
git add apps/api/server/gisRouter.ts apps/api/data/editor-v3-sample.json apps/api/src/server.ts
git commit -m "feat(gis): Add market map endpoint for Editor v3 integration"
git push origin feature/gis-market-map-v1

# Frontend
cd /home/ubuntu/dms-hub-app-new
git add client/src/pages/MarketGISPage.tsx client/src/App.tsx
git commit -m "feat(gis): Add market GIS page with Leaflet map"
git push origin feature/gis-market-map-v1

# Blueprint
cd /home/ubuntu/dms-system-blueprint
git add MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md CREDENZIALI_E_ACCESSI.md REPORT_STATO_MAPPA_GIS_v2_21Nov2025.md
git commit -m "docs: Update blueprint with GIS integration progress"
git push origin main
```

---

## 10. CONTATTI E SUPPORTO

### Account Principali

```
Email principale: chcndr@gmail.com
```

### Link Utili

```
Dashboard Vercel: https://vercel.com/dashboard
GitHub: https://github.com/Chcndr
Neon Console: https://console.neon.tech
Hetzner Cloud: https://console.hetzner.cloud
```

### Documentazione

```
Blueprint Repository: https://github.com/Chcndr/dms-system-blueprint
Master System Plan: MASTER_SYSTEM_PLAN_DMS_MIO-HUB.md
Report GIS v2: REPORT_STATO_MAPPA_GIS_v2_21Nov2025.md
```

---

## 11. CHECKLIST DEPLOY

### Pre-Deploy

- [ ] Verificare build backend senza errori
- [ ] Verificare build frontend senza errori
- [ ] Testare endpoint localmente
- [ ] Verificare variabili ambiente
- [ ] Backup database (se necessario)

### Deploy Backend

- [ ] SSH su Hetzner
- [ ] Pull branch `feature/gis-market-map-v1`
- [ ] `npm install`
- [ ] `npm run build`
- [ ] `pm2 restart mihub-backend`
- [ ] Test endpoint produzione

### Deploy Frontend

- [ ] Push branch su GitHub
- [ ] Configurare variabile `VITE_API_URL` su Vercel
- [ ] Deploy su Vercel (auto o manuale)
- [ ] Test pagina `/market-gis` in produzione

### Post-Deploy

- [ ] Verificare endpoint `/api/gis/market-map`
- [ ] Verificare pagina `/market-gis`
- [ ] Test caricamento mappa
- [ ] Test visualizzazione piazzole
- [ ] Aggiornare documentazione blueprint

---

**Documento creato da:** Manus AI  
**Data creazione:** 21 Novembre 2025 - 18:20 GMT+1  
**Versione:** 1.0  
**⚠️ Proteggere questo documento - Contiene credenziali sensibili**
