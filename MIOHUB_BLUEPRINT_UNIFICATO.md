# 🏗️ MIO HUB - BLUEPRINT UNIFICATO DEL SISTEMA

> **Versione:** 7.9  
> **Data:** 20 Febbraio 2026 (Aggiornato ore 02:30)  
> **Autore:** Sistema documentato da Manus AI  
> **Stato:** PRODUZIONE

---

## 📋 INDICE

1. [Panoramica Sistema](#panoramica-sistema)
2. [Architettura Completa](#architettura-completa)
3. [Repository GitHub](#repository-github)
4. [Servizi e Componenti](#servizi-e-componenti)
5. [MIO Agent - Sistema Multi-Agente](#mio-agent---sistema-multi-agente)
6. [Guardian - Sistema di Monitoraggio](#guardian---sistema-di-monitoraggio)
7. [Database e Storage](#database-e-storage)
8. [API Endpoints](#api-endpoints)
9. [Deploy e CI/CD](#deploy-e-cicd)
10. [Credenziali e Accessi](#credenziali-e-accessi)
11. [Troubleshooting](#troubleshooting)
12. [Regole per Agenti AI](#regole-per-agenti-ai)

---

## 🎯 PANORAMICA SISTEMA

### Cos'è MIO HUB?

**MIO HUB** è un ecosistema digitale per la gestione dei mercati ambulanti italiani. Include:

- **DMS HUB** - Dashboard principale per Pubblica Amministrazione
- **MIO Agent** - Sistema multi-agente AI per automazione
- **Guardian** - Sistema di logging e monitoraggio API
- **Gestionale** - Backend per operazioni CRUD

### Stack Tecnologico

| Layer | Tecnologia |
|-------|------------|
| **Frontend** | React 19 + TypeScript + Tailwind CSS 4 + shadcn/ui |
| **Backend** | Node.js + Express + tRPC |
| **Database** | PostgreSQL (Neon) |
| **AI/LLM** | Google Gemini API |
| **Hosting Frontend** | Vercel |
| **Hosting Backend** | Hetzner VPS (157.90.29.66) |
| **CI/CD** | GitHub Actions + PM2 |

---

## 🏛️ ARCHITETTURA COMPLETA

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              INTERNET                                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌───────────────┐         ┌─────────────────┐         ┌─────────────────┐
│   VERCEL      │         │  HETZNER VPS    │         │   NEON DB       │
│               │         │  157.90.29.66   │         │                 │
│ dms-hub-app-  │ ◄─────► │                 │ ◄─────► │  PostgreSQL     │
│ new.vercel.app│  API    │ orchestratore.  │  SQL    │  (Serverless)   │
│               │         │ mio-hub.me      │         │                 │
│ ┌───────────┐ │         │ ┌─────────────┐ │         │ ┌─────────────┐ │
│ │ React App │ │         │ │ Express API │ │         │ │ 542 mercati │ │
│ │ + tRPC    │ │         │ │ + PM2       │ │         │ │ + logs      │ │
│ │ client    │ │         │ │             │ │         │ │ + agents    │ │
│ └───────────┘ │         │ └─────────────┘ │         │ └─────────────┘ │
└───────────────┘         └─────────────────┘         └─────────────────┘
        │                           │
        │                           │
        ▼                           ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                         MODULI INTERNI BACKEND                             │
│                                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   GUARDIAN   │  │  MIO AGENT   │  │    LOGS      │  │   HEALTH     │  │
│  │              │  │              │  │              │  │   MONITOR    │  │
│  │ /api/guardian│  │ /api/mihub/  │  │ /api/logs/*  │  │ /api/health/ │  │
│  │ - health     │  │ orchestrator │  │ - createLog  │  │ - full       │  │
│  │ - testEndpoint│ │ - chats      │  │ - getLogs    │  │ - history    │  │
│  │ - logs       │  │ - messages   │  │ - stats      │  │ - alerts     │  │
│  │ - permissions│  │              │  │              │  │              │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
│                           │                                               │
│                           ▼                                               │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                    ORCHESTRATORE MIO                                │  │
│  │                                                                     │  │
│  │   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐           │  │
│  │   │   MIO   │   │ GPT Dev │   │  Manus  │   │ Abacus  │           │  │
│  │   │ (GPT-5) │──►│ GitHub  │   │ Server  │   │  SQL    │           │  │
│  │   │Coordina │   │  Code   │   │  PM2    │   │ Query   │           │  │
│  │   └─────────┘   └─────────┘   └─────────┘   └─────────┘           │  │
│  │        │                                          │                │  │
│  │        │        ┌─────────┐                       │                │  │
│  │        └───────►│ Zapier  │◄──────────────────────┘                │  │
│  │                 │ Email   │                                        │  │
│  │                 │WhatsApp │                                        │  │
│  │                 │Calendar │                                        │  │
│  │                 └─────────┘                                        │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────────┘
```

---

## 📁 REPOSITORY GITHUB

| Repository | Descrizione | URL |
|------------|-------------|-----|
| **dms-hub-app-new** | Frontend React + tRPC | https://github.com/Chcndr/dms-hub-app-new |
| **mihub-backend-rest** | Backend Express + API | https://github.com/Chcndr/mihub-backend-rest |
| **dms-system-blueprint** | Documentazione sistema | https://github.com/Chcndr/dms-system-blueprint |
| **mio-hub-implementation-deploy** | Script deploy | https://github.com/Chcndr/mio-hub-implementation-deploy |

### Struttura Repository Principale

```
dms-hub-app-new/
├── client/                 # Frontend React
│   ├── src/
│   │   ├── pages/         # Pagine dashboard
│   │   │   ├── DashboardPA.tsx      # Dashboard PA principale
│   │   │   └── MappaItaliaPage.tsx  # Pagina mappa pubblica
│   │   ├── components/    # Componenti UI
│   │   │   ├── GestioneHubPanel.tsx     # 🆕 Cabina di regia Hub (6 sub-tab)
│   │   │   ├── MappaItaliaComponent.tsx # Mappa admin (completa)
│   │   │   └── MappaItaliaPubblica.tsx  # 🆕 Mappa pubblica (sola lettura)
│   │   └── lib/           # Utilities
│   └── public/            # Asset statici
├── server/                 # Backend tRPC (Vercel)
│   ├── routers.ts         # Router principale
│   ├── guardianRouter.ts  # Guardian API
│   └── services/          # Servizi business
└── shared/                 # Tipi condivisi

mihub-backend-rest/
├── routes/
│   ├── orchestrator.js    # MIO Agent orchestratore
│   ├── guardian.js        # Guardian API
│   ├── health-monitor.js  # Health check
│   ├── logs.js            # Sistema logging
│   └── integrations.js    # Integrazioni esterne
├── src/
│   └── modules/
│       └── orchestrator/  # Logica multi-agente
│           ├── llm.js     # Chiamate Gemini
│           ├── database.js # DB orchestratore
│           └── *.js       # Tool agenti
└── index.js               # Entry point
```

---

## 🤖 MIO AGENT - SISTEMA MULTI-AGENTE

### Cos'è MIO Agent?

MIO Agent è un **sistema multi-agente interno** che coordina 5 agenti AI specializzati. **NON è un servizio esterno** su un sottodominio separato.

### Endpoint Principale

```
POST https://orchestratore.mio-hub.me/api/mihub/orchestrator
```

### I 5 Agenti

| Agente | Ruolo | Capabilities |
|--------|-------|--------------|
| **MIO** | Coordinatore (GPT-5) | Smista task, coordina agenti |
| **GPT Dev** | Sviluppatore | GitHub, commit, PR, codice |
| **Manus** | Operatore | SSH, PM2, file system, server |
| **Abacus** | Analista | Query SQL, analisi dati |
| **Zapier** | Automazioni | Email, WhatsApp, Calendar, Gmail |

### Modalità di Funzionamento

```javascript
// Mode AUTO - MIO decide quale agente usare
POST /api/mihub/orchestrator
{
  "mode": "auto",
  "message": "Quanti mercati ci sono nel database?"
}
// MIO smista ad Abacus

// Mode DIRECT - Chiama agente specifico
POST /api/mihub/orchestrator
{
  "mode": "direct",
  "targetAgent": "manus",
  "message": "Mostra lo stato di PM2"
}
```

### Tabelle Database

```sql
-- Messaggi degli agenti
CREATE TABLE agent_messages (
  id SERIAL PRIMARY KEY,
  conversation_id VARCHAR(255),
  sender VARCHAR(50),
  recipient VARCHAR(50),
  agent VARCHAR(50),
  role VARCHAR(20),
  message TEXT,
  meta JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Log delle chiamate
CREATE TABLE mio_agent_logs (
  id SERIAL PRIMARY KEY,
  timestamp TIMESTAMP DEFAULT NOW(),
  agent VARCHAR(50),
  service_id VARCHAR(100),
  endpoint VARCHAR(255),
  method VARCHAR(10),
  status_code INTEGER,
  risk VARCHAR(20),
  success BOOLEAN,
  message TEXT,
  meta_json JSONB
);
```

---

## 🛡️ GUARDIAN - SISTEMA DI MONITORAGGIO

### Cos'è Guardian?

Guardian è un **modulo interno del backend** che gestisce:
- Logging centralizzato di tutte le chiamate API
- Test endpoint (API Playground)
- Permessi degli agenti
- Statistiche di utilizzo

### Endpoint Guardian

| Endpoint | Metodo | Descrizione |
|----------|--------|-------------|
| `/api/guardian/health` | GET | Health check Guardian |
| `/api/guardian/debug/testEndpoint` | POST | Testa un endpoint API |
| `/api/guardian/logs` | GET | Recupera log agenti |
| `/api/guardian/permissions` | GET | Permessi agenti |
| `/api/logs/createLog` | POST | Crea nuovo log |
| `/api/logs/getLogs` | GET | Lista log con filtri |
| `/api/logs/stats` | GET | Statistiche log |

### Esempio Test Endpoint

```javascript
POST /api/guardian/debug/testEndpoint
{
  "serviceId": "test.api",
  "method": "GET",
  "path": "/api/health",
  "headers": {}
}

// Response
{
  "success": true,
  "request": { "method": "GET", "url": "...", "headers": {...} },
  "response": { "statusCode": 200, "durationMs": 42, "body": {...} }
}
```

---

## 💾 DATABASE E STORAGE

### Database Neon (PostgreSQL)

**Connection String:** Vedi variabile `DATABASE_URL` o `NEON_POSTGRES_URL`

### Tabelle Principali

| Tabella | Descrizione | Records (stima) |
|---------|-------------|-----------------|
| `markets` | Mercati | 542 |
| `vendors` | Operatori | ~2000 |
| `stalls` | Posteggi | ~5000 |
| `concessions` | Concessioni | ~3000 |
| `agent_messages` | Chat agenti | ~400 |
| `mio_agent_logs` | Log API | ~1200 |
| `suap_eventi` | Eventi SUAP | variabile |

### Storage S3

- **Provider:** Cloudflare R2 (compatibile S3)
- **Stato:** In configurazione
- **Uso:** Documenti, allegati, export

---

## 🔌 API ENDPOINTS — INVENTARIO COMPLETO

> **Ultimo scan:** 19/02/2026 21:22  
> **Backend:** `mihub-backend-rest` su Hetzner (157.90.29.66:3000)  
> **Base URL:** `https://mihub.157-90-29-66.nip.io` / `https://orchestratore.mio-hub.me`

### Sommario

| Metrica | Valore |
|---------|--------|
| **Endpoint totali** | 635 |
| **File route** | 70 |
| **GET** | 328 |
| **POST** | 214 |
| **PUT** | 48 |
| **DELETE** | 36 |
| **PATCH** | 9 |

### Risultati Test GET (produzione)

| Stato | Conteggio | Significato |
|-------|-----------|-------------|
| **200 OK** | 265 | Funzionanti |
| **400 Bad Request** | 12 | Parametri mancanti (atteso) |
| **401 Unauthorized** | 6 | Auth richiesta (atteso) |
| **404 Not Found** | 22 | Record test non esiste (atteso) |
| **500 Server Error** | 22 | Bug reali da investigare |
| **Timeout** | 1 | Connessione lenta/bloccata |

### Endpoint con errore 500 (da investigare)

| Endpoint | File | Note |
|----------|------|------|
| `/api/autorizzazioni/next-number` | autorizzazioni.js | Server error |
| `/api/bandi/matching/1` | bandi.js | Server error |
| `/api/documents` | documents.js | Server error |
| `/api/documents/1` | documents.js | Server error |
| `/api/documents/1/download` | documents.js | Server error |
| `/api/inspections/1` | inspections.js | Server error |
| `/api/mihub/chats/1` | chats.js | Server error |
| `/api/presenze/storico/dettaglio/1/1` | presenze.js | Server error |
| `/api/qualificazioni/durc/1` | qualificazioni.js | Server error |
| `/api/qualificazioni/suap/1` | qualificazioni.js | Server error |
| `/api/security/threats/alerts` | security.js | Server error |
| `/api/security/threats/patterns` | security.js | Server error |
| `/api/storico/dettaglio/1/1` | presenze.js | Server error |
| `/api/suap/documenti/1/download` | suap.js | Server error |
| `/api/suap/pratiche/1` | suap.js | Server error |
| `/api/suap/pratiche/1/azioni` | suap.js | Server error |
| `/api/suap/pratiche/1/checks` | suap.js | Server error |
| `/api/suap/pratiche/1/documenti` | suap.js | Server error |
| `/api/suap/pratiche/1/eventi` | suap.js | Server error |
| `/api/tcc/merchant/1/reimbursements` | tcc.js | Server error |
| `/api/tcc/v2/impresa/1/wallet/transactions` | tcc-v2.js | Server error |
| `/api/verbali/impresa/1` | verbali.js | Server error |

### Endpoint Timeout

| Endpoint | File | Note |
|----------|------|------|
| `/api/imprese` | imprese.js | Timeout/Unreachable |

### Inventario per Modulo

#### `abacusGithub.js` — 3 endpoint
**Mount:** `/api/abacus/github`

| Metodo | Path |
|--------|------|
| POST | `/api/abacus/github/list` |
| POST | `/api/abacus/github/get` |
| POST | `/api/abacus/github/update` |

#### `abacusSql.js` — 4 endpoint
**Mount:** `/api/abacus/sql`

| Metodo | Path |
|--------|------|
| POST | `/api/abacus/sql/query` |
| POST | `/api/abacus/sql/count` |
| GET | `/api/abacus/sql/tables` |
| POST | `/api/abacus/sql/schema` |

#### `admin.js` — 5 endpoint
**Mount:** `/admin`

| Metodo | Path |
|--------|------|
| POST | `/admin/secrets` |
| GET | `/admin/secrets` |
| DELETE | `/admin/secrets/:name` |
| POST | `/admin/secrets/init` |
| POST | `/admin/deploy-backend` |

#### `adminDeploy.js` — 2 endpoint
**Mount:** `/api/admin`

| Metodo | Path |
|--------|------|
| POST | `/api/admin/deploy-backend` |
| GET | `/api/admin/deploy-status` |

#### `adminMigrate.js` — 2 endpoint
**Mount:** `/api/admin`

| Metodo | Path |
|--------|------|
| POST | `/api/admin/migrate` |
| GET | `/api/admin/tables` |

#### `adminSecrets.js` — 3 endpoint
**Mount:** `/api/admin/secrets`

| Metodo | Path |
|--------|------|
| GET | `/api/admin/secrets/meta` |
| GET | `/api/admin/secrets/categories` |
| GET | `/api/admin/secrets/stats` |

#### `apiSecrets.js` — 3 endpoint
**Mount:** `/api/secrets`

| Metodo | Path |
|--------|------|
| POST | `/api/secrets/:key` |
| GET | `/api/secrets/:key` |
| DELETE | `/api/secrets/:key` |

#### `auth.js` — 9 endpoint
**Mount:** `/api/auth`

| Metodo | Path |
|--------|------|
| GET | `/api/auth/config` |
| GET | `/api/auth/login` |
| POST | `/api/auth/callback` |
| POST | `/api/auth/refresh` |
| POST | `/api/auth/logout` |
| GET | `/api/auth/me` |
| GET | `/api/auth/verify` |
| POST | `/api/auth/register` |
| POST | `/api/auth/login` |

#### `autorizzazioni.js` — 6 endpoint
**Mount:** `/api/autorizzazioni`

| Metodo | Path |
|--------|------|
| GET | `/api/autorizzazioni` |
| GET | `/api/autorizzazioni/:id` |
| POST | `/api/autorizzazioni` |
| PUT | `/api/autorizzazioni/:id` |
| DELETE | `/api/autorizzazioni/:id` |
| GET | `/api/autorizzazioni/next-number` |

#### `bandi.js` — 18 endpoint
**Mount:** `/api/bandi`

| Metodo | Path |
|--------|------|
| GET | `/api/bandi/associazioni` |
| GET | `/api/bandi/associazioni/:id` |
| POST | `/api/bandi/associazioni` |
| PUT | `/api/bandi/associazioni/:id` |
| GET | `/api/bandi/catalogo` |
| GET | `/api/bandi/catalogo/:id` |
| POST | `/api/bandi/catalogo` |
| GET | `/api/bandi/matching/:bando_id` |
| GET | `/api/bandi/servizi` |
| GET | `/api/bandi/servizi/categorie` |
| GET | `/api/bandi/richieste` |
| POST | `/api/bandi/richieste` |
| PUT | `/api/bandi/richieste/:id` |
| GET | `/api/bandi/richieste/stats` |
| GET | `/api/bandi/regolarita` |
| GET | `/api/bandi/regolarita/stats` |
| PUT | `/api/bandi/regolarita/:id` |
| GET | `/api/bandi/stats` |

#### `canone-unico.js` — 20 endpoint
**Mount:** `/api/canone-unico`

| Metodo | Path |
|--------|------|
| GET | `/api/canone-unico/riepilogo` |
| POST | `/api/canone-unico/genera-canone-annuo` |
| POST | `/api/canone-unico/genera-pagamento-straordinario` |
| POST | `/api/canone-unico/calcola-mora` |
| PUT | `/api/canone-unico/concessions/:id/status` |
| GET | `/api/canone-unico/impostazioni/:comune_id` |
| PUT | `/api/canone-unico/impostazioni/:comune_id` |
| GET | `/api/canone-unico/imprese-concessioni` |
| GET | `/api/canone-unico/semaforo-rate/:wallet_id/:anno` |
| GET | `/api/canone-unico/semaforo-impresa/:impresa_id/:anno` |
| DELETE | `/api/canone-unico/scadenze/:anno` |
| DELETE | `/api/canone-unico/scadenza/:id` |
| POST | `/api/canone-unico/wallet/:id/azzera` |
| GET | `/api/canone-unico/scadenza/:id` |
| POST | `/api/canone-unico/wallets/azzera-tutti` |
| GET | `/api/canone-unico/impostazioni-mora` |
| PUT | `/api/canone-unico/impostazioni-mora` |
| POST | `/api/canone-unico/aggiorna-mora` |
| GET | `/api/canone-unico/posteggi-mercato/:mercato_id` |
| GET | `/api/canone-unico/ricariche-spunta` |

#### `chats.js` — 3 endpoint
**Mount:** `/api/mihub/chats`

| Metodo | Path |
|--------|------|
| GET | `/api/mihub/chats/:room` |
| POST | `/api/mihub/chats/:room` |
| DELETE | `/api/mihub/chats/:room` |

#### `citizens.js` — 4 endpoint
**Mount:** `/api/citizens`

| Metodo | Path |
|--------|------|
| GET | `/api/citizens` |
| GET | `/api/citizens/:id` |
| GET | `/api/citizens/:id/transactions` |
| PATCH | `/api/citizens/:id/eco-credit` |

#### `civic-reports.js` — 11 endpoint
**Mount:** `/api/civic-reports`

| Metodo | Path |
|--------|------|
| GET | `/api/civic-reports` |
| POST | `/api/civic-reports` |
| GET | `/api/civic-reports/stats` |
| POST | `/api/civic-reports/fix-comune-ids` |
| GET | `/api/civic-reports/config` |
| PUT | `/api/civic-reports/config` |
| GET | `/api/civic-reports/:id` |
| PATCH | `/api/civic-reports/:id/status` |
| PATCH | `/api/civic-reports/:id/assign` |
| POST | `/api/civic-reports/:id/resolve` |
| POST | `/api/civic-reports/:id/link-sanction` |

#### `collaboratori.js` — 5 endpoint
**Mount:** `/api/collaboratori`

| Metodo | Path |
|--------|------|
| GET | `/api/collaboratori` |
| POST | `/api/collaboratori` |
| PUT | `/api/collaboratori/:id` |
| DELETE | `/api/collaboratori/:id` |
| PATCH | `/api/collaboratori/:id/toggle-presenze` |

#### `comuni.js` — 26 endpoint
**Mount:** `/api/comuni`

| Metodo | Path |
|--------|------|
| GET | `/api/comuni` |
| GET | `/api/comuni/:id` |
| POST | `/api/comuni` |
| PUT | `/api/comuni/:id` |
| DELETE | `/api/comuni/:id` |
| GET | `/api/comuni/settori/tipi` |
| GET | `/api/comuni/:id/settori` |
| POST | `/api/comuni/:id/settori` |
| PUT | `/api/comuni/settori/:settoreId` |
| DELETE | `/api/comuni/settori/:settoreId` |
| GET | `/api/comuni/:id/contratti` |
| POST | `/api/comuni/:id/contratti` |
| PUT | `/api/comuni/contratti/:contrattoId` |
| DELETE | `/api/comuni/contratti/:contrattoId` |
| GET | `/api/comuni/:id/fatture` |
| POST | `/api/comuni/:id/fatture` |
| PUT | `/api/comuni/fatture/:fatturaId` |
| GET | `/api/comuni/:id/utenti` |
| POST | `/api/comuni/:id/utenti` |
| PUT | `/api/comuni/utenti/:assegnazioneId` |
| DELETE | `/api/comuni/utenti/:assegnazioneId` |
| GET | `/api/comuni/:id/utenti/stats` |
| GET | `/api/comuni/:id/mercati` |
| GET | `/api/comuni/:id/hub` |
| POST | `/api/comuni/:id/provision-admin` |
| GET | `/api/comuni/:id/admin-credentials` |

#### `concessions.js` — 8 endpoint
**Mount:** `/api/concessions`

| Metodo | Path |
|--------|------|
| GET | `/api/concessions` |
| GET | `/api/concessions/:id` |
| POST | `/api/concessions` |
| POST | `/api/concessions/full` |
| PATCH | `/api/concessions/:id` |
| PUT | `/api/concessions/:id` |
| DELETE | `/api/concessions/:id` |
| POST | `/api/concessions/:id/associa-posteggio` |

#### `dms-legacy.js` — 24 endpoint
**Mount:** `/api/integrations/dms-legacy`

| Metodo | Path |
|--------|------|
| GET | `/api/integrations/dms-legacy/health` |
| GET | `/api/integrations/dms-legacy/status` |
| GET | `/api/integrations/dms-legacy/markets` |
| GET | `/api/integrations/dms-legacy/vendors` |
| GET | `/api/integrations/dms-legacy/concessions` |
| GET | `/api/integrations/dms-legacy/presences/:marketId` |
| GET | `/api/integrations/dms-legacy/market-sessions/:marketId` |
| GET | `/api/integrations/dms-legacy/stalls/:marketId` |
| GET | `/api/integrations/dms-legacy/spuntisti` |
| GET | `/api/integrations/dms-legacy/documents` |
| GET | `/api/integrations/dms-legacy/stats` |
| POST | `/api/integrations/dms-legacy/sync-out/vendor` |
| POST | `/api/integrations/dms-legacy/sync-out/market` |
| POST | `/api/integrations/dms-legacy/sync-out/stall` |
| POST | `/api/integrations/dms-legacy/sync-out/concession` |
| POST | `/api/integrations/dms-legacy/sync-out/spuntista` |
| POST | `/api/integrations/dms-legacy/sync-out/user` |
| POST | `/api/integrations/dms-legacy/sync-out/wallet` |
| POST | `/api/integrations/dms-legacy/sync-out/mkt-prezzo` |
| POST | `/api/integrations/dms-legacy/sync-out/all` |
| POST | `/api/integrations/dms-legacy/sync-in/presences/:marketId` |
| POST | `/api/integrations/dms-legacy/sync-in/sessions/:marketId` |
| POST | `/api/integrations/dms-legacy/sync` |
| POST | `/api/integrations/dms-legacy/cron-sync` |

#### `dmsHub.js` — 5 endpoint
**Mount:** `/api/dmsHub`

| Metodo | Path |
|--------|------|
| GET | `/api/dmsHub/markets/list` |
| GET | `/api/dmsHub/markets/getById` |
| GET | `/api/dmsHub/stalls/listByMarket` |
| GET | `/api/dmsHub/vendors/list` |
| GET | `/api/dmsHub/concessions/list` |

#### `documents.js` — 6 endpoint
**Mount:** `/api/documents`

| Metodo | Path |
|--------|------|
| POST | `/api/documents/upload` |
| GET | `/api/documents` |
| GET | `/api/documents/:id` |
| GET | `/api/documents/:id/download` |
| DELETE | `/api/documents/:id` |
| POST | `/api/documents/init-schema` |

#### `domande-spunta.js` — 9 endpoint
**Mount:** `/api/domande-spunta`

| Metodo | Path |
|--------|------|
| GET | `/api/domande-spunta` |
| GET | `/api/domande-spunta/:id` |
| POST | `/api/domande-spunta` |
| PUT | `/api/domande-spunta/:id` |
| DELETE | `/api/domande-spunta/:id` |
| POST | `/api/domande-spunta/:id/approva` |
| POST | `/api/domande-spunta/:id/rifiuta` |
| POST | `/api/domande-spunta/:id/revisione` |
| GET | `/api/domande-spunta/presenze/:impresa_id/:mercato_id` |

#### `formazione.js` — 17 endpoint
**Mount:** `/api/formazione`

| Metodo | Path |
|--------|------|
| GET | `/api/formazione/enti` |
| GET | `/api/formazione/enti/:id` |
| POST | `/api/formazione/enti` |
| PUT | `/api/formazione/enti/:id` |
| GET | `/api/formazione/corsi` |
| GET | `/api/formazione/corsi/:id` |
| POST | `/api/formazione/corsi` |
| GET | `/api/formazione/stats` |
| GET | `/api/formazione/imprese/search` |
| POST | `/api/formazione/attestati` |
| GET | `/api/formazione/attestati` |
| GET | `/api/formazione/iscrizioni` |
| POST | `/api/formazione/iscrizioni` |
| PUT | `/api/formazione/iscrizioni/:id` |
| DELETE | `/api/formazione/iscrizioni/:id` |
| GET | `/api/formazione/iscrizioni/stats` |
| GET | `/api/formazione/tipi-attestato` |

#### `gaming-rewards.js` — 32 endpoint
**Mount:** `/api/gaming-rewards`

| Metodo | Path |
|--------|------|
| GET | `/api/gaming-rewards/config` |
| GET | `/api/gaming-rewards/config/all` |
| POST | `/api/gaming-rewards/config` |
| PUT | `/api/gaming-rewards/config` |
| POST | `/api/gaming-rewards/mobility/start-tracking` |
| POST | `/api/gaming-rewards/mobility/complete` |
| POST | `/api/gaming-rewards/mobility/checkin` |
| GET | `/api/gaming-rewards/mobility/history` |
| GET | `/api/gaming-rewards/mobility/stats` |
| GET | `/api/gaming-rewards/mobility/heatmap` |
| POST | `/api/gaming-rewards/culture/checkin` |
| GET | `/api/gaming-rewards/culture/history` |
| GET | `/api/gaming-rewards/culture/stats` |
| GET | `/api/gaming-rewards/culture/heatmap` |
| GET | `/api/gaming-rewards/nearby-pois` |
| GET | `/api/gaming-rewards/top-shops` |
| GET | `/api/gaming-rewards/trend` |
| GET | `/api/gaming-rewards/stats` |
| GET | `/api/gaming-rewards/heatmap` |
| POST | `/api/gaming-rewards/referral/generate` |
| GET | `/api/gaming-rewards/referral/validate/:code` |
| POST | `/api/gaming-rewards/referral/register` |
| POST | `/api/gaming-rewards/referral/first-purchase` |
| GET | `/api/gaming-rewards/referral/stats/:user_id` |
| GET | `/api/gaming-rewards/referral/heatmap` |
| GET | `/api/gaming-rewards/referral/list` |
| GET | `/api/gaming-rewards/challenges` |
| POST | `/api/gaming-rewards/challenges` |
| PUT | `/api/gaming-rewards/challenges/:id` |
| DELETE | `/api/gaming-rewards/challenges/:id` |
| POST | `/api/gaming-rewards/challenges/:id/join` |
| POST | `/api/gaming-rewards/challenges/:id/progress` |

#### `gis.js` — 4 endpoint
**Mount:** `/api/gis`

| Metodo | Path |
|--------|------|
| GET | `/api/gis/health` |
| GET | `/api/gis/market-map` |
| GET | `/api/gis/market-map/:marketId` |
| POST | `/api/gis/import-market` |

#### `giustificazioni.js` — 7 endpoint
**Mount:** `/api/giustificazioni`

| Metodo | Path |
|--------|------|
| POST | `/api/giustificazioni` |
| GET | `/api/giustificazioni/file/:id` |
| GET | `/api/giustificazioni/impresa/:id` |
| GET | `/api/giustificazioni/pendenti` |
| GET | `/api/giustificazioni` |
| PUT | `/api/giustificazioni/:id/review` |
| DELETE | `/api/giustificazioni/:id` |

#### `gtfs.js` — 6 endpoint
**Mount:** `/api/gtfs`

| Metodo | Path |
|--------|------|
| GET | `/api/gtfs/stops` |
| GET | `/api/gtfs/stops/nearby` |
| GET | `/api/gtfs/routes` |
| GET | `/api/gtfs/stats` |
| POST | `/api/gtfs/stops/bulk` |
| POST | `/api/gtfs/routes/bulk` |

#### `guardian.js` — 4 endpoint
**Mount:** `/api/guardian`

| Metodo | Path |
|--------|------|
| POST | `/api/guardian/debug/testEndpoint` |
| GET | `/api/guardian/health` |
| GET | `/api/guardian/logs` |
| GET | `/api/guardian/permissions` |

#### `guardianSync.js` — 1 endpoint
**Mount:** `/api/mihub`

| Metodo | Path |
|--------|------|
| POST | `/api/mihub/sync-blueprint` |

#### `health-monitor.js` — 6 endpoint
**Mount:** `/api/health`

| Metodo | Path |
|--------|------|
| GET | `/api/health` |
| GET | `/api/health/full` |
| GET | `/api/health/history` |
| GET | `/api/health/service/:name` |
| GET | `/api/health/alerts` |
| GET | `/api/health/config` |

#### `hub.js` — 10 endpoint
**Mount:** `/api/hub`

| Metodo | Path |
|--------|------|
| GET | `/api/hub/locations` |
| GET | `/api/hub/locations/:id` |
| POST | `/api/hub/locations` |
| PUT | `/api/hub/locations/:id` |
| DELETE | `/api/hub/locations/:id` |
| GET | `/api/hub/shops` |
| POST | `/api/hub/shops` |
| POST | `/api/hub/shops/create-with-impresa` |
| GET | `/api/hub/services` |
| POST | `/api/hub/services` |

#### `imprese.js` — 13 endpoint
**Mount:** `/api/imprese`

| Metodo | Path |
|--------|------|
| GET | `/api/imprese` |
| GET | `/api/imprese/:id` |
| POST | `/api/imprese` |
| PUT | `/api/imprese/:id` |
| DELETE | `/api/imprese/:id` |
| GET | `/api/imprese/:id/rating` |
| PUT | `/api/imprese/:id/vetrina` |
| POST | `/api/imprese/:id/vetrina/upload` |
| DELETE | `/api/imprese/:id/vetrina/gallery/:index` |
| GET | `/api/imprese/:id/qualificazioni` |
| POST | `/api/imprese/:id/qualificazioni` |
| PUT | `/api/imprese/:id/qualificazioni/:qualId` |
| DELETE | `/api/imprese/:id/qualificazioni/:qualId` |

#### `index.js` — 2 endpoint
**Mount:** `/health`

| Metodo | Path |
|--------|------|
| GET | `/health` |
| GET | `/` |

#### `inspections.js` — 6 endpoint
**Mount:** `/api/inspections`

| Metodo | Path |
|--------|------|
| GET | `/api/inspections` |
| GET | `/api/inspections/stats` |
| POST | `/api/inspections` |
| GET | `/api/inspections/:id` |
| PUT | `/api/inspections/:id` |
| DELETE | `/api/inspections/:id` |

#### `integrations.js` — 3 endpoint
**Mount:** `/api/dashboard/integrations`

| Metodo | Path |
|--------|------|
| GET | `/api/dashboard/integrations` |
| GET | `/api/dashboard/integrations/endpoint-count` |
| GET | `/api/dashboard/integrations/status` |

#### `ipa.js` — 6 endpoint
**Mount:** `/api/ipa`

| Metodo | Path |
|--------|------|
| GET | `/api/ipa/status` |
| GET | `/api/ipa/search` |
| GET | `/api/ipa/ente/:codice_ipa` |
| POST | `/api/ipa/import` |
| GET | `/api/ipa/uo/:codice_ipa` |
| GET | `/api/ipa/tipologie` |

#### `logs.js` — 7 endpoint
**Mount:** `/api/logs`

| Metodo | Path |
|--------|------|
| POST | `/api/logs/recreateSchema` |
| POST | `/api/logs/initSchema` |
| POST | `/api/logs/createLog` |
| GET | `/api/logs/getLogs` |
| GET | `/api/logs/guardian` |
| GET | `/api/logs/stats` |
| DELETE | `/api/logs/clear` |

#### `market-settings.js` — 10 endpoint
**Mount:** `/api/market-settings`

| Metodo | Path |
|--------|------|
| GET | `/api/market-settings/:marketId` |
| POST | `/api/market-settings/:marketId` |
| GET | `/api/market-settings/transgressions/list` |
| GET | `/api/market-settings/transgressions/pending-justifications` |
| POST | `/api/market-settings/transgressions/:id/justify` |
| PUT | `/api/market-settings/transgressions/:id/review` |
| PUT | `/api/market-settings/transgressions/:id/archive` |
| POST | `/api/market-settings/transgressions/:id/sanction` |
| GET | `/api/market-settings/transgressions/stats` |
| POST | `/api/market-settings/cron/run` |

#### `markets.js` — 10 endpoint
**Mount:** `/api/markets`

| Metodo | Path |
|--------|------|
| GET | `/api/markets` |
| GET | `/api/markets/:id` |
| GET | `/api/markets/:id/stalls` |
| POST | `/api/markets` |
| PATCH | `/api/markets/:id` |
| DELETE | `/api/markets/:id` |
| GET | `/api/markets/:marketId/companies` |
| POST | `/api/markets/:marketId/companies` |
| GET | `/api/markets/:marketId/concessions` |
| POST | `/api/markets/:marketId/concessions` |

#### `mercaweb.js` — 9 endpoint
**Mount:** `/api/integrations/mercaweb`

| Metodo | Path |
|--------|------|
| GET | `/api/integrations/mercaweb/health` |
| GET | `/api/integrations/mercaweb/status` |
| POST | `/api/integrations/mercaweb/import/ambulanti` |
| POST | `/api/integrations/mercaweb/import/mercati` |
| POST | `/api/integrations/mercaweb/import/piazzole` |
| POST | `/api/integrations/mercaweb/import/concessioni` |
| POST | `/api/integrations/mercaweb/import/spuntisti` |
| GET | `/api/integrations/mercaweb/export/presenze/:marketId` |
| GET | `/api/integrations/mercaweb/export/mapping/:entity` |

#### `migratePDND.js` — 1 endpoint
**Mount:** `/api/admin`

| Metodo | Path |
|--------|------|
| POST | `/api/admin/migrate-pdnd` |

#### `mihub.js` — 7 endpoint
**Mount:** `/api/mihub`

| Metodo | Path |
|--------|------|
| GET | `/api/mihub/tasks` |
| POST | `/api/mihub/tasks` |
| POST | `/api/mihub/deploy/vercel` |
| GET | `/api/mihub/secrets-meta` |
| POST | `/api/mihub/secrets/:envVar` |
| GET | `/api/mihub/logs` |
| GET | `/api/mihub/direct-messages/:conversationId` |

#### `mioAgent.js` — 2 endpoint
**Mount:** `/api/mio`

| Metodo | Path |
|--------|------|
| GET | `/api/mio/agent-logs` |
| POST | `/api/mio/agent-logs` |

#### `notifiche.js` — 15 endpoint
**Mount:** `/api/notifiche`

| Metodo | Path |
|--------|------|
| POST | `/api/notifiche/send` |
| GET | `/api/notifiche/impresa/:id` |
| GET | `/api/notifiche/conversazione/:id` |
| PUT | `/api/notifiche/leggi/:id` |
| POST | `/api/notifiche/reply` |
| GET | `/api/notifiche/stats` |
| GET | `/api/notifiche/targets` |
| DELETE | `/api/notifiche/:id` |
| PUT | `/api/notifiche/archivia/:id` |
| GET | `/api/notifiche/risposte` |
| GET | `/api/notifiche/markets` |
| GET | `/api/notifiche/imprese` |
| GET | `/api/notifiche/messaggi/:mittente_tipo/:mittente_id` |
| PUT | `/api/notifiche/segna-letto/:id` |
| PUT | `/api/notifiche/risposte/:id/letta` |

#### `orchestrator.js` — 9 endpoint
**Mount:** `/api/mihub`

| Metodo | Path |
|--------|------|
| POST | `/api/mihub/orchestrator` |
| GET | `/api/mihub/orchestrator/conversations` |
| GET | `/api/mihub/orchestrator/conversations/:id` |
| POST | `/api/mihub/orchestrator/init` |
| POST | `/api/mihub/webhook/email` |
| GET | `/api/mihub/mail/inbox` |
| GET | `/api/mihub/notifications/sent` |
| GET | `/api/mihub/notifications/all` |
| POST | `/api/mihub/notifications/send` |

#### `orchestratorMock.js` — 1 endpoint
**Mount:** `/api/mihub`

| Metodo | Path |
|--------|------|
| POST | `/api/mihub/orchestrator/mock` |

#### `presenze.js` — 34 endpoint
**Mount:** `/api/presenze`

| Metodo | Path |
|--------|------|
| GET | `/api/presenze/mercato/:id` |
| POST | `/api/presenze/registra` |
| POST | `/api/presenze/checkout` |
| GET | `/api/presenze/graduatoria/mercato/:id` |
| PUT | `/api/presenze/graduatoria/aggiorna` |
| PUT | `/api/presenze/graduatoria/assenze` |
| POST | `/api/presenze/graduatoria/ricalcola-posizioni` |
| POST | `/api/presenze/graduatoria/crea` |
| POST | `/api/presenze/graduatoria/aggiorna-storico` |
| GET | `/api/presenze/spuntisti/mercato/:id` |
| POST | `/api/presenze/registra-uscita` |
| GET | `/api/presenze/storico/sessioni` |
| GET | `/api/presenze/storico/dettaglio/:marketId/:data` |
| POST | `/api/presenze/mercato/:id/chiudi` |
| GET | `/api/presenze/sessioni/:id/dettaglio` |
| GET | `/api/presenze/sessioni` |
| GET | `/api/presenze/impresa/:id` |
| GET | `/api/mercato/:id` |
| POST | `/api/registra` |
| POST | `/api/checkout` |
| GET | `/api/graduatoria/mercato/:id` |
| PUT | `/api/graduatoria/aggiorna` |
| PUT | `/api/graduatoria/assenze` |
| POST | `/api/graduatoria/ricalcola-posizioni` |
| POST | `/api/graduatoria/crea` |
| POST | `/api/graduatoria/aggiorna-storico` |
| GET | `/api/spuntisti/mercato/:id` |
| POST | `/api/registra-uscita` |
| GET | `/api/storico/sessioni` |
| GET | `/api/storico/dettaglio/:marketId/:data` |
| POST | `/api/mercato/:id/chiudi` |
| GET | `/api/sessioni/:id/dettaglio` |
| GET | `/api/sessioni` |
| GET | `/api/impresa/:id` |

#### `public-search.js` — 1 endpoint
**Mount:** `/api/public`

| Metodo | Path |
|--------|------|
| GET | `/api/public/search` |

#### `qualificazioni.js` — 7 endpoint
**Mount:** `/api/qualificazioni`

| Metodo | Path |
|--------|------|
| GET | `/api/qualificazioni` |
| GET | `/api/qualificazioni/impresa/:impresa_id` |
| GET | `/api/qualificazioni/durc/:impresa_id` |
| POST | `/api/qualificazioni/durc` |
| GET | `/api/qualificazioni/suap/:impresa_id` |
| POST | `/api/qualificazioni/suap` |
| PUT | `/api/qualificazioni/suap/:id` |

#### `regioni.js` — 5 endpoint
**Mount:** `/api/regioni`

| Metodo | Path |
|--------|------|
| GET | `/api/regioni` |
| GET | `/api/regioni/:id` |
| GET | `/api/regioni/:id/province` |
| GET | `/api/regioni/province/all` |
| POST | `/api/regioni/migrate` |

#### `routing.js` — 2 endpoint
**Mount:** `/api/routing`

| Metodo | Path |
|--------|------|
| POST | `/api/routing/calculate` |
| GET | `/api/routing/tpl-stops` |

#### `sanctions.js` — 9 endpoint
**Mount:** `/api/sanctions`

| Metodo | Path |
|--------|------|
| GET | `/api/sanctions` |
| GET | `/api/sanctions/types` |
| POST | `/api/sanctions` |
| GET | `/api/sanctions/riepilogo-pagamenti` |
| GET | `/api/sanctions/:id` |
| POST | `/api/sanctions/:id/pay` |
| PUT | `/api/sanctions/:id` |
| GET | `/api/sanctions/impresa/:id/da-pagare` |
| POST | `/api/sanctions/:id/paga-pagopa` |

#### `security.js` — 64 endpoint
**Mount:** `/api/security`

| Metodo | Path |
|--------|------|
| GET | `/api/security/stats` |
| GET | `/api/security/users` |
| GET | `/api/security/users/:id` |
| POST | `/api/security/users` |
| PUT | `/api/security/users/:id` |
| DELETE | `/api/security/users/:id` |
| POST | `/api/security/users/:id/lock` |
| POST | `/api/security/users/:id/unlock` |
| POST | `/api/security/users/:id/reset-password` |
| POST | `/api/security/users/:id/impersonate` |
| GET | `/api/security/users/:id/roles` |
| GET | `/api/security/roles` |
| GET | `/api/security/roles/:id` |
| POST | `/api/security/roles` |
| PUT | `/api/security/roles/:id` |
| DELETE | `/api/security/roles/:id` |
| POST | `/api/security/roles/:id/assign` |
| POST | `/api/security/roles/:id/revoke` |
| GET | `/api/security/roles/:id/permissions` |
| GET | `/api/security/permissions` |
| POST | `/api/security/permissions` |
| POST | `/api/security/permissions/:code/assign` |
| GET | `/api/security/matrix` |
| GET | `/api/security/access-logs` |
| GET | `/api/security/events` |
| POST | `/api/security/events` |
| GET | `/api/security/login-attempts` |
| GET | `/api/security/ip-blacklist` |
| POST | `/api/security/ip-blacklist` |
| DELETE | `/api/security/ip-blacklist/:ip` |
| GET | `/api/security/audit` |
| GET | `/api/security/audit/stats` |
| GET | `/api/security/audit/export` |
| GET | `/api/security/audit/by-entity/:type/:id` |
| GET | `/api/security/audit/by-user/:email` |
| GET | `/api/security/audit/gdpr-report/:userId` |
| GET | `/api/security/audit/:id` |
| GET | `/api/security/compliance/status` |
| GET | `/api/security/compliance/certificates` |
| GET | `/api/security/compliance/report` |
| GET | `/api/security/compliance/gdpr-requests` |
| POST | `/api/security/compliance/gdpr-export/:userId` |
| POST | `/api/security/compliance/gdpr-delete/:userId` |
| GET | `/api/security/delegations` |
| GET | `/api/security/delegations/:id` |
| POST | `/api/security/delegations` |
| POST | `/api/security/delegations/:id/revoke` |
| GET | `/api/security/delegations/user/:userId` |
| GET | `/api/security/threats/dashboard` |
| GET | `/api/security/threats/alerts` |
| POST | `/api/security/threats/alerts/:id/acknowledge` |
| GET | `/api/security/threats/patterns` |
| GET | `/api/security/threats/report` |
| POST | `/api/security/threats/scan` |
| GET | `/api/security/threats/recommendations` |
| GET | `/api/security/health` |
| GET | `/api/security/sessions` |
| GET | `/api/security/sessions/stats` |
| GET | `/api/security/sessions/:id` |
| DELETE | `/api/security/sessions/:id` |
| DELETE | `/api/security/sessions/user/:userId` |
| PUT | `/api/security/roles/:id/permissions` |
| GET | `/api/security/permissions/tabs` |
| GET | `/api/security/permissions/quick-access` |

#### `stalls.js` — 6 endpoint
**Mount:** `/api/stalls`

| Metodo | Path |
|--------|------|
| GET | `/api/stalls` |
| GET | `/api/stalls/:id` |
| PATCH | `/api/stalls/:id` |
| POST | `/api/stalls` |
| DELETE | `/api/stalls/:id` |
| GET | `/api/stalls/stats/totals` |

#### `stats-qualificazione.js` — 6 endpoint
**Mount:** `/api/stats/qualificazione`

| Metodo | Path |
|--------|------|
| GET | `/api/stats/qualificazione/overview` |
| GET | `/api/stats/qualificazione/scadenze` |
| GET | `/api/stats/qualificazione/demografia` |
| GET | `/api/stats/qualificazione/top-imprese` |
| GET | `/api/stats/qualificazione/indici` |
| GET | `/api/stats/qualificazione/all` |

#### `stats.js` — 4 endpoint
**Mount:** `/api/stats`

| Metodo | Path |
|--------|------|
| GET | `/api/stats/overview` |
| GET | `/api/stats/realtime` |
| GET | `/api/stats/inspections` |
| GET | `/api/stats/growth` |

#### `suap.js` — 20 endpoint
**Mount:** `/api/suap`

| Metodo | Path |
|--------|------|
| GET | `/api/suap/stats` |
| GET | `/api/suap/pratiche` |
| GET | `/api/suap/pratiche/:id` |
| POST | `/api/suap/pratiche` |
| POST | `/api/suap/pratiche/:id/valuta` |
| POST | `/api/suap/import` |
| POST | `/api/suap/pratiche/:id/refresh` |
| POST | `/api/suap/pratiche/:id/stato` |
| POST | `/api/suap/pratiche/:id/documenti` |
| GET | `/api/suap/pratiche/:id/documenti` |
| GET | `/api/suap/documenti/:docId/download` |
| GET | `/api/suap/pratiche/:id/eventi` |
| POST | `/api/suap/pratiche/:id/checks` |
| GET | `/api/suap/pratiche/:id/checks` |
| POST | `/api/suap/pratiche/:id/azioni` |
| GET | `/api/suap/pratiche/:id/azioni` |
| GET | `/api/suap/regole` |
| PUT | `/api/suap/regole/:checkCode` |
| PATCH | `/api/suap/pratiche/:id` |
| GET | `/api/suap/notifiche-pm` |

#### `system.js` — 2 endpoint
**Mount:** `/api/system`

| Metodo | Path |
|--------|------|
| GET | `/api/system/health` |
| GET | `/api/system/pm2-status` |

#### `tariffs.js` — 2 endpoint
**Mount:** `/api/tariffs`

| Metodo | Path |
|--------|------|
| GET | `/api/tariffs/:marketId` |
| POST | `/api/tariffs` |

#### `tcc-v2.js` — 32 endpoint
**Mount:** `/api/tcc/v2`

| Metodo | Path |
|--------|------|
| POST | `/api/tcc/v2/generate-spend-qr` |
| GET | `/api/tcc/v2/operator/wallet/:operatorId` |
| POST | `/api/tcc/v2/operator/issue` |
| POST | `/api/tcc/v2/operator/validate-spend-qr` |
| POST | `/api/tcc/v2/operator/redeem-spend` |
| POST | `/api/tcc/v2/operator/settlement` |
| GET | `/api/tcc/v2/operator/transactions/:operatorId` |
| PUT | `/api/tcc/v2/config/update` |
| GET | `/api/tcc/v2/config` |
| GET | `/api/tcc/v2/fund/stats` |
| GET | `/api/tcc/v2/fund/transactions` |
| GET | `/api/tcc/v2/comuni` |
| GET | `/api/tcc/v2/rules` |
| POST | `/api/tcc/v2/rules` |
| PUT | `/api/tcc/v2/rules/:id` |
| DELETE | `/api/tcc/v2/rules/:id` |
| GET | `/api/tcc/v2/fund/stats/:hubId` |
| PUT | `/api/tcc/v2/config/update/:hubId` |
| GET | `/api/tcc/v2/environment/:hubId` |
| PUT | `/api/tcc/v2/ets-price` |
| POST | `/api/tcc/v2/process-batch-reimbursements` |
| GET | `/api/tcc/v2/pending-reimbursements` |
| GET | `/api/tcc/v2/reimbursement-history` |
| GET | `/api/tcc/v2/fund-status` |
| POST | `/api/tcc/v2/admin/reset-test-data` |
| GET | `/api/tcc/v2/impresa/:impresaId/wallet` |
| POST | `/api/tcc/v2/impresa/:impresaId/wallet/create` |
| GET | `/api/tcc/v2/impresa/:impresaId/qualification-status` |
| PUT | `/api/tcc/v2/impresa/:impresaId/wallet/status` |
| GET | `/api/tcc/v2/impresa/:impresaId/wallet/transactions` |
| POST | `/api/tcc/v2/impresa/:impresaId/wallet/sync-qualification` |
| GET | `/api/tcc/v2/wallets/all` |

#### `tcc.js` — 15 endpoint
**Mount:** `/api/tcc`

| Metodo | Path |
|--------|------|
| GET | `/api/tcc/wallet/:userId` |
| GET | `/api/tcc/wallet/:userId/transactions` |
| POST | `/api/tcc/earn` |
| POST | `/api/tcc/spend` |
| GET | `/api/tcc/fund` |
| GET | `/api/tcc/config` |
| GET | `/api/tcc/stats` |
| POST | `/api/tcc/merchant/redeem` |
| GET | `/api/tcc/qrcode/:userId` |
| POST | `/api/tcc/scan` |
| POST | `/api/tcc/validate-qr` |
| GET | `/api/tcc/merchant/:shopId` |
| POST | `/api/tcc/merchant/redeem` |
| GET | `/api/tcc/merchant/:shopId/reimbursements` |
| GET | `/api/tcc/merchants` |

#### `test-mercato.js` — 10 endpoint
**Mount:** `/api/test-mercato`

| Metodo | Path |
|--------|------|
| POST | `/api/test-mercato/start` |
| POST | `/api/test-mercato/registra-presenza-concessionario` |
| POST | `/api/test-mercato/avvia-spunta` |
| POST | `/api/test-mercato/assegna-posteggio-spunta` |
| GET | `/api/test-mercato/stato/:market_id` |
| POST | `/api/test-mercato/chiudi-spunta` |
| POST | `/api/test-mercato/reset` |
| POST | `/api/test-mercato/registra-rifiuti` |
| POST | `/api/test-mercato/chiudi-mercato` |
| POST | `/api/test-mercato/inizia-mercato` |

#### `vendors.js` — 5 endpoint
**Mount:** `/api/vendors`

| Metodo | Path |
|--------|------|
| GET | `/api/vendors` |
| GET | `/api/vendors/:id` |
| POST | `/api/vendors` |
| PATCH | `/api/vendors/:id` |
| DELETE | `/api/vendors/:id` |

#### `verbali.js` — 7 endpoint
**Mount:** `/api/verbali`

| Metodo | Path |
|--------|------|
| GET | `/api/verbali/config` |
| GET | `/api/verbali/infrazioni` |
| GET | `/api/verbali/impresa/:id` |
| POST | `/api/verbali` |
| GET | `/api/verbali/:id` |
| GET | `/api/verbali/:id/pdf` |
| POST | `/api/verbali/:id/invia` |

#### `wallet-history.js` — 5 endpoint
**Mount:** `/api/wallet-history`

| Metodo | Path |
|--------|------|
| GET | `/api/wallet-history` |
| GET | `/api/wallet-history/rimborsi-pendenti` |
| GET | `/api/wallet-history/:walletId` |
| GET | `/api/wallet-history/:walletId/balance-history` |
| POST | `/api/wallet-history` |

#### `wallet-scadenze.js` — 7 endpoint
**Mount:** `/api/wallet-scadenze`

| Metodo | Path |
|--------|------|
| GET | `/api/wallet-scadenze` |
| GET | `/api/wallet-scadenze/scadute` |
| GET | `/api/wallet-scadenze/:walletId` |
| POST | `/api/wallet-scadenze` |
| POST | `/api/wallet-scadenze/:id/calcola-mora` |
| POST | `/api/wallet-scadenze/:id/paga` |
| POST | `/api/wallet-scadenze/aggiorna-ritardi` |

#### `wallets.js` — 8 endpoint
**Mount:** `/api/wallets`

| Metodo | Path |
|--------|------|
| GET | `/api/wallets` |
| POST | `/api/wallets/calculate-annual-fee` |
| GET | `/api/wallets/company/:companyId` |
| GET | `/api/wallets/:id/transactions` |
| POST | `/api/wallets/deposit` |
| POST | `/api/wallets/withdraw` |
| POST | `/api/wallets/init` |
| DELETE | `/api/wallets/:id` |

#### `watchlist.js` — 6 endpoint
**Mount:** `/api/watchlist`

| Metodo | Path |
|--------|------|
| GET | `/api/watchlist` |
| GET | `/api/watchlist/stats` |
| POST | `/api/watchlist` |
| PUT | `/api/watchlist/:id` |
| DELETE | `/api/watchlist/:id` |
| POST | `/api/watchlist/webhook/suap` |

#### `webhook.js` — 3 endpoint
**Mount:** `/webhook`

| Metodo | Path |
|--------|------|
| POST | `/webhook/deploy-mihub` |
| GET | `/webhook/status` |
| POST | `/webhook/force-deploy-emergency` |

#### `webhooks.js` — 5 endpoint
**Mount:** `/api/hooks`

| Metodo | Path |
|--------|------|
| POST | `/api/hooks/zapier/new-concession` |
| POST | `/api/hooks/zapier/stall-status-changed` |
| GET | `/api/hooks/zapier/test` |
| POST | `/api/hooks/github/deploy` |
| GET | `/api/hooks/github/deploy/status` |

#### `workspace.js` — 6 endpoint
**Mount:** `/api/workspace`

| Metodo | Path |
|--------|------|
| POST | `/api/workspace/save` |
| GET | `/api/workspace/load` |
| GET | `/api/workspace/list` |
| DELETE | `/api/workspace/delete` |
| POST | `/api/workspace/add-shape` |
| POST | `/api/workspace/add-image` |

---

## 🚀 DEPLOY E CI/CD

### ⚠️ REGOLA FONDAMENTALE

```
╔═══════════════════════════════════════════════════════════════════╗
║  NON FARE MAI SSH MANUALE PER DEPLOY!                             ║
║  Il sistema è AUTO-DEPLOY tramite GitHub Actions                  ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Flusso Deploy

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   COMMIT    │────►│    PUSH     │────►│   GITHUB    │────►│   DEPLOY    │
│   locale    │     │   GitHub    │     │   Actions   │     │ automatico  │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                              │
                    ┌─────────────────────────┼─────────────────────────┐
                    │                         │                         │
                    ▼                         ▼                         ▼
            ┌─────────────┐           ┌─────────────┐           ┌─────────────┐
            │   VERCEL    │           │   HETZNER   │           │    NEON     │
            │  Frontend   │           │   Backend   │           │  Database   │
            │  (auto)     │           │  (webhook)  │           │  (migrate)  │
            └─────────────┘           └─────────────┘           └─────────────┘
```

### Procedura Corretta

```bash
# 1. Modifica codice
# 2. Commit
git add .
git commit -m "feat: descrizione modifica"

# 3. Push (triggera auto-deploy)
git push origin master

# 4. Verifica (dopo 2-3 minuti)
curl https://orchestratore.mio-hub.me/api/health
```

---

## 🔐 CREDENZIALI E ACCESSI

### Variabili d'Ambiente Backend

| Variabile | Descrizione |
|-----------|-------------|
| `DATABASE_URL` | Connection string Neon |
| `GEMINI_API_KEY` | API key Google Gemini |
| `GITHUB_TOKEN` | Token GitHub per GPT Dev |
| `SSH_PRIVATE_KEY` | Chiave SSH per Manus |
| `ZAPIER_WEBHOOK_URL` | Webhook Zapier |
| `VERCEL_TOKEN` | Token deploy Vercel |

### Accessi Server

| Risorsa | Accesso |
|---------|---------|
| **Hetzner VPS** | SSH con chiave (solo per emergenze) |
| **Neon Dashboard** | https://console.neon.tech |
| **Vercel Dashboard** | https://vercel.com/dashboard |
| **GitHub** | https://github.com/Chcndr |

---

## 🔧 TROUBLESHOOTING

### Health Monitor mostra servizi Offline

| Servizio | Problema | Soluzione |
|----------|----------|-----------|
| Guardian | Era configurato su URL esterno inesistente | ✅ Fixato v2.1.0 - ora check interno |
| MIO Agent | Era configurato su URL esterno inesistente | ✅ Fixato v2.1.0 - ora check interno |
| S3 | Non configurato | Configurare quando necessario |
| PDND | Non configurato | Normale - per uso futuro |

### Configurazione CORS e Sicurezza

**File:** `mihub-backend-rest/index.js`

**Origini CORS Permesse:**
```javascript
const allowedOrigins = [
  'https://mio-hub.me',
  'https://www.mio-hub.me',
  'https://dms-hub-app-new.vercel.app',
  'https://orchestratore.mio-hub.me',
  'https://api.mio-hub.me',
  'https://ms-hub-app-new.vercel.app',
  'https://dms-cittadini.vercel.app',
  'https://dms-operatore.vercel.app',
  'http://localhost:5173',
  'http://localhost:3000'
];
```

**Configurazione Helmet:**
- `frameguard: false` - Disabilitato per permettere iframe da Vercel
- `frameAncestors` nel CSP permette embedding da domini autorizzati

**⚠️ IMPORTANTE:** Se aggiungi un nuovo dominio frontend:
1. Aggiungerlo a `allowedOrigins` in `index.js`
2. Aggiungerlo a `frameAncestors` nella configurazione helmet
3. Riavviare PM2: `pm2 restart mihub-backend`

### Backend non risponde

```bash
# Verifica stato PM2 (solo emergenza)
ssh user@157.90.29.66 "pm2 status"

# Riavvia (solo emergenza)
ssh user@157.90.29.66 "pm2 restart mihub-backend"
```

### Frontend non si aggiorna

1. Verifica deploy Vercel: https://vercel.com/dashboard
2. Controlla build logs
3. Forza rebuild: push commit vuoto

---

## 🤖 REGOLE PER AGENTI AI

### ❌ NON FARE MAI

1. **NON** fare SSH manuale per deploy
2. **NON** modificare file direttamente sul server
3. **NON** creare nuovi repository paralleli
4. **NON** hardcodare URL endpoint nel frontend
5. **NON** modificare senza leggere prima questo Blueprint

### ✅ FARE SEMPRE

1. **LEGGERE** questo Blueprint prima di ogni modifica
2. **USARE** git commit + push per deploy
3. **VERIFICARE** api-index.json per endpoint
4. **TESTARE** con /api/health/full dopo modifiche
5. **DOCUMENTARE** ogni modifica significativa

### Checklist Pre-Modifica

- [ ] Ho letto il Blueprint?
- [ ] Ho verificato l'architettura esistente?
- [ ] Sto usando i repository corretti?
- [ ] Il mio deploy usa git push (non SSH)?
- [ ] Ho aggiornato la documentazione?

---

## 📊 STATO ATTUALE SISTEMA

### Servizi Online ✅

| Servizio | URL | Stato |
|----------|-----|-------|
| Frontend | https://dms-hub-app-new.vercel.app | ✅ Online |
| Backend | https://orchestratore.mio-hub.me | ✅ Online |
| Database | Neon PostgreSQL | ✅ Online |
| MIO Agent | /api/mihub/orchestrator | ✅ Funzionante |
| Guardian | /api/guardian/* | ✅ Funzionante |

### Statistiche

- **Endpoint totali:** 635
- **Mercati nel DB:** 542
- **Log totali:** 1232
- **Agenti attivi:** 5 (MIO, GPT Dev, Manus, Abacus, Zapier)
- **Secrets configurati:** 10/10

---

## 📚 DOCUMENTAZIONE CORRELATA

Questo Blueprint unificato si integra con la documentazione esistente nel repository:

### LIVE_SYSTEM_DEC2025/

Documentazione del sistema funzionante in produzione:

| Cartella | Contenuto |
|----------|----------|
| `01_ARCHITECTURE/` | Architettura "8 Isole", flusso dati, deployment |
| `02_BACKEND_CORE/` | API map, LLM Engine, sistema tools |
| `03_DATABASE_SCHEMA/` | Schema PostgreSQL, query, migrazioni |
| `04_FRONTEND_DASHBOARD/` | 27 tabs dashboard, componenti, state management |

### 00_LEGACY_ARCHIVE/

Archivio storico con 87 documenti Markdown:

| Cartella | Contenuto |
|----------|----------|
| `01_architettura/` | MASTER_SYSTEM_PLAN, AS-IS/TO-BE, integrazioni |
| `01_architettura/legacy/` | Documentazione teorica vecchia |
| `01_architettura/legacy/root_legacy/` | CREDENZIALI, BACKEND_UFFICIALE, GIS_SYSTEM |
| `07_guide_operative/` | Guide deploy e troubleshooting |

### ROADMAP_2025/

Piano sviluppo organizzato per quarter:

| Quarter | Obiettivi Principali |
|---------|---------------------|
| **Q1 2025** | TAB Clienti/Prodotti, PDND, performance <2s |
| **Q2 2025** | TAB Sostenibilità/TPAS, IoT, 1000+ utenti |
| **Q3-Q4 2025** | Carbon Credits blockchain, TPER, 10.000+ utenti |

---

## 📌 TODO - CONFIGURAZIONI FUTURE

### ⚠️ S3 Storage (Cloudflare R2) - DA CONFIGURARE

**Stato:** Predisposto ma NON ancora configurato

**Cosa serve:** Storage esterno per documenti, allegati, file di grandi dimensioni.

**Quando configurare:** Quando il sistema inizierà a gestire molti documenti/allegati.

**Come configurare:**
1. Creare account Cloudflare (se non esiste)
2. Andare su Cloudflare Dashboard → R2 Object Storage
3. Creare un bucket (es. "miohub-documents")
4. Generare API Token con permessi R2
5. Configurare le variabili d'ambiente su Hetzner:
   ```bash
   ssh root@157.90.29.66
   nano /root/mihub-backend-rest/.env
   # Aggiungere:
   R2_ACCOUNT_ID=xxx
   R2_ACCESS_KEY_ID=xxx
   R2_SECRET_ACCESS_KEY=xxx
   R2_BUCKET_NAME=miohub-documents
   ```
6. Riavviare PM2: `pm2 restart mihub-backend`

**Endpoint già pronti:**
- `POST /api/documents/upload` - Upload file
- `GET /api/documents/:id` - Download file  
- `GET /api/documents` - Lista documenti
- `DELETE /api/documents/:id` - Elimina documento

**Tabella database:** `documents` (già creata in Neon)

---

### ⚠️ PDND API - DA CONFIGURARE

**Stato:** Predisposto ma NON ancora configurato

**Cosa serve:** Interoperabilità con Piattaforma Digitale Nazionale Dati (PagoPA)

**Quando configurare:** Quando servirà integrazione con altri sistemi PA.

---

## 📝 CHANGELOG

### v3.5.13 (06/01/2026 20:50) - "Fix CORS e Slot Editor V3"

- 🐛 **Bug Fix CORS - Richieste API da iframe:**
  - Problema: "Load failed" quando Slot Editor (in iframe da Vercel) chiamava api.mio-hub.me
  - Causa 1: `X-Frame-Options: SAMEORIGIN` bloccava l'iframe
  - Causa 2: CORS non restituiva `Access-Control-Allow-Origin` per origini nella lista
  - Soluzione:
    - Aggiunto `frameguard: false` alla configurazione helmet
    - Riscritto middleware CORS con funzione callback per restituire sempre l'origine

- 🐛 **Bug Fix Slot Editor V3:**
  - Corretto errore sintassi JavaScript (mancava chiusura `}` nel catch)
  - Popup HUB Area: aggiunto pulsante ✕ per chiudere
  - Controlli colore/spessore/trasparenza: usato addEventListener invece di onchange inline
  - Cancella Tutti i Negozi: usato addEventListener invece di onclick
  - Reset Pianta: aggiunta cancellazione di `dms_editor_autosave`
  - Aggiunta colonna `area_sqm` alla tabella `hub_locations` per superficie area

- 📁 **File Modificati:**
  - `mihub-backend-rest/index.js` - Configurazione CORS e helmet
  - `mihub-backend-rest/src/routes/hub.js` - Aggiunto campo areaSqm
  - `mihub-backend-rest/public/tools/slot_editor_v3_unified.html` - Correzioni varie

- 🔧 **Commits GitHub:**
  - `7e81008` - fix: corretto CORS per restituire Access-Control-Allow-Origin
  - `f1fdaba` - fix: corretto errore sintassi JavaScript
  - `0d60b74` - feat: aggiunto areaSqm per superficie area HUB

### v3.5.11 (05/01/2026 23:15) - "Fix Ricaricamento Dati GIS al Cambio Mercato"

- 🐛 **Bug Fix - Dati GIS non ricaricati al cambio mercato via marker:**
  - Problema: Cliccando su un marker M, i dati GIS (poligoni posteggi) non venivano ricaricati
  - Causa: Lo stato loading/mapData non veniva resettato quando cambiava marketId
  - Soluzione in PosteggiTabPubblica:
    - Reset loading = true (mostra spinner durante caricamento)
    - Reset mapData = null (forza ricaricamento GIS)
    - Reset selectedStallId = null (deseleziona posteggio)
  - Soluzione in MarketDetailPubblica:
    - Reset stalls = [] (svuota lista posteggi)
  - Ora i dati vengono ricaricati correttamente quando si clicca su un marker M

- 📁 **File Modificati:**
  - `dms-hub-app-new/client/src/components/MappaItaliaPubblica.tsx`
    - useEffect in PosteggiTabPubblica: aggiunto reset stato al cambio marketId
    - useEffect in MarketDetailPubblica: aggiunto reset stalls al cambio market.id

- 🔧 **Commit GitHub:** 04f32ec
- 🚀 **Deploy:** Auto-deploy Vercel completato

### v3.5.10 (05/01/2026 22:30) - "Bug Fix Mappa Italia Pubblica"

- 🐛 **Bug Fix #1 - Selezione Mercato:**
  - Problema: Click su marker "Vista Mercato" zoomava sul mercato sbagliato
  - Causa: onMarketClick non propagava l'ID del mercato cliccato al componente padre
  - Soluzione: Aggiunto callback onMarketChange che propaga l'ID attraverso la catena:
    - MappaItaliaPubblica → MarketDetailPubblica → PosteggiTabPubblica → MarketMapComponent
  - Ora il click su un marker aggiorna correttamente selectedMarket prima dello zoom

- 🐛 **Bug Fix #2 - Macchie Verdi durante Zoom:**
  - Problema: Poligoni verdi (posteggi) apparivano durante la transizione zoom Italia→Mercato
  - Causa: Condizione showItalyView cambiava prima che l'animazione fosse completata
  - Soluzione: Aggiunto controllo isAnimating dal AnimationContext
  - I poligoni ora appaiono solo DOPO il completamento dell'animazione di zoom

- 📁 **File Modificati:**
  - `dms-hub-app-new/client/src/components/MappaItaliaPubblica.tsx`
    - Aggiunto prop onMarketChange a MarketDetailPubblica
    - Aggiunto prop onMarketChange a PosteggiTabPubblica
    - Aggiornato onMarketClick per chiamare onMarketChange
  - `dms-hub-app-new/client/src/components/MarketMapComponent.tsx`
    - Importato useAnimation da AnimationContext
    - Aggiunto isAnimating alla condizione di rendering poligoni

- 🔧 **Commit GitHub:** 0aef28a
- 🚀 **Deploy:** Auto-deploy Vercel attivato

### v3.5.7 (05/01/2026 18:30) - "Gestione Hub e Mappa Italia Pubblica"

- 🆕 **Nuovo Componente GestioneHubPanel.tsx:**
  - Cabina di regia per Hub Urbani e di Prossimità
  - 6 sub-tab: Cruscotto, Rete Hub, Imprese, EcoCarbon, Comunicazione, Report ESG
  - KPI real-time: Hub Attivi, Imprese Aderenti, Flussi Mensili, Crediti Emessi, Rating ESG
  - Mappa Hub Territoriali con marker interattivi
  - Alert & Notifiche: scadenze concessioni, report, adesioni, eventi
  - Sezione "Hub Attivi nel Territorio" con card dettaglio
  - Integrato nella DashboardPA al posto del placeholder

- 🆕 **Nuovo Componente MappaItaliaPubblica.tsx:**
  - Versione pubblica semplificata di MappaItaliaComponent
  - RIMOSSO: Tab Anagrafica (dati sensibili mercato)
  - RIMOSSO: Tab Imprese/Concessioni (dati sensibili imprese)
  - RIMOSSO: Editing posteggi (solo visualizzazione)
  - RIMOSSO: Scheda impresa dettagliata
  - RIMOSSO: Pulsante Spunta per assegnazioni
  - MANTENUTO: Mappa interattiva con zoom Italia/Mercato
  - MANTENUTO: Lista mercati con ricerca
  - MANTENUTO: Statistiche posteggi (occupati/liberi/riservati)
  - MANTENUTO: Lista posteggi in sola lettura

- 📁 **File Creati:**
  - `dms-hub-app-new/client/src/components/GestioneHubPanel.tsx` - Pannello Gestione Hub
  - `dms-hub-app-new/client/src/components/MappaItaliaPubblica.tsx` - Mappa pubblica

- 📁 **File Modificati:**
  - `dms-hub-app-new/client/src/pages/DashboardPA.tsx` - Integrato GestioneHubPanel
  - `dms-hub-app-new/client/src/pages/MappaItaliaPage.tsx` - Usa MappaItaliaPubblica

- 🔒 **Sicurezza:**
  - Separazione netta tra vista admin (MappaItaliaComponent) e vista pubblica (MappaItaliaPubblica)
  - Utenti pubblici non possono accedere a dati sensibili di imprese e concessioni

### v3.5.6 (03/01/2026 12:45) - "Sincronizzazione Liste Concessioni e Vista Statica"
- 🔄 **Sincronizzazione Liste Concessioni:**
  - Liste concessioni sincronizzate tra SSO SUAP e Gestione Mercati
  - Entrambe le liste leggono dalla stessa tabella `concessions`
  - Aggiunto `stato_calcolato` dinamico: SCADUTA se `valid_to < oggi`
  - Semafori ora mostrano correttamente lo stato reale delle concessioni
- 📝 **Vista Statica Concessione (stile SCIA):**
  - Nuova vista dettaglio concessione identica alla SCIA generata
  - Sezioni: Frontespizio, Concessionario, Dati Posteggio e Mercato
  - Card con sfondo scuro e bordi colorati (teal/cyan)
  - Layout a 3 colonne con label uppercase grigie e valori bianchi
  - Vista inline nella pagina (non più modale popup)
- 🆕 **Nuovo Componente:**
  - `ConcessionForm.tsx` - Form separato per modifica concessione
- 📁 **File Modificati:**
  - `mihub-backend-rest/routes/markets.js` - Endpoint con stato_calcolato
  - `mihub-backend-rest/routes/concessions.js` - Calcolo dinamico stato
  - `dms-hub-app-new/client/src/components/markets/MarketCompaniesTab.tsx` - Vista statica
  - `dms-hub-app-new/client/src/components/markets/ConcessionForm.tsx` - Nuovo form

### v3.5.5 (03/01/2026 07:15) - "API Associazione Posteggio Subingresso"
- 🆕 **Nuovo Endpoint Backend:**
  - `POST /api/concessions/:id/associa-posteggio` - Trasferimento posteggio per subingresso
  - Disassocia posteggio dal cedente e associa al subentrante
  - Chiude concessione cedente (stato → CESSATA)
  - Trasferisce wallet CONCESSION dal cedente al subentrante
  - Attiva nuova concessione (stato → ATTIVA)
- 📁 **File Modificati:**
  - `mihub-backend-rest/routes/concessions.js` (linee 734-917)
- 🔄 **Frontend già pronto:**
  - Tab "Aggiorna Posteggi" in modale dettaglio concessione
  - Pulsante "Associa Posteggio" per concessioni subingresso DA_ASSOCIARE
  - File: `dms-hub-app-new/client/src/components/suap/SuapPanel.tsx` (linee 1276-1425)

### v3.5.4 (03/01/2026) - "Auto-compilazione Concessione da SCIA"
- 📝 **Form Concessione:**
  - Auto-selezione Mercato quando si genera da SCIA
  - Auto-selezione Posteggio quando si genera da SCIA
  - Auto-selezione Merceologia quando si genera da SCIA
  - Aggiunta P.IVA e Provincia al preData dalla SCIA
  - Select Mercato e Posteggio ora mostrano il valore selezionato
- 🔄 **Logica:**
  - Bypass dei selettori quando i dati arrivano dalla SCIA
  - Tutti i campi si auto-compilano senza intervento manuale

### v3.5.3 (03/01/2026) - "Vista Dettaglio SCIA e PEC Delegato"
- 📝 **Vista Dettaglio SCIA:**
  - P.IVA e CF ora separati per Subentrante e Cedente
  - Rimossi "(Cessionario)" e "(Dante Causa)" dai titoli sezioni
  - Aggiunto campo PEC nella sezione Delegato
- 📝 **Form Compilazione SCIA:**
  - Aggiunto campo "PEC Delegato" (obbligatorio quando c'è delegato)
- 🗄️ **Database:**
  - Aggiunta colonna `del_pec` alla tabella `suap_pratiche`
- 🔄 **Backend:**
  - Query INSERT aggiornata per salvare PEC delegato

### v3.5.2 (03/01/2026) - "Campi P.IVA Separati in SCIA e Concessione"
- 🗄️ **Database:**
  - Aggiunte colonne `sub_partita_iva` e `ced_partita_iva` alla tabella `suap_pratiche`
  - Migrazione 015 eseguita su Neon PostgreSQL
- 📝 **Form SCIA:**
  - Nuova struttura a 4 colonne: Cerca Impresa | P.IVA | CF | Ragione Sociale
  - Campi P.IVA e CF ora separati (prima era campo unico)
  - Autocomplete mostra P.IVA e CF separatamente nel dropdown
- 📝 **Form Concessione:**
  - Stessa struttura a 4 colonne per Concessionario e Cedente
  - Campi auto-popolati dopo selezione impresa
- 🔄 **Backend:**
  - Service SUAP aggiornato per salvare `sub_partita_iva` e `ced_partita_iva`
  - Mappatura frontend→backend aggiornata in `handleSciaSubmit`

### v3.5.1 (03/01/2026) - "Progetto Sistema Concessioni Completo v2"
- 📋 **Analisi concessione reale Comune di Bologna:**
  - Documento 2 pagine: Autorizzazione + Concessione Suolo Pubblico
  - Identificati 56 campi totali, 25 implementati, 31 mancanti
- 🆕 **5 TIPI DI CONCESSIONE SUPPORTATI:**
  - **Nuova Autorizzazione** - Prima concessione per posteggio libero
  - **Subingresso** - Trasferimento da cedente a subentrante (con SCIA)
  - **Conversione** - 3 sottotipi: Tipo B→A, Merceologia, Dimensioni
  - **Rinnovo** - Rinnovo concessione in scadenza
  - **Voltura** - Cambio dati anagrafici/impresa
- 📝 **Form dinamico:**
  - Selettore tipo con radio buttons
  - Campi condizionali che appaiono/scompaiono per tipo
  - Auto-popolamento da CF e concessione precedente
- 🚦 **Logica semaforo per tipo:**
  - Nuova/Conversione/Rinnovo/Voltura: 🟢 Associato (automatico)
  - Subingresso: 🔴 Da Associare → 🟢 Associato (manuale)
- 🔄 **Tab "Aggiorna Posteggi":**
  - Visibile SOLO per subingresso con stato "da_associare"
  - Trasferisce posteggio + wallet da cedente a subentrante
- 🗄️ **Database:**
  - Tabella `concessioni` con campi per tutti i tipi
  - Enum: tipo_concessione, sottotipo_conversione, stato_concessione
- 📄 **Documentazione:**
  - PROGETTO_SISTEMA_CONCESSIONI_v2.md
  - TIPI_CONCESSIONE_COMPLETI.md
  - Piano implementazione 17-24 giorni

### v3.4.0 (02/01/2026) - "Bug Fix Completo Sistema Verifiche"
- ✅ **Date picker nei modal:** Fix chiusura automatica con stopPropagation su tutti gli input date
- ✅ **Toggle storico verifiche SCIA:** 
  - Mostra "Ultima Verifica" o "Storico Completo"
  - Punteggio affidabilità calcolato solo su ultima verifica
- ✅ **Form SCIA mercato/posteggio:** Ora usa CEDENTE invece di subentrante per filtrare
- ✅ **Semaforo qualificazioni:** Calcolo dinamico stato da data_scadenza (ignora stato DB obsoleto)
- ✅ **Tag stabile:** v1.6.0-stable creato come punto di ripristino
- Sistema verifica SCIA completamente funzionante

### v3.3.0 (02/01/2026) - "Sistema Qualifiche e Concessioni"
- ✅ **Modal Nuova Concessione migliorato:**
  - Campo "Mercato" preselezionato in sola visualizzazione
  - Dropdown "Posteggio" filtra solo posteggi liberi (senza concessione attiva o scaduta)
  - Conteggio posteggi liberi/totali visualizzato
  - Ordinamento numerico dei posteggi
- ✅ **Fix filtro posteggi:** Conversione stall_id a stringa per confronto corretto
- ✅ **Tag stabile:** v1.5.0-stable creato come punto di ripristino
- Sistema pronto per test qualifiche SCIA

### v3.2.0 (30/12/2025) - "Collaudo MIO Agent Completo"
- ✅ **Collaudo completo MIO Agent** - Tutti gli agenti testati e funzionanti
- ✅ **Fix orchestratorClient.ts** - Gestione errori non-JSON (rate limiting, timeout, server non disponibile)
- ✅ **Fix duplicati frontend** - Sistema "fingerprint" anti-duplicati nel polling
- ✅ **Fix sezione Attività Agenti** - Ripristinata sezione che carica da `agent_messages` invece di `guardian_logs`
- ✅ **Fix ordinamento messaggi** - Parametro `order=desc` in get-messages.ts per messaggi recenti
- ✅ **Test completati:** MIO coordinamento multi-agente, Zapier, GPT Dev, Abacus, Manus
- Sistema operativo all'85%+

### v3.5.7 (05/01/2026)
- **VETRINE - Design Migliorato:**
  - Hero Section con gradient overlay e info overlay (nome, settore, rating)
  - Effetto zoom al hover sull'immagine principale
  - Card Info Negozio con design moderno (descrizione quote, badge gradient, contatti in card)
  - Social Media con pulsanti colorati e gradient (Facebook blu, Instagram multicolor, WhatsApp verde)
  - Fix Instagram: ora accetta @username, URL completo, o solo username
  - Fix Facebook: aggiunge automaticamente https://facebook.com/ se necessario
  - Gallery con effetti hover, overlay gradient e badge numerati
  - Pulsanti azione più grandi con effetti hover
- **VETRINE - Fix Navigazione:**
  - Freccia indietro ora usa `window.history.back()` per tornare alla pagina precedente
- **VETRINE - Fix Dialog:**
  - Risolto bug del Dialog che mostrava contenuto fuori dal modal
  - Aggiunto reset stati form quando cambia impresa selezionata

### v3.1.0 (30/12/2025)
- **FIX Health Monitor:** Guardian e MIO Agent ora verificati come moduli interni (non più URL esterni inesistenti)
- **FIX API Logger:** Corretto middleware per catturare `req.originalUrl` invece di `req.path` (che viene modificato dai router Express)
- **Imprese API Logs:** Ora le chiamate a `/api/imprese/*` vengono loggate correttamente con `service_id: imprese.api`
- **Dipendenze installate:** `@aws-sdk/client-s3`, `@aws-sdk/s3-request-presigner`, `adm-zip`, `xml2js`
- Aggiornato middleware `apiLogger.js` v1.1.0

### v3.0.0 (30/12/2025)
- Creato Blueprint unificato
- Documentata architettura completa
- Chiarito che Guardian e MIO Agent sono moduli interni
- Fixato Health Monitor (v2.1.0)
- Integrato riferimenti a documentazione legacy

### v2.2.0 (21/12/2025)
- Fix duplicazione messaggi chat singole
- Fix visualizzazione risposte agenti
- Nuovi conversation_id (`user-{agent}-direct`)
- Sistema Doppio Canale FRONTSTAGE/BACKSTAGE

### v2.1.0 (12/12/2025)
- Documentazione LIVE_SYSTEM_DEC2025 completa
- ROADMAP_2025 organizzata per quarter
- Endpoint `/api/guardian/logs` per dashboard
- Riorganizzazione completa repository

### v2.0.0 (11/12/2025) - "Operazione Specchio Reale"
- Separazione documentazione legacy da sistema live
- Implementato Health Monitor
- Aggiunto sistema logging Guardian
- Integrazione completa MIO Agent

---

## 🔗 LINK RAPIDI

### Produzione
- **Dashboard PA:** https://dms-hub-app-new.vercel.app/dashboard-pa
- **Backend API:** https://orchestratore.mio-hub.me
- **Health Check:** https://orchestratore.mio-hub.me/api/health/full

### Repository GitHub
- **Frontend:** https://github.com/Chcndr/dms-hub-app-new
- **Backend:** https://github.com/Chcndr/mihub-backend-rest
- **Blueprint:** https://github.com/Chcndr/dms-system-blueprint

### Documentazione Esterna
- **PDND:** https://docs.pdnd.italia.it
- **Neon PostgreSQL:** https://neon.tech/docs
- **Google Gemini:** https://ai.google.dev/docs

---

---

## 📌 REGOLE FONDAMENTALI PER AGENTI AI

### 🚨 REGOLA #1: UN SOLO BLUEPRINT

**File unico di riferimento:** `MIOHUB_BLUEPRINT_UNIFICATO.md`

- **SEMPRE** aggiornare SOLO questo file
- **MAI** creare file di documentazione duplicati
- **MAI** modificare altri file di documentazione senza aggiornare questo
- Prima di ogni modifica al sistema, leggere questo file
- Dopo ogni modifica al sistema, aggiornare il CHANGELOG di questo file

### 🚨 REGOLA #2: WORKFLOW MODIFICHE

1. Leggere il Blueprint prima di iniziare
2. Fare le modifiche al codice
3. Testare le modifiche
4. Aggiornare il CHANGELOG del Blueprint
5. Committare Blueprint insieme al codice
6. **MAI** lasciare il Blueprint non aggiornato

### 🚨 REGOLA #3: DOVE SONO I FILE

| Cosa | Dove |
|------|------|
| Blueprint UNICO | `dms-system-blueprint/MIOHUB_BLUEPRINT_UNIFICATO.md` |
| Frontend | `dms-hub-app-new/` |
| Backend | `mihub-backend-rest/` |
| Credenziali | File locale utente (MAI su GitHub) |

### 🚨 REGOLA #4: PRIMA DI MODIFICARE

```
✅ Ho letto il Blueprint?
✅ So quale file modificare?
✅ Ho verificato che non creo duplicati?
✅ Aggiornerò il CHANGELOG dopo?
```

---

> **Nota:** Questo documento è la **UNICA fonte di verità** per il sistema MIO HUB.
> Ogni agente AI **DEVE** leggerlo prima di effettuare modifiche.
> **NON esistono altri file di documentazione validi.**

### v3.5.8 (05/01/2026 19:15) - "GestioneHub con Dati Reali"

- 🔄 **GestioneHubPanel.tsx aggiornato:**
  - KPI calcolati dinamicamente dalle API reali
  - Endpoint utilizzati: /api/markets, /api/vendors, /api/stalls, /api/concessions
  - Lista Hub basata sui mercati reali con statistiche posteggi
  - Alert dinamici: concessioni in scadenza, tasso occupazione
  - Pulsante "Aggiorna" per refresh dati in tempo reale
  - Tab Report ESG con indicatori Environmental/Social/Governance
  - Riepilogo dati sistema con contatori reali


### v3.5.9 (05/01/2026 19:45) - "Gestione Hub Completa"

#### 🎯 SEZIONE GESTIONE HUB - RIEPILOGO COMPLETO

La sezione "Gestione Hub" è ora completamente operativa nella Dashboard PA.
Fornisce una "Cabina di Regia Territoriale" per stakeholder (Associazioni, Cluster, Regione).

**Struttura a 6 Tab:**

| Tab | Componente | Endpoint API | Funzionalità |
|-----|------------|--------------|--------------|
| **Cruscotto** | Custom | /api/markets, /api/vendors, /api/stalls, /api/concessions | KPI real-time, Alert dinamici, Lista Hub |
| **Rete Hub** | MappaItaliaComponent | /api/markets/{id}/stalls, /api/gis/market-map | Mappa interattiva mercati e posteggi |
| **Imprese** | ImpreseQualificazioniPanel | /api/vendors, /api/imprese | Gestione e qualificazione imprese |
| **EcoCarbon** | WalletPanel | /api/trpc/wallet.* | Sistema crediti carbonio e wallet |
| **Comunicazione** | NotificationsPanel | /api/notifications | Email, WhatsApp, campagne |
| **Report ESG** | Custom | Dati calcolati | Indicatori Environmental/Social/Governance |

**KPI Dinamici nel Cruscotto:**
- Hub Attivi (conteggio mercati)
- Imprese Aderenti (conteggio vendors)
- Posteggi Totali (conteggio stalls)
- Concessioni Attive (conteggio concessions)
- Tasso Occupazione (calcolato)

**Alert Automatici:**
- Concessioni in scadenza (entro 30 giorni)
- Report mensile disponibile
- Numero imprese attive
- Prossimo mercato
- Tasso occupazione posteggi

**File Coinvolti:**
- `client/src/components/GestioneHubPanel.tsx` - Componente principale (v2.0)
- `client/src/pages/DashboardPA.tsx` - Integrazione nella dashboard

**Commit GitHub:** 2dec709



### v3.5.12 (06/01/2026 20:00) - "Slot Editor V3 e API HUB"

#### 🛠️ TOOLS - SLOT EDITOR V3

Lo **Slot Editor V3** è uno strumento per creare e gestire aree HUB con negozi sulla mappa.

**URL:** `https://orchestratore.mio-hub.me/tools/slot_editor_v3_unified.html`

**Funzionalità:**
| Funzione | Descrizione |
|----------|-------------|
| **Disegna Area HUB** | Crea poligono dell'area sulla mappa |
| **Imposta Centro HUB** | Marker per il centro dell'HUB |
| **Aggiungi Negozi** | Marker con lettera (A, B, C...) per ogni negozio |
| **Personalizza Stile** | Colore, trasparenza, spessore bordo dell'area |
| **Esporta GeoJSON** | Scarica file JSON con tutti i dati |
| **Salva nel Database** | Salva direttamente su api.mio-hub.me |

**Pulsanti Esporta:**
| Pulsante | API | Descrizione |
|----------|-----|-------------|
| **Esporta GeoJSON** | - | Download file JSON locale |
| **Esporta per Dashboard Admin** | POST /api/markets | Crea mercato con posteggi |
| **Salva nel Bus** | DMSBUS | Salva su BUS Hub (richiede connessione) |
| **Salva nel Database (Pepe GIS)** | POST /api/hub/locations | Crea HUB con negozi |

---

#### 🗄️ DATABASE - SCHEMA HUB

**Tabella `hub_locations`:**
```sql
CREATE TABLE hub_locations (
  id SERIAL PRIMARY KEY,
  market_id INTEGER REFERENCES markets(id),
  name VARCHAR(255) NOT NULL,
  address VARCHAR(255) NOT NULL,
  city VARCHAR(100) NOT NULL,
  lat DECIMAL(10,8) NOT NULL,
  lng DECIMAL(11,8) NOT NULL,
  center_lat DECIMAL(10,8),
  center_lng DECIMAL(11,8),
  area_geojson JSONB,
  corner_geojson JSONB,
  opening_hours VARCHAR(255),
  active INTEGER DEFAULT 1,
  is_independent INTEGER DEFAULT 0,
  description TEXT,
  photo_url VARCHAR(500),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Tabella `hub_shops`:**
```sql
CREATE TABLE hub_shops (
  id SERIAL PRIMARY KEY,
  hub_id INTEGER REFERENCES hub_locations(id),
  shop_number INTEGER,
  letter VARCHAR(5),
  name VARCHAR(255),
  category VARCHAR(100),
  certifications TEXT,
  owner_id INTEGER,
  business_name VARCHAR(255),
  vat_number VARCHAR(20),
  phone VARCHAR(50),
  email VARCHAR(255),
  vetrina_url VARCHAR(500),
  lat DECIMAL(10,8),
  lng DECIMAL(11,8),
  area_mq DECIMAL(10,2),
  status VARCHAR(50) DEFAULT 'active',
  opening_hours VARCHAR(255),
  description TEXT,
  photo_url VARCHAR(500),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

---

#### 📡 API ENDPOINTS - HUB

**Base URL:** `https://api.mio-hub.me/api/hub`

| Metodo | Endpoint | Descrizione |
|--------|----------|-------------|
| GET | /locations | Lista tutti gli HUB |
| GET | /locations/:id | Dettaglio HUB con negozi |
| POST | /locations | Crea nuovo HUB |
| PATCH | /locations/:id | Aggiorna HUB |
| DELETE | /locations/:id | Elimina HUB |
| POST | /locations/:hubId/shops | Aggiungi negozio a HUB |
| PATCH | /shops/:id | Aggiorna negozio |
| DELETE | /shops/:id | Elimina negozio |

**Esempio POST /api/hub/locations:**
```json
{
  "name": "Hub Centro Grosseto",
  "address": "Via Roma 1",
  "city": "Grosseto",
  "lat": 42.7589,
  "lng": 11.1135,
  "areaGeojson": {
    "type": "Polygon",
    "coordinates": [[[11.11, 42.75], [11.12, 42.76], ...]]
  },
  "isIndependent": true,
  "description": "HUB commerciale centro città",
  "shops": [
    {
      "shopNumber": 1,
      "letter": "A",
      "name": "Bar Roma",
      "category": "bar",
      "lat": 42.7590,
      "lng": 11.1136
    }
  ]
}
```

**Risposta:**
```json
{
  "success": true,
  "hubId": 3,
  "shopsCreated": 1,
  "message": "HUB \"Hub Centro Grosseto\" creato con successo"
}
```

---

#### 📡 API ENDPOINTS - MERCATI

**Base URL:** `https://api.mio-hub.me/api/markets`

| Metodo | Endpoint | Descrizione |
|--------|----------|-------------|
| GET | / | Lista tutti i mercati |
| GET | /:id | Dettaglio mercato |
| GET | /:id/stalls | Posteggi del mercato |
| POST | / | Crea nuovo mercato |
| PATCH | /:id | Aggiorna mercato |
| DELETE | /:id | Elimina mercato |

**Esempio POST /api/markets:**
```json
{
  "code": "GR001",
  "name": "Mercato Centro Grosseto",
  "municipality": "Grosseto",
  "days": "Lunedì, Giovedì",
  "total_stalls": 50,
  "latitude": 42.7589,
  "longitude": 11.1135
}
```

---

#### 🔄 DIFFERENZA HUB vs MERCATI

| Aspetto | HUB | Mercati |
|---------|-----|---------|
| **Tabelle** | hub_locations, hub_shops | markets, stalls |
| **Uso** | Centri commerciali, zone negozi fissi | Mercati ambulanti periodici |
| **Elementi** | Negozi (shops) | Posteggi (stalls) |
| **Indipendenza** | Può essere indipendente | Sempre legato a comune |
| **API** | /api/hub/locations | /api/markets |

---

**File Modificati:**
- `mihub-backend-rest/public/tools/slot_editor_v3_unified.html`
- `mihub-backend-rest/routes/hub.js`

**Commit GitHub:** 5afb2d0



---

### v3.5.13 (06/01/2026 23:30) - "Sistema HUB Completo con Mappa Interattiva"

#### 🎯 PANORAMICA SISTEMA HUB

Il sistema HUB è stato completamente implementato con:
- **12 HUB Market** per le principali città italiane
- **Mappa interattiva** con Vista Italia ↔ Vista HUB
- **Visualizzazione negozi** come punti/marker sulla mappa
- **Selettore Mercato/HUB** per switchare tra le due modalità

---

#### 🗺️ COMPONENTI FRONTEND

**File Principali:**

| Componente | Path | Descrizione |
|------------|------|-------------|
| `HubMarketMapComponent` | `client/src/components/HubMarketMapComponent.tsx` | Clone di MarketMapComponent con supporto HUB |
| `GestioneHubMapWrapper` | `client/src/components/GestioneHubMapWrapper.tsx` | Wrapper con selettore Mercato/HUB |
| `GestioneHubPanel` | `client/src/components/GestioneHubPanel.tsx` | Pannello principale Gestione Hub Territoriale |

**HubMarketMapComponent - Caratteristiche:**

1. **Vista Italia** - Zoom 6, centro [42.5, 12.5]
   - Marker "M" rossi per i Mercati
   - Marker "H" viola per gli HUB

2. **Vista HUB** - Zoom 16, centro su HUB selezionato
   - Poligono area HUB (viola tratteggiato)
   - Marker negozi con lettera (A, B, C...)
   - Colori negozi: verde=attivo, grigio=inattivo, rosso=chiuso

3. **Animazioni**
   - `flyTo` con durata dinamica basata su differenza zoom
   - Fine corsa basato su `area_geojson` (hubBounds)

**Props HubMarketMapComponent:**

```typescript
interface HubMarketMapComponentProps {
  // Props base mappa
  mapData?: MapData;
  center?: [number, number];
  zoom?: number;
  height?: string;
  
  // Props Vista Italia
  allMarkets?: Market[];
  allHubs?: HubLocation[];
  showItalyView?: boolean;
  viewTrigger?: number;
  
  // Props HUB
  mode?: 'mercato' | 'hub';
  selectedHub?: HubLocation;
  hubCenterFixed?: [number, number];
  onHubClick?: (hubId: number) => void;
  onShopClick?: (shop: HubShop) => void;
}
```

---

#### 📡 API ENDPOINTS - HUB

**Base URL:** `https://api.mio-hub.me/api/hub`

| Metodo | Endpoint | Descrizione | Risposta |
|--------|----------|-------------|----------|
| GET | /locations | Lista tutti gli HUB | `{success, data: HubLocation[], count}` |
| GET | /locations/:id | Dettaglio HUB con negozi | `{success, data: HubLocation}` |
| POST | /locations | Crea nuovo HUB | `{success, hubId, shopsCreated, message}` |
| PATCH | /locations/:id | Aggiorna HUB | `{success, message}` |
| DELETE | /locations/:id | Elimina HUB | `{success, message}` |
| POST | /locations/:hubId/shops | Aggiungi negozio | `{success, shopId}` |
| PATCH | /shops/:id | Aggiorna negozio | `{success, message}` |
| DELETE | /shops/:id | Elimina negozio | `{success, message}` |

**Esempio GET /api/hub/locations:**

```json
{
  "success": true,
  "data": [
    {
      "id": 7,
      "name": "Hub Centro",
      "address": "Via del Centro",
      "city": "Grosseto",
      "lat": "42.7589",
      "lng": "11.1135",
      "area_geojson": "{\"type\":\"Polygon\",\"coordinates\":[...]}",
      "shops": [
        {
          "id": 1,
          "letter": "A",
          "name": "Bar Roma",
          "lat": "42.7590",
          "lng": "11.1136",
          "status": "active"
        }
      ]
    }
  ],
  "count": 12
}
```

---

#### 🗄️ DATABASE - SCHEMA HUB

**Tabella `hub_locations`:**

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| id | SERIAL | Primary key |
| market_id | INTEGER | FK a markets (opzionale) |
| name | VARCHAR(255) | Nome HUB |
| address | VARCHAR(255) | Indirizzo |
| city | VARCHAR(100) | Città |
| lat | DECIMAL(10,8) | Latitudine |
| lng | DECIMAL(11,8) | Longitudine |
| center_lat | DECIMAL(10,8) | Latitudine centro |
| center_lng | DECIMAL(11,8) | Longitudine centro |
| area_geojson | JSONB | Poligono area GeoJSON |
| corner_geojson | JSONB | Corner points (fine corsa) |
| opening_hours | VARCHAR(255) | Orari apertura |
| active | INTEGER | 1=attivo, 0=inattivo |
| is_independent | INTEGER | 1=indipendente, 0=legato a mercato |
| description | TEXT | Descrizione |
| photo_url | VARCHAR(500) | URL foto |
| area_sqm | DECIMAL(10,2) | Superficie in mq |
| created_at | TIMESTAMP | Data creazione |
| updated_at | TIMESTAMP | Data aggiornamento |

**Tabella `hub_shops`:**

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| id | SERIAL | Primary key |
| hub_id | INTEGER | FK a hub_locations |
| shop_number | INTEGER | Numero negozio |
| letter | VARCHAR(5) | Lettera (A, B, C...) |
| name | VARCHAR(255) | Nome negozio |
| category | VARCHAR(100) | Categoria |
| lat | DECIMAL(10,8) | Latitudine |
| lng | DECIMAL(11,8) | Longitudine |
| status | VARCHAR(50) | active/inactive/closed |
| phone | VARCHAR(50) | Telefono |
| email | VARCHAR(255) | Email |
| vetrina_url | VARCHAR(500) | URL vetrina |
| description | TEXT | Descrizione |
| created_at | TIMESTAMP | Data creazione |
| updated_at | TIMESTAMP | Data aggiornamento |

---

#### 🏙️ HUB MARKET CITTÀ ITALIANE

**12 HUB Market creati:**

| ID | Nome | Città | Coordinate |
|----|------|-------|------------|
| 7 | Hub Centro | Grosseto | 42.7589, 11.1135 |
| 8 | HUB Market Roma | Roma | 41.9028, 12.4964 |
| 9 | HUB Market Milano | Milano | 45.4642, 9.1900 |
| 11 | HUB Market Torino | Torino | 45.0703, 7.6869 |
| 12 | HUB Market Firenze | Firenze | 43.7696, 11.2558 |
| 13 | HUB Market Bologna | Bologna | 44.4949, 11.3426 |
| 14 | HUB Market Venezia | Venezia | 45.4408, 12.3155 |
| 15 | HUB Market Genova | Genova | 44.4056, 8.9463 |
| 16 | HUB Market Palermo | Palermo | 38.1157, 13.3615 |
| 17 | HUB Market Bari | Bari | 41.1171, 16.8719 |
| 19 | HUB Market Modena | Modena | 44.6471, 10.9252 |
| - | HUB Market Napoli | Napoli | 40.8518, 14.2681 |

---

#### 🔄 FLUSSO VISTA ITALIA ↔ VISTA HUB

```
┌─────────────────────────────────────────────────────────────┐
│                      VISTA ITALIA                           │
│  - Zoom: 6                                                  │
│  - Centro: [42.5, 12.5]                                     │
│  - Marker "H" viola per ogni HUB                            │
│  - Pulsante: "Vista HUB" (disabilitato)                     │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ Click su HUB
                            │ (animazione flyTo)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       VISTA HUB                             │
│  - Zoom: 16                                                 │
│  - Centro: coordinate HUB selezionato                       │
│  - Poligono area HUB (viola tratteggiato)                   │
│  - Marker negozi (A, B, C...) colorati per stato            │
│  - Fine corsa: hubBounds da area_geojson                    │
│  - Pulsante: "Vista Italia" (attivo)                        │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ Click "Vista Italia"
                            │ (animazione flyTo)
                            ▼
                    [Torna a VISTA ITALIA]
```

---

#### 🔄 DIFFERENZA HUB vs MERCATI

| Aspetto | HUB | Mercati |
|---------|-----|---------|
| **Tabelle DB** | hub_locations, hub_shops | markets, stalls |
| **Uso** | Centri commerciali, zone negozi fissi | Mercati ambulanti periodici |
| **Elementi** | Negozi (shops) - **PUNTI** | Posteggi (stalls) - **POLIGONI** |
| **Marker mappa** | "H" viola | "M" rosso |
| **Zoom dettaglio** | 16 | 17 |
| **API base** | /api/hub/locations | /api/markets |
| **Indipendenza** | Può essere indipendente | Sempre legato a comune |
| **Visualizzazione** | Marker con lettera | Rettangoli colorati |

---

#### 📁 FILE MODIFICATI

**Frontend (dms-hub-app-new):**
- `client/src/components/HubMarketMapComponent.tsx` - Nuovo componente mappa HUB
- `client/src/components/GestioneHubMapWrapper.tsx` - Wrapper con selettore
- `client/src/components/GestioneHubPanel.tsx` - Integrazione wrapper

**Backend (mihub-backend-rest):**
- `routes/hub.js` - API endpoints HUB (corretto INSERT)

**Commit GitHub:**
- `95ba737` - feat: HubMarketMapComponent clone con supporto HUB
- `87db919` - fix: correzioni mappa HUB - altezza 700px, centro Italia fisso
- `6053d6f` - feat: aggiunto hubBounds per fine corsa mappa HUB

---

