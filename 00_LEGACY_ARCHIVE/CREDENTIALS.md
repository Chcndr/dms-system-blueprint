# 🔐 Credenziali e Accessi Sistema DMS/MIHUB

**⚠️ DOCUMENTO SENSIBILE - NON CONDIVIDERE PUBBLICAMENTE**

---

## 🖥️ Server Hetzner (Backend MIHUB)

### Informazioni Server

**⚠️ IMPORTANTE:** Questo server ospita il **Backend Mercati/GIS**, NON il backend MIO-hub generale.
- **Backend ospitato:** Mercati/GIS API (Dashboard PA)
- **Repository:** `Chcndr/mihub` (monorepo)
- **Path server:** `/root/mihub-backend-rest`
- **URL pubblico:** `https://mihub.157-90-29-66.nip.io/api`

| Campo | Valore |
| :--- | :--- |
| **IP Pubblico** | `157.90.29.66` |
| **Hostname** | `orchestratore.mio-hub.me` |
| **OS** | Ubuntu 22.04.5 LTS |
| **User** | `root` |

### Accesso SSH

**Metodo 1: SSH Key (RACCOMANDATO)**
```bash
ssh -i ~/.ssh/hetzner_deploy_key root@157.90.29.66
```

**Chiave SSH Attuale:**
- **Nome:** `manus-deploy-20251128`
- **Tipo:** ed25519
- **Fingerprint:** `86:e6:34:de:f1:9f:2f:c3:c0:a3:ab:88:71`
- **Path sandbox:** `/home/ubuntu/.ssh/hetzner_deploy_key`
- **Aggiunta:** 28 Novembre 2024

**Metodo 2: Password (Backup)**
```bash
ssh root@157.90.29.66
# Password: maR9riihma3q
```

**⚠️ Nota:** Password aggiornata il 28/11/2024 tramite Hetzner Console

**Posizione Chiave SSH:**
- Path nel sandbox: `/home/ubuntu/.ssh/hetzner_server_key`
- Tipo: ed25519
- Permessi: `600` (solo lettura per owner)

### Directory Principali

| Directory | Contenuto |
| :--- | :--- |
| `/root/mihub-backend-rest/` | Backend Mercati/GIS (monorepo mihub) |
| `/root/.pm2/` | PM2 process manager |
| `/root/.pm2/logs/` | Log applicazioni PM2 |

### Backend Mercati/GIS - Dettagli

**Repository GitHub:**
- **Owner:** `Chcndr`
- **Nome:** `mihub`
- **URL:** https://github.com/Chcndr/mihub
- **Branch:** master
- **Tipo:** Monorepo (pnpm workspaces)

**Struttura Monorepo:**
```
/root/mihub-backend-rest/
├── apps/
│   ├── api/              ← Backend REST API
│   │   ├── server/       ← Router e logica
│   │   ├── src/          ← Entry point
│   │   └── dist/         ← Codice compilato
│   └── frontend/         ← Dashboard (non deployato qui)
├── packages/             ← Shared packages (mancante - causa errori)
└── pnpm-workspace.yaml
```

**PM2 Configuration:**
- **Process name:** `mihub-backend`
- **Script:** `apps/api/dist/apps/api/src/server.js`
- **Status attuale:** ❌ errored (manca packages/shared)
- **Porta:** 3000

**Endpoint Principali:**

| Endpoint | Descrizione | Router |
| :--- | :--- | :--- |
| `/api/gis/market-map` | Mappa mercati per Editor v3 | gisRouter.ts |
| `/api/markets/:id/stalls` | Lista banchi per mercato | dmsHubRouter.ts |
| `/api/markets/:id/companies` | Aziende per mercato | dmsHubRouter.ts |
| `/api/markets/list` | Lista tutti i mercati | dmsHubRouter.ts |
| `/api/dmsHub/status` | Health check | dmsHubRouter.ts |
| `/api/gis/health` | Health check GIS | gisRouter.ts |

**⚠️ Problema Attuale (28/11/2024):**
- Il monorepo è configurato con `workspaces: ["packages/*"]`
- La directory `packages/` non esiste nel repository
- Il codice compilato importa `@shared/const` che non può essere risolto
- PM2 crasha con `ERR_MODULE_NOT_FOUND`
- Tutti gli endpoint restituiscono 502 Bad Gateway

