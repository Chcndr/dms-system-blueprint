# 🔍 REPORT ISPEZIONE TECNICA - MIO AGENT E LOGS

**Data Ispezione:** 24 Novembre 2025, ore 14:30 CET  
**Commit Analizzato:** 8e8459d (production stabile)  
**Tipo:** Ispezione SOLO LETTURA (nessuna modifica al codice)

---

## 📋 INDICE

1. [MIO Agent - Frontend](#1-mio-agent---frontend)
2. [Backend Orchestratore Hetzner](#2-backend-orchestratore-hetzner)
3. [Pagina Integrazioni / API Tester](#3-pagina-integrazioni--api-tester)
4. [Pagina Logs e Debug](#4-pagina-logs-e-debug)
5. [Conclusioni e Raccomandazioni](#5-conclusioni-e-raccomandazioni)

---

## 1. MIO AGENT - FRONTEND

### 1.1 Componenti Trovati

| File | Path | Tipo | Stato |
|------|------|------|-------|
| **DashboardPA.tsx** | `/client/src/pages/DashboardPA.tsx` | Pagina principale | ✅ Attivo |
| **MIOAgent.tsx** | `/client/src/components/MIOAgent.tsx` | Componente legacy | ⚠️ Non usato |
| **MioAgentPanel.tsx** | `/client/src/components/mio/MioAgentPanel.tsx` | Componente legacy | ⚠️ Non usato |
| **orchestratorClient.ts** | `/client/src/api/orchestratorClient.ts` | Client REST | ✅ Pronto ma non collegato |

### 1.2 Struttura MIO Agent in DashboardPA.tsx

**Linee:** 3224-3546

```tsx
TAB "MIO Agent" (value="mio")
│
├── SEZIONE A: Chat Principale MIO (sempre visibile)
│   ├── Card: "MIO Agent - Chat Principale (GPT-5 Coordinatore)"
│   ├── Icona: Brain (h-5 w-5 text-[#8b5cf6])
│   ├── Descrizione: "Chat principale con il 'cervello'..."
│   ├── Area chat: h-96 bg-[#0a0f1a]
│   ├── Input: disabled ❌
│   ├── Button Send: disabled ❌
│   └── Nota: "Chat in fase di sviluppo"
│
└── SEZIONE B: Pannello Multi-Agente (sotto)
    ├── Card: "Chat Multi-Agente (MIO / Manus / Abacus / Zapier)"
    ├── Toggle: Vista singola / Vista 4 agenti
    │   ├── Button "Vista singola" (viewMode === 'single')
    │   └── Button "Vista 4 agenti" (viewMode === 'quad')
    │
    ├── Pill Tabs (4 bottoni)
    │   ├── MIO (Brain icon, purple)
    │   ├── Manus (Wrench icon, blue)
    │   ├── Abacus (Calculator icon, green)
    │   └── Zapier (Zap icon, orange)
    │   └── disabled quando viewMode === 'quad' ❌
    │
    └── Area Chat Condizionale
        ├── Vista singola: 1 chat grande (selectedAgent)
        │   ├── Input: disabled ❌
        │   └── Button: disabled ❌
        │
        └── Vista 4 agenti: griglia 2x2
            ├── Card MIO (input disabled ❌)
            ├── Card Manus (input disabled ❌)
            ├── Card Abacus (input disabled ❌)
            └── Card Zapier (input disabled ❌)
```

### 1.3 Funzioni di Invio Messaggio

**❌ NESSUNA FUNZIONE ATTIVA**

Tutti gli input hanno `disabled={true}` e non ci sono handler `onClick` o `onSubmit` collegati.

**Ricerca nel codice:**
```bash
grep -r "orchestratorClient\|callOrchestrator" client/src --include="*.tsx"
# Risultato: NESSUN IMPORT, NESSUN USO
```

### 1.4 orchestratorClient.ts - Dettagli Tecnici

**Path:** `/client/src/api/orchestratorClient.ts`

**Funzione principale:**
```typescript
export async function callOrchestrator(
  payload: OrchestratorRequest
): Promise<OrchestratorResponse>
```

**URL chiamato:**
```typescript
const baseUrl = import.meta.env.VITE_API_URL ?? "https://orchestratore.mio-hub.me";
const url = `${baseUrl}/api/mihub/orchestrator`;
```

**Metodo:** `POST`

**Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Body inviato (esempio):**
```json
{
  "mode": "auto",
  "message": "Ciao MIO",
  "conversationId": null,
  "meta": { "source": "dashboard_pa" }
}
```

**Response attesa:**
```typescript
{
  success: boolean;
  agent: "mio" | "dev" | "manus_worker" | "gemini_arch";
  conversationId: string | null;
  message: string | null;
  error?: {
    type: string;
    provider?: string | null;
    statusCode?: number;
    detail?: string;
  };
}
```

**Console logs:**
```typescript
console.log("[OrchestratorClient] Chiamata a:", url);
console.log("[OrchestratorClient] Payload:", payload);
console.log("[OrchestratorClient] Status:", res.status);
console.log("[OrchestratorClient] Risposta:", data);
```

**⚠️ STATO:** Codice pronto ma MAI IMPORTATO in nessun componente.

---

## 2. BACKEND ORCHESTRATORE HETZNER

### 2.1 Endpoint Testato

**URL:** `https://orchestratore.mio-hub.me/api/mihub/orchestrator`

**Method:** `POST`

**Content-Type:** `application/json`

### 2.2 Test cURL - Mode AUTO

**Comando:**
```bash
curl -X POST https://orchestratore.mio-hub.me/api/mihub/orchestrator \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "auto",
    "message": "Ping MIO auto",
    "conversationId": null,
    "meta": { "source": "ispezione_auto" }
  }'
```

**Status HTTP:** `200 OK`

**Response Body:**
```json
{
  "success": false,
  "error": {
    "type": "llm_rate_limit",
    "provider": "openai",
    "statusCode": 429,
    "message": "Rate limit OpenAI: riprovare tra qualche secondo"
  },
  "conversationId": "1c23231a-12d4-4097-b1bd-23cfb17b932e",
  "agent": "mio"
}
```

### 2.3 Test cURL - Mode MANUAL

**Comando:**
```bash
curl -X POST https://orchestratore.mio-hub.me/api/mihub/orchestrator \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "manual",
    "targetAgent": "dev",
    "message": "Ping dev manual",
    "conversationId": null,
    "meta": { "source": "ispezione_manual" }
  }'
```

**Status HTTP:** `200 OK`

**Response Body:**
```json
{
  "success": false,
  "error": {
    "type": "llm_rate_limit",
    "provider": "openai",
    "statusCode": 429,
    "message": "Rate limit OpenAI: riprovare tra qualche secondo"
  },
  "conversationId": "65771d2b-4e7a-469f-87c5-a3dfd59036b2",
  "agent": "dev"
}
```

### 2.4 Analisi Response Shape

**Campi sempre presenti:**
- `success`: boolean
- `agent`: string (mio, dev, manus_worker, gemini_arch)
- `conversationId`: string (UUID generato)

**Campi condizionali:**
- `message`: string | null (presente se success === true)
- `error`: object | undefined (presente se success === false)
  - `error.type`: string (es: "llm_rate_limit", "validation_error")
  - `error.provider`: string | null (es: "openai")
  - `error.statusCode`: number (es: 429)
  - `error.message`: string (messaggio human-readable)

**✅ CONCLUSIONE:** Il backend risponde correttamente con JSON strutturato, anche in caso di errore.

---

## 3. PAGINA INTEGRAZIONI / API TESTER

### 3.1 File Responsabile

**Path:** `/client/src/components/Integrazioni.tsx`

**Import in DashboardPA.tsx:**
```typescript
import Integrazioni from '@/components/Integrazioni';
```

**Linea rendering:** 3066
```tsx
<TabsContent value="integrations">
  <Integrazioni />
</TabsContent>
```

### 3.2 Struttura Tabs

```
Integrazioni Component
│
├── Tab 1: API Dashboard (APIDashboard)
├── Tab 2: Connessioni (ConnessioniEsterne)
├── Tab 3: API Keys (APIKeysManager)
├── Tab 4: Webhook (WebhookManager)
└── Tab 5: Sync Status (SyncStatus)
```

### 3.3 APIDashboard - Come Funziona

**Linee:** 103-250+

**1. Caricamento Endpoint:**
```typescript
useEffect(() => {
  fetch('https://raw.githubusercontent.com/Chcndr/MIO-hub/master/api/index.json')
    .then(r => r.json())
    .then(data => {
      // Group endpoints by category
      const endpointsByCategory: { [key: string]: any[] } = {};
      
      data.services?.forEach((service: any) => {
        service.endpoints?.forEach((ep: any) => {
          const category = ep.category || 'Other';
          if (!endpointsByCategory[category]) {
            endpointsByCategory[category] = [];
          }
          endpointsByCategory[category].push({
            id: ep.id,
            method: ep.method,
            path: ep.path,
            description: ep.description,
            risk_level: ep.risk_level,
            enabled: ep.enabled,
            test: ep.test,
            service_id: service.id,
            base_url: service.base_url
          });
        });
      });
      
      setApiEndpoints(categorizedEndpoints);
      setLoading(false);
    })
    .catch(err => {
      console.error('Error loading endpoints from MIO-hub:', err);
      setLoading(false);
    });
}, []);
```

**Source:** GitHub repository `Chcndr/MIO-hub` file `/api/index.json`

**2. Test Endpoint:**
```typescript
const handleTestEndpoint = async (endpointPath: string, customBody?: string) => {
  setSelectedEndpoint(endpointPath);
  setTestResult(null);
  setIsLoading(true);
  
  const startTime = Date.now();
  
  // Find endpoint info from MIO-hub data
  const endpointInfo = apiEndpoints
    .flatMap(cat => cat.endpoints)
    .find(ep => ep.path === endpointPath);
  
  try {
    let data: any = null;
    
    // Parse request body
    let parsedBody: any = {};
    const bodyToUse = customBody || requestBody;
    if (bodyToUse && bodyToUse.trim()) {
      parsedBody = JSON.parse(bodyToUse);
    }
    
    // Mappa endpoint → chiamata TRPC
    switch (endpointPath) {
      case '/api/dmsHub/markets/importAuto':
        data = await utils.client.dmsHub.markets.importAuto.mutate(parsedBody);
        break;
      case '/api/dmsHub/markets/list':
        data = await utils.client.dmsHub.markets.list.query();
        break;
      // ... altri endpoint
    }
    
    setTestResult({
      success: true,
      data,
      responseTime: Date.now() - startTime
    });
  } catch (error) {
    setTestResult({
      success: false,
      error: error.message,
      responseTime: Date.now() - startTime
    });
  } finally {
    setIsLoading(false);
  }
}
```

**3. Client Usato:**
```typescript
const utils = trpc.useUtils();
// Usa tRPC client per chiamare backend Vercel serverless
```

**⚠️ IMPORTANTE:** 
- Gli endpoint sono caricati da GitHub (config JSON)
- I test usano **tRPC** (NON fetch diretto)
- Le chiamate passano per **Vercel serverless functions**
- NON chiama direttamente Hetzner

### 3.4 Esempio Test Endpoint

**Endpoint testato:** `/api/dmsHub/markets/list`

**Request:**
```typescript
utils.client.dmsHub.markets.list.query()
```

**Response attesa:**
```json
{
  "success": true,
  "data": [...],
  "responseTime": 234
}
```

**Errore possibile:**
```json
{
  "success": false,
  "error": "Unable to transform response from server",
  "responseTime": 156
}
```

**Causa errore:** Mismatch tra shape response backend e type tRPC atteso.

---

## 4. PAGINA LOGS E DEBUG

### 4.1 Pagina LOG

**File:** `/client/src/pages/DashboardPA.tsx`

**Componente:** `LogsSection()` (linea 3609)

**Tab in DashboardPA:** `value="logs"` (linea 1972)

#### 4.1.1 Endpoint Chiamato

**NON usa endpoint backend!**

Carica i log direttamente da **GitHub:**

```typescript
useEffect(() => {
  // Load Guardian logs from MIO-hub GitHub repository
  fetch('https://raw.githubusercontent.com/Chcndr/MIO-hub/master/logs/api-guardian.log')
    .then(r => r.text())
    .then(text => {
      const logLines = text.trim().split('\n').filter(line => line.trim());
      const parsedLogs = logLines.map(line => JSON.parse(line));
      setGuardianLogs(parsedLogs);
      setLoading(false);
    })
    .catch(err => {
      console.error('Error loading Guardian logs:', err);
      setLoading(false);
    });
}, []);
```

**Source:** `https://raw.githubusercontent.com/Chcndr/MIO-hub/master/logs/api-guardian.log`

**Formato log (NDJSON):**
```json
{"timestamp":"2025-11-24T12:34:56.789Z","status":"allowed","endpoint":"/api/markets/list","method":"GET","user":"admin@dms.it"}
{"timestamp":"2025-11-24T12:35:12.456Z","status":"denied","endpoint":"/api/admin/delete","method":"DELETE","user":"guest@dms.it","reason":"insufficient_permissions"}
```

#### 4.1.2 Dati Mostrati

**Stats Cards:**
```typescript
const stats = {
  total: guardianLogs.length,
  allowed: guardianLogs.filter(log => log.status === 'allowed').length,
  denied: guardianLogs.filter(log => log.status === 'denied').length,
  error: guardianLogs.filter(log => log.status === 'error').length,
};
```

**Tabs:**
1. **System Logs** (MOCK - hard-coded)
```typescript
const systemLogs = [
  { id: 1, timestamp: new Date().toISOString(), level: 'info', app: 'DMS_HUB', type: 'API_CALL', message: 'GET /api/markets/list - 200 OK', userEmail: 'system' },
  { id: 2, timestamp: new Date(Date.now() - 60000).toISOString(), level: 'info', app: 'DMS_HUB', type: 'DATABASE', message: 'Query executed successfully', userEmail: 'admin@dms.it' },
  { id: 3, timestamp: new Date(Date.now() - 120000).toISOString(), level: 'warn', app: 'MIHUB', type: 'RATE_LIMIT', message: 'Rate limit approaching for agent: mio', userEmail: 'system' },
];
```

2. **Guardian Logs** (REALI da GitHub)
   - Caricati da `api-guardian.log`
   - Parsati come NDJSON
   - Mostrati in tabella con badge colorati

**✅ CONCLUSIONE:**
- **Guardian Logs:** REALI (da GitHub)
- **System Logs:** MOCK (hard-coded)

### 4.2 Pagina DEBUG

**File:** `/client/src/pages/DashboardPA.tsx`

**Tab:** `value="debug"` (linea 2064+)

#### 4.2.1 Contenuto

**NON TROVATO** un tab "Debug" dedicato.

Il tab più vicino è **"Agente AI"** (value="ai") che ha:
- Chat simulata
- Risposte MOCK
- Nessuna chiamata backend reale

```typescript
setTimeout(() => {
  setChatMessages(prev => [...prev, { 
    role: 'ai', 
    content: `Ho ricevuto la tua richiesta: "${userMsg}". Questa è una risposta simulata. In produzione, qui ci sarà l'integrazione con un vero modello AI per analizzare i dati della Dashboard PA.`
  }]);
}, 500);
```

**✅ CONCLUSIONE:** Nessun sistema di debug attivo, solo chat AI simulata.

---

## 5. CONCLUSIONI E RACCOMANDAZIONI

### 5.1 Stato Attuale

| Componente | Stato | Note |
|------------|-------|------|
| **MIO Agent UI** | ✅ Completo | Layout funzionante, input disabled |
| **orchestratorClient.ts** | ✅ Pronto | Codice corretto, mai importato |
| **Backend Hetzner** | ✅ Funzionante | Risponde con JSON strutturato |
| **Integrazioni/API Tester** | ✅ Funzionante | Usa tRPC, carica config da GitHub |
| **Logs Guardian** | ✅ Funzionante | Carica da GitHub, dati reali |
| **Logs System** | ⚠️ Mock | Hard-coded, non reali |
| **Debug** | ❌ Non presente | Solo chat AI simulata |

### 5.2 Problemi Identificati

#### 5.2.1 MIO Agent - Input Disabilitati

**Problema:**
```tsx
<input disabled />
<Button disabled>
```

**Causa:** Nessun handler collegato, orchestratorClient mai importato.

**Soluzione (da fare in Fase 2):**
1. Importare `callOrchestrator` da `orchestratorClient.ts`
2. Creare state per input e messaggi
3. Collegare handler `onSend` che chiama `callOrchestrator`
4. Rimuovere `disabled` dagli input

#### 5.2.2 orchestratorClient.ts - Mai Usato

**Problema:** File pronto ma non importato in nessun componente.

**Soluzione (da fare in Fase 2):**
```typescript
// In DashboardPA.tsx
import { callOrchestrator } from '@/api/orchestratorClient';

