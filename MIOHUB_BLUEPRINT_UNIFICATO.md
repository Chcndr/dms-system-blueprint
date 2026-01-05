# 🏗️ MIO HUB - BLUEPRINT UNIFICATO DEL SISTEMA

> **Versione:** 3.5.11  
> **Data:** 05 Gennaio 2026 (Aggiornato ore 23:15)  
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

## 🔌 API ENDPOINTS

### Endpoint Index (153 endpoint totali)

Gli endpoint sono documentati in:
```
/home/ubuntu/dms-hub-app-new/client/public/api-index.json
```

### Categorie Principali

| Categoria | Prefisso | Esempi |
|-----------|----------|--------|
| **DMS Hub** | `/api/trpc/dmsHub.*` | bookings, inspections, locations |
| **Guardian** | `/api/guardian/*` | health, logs, testEndpoint |
| **MIO Hub** | `/api/mihub/*` | orchestrator, chats, messages |
| **Logs** | `/api/logs/*` | createLog, getLogs, stats |
| **Health** | `/api/health/*` | full, history, alerts |
| **GIS** | `/api/gis/*` | market-map |
| **Imprese** | `/api/imprese/*` | qualificazioni, rating |

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

- **Endpoint totali:** 153
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