---

## 🔑 Secrets e API Tokens

I seguenti secrets sono configurati nel **DMS Hub** → **Configurazioni** → **API & Agent Tokens**:

### GitHub Personal Access Token

- **Nome nel sistema:** `GITHUB_PERSONAL_ACCESS_TOKEN`
- **Scope:** `repo` (accesso completo ai repository)
- **Usato da:** Agente Abacus (via proxy MIHUB)
- **Stato:** ✅ Configurato
- **Ultime 4 cifre:** `airq`
- **Aggiornato:** 27/11/2025, 04:51:26

### Vercel Token

- **Nome nel sistema:** `VERCEL_TOKEN`
- **Scope:** `deploy`
- **Usato da:** Deploy automatici frontend
- **Stato:** ⚠️ Non configurato

### TPER API Key

- **Nome nel sistema:** `TPER_API_KEY`
- **Scope:** `mobility`
- **Usato da:** Integrazioni trasporto pubblico
- **Stato:** ⚠️ Non configurato

**Nota:** I token sono cifrati con AES-256-GCM usando una master key sul server prima di essere salvati nel database.

---

## 🗄️ Database PostgreSQL (Neon)

### Connessione

| Campo | Valore |
| :--- | :--- |
| **Host** | `ep-bold-silence-a2w3xqbw.eu-central-1.aws.neon.tech` |
| **Database** | `neondb` |
| **User** | `neondb_owner` |
| **Password** | `npg_lYG6JQ5Krtsi` |
| **SSL Mode** | `require` |
| **Channel Binding** | `require` |

### Connection String

```
postgresql://neondb_owner:npg_lYG6JQ5Krtsi@ep-bold-silence-a2w3xqbw.eu-central-1.aws.neon.tech/neondb?sslmode=require
```

### Console Web

- **URL:** https://console.neon.tech
- **Email:** `chcndr@gmail.com`
- **Password:** (verificare con utente)

---

## ☁️ Vercel (Frontend)

### Account

- **URL:** https://vercel.com/dashboard
- **Email:** `chcndr@gmail.com`
- **Password:** (verificare con utente)

### Progetti Deployati

| Progetto | URL | Repository |
| :--- | :--- | :--- |
| **Dashboard PA** | https://dms-hub-app-new.vercel.app | `Chcndr/dms-hub-app-new` |
| **App Cittadini** | https://dms-cittadini.vercel.app | `Chcndr/dms-cittadini` |
| **App Operatore** | https://dms-operatore.vercel.app | `Chcndr/dms-operatore` |

---

## 🐙 GitHub

### Account

- **Username:** `Chcndr`
- **Email:** `chcndr@gmail.com`

### Repository Principali

| Repository | Descrizione |
| :--- | :--- |
| `Chcndr/dms-system-blueprint` | Blueprint e documentazione sistema |
| `Chcndr/mihub-backend-rest` | Backend MIHUB REST API |
| `Chcndr/dms-hub-app-new` | Frontend Dashboard PA |
| `Chcndr/MIO-hub` | Catalogo API e configurazioni |

---

## 🔒 Note di Sicurezza

- **Master Key Secrets:** Configurata in `/root/mihub-backend-rest/.env` come `SECRETS_MASTER_KEY`
- **Backup Credenziali:** Disponibile in `/home/ubuntu/upload/mihub_credentials(3).pdf`
- **Rotazione Token:** I token GitHub e Vercel dovrebbero essere ruotati ogni 90 giorni
- **Accesso SSH:** Preferire sempre la chiave SSH alla password

---

## 📚 Riferimenti

- **Guida Deploy Backend:** `07_guide_operative/DEPLOY_BACKEND_HETZNER.md`
- **Guida Deploy Frontend:** `07_guide_operative/DEPLOY_FRONTEND_VERCEL.md`
- **Architettura Sistema:** `01_architettura/MASTER_SYSTEM_PLAN.md`
- **Credenziali Legacy:** `01_architettura/legacy/root_legacy/CREDENZIALI_E_ACCESSI.md`

---

**Ultimo Aggiornamento:** 28 Novembre 2024  
**Autore:** Manus AI
