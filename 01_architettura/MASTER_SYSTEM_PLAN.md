# MASTER SYSTEM PLAN - DMS HUB + MIO HUB

**Ultima Modifica:** 25 Novembre 2025  
**Versione:** 2.1

---

## Indice

1. [Architettura Generale](#1-architettura-generale)
2. [MIO Agent - Fase 1](#2-mio-agent---fase-1)
3. [Componenti Sistema](#3-componenti-sistema)
4. [Integrazioni](#4-integrazioni)
5. [Sistema Mappe GIS](#5-sistema-mappe-gis)
6. [Deployment](#6-deployment)
7. [Sicurezza](#7-sicurezza)
8. [Monitoring](#8-monitoring)
9. [Roadmap](#9-roadmap)

---

## 1. Architettura Generale

Il sistema DMS HUB è composto da:

- **Frontend Vercel:** Dashboard PA (Next.js + React + tRPC)
- **Backend Vercel:** Serverless functions (tRPC router)
- **Backend Hetzner:** REST API per orchestratore multi-agente
- **Database:** PostgreSQL (Vercel Postgres)
- **Storage:** File system locale + GitHub logs

---

## 2. MIO Agent - Fase 1

### 2.1. Obiettivo Fase 1

Implementare la **vista singola** del MIO Agent con connessione diretta all'orchestratore REST su Hetzner.

**Scope limitato:**
- ✅ Vista singola MIO (chat principale)
- ✅ Chiamate REST dirette a Hetzner
- ✅ Backend orchestratore uniformato con contratto standard
- ✅ Backend sotto controllo Git (repository mihub-backend-rest)
- ⚠️ Handler MIO da sistemare (manca check `if (mioLoading) return;`)
- ❌ Vista 4 agenti (rimane disabled)

### 2.2. Endpoint Backend

**URL:** `POST https://orchestratore.mio-hub.me/api/mihub/orchestrator`

**Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Payload Richiesto:**
```json
{
  "mode": "auto",
  "conversationId": "string|null",
  "message": "string",
  "meta": {
    "source": "dashboard_main"
  }
}
```

**Campi Payload:**
- `mode`: `"auto"` | `"manual"` (Fase 1 usa solo `"auto"`)
- `conversationId`: UUID della conversazione (null per nuova conversazione)
- `message`: Testo del messaggio utente
- `meta`: Metadati opzionali (source, timestamp, etc.)

### 2.3. Response Standard

#### Successo

```json
{
  "success": true,
  "agent": "mio",
  "conversationId": "uuid-string",
  "message": "Risposta dell'agente MIO",
  "error": null
}
```

#### Errore

```json
{
  "success": false,
  "agent": "mio",
  "conversationId": "uuid-string",
  "message": null,
  "error": {
    "type": "llm_rate_limit | llm_provider_error | validation_error | internal_error",
    "provider": "openai | gemini | null",
    "statusCode": 429,
    "message": "Rate limit OpenAI: riprovare tra qualche secondo"
  }
}
```

**Campi Response:**
- `success`: boolean - indica se la richiesta è andata a buon fine
- `agent`: string - ID dell'agente che ha risposto (`"mio"`, `"dev"`, `"gemini_arch"`)
- `conversationId`: string | null - UUID della conversazione (generato se null in input)
- `message`: string | null - Risposta dell'agente (presente solo se success === true)
- `error`: object | null - Dettagli errore (presente solo se success === false)

**Tipi di Errore:**
- `llm_rate_limit`: Rate limit del provider LLM (OpenAI, Gemini)
- `llm_provider_error`: Errore generico del provider LLM
- `validation_error`: Payload non valido
- `internal_error`: Errore interno orchestratore

**⚠️ NOTA:** L'agente `manus_worker` NON è configurato nel backend orchestratore.

### 2.4. Flusso MIO (Vista Singola)

```
┌─────────────────────────────────────────────────────────────┐
│                  Dashboard PA (Vercel)                      │
│                                                             │
│  Tab "MIO Agent" (vista singola)                           │
│  ├── User input: "Ciao MIO"                                │
│  ├── Click Send                                            │
│  └── handleSendMio()                                       │
│      ├── if (mioLoading) return; ⚠️ DA AGGIUNGERE         │
│      └── callOrchestrator({                                │
│            mode: "auto",                                   │
│            conversationId: mioConversationId,              │
│            message: "Ciao MIO",                            │
│            meta: { source: "dashboard_main" }              │
│          })                                                │
└─────────────────────────────────────────────────────────────┘
                         │
                         │ POST (fetch)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│           Backend Hetzner (mihub-backend-rest)              │
│                                                             │
│  POST /api/mihub/orchestrator                              │
│  ├── Validate payload                                      │
│  ├── Load/Create conversation                              │
│  ├── Call LLM (OpenAI GPT-4 / Gemini)                     │
│  ├── Save message to DB                                    │
│  ├── Log to api-guardian.log                               │
│  └── Return JSON response (contratto uniformato)           │
└─────────────────────────────────────────────────────────────┘
                         │
                         │ JSON response
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  Dashboard PA (Vercel)                      │
│                                                             │
│  handleSendMio() riceve response                           │
│  ├── if (response.success)                                 │
│  │   └── Mostra messaggio agente                           │
│  └── else                                                  │
│      └── Mostra errore leggibile                           │
└─────────────────────────────────────────────────────────────┘
```

### 2.5. File Modificati in Fase 1

**Backend (mihub-backend-rest):**
- ✅ `routes/orchestrator.js` - Response uniformate con contratto standard
- ✅ Repository sotto Git: `Chcndr/mihub-backend-rest`
- ✅ Backup legacy creato: `mihub-backend-rest-legacy-20251124`

**Frontend (dms-hub-app-new):**
- ✅ `client/src/api/orchestratorClient.ts` - Client REST diretto
- ✅ `client/src/pages/DashboardPA.tsx` - Tab MIO Agent (vista singola)
  - ✅ Input/button abilitati
  - ✅ `handleSendMio` collegato a `callOrchestrator`
  - ✅ Gestione conversationId
  - ✅ Mostra messaggi e errori
  - ⚠️ **DA SISTEMARE:** Aggiungere `if (mioLoading) return;` per evitare re-render multipli

**Blueprint (dms-system-blueprint):**
- ✅ `01_architettura/MASTER_SYSTEM_PLAN.md` - Questo file

### 2.6. File NON Modificati in Fase 1

**⚠️ ESPLICITO: Nessuna modifica a:**
- ✅ GIS/Mappa Mercato Grosseto (SACRA - funzionante)
- ✅ Vista 4 agenti (rimane disabled)
- ✅ Componenti legacy (MIOAgent.tsx, MioAgentPanel.tsx)

### 2.7. Verifica Mappa GIS (SACRA)

**⚠️ PROTOCOLLO OBBLIGATORIO:**

**Prima di ogni modifica:**
1. Aprire Dashboard PA → Gestione Mercati → Mercato Grosseto → Posteggi
2. Verificare che la mappa sia visibile con 160 posteggi colorati
3. Se mappa non funziona → STOP, nessuna modifica

**Dopo ogni deploy:**
1. Verificare mappa Grosseto PRIMA di qualsiasi altra cosa
2. Se mappa rotta → ROLLBACK immediato
3. Solo se mappa OK → verificare altre funzionalità

**Causa rottura mappa:**
- Handler MIO senza check `if (mioLoading) return;` causa re-render multipli che rompono inizializzazione Leaflet
- Documentato in: `/home/ubuntu/ANALISI_ROTTURA_MAPPA_GIS.md`

---

## 3. Componenti Sistema

### 3.1. Frontend (Vercel)

**Repository:** `Chcndr/dms-hub-app-new`

**Stack:**
- Next.js 14
- React 18
- TypeScript
- tRPC
- TailwindCSS
- shadcn/ui
- Leaflet + OpenStreetMap (mappe GIS)

**Pagine Principali:**
- `/dashboard-pa` - Dashboard PA con tabs
- `/map` - Pagina Mappa (rifatta con design Dashboard PA)

**Componenti Mappe:**
- `GISMap.tsx` - Componente base Leaflet generico
- `MarketMapComponent.tsx` - Mappa mercati specifica (usa GISMap)
- `MobilityMap.tsx` - Mappa mobilità Centro Mobilità
- `Map.tsx` - Google Maps (NON usato, non causa conflitti)

### 3.2. Backend Vercel (Serverless)

**Repository:** `Chcndr/dms-hub-app-new` (cartella `/server`)

**Stack:**
- tRPC router
- Vercel Postgres
- Prisma ORM

**Router Principali:**
- `dmsHub.*` - Gestione mercati, posteggi, operatori
- `logs.*` - Sistema logging
- `integrations.*` - Statistiche API

### 3.3. Backend Hetzner (REST)

**Repository:** `Chcndr/mihub-backend-rest`

**Server:** 157.90.29.66

**Stack:**
- Node.js + Express
- PostgreSQL
- OpenAI API
- Google Gemini API
- PM2 (process manager)

**Endpoint Principali:**
- `POST /api/mihub/orchestrator` - Orchestratore multi-agente
- `GET /api/guardian/logs` - Log Guardian

**PM2 Process:**
- Nome: `mihub-backend`
- Path: `/root/mihub-backend-rest`
- Restart: `pm2 restart mihub-backend`

**Deploy:**
- Sempre via Git: commit → push → pull su Hetzner → pm2 restart
- MAI modifiche manuali su server

---

## 4. Integrazioni

### 4.1. GitHub

**Repository MIO-hub:** `Chcndr/MIO-hub`

**Contenuti:**
- `/api/index.json` - Configurazione endpoint API
- `/logs/*.log` - Log Guardian (NDJSON)

**Uso:**
- Dashboard PA carica endpoint da `index.json`
- Dashboard PA carica log da `*.log`

**Autenticazione:**
- ⚠️ NON usare token hardcoded
- Usare sempre GitHub CLI: `gh auth token`

### 4.2. OpenAI

**Modelli:**
- GPT-4 Turbo (agente MIO)
- GPT-4 (agente dev)

**Rate Limit:**
- Tier 1: 3 RPM (requests per minute)
- Gestito con retry + exponential backoff

### 4.3. Google Gemini

**Modelli:**
- Gemini 1.5 Pro (agente gemini_arch)

**Fallback:**
- Se OpenAI rate limit → usa Gemini

### 4.4. Zapier (Futuro)

**Obiettivo:**
- Trigger GitHub `repository_dispatch` per automazioni

**Stato:**
- ⚠️ Da configurare

---

## 5. Sistema Mappe GIS

### 5.1. Architettura Mappe

**Componenti:**

1. **GISMap.tsx** (componente base)
   - Leaflet + OpenStreetMap
   - Generico e riutilizzabile
   - Props: center, zoom, markers, polygons, etc.

2. **MarketMapComponent.tsx** (mappa mercati)
   - Usa GISMap
   - Gestione posteggi mercati
   - Colori per stato (libero, occupato, riservato)
   - **SACRO:** usato in Gestione Mercati → Mercato Grosseto

3. **MobilityMap.tsx** (mappa mobilità)
   - Usa GISMap
   - Centro Mobilità
   - Percorsi e zone

4. **Map.tsx** (Google Maps)
   - NON usato
   - Non causa conflitti

### 5.2. Mappa Grosseto (SACRA)

**Ubicazione:**
- Dashboard PA → Gestione Mercati → Mercato Grosseto → Posteggi

**Caratteristiche:**
- 160 posteggi colorati
- Leaflet + OpenStreetMap
- Dati VERI da database

**Componenti coinvolti:**
- `GestioneMercati.tsx`
- `MarketMapComponent.tsx`
- `GISMap.tsx`

**⚠️ VINCOLO ASSOLUTO:**
- NON toccare MAI questi componenti
- Se si rompe → ROLLBACK immediato

### 5.3. Mappe Mock (RIMOSSE)

**Stato:** ✅ Tutte rimosse (4/4)

**Rimosse da:**
1. ✅ Overview (card "Mappa Interattiva")
2. ✅ Tab Mercati (mappa mock)
3. ✅ Segnalazioni Civiche (mappa mock)
4. ✅ Controlli e Sanzioni (mappa mock)

**Commit:** `c7b1c76` - "cleanup: remove all mock maps + fix DMS AI widget z-index"

### 5.4. MapPage.tsx (Rifatta)

**Stato:** ✅ Rifatta completamente

**Caratteristiche:**
- Design Dashboard PA (colori, stile, card)
- NO barra laterale mobile
- Campo ricerca: Comune o Mercato
- Layout pulito e responsive
- Placeholder per mappa (NO mappa vera ancora)
- Icone e UI coerenti

**Uso futuro:**
- Preview strumenti Dashboard Negozianti/Ambulanti

**File:** `client/src/pages/MapPage.tsx`

### 5.5. Ispezione Codice Mappe

**Documento:** `/home/ubuntu/ISPEZIONE_COMPLETA_CODICE_MAPPE.md`

**Risultato:**
- ✅ Codebase pulito
- ✅ Nessun conflitto critico
- ✅ Import corretti
- ✅ Nessun codice morto

---

## 6. Deployment

### 6.1. Frontend (Vercel)

**Production:** `https://dms-hub-app-new.vercel.app`

**Branch Strategy:**
- `master` → Production (auto-deploy)
- `feature/*` → Preview deployments

**Environment Variables:**
- `VITE_API_URL` → `https://orchestratore.mio-hub.me`

**Commit Stabile:**
- Documentato in: `dms-system-blueprint/COMMIT_STABILE_PRODUZIONE.md`

### 6.2. Backend Hetzner

**Production:** `https://orchestratore.mio-hub.me`

**Deploy:**
```bash
# Su Hetzner (SSH)
cd /root/mihub-backend-rest
git pull origin master
pm2 restart mihub-backend
```

**Test:**
```bash
curl -X POST https://orchestratore.mio-hub.me/api/mihub/orchestrator \
  -H "Content-Type: application/json" \
  -d '{"mode":"auto","conversationId":null,"message":"test","meta":{"source":"curl"}}'
```

**Backup Legacy:**
- Path: `/root/mihub-backend-rest-legacy-20251124`

---

## 7. Sicurezza

### 7.1. Guardian System

**Controllo accessi API:**
- Ogni endpoint ha `risk_level` (low, medium, high, critical)
- Ogni agente ha permessi specifici
- Log completo di tutte le chiamate

**File:** `mihub-backend-rest/middleware/guardian.js`

### 7.2. Rate Limiting

**OpenAI:**
- 3 RPM (Tier 1)
- Gestito con retry

**Hetzner:**
- Nessun rate limit interno (per ora)

### 7.3. Credenziali

**File:** `/home/ubuntu/mihub_credentials.txt`

**Contenuto:**
- ✅ Credenziali aggiornate
- ✅ Rimosso token GitHub hardcoded
- ✅ Documentato uso GitHub CLI

---

## 8. Monitoring

### 8.1. Logs

**Guardian Logs:**
- File: `MIO-hub/logs/api-guardian.log`
- Formato: NDJSON
- Contenuto: timestamp, status, endpoint, method, user

**System Logs:**
- Console logs backend
- PM2 logs

### 8.2. Vercel Analytics

**Metriche:**
- Page views
- API calls
- Error rate

---

## 9. Roadmap

### Fase 1 (Corrente) ✅ 95% Completata

- ✅ Vista singola MIO Agent
- ✅ Chiamate REST dirette a Hetzner
- ✅ Backend orchestratore uniformato
- ✅ Backend sotto Git
- ✅ Mappe mock rimosse (4/4)
- ✅ MapPage rifatta con design Dashboard PA
- ✅ Widget DMS AI Assistant sistemato (z-index 9999)
- ⚠️ Handler MIO da sistemare (manca check loading)
- ⚠️ Pagina Chiavi Sicurezza da ricostruire

### Fase 2 (Prossima)

- Vista 4 agenti funzionante
- Selezione manuale agente
- Storico conversazioni
- Pagina Chiavi Sicurezza completa

### Fase 3 (Futura)

- Tool calling (API DMS)
- Automazioni Zapier (repository_dispatch)
- Notifiche real-time

---

## 10. Problemi Noti e Soluzioni

### 10.1. Rottura Mappa GIS

**Problema:**
- Handler MIO senza check `if (mioLoading) return;` causa re-render multipli
- Re-render multipli rompono inizializzazione Leaflet
- Mappa Grosseto diventa vuota

**Soluzione:**
- Aggiungere `if (mioLoading) return;` all'inizio di `handleSendMio`
- Aggiungere `disabled={mioLoading}` al bottone invio

**Documentazione:**
- `/home/ubuntu/ANALISI_ROTTURA_MAPPA_GIS.md`

### 10.2. Widget DMS AI Assistant

**Problema:**
- Widget sotto altri elementi (z-index basso)

**Soluzione:**
- ✅ z-index aumentato a 9999

**Nota:**
- Widget NON usa orchestratore (API esterna separata)

### 10.3. Pagina Chiavi Sicurezza

**Problema:**
- Sparita dopo rollback `2f9f840`

**Soluzione:**
- Da ricostruire in Impostazioni → Chiavi Sicurezza
- Design coerente con Dashboard PA

---

## 11. File Chiave

**Backend Hetzner:**
- `/root/mihub-backend-rest/routes/orchestrator.js` - Orchestratore uniformato

**Frontend:**
- `client/src/api/orchestratorClient.ts` - Client orchestratore
- `client/src/pages/DashboardPA.tsx` - Dashboard PA con MIO Agent
- `client/src/pages/MapPage.tsx` - Pagina Mappa (rifatta)
- `client/src/components/GestioneMercati.tsx` - Gestione Mercati (SACRO)
- `client/src/components/MarketMapComponent.tsx` - Mappa GIS (SACRO)
- `client/src/components/GISMap.tsx` - Componente base Leaflet (SACRO)

**Documentazione:**
- `/home/ubuntu/mihub_credentials.txt` - Credenziali aggiornate
- `/home/ubuntu/ANALISI_ROTTURA_MAPPA_GIS.md` - Analisi rottura mappa
- `/home/ubuntu/ISPEZIONE_COMPLETA_CODICE_MAPPE.md` - Ispezione codice mappe

**Blueprint:**
- `dms-system-blueprint/01_architettura/MASTER_SYSTEM_PLAN.md` - Questo file
- `dms-system-blueprint/COMMIT_STABILE_PRODUZIONE.md` - Commit stabili

---

**Documento Vivo:** Questo file viene aggiornato ad ogni fase del progetto.

**Ultimo Aggiornamento:** 25 Novembre 2025 - Fase 1 quasi completata, mappe mock rimosse, MapPage rifatta

**Prossimo Aggiornamento:** Fine Fase 1 (dopo sistemazione handler MIO e deploy finale)