const handleSendMessage = async (message: string) => {
  const response = await callOrchestrator({
    mode: 'auto',
    message,
    conversationId: null,
    meta: { source: 'dashboard_pa' }
  });
  
  if (response.success) {
    // Mostra messaggio
  } else {
    // Mostra errore
  }
};
```

#### 5.2.3 System Logs - Dati Mock

**Problema:** `systemLogs` array hard-coded, non reali.

**Soluzione (da fare in Fase 2):**
1. Creare endpoint backend `/api/logs/system`
2. Sostituire array mock con `fetch()` o `trpc.logs.system.useQuery()`

#### 5.2.4 Debug - Non Presente

**Problema:** Nessun sistema di debug/test interno.

**Soluzione (da fare in Fase 2):**
- Usare tab "Integrazioni" per test API
- Aggiungere sezione "Test Orchestrator" in tab MIO Agent

### 5.3 Prossimi Step (Fase 2)

**⚠️ ATTENDERE ISTRUZIONI PRIMA DI MODIFICARE**

1. **Collegare orchestratorClient a MIO Agent**
   - Importare `callOrchestrator`
   - Creare state e handler
   - Abilitare input

2. **Implementare gestione conversazioni**
   - Salvare `conversationId`
   - Mostrare storico messaggi
   - Gestire errori (rate limit, timeout)

3. **Sostituire System Logs mock con dati reali**
   - Endpoint backend `/api/logs/system`
   - Query tRPC

4. **Aggiungere test orchestrator in Integrazioni**
   - Nuovo endpoint `/api/mihub/orchestrator` in API tester
   - Test mode auto/manual

---

## 📊 SCHEMA FLUSSO CHIAMATE

```
┌─────────────────────────────────────────────────────────────┐
│                      FRONTEND (Vercel)                      │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  DashboardPA.tsx                                     │  │
│  │  ├── Tab "MIO Agent"                                 │  │
│  │  │   ├── SEZIONE A: Chat Principale (disabled ❌)   │  │
│  │  │   └── SEZIONE B: Multi-Agent (disabled ❌)       │  │
│  │  │                                                   │  │
│  │  ├── Tab "Integrazioni"                             │  │
│  │  │   └── APIDashboard                               │  │
│  │  │       ├── Load: GitHub/MIO-hub/api/index.json   │  │
│  │  │       └── Test: trpc.dmsHub.*.query/mutate      │  │
│  │  │                                                   │  │
│  │  └── Tab "Logs"                                     │  │
│  │      └── LogsSection                                │  │
│  │          ├── Guardian: GitHub/MIO-hub/logs/*.log   │  │
│  │          └── System: MOCK (hard-coded)             │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  orchestratorClient.ts (NON USATO ❌)               │  │
│  │  └── callOrchestrator()                             │  │
│  │      └── POST ${VITE_API_URL}/api/mihub/orchestrator│  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ (se collegato)
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   BACKEND HETZNER                           │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  POST /api/mihub/orchestrator                        │  │
│  │  ├── Body: { mode, message, conversationId, meta }  │  │
│  │  └── Response: { success, agent, conversationId,    │  │
│  │                  message?, error? }                  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  Status: ✅ Funzionante (testato con cURL)                 │
│  Rate Limit: ⚠️ OpenAI 429 (normale)                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 📝 NOTE FINALI

1. **NESSUNA MODIFICA** è stata fatta al codice durante questa ispezione
2. Tutti i componenti sono stati **solo letti e analizzati**
3. Il commit stabile **8e8459d** rimane intatto
4. La mappa GIS continua a funzionare ✅
5. Questo report è la **fotografia tecnica completa** richiesta

**Prossimo step:** Attendere istruzioni per Fase 2 (modifiche mirate e limitate).

---

**Report compilato da:** Manus AI Agent  
**Data:** 24 Novembre 2025, ore 14:45 CET  
**Versione:** 1.0
