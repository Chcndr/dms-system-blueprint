# MASTER SYSTEM PLAN - DMS HUB + MIO HUB

**Ultima Modifica:** 24 Novembre 2025  
**Versione:** 2.0

---

## Indice

1. [Architettura Generale](#1-architettura-generale)
2. [MIO Agent - Fase 1](#2-mio-agent---fase-1)
3. [Componenti Sistema](#3-componenti-sistema)
4. [Integrazioni](#4-integrazioni)

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
- ❌ Vista 4 agenti (rimane disabled)
- ❌ Modifiche a GIS/mappa
- ❌ Modifiche a Integrazioni, Log, Debug

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
- `agent`: string - ID dell'agente che ha risposto (`"mio"`, `"dev"`, `"manus_worker"`, `"gemini_arch"`)
- `conversationId`: string | null - UUID della conversazione (generato se null in input)
- `message`: string | null - Risposta dell'agente (presente solo se success === true)
- `error`: object | null - Dettagli errore (presente solo se success === false)

**Tipi di Errore:**
- `llm_rate_limit`: Rate limit del provider LLM (OpenAI, Gemini)
- `llm_provider_error`: Errore generico del provider LLM
- `validation_error`: Payload non valido
- `internal_error`: Errore interno orchestratore

### 2.4. Flusso MIO (Vista Singola)

```
┌─────────────────────────────────────────────────────────────┐
│                  Dashboard PA (Vercel)                      │
│                                                             │
│  Tab "MIO Agent" (vista singola)                           │
│  ├── User input: "Ciao MIO"                                │
│  ├── Click Send                                            │
│  └── handleSendMio()                                       │
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
│  └── Return JSON response                                  │
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
- `routes/orchestrator.js` - Uniformare response JSON al contratto

**Frontend (dms-hub-app-new):**
- `client/src/api/orchestratorClient.ts` - Client REST diretto
- `client/src/pages/DashboardPA.tsx` - Tab MIO Agent (vista singola)
  - Abilitare input/button (rimuovere `disabled`)
  - Collegare `handleSendMio` a `callOrchestrator`
  - Gestire conversationId
  - Mostrare messaggi e errori

**Blueprint (dms-system-blueprint):**
- `01_architettura/MASTER_SYSTEM_PLAN.md` - Questo file

### 2.6. File NON Modificati in Fase 1

**⚠️ ESPLICITO: Nessuna modifica a:**
- GIS/Mappa Mercato Grosseto
- Vista 4 agenti (rimane disabled)
- Pagina Integrazioni
- Pagina Log
- Pagina Debug
- Componenti legacy (MIOAgent.tsx, MioAgentPanel.tsx)

### 2.7. Verifica Mappa GIS (SACRA)

**Prima di ogni modifica:**
1. Aprire Dashboard PA → Gestione Mercati → Mercato Grosseto → Posteggi
2. Verificare che la mappa sia visibile con 160 posteggi colorati
3. Se mappa non funziona → STOP, nessuna modifica

**Dopo ogni deploy preview:**
1. Verificare mappa in preview
2. Se mappa non identica a produzione → rollback branch, STOP
3. Solo se mappa OK → verificare MIO Agent
4. Solo se entrambi OK → merge

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

**Pagine Principali:**
- `/dashboard-pa` - Dashboard PA con tabs
- `/market-gis` - Mappa GIS standalone

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

**Stack:**
- Node.js + Express
- PostgreSQL
- OpenAI API
- Google Gemini API

**Endpoint Principali:**
- `POST /api/mihub/orchestrator` - Orchestratore multi-agente
- `GET /api/guardian/logs` - Log Guardian

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

---

## 5. Deployment

### 5.1. Frontend (Vercel)

**Production:** `https://dms-hub-app-new.vercel.app`

**Branch Strategy:**
- `master` → Production
- `feature/*` → Preview deployments

**Environment Variables:**
- `VITE_API_URL` → `https://orchestratore.mio-hub.me`

### 5.2. Backend Hetzner

**Production:** `https://orchestratore.mio-hub.me`

**Deploy:**
- SSH + PM2
- Restart: `pm2 restart mihub-backend-rest`

---

## 6. Sicurezza

### 6.1. Guardian System

**Controllo accessi API:**
- Ogni endpoint ha `risk_level` (low, medium, high, critical)
- Ogni agente ha permessi specifici
- Log completo di tutte le chiamate

**File:** `mihub-backend-rest/middleware/guardian.js`

### 6.2. Rate Limiting

**OpenAI:**
- 3 RPM (Tier 1)
- Gestito con retry

**Hetzner:**
- Nessun rate limit interno (per ora)

---

## 7. Monitoring

### 7.1. Logs

**Guardian Logs:**
- File: `MIO-hub/logs/api-guardian.log`
- Formato: NDJSON
- Contenuto: timestamp, status, endpoint, method, user

**System Logs:**
- Console logs backend
- PM2 logs

### 7.2. Vercel Analytics

**Metriche:**
- Page views
- API calls
- Error rate

---

## 8. Roadmap

### Fase 1 (Corrente)
- ✅ Vista singola MIO Agent
- ✅ Chiamate REST dirette a Hetzner

### Fase 2 (Futura)
- Vista 4 agenti funzionante
- Selezione manuale agente
- Storico conversazioni

### Fase 3 (Futura)
- Tool calling (API DMS)
- Automazioni Zapier
- Notifiche real-time

---

**Documento Vivo:** Questo file viene aggiornato ad ogni fase del progetto.

**Prossimo Aggiornamento:** Fine Fase 1 (dopo merge)
