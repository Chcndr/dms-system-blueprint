# Orchestratore Multi-Agente MIO HUB

**Data Creazione:** 24 Novembre 2025  
**Versione:** 1.0  
**Autore:** Manus AI

---

## 1. Introduzione

L'**Orchestratore Multi-Agente** è il sistema centrale di MIO HUB che coordina l'esecuzione di task complessi distribuendoli tra agenti specializzati. Ogni agente ha competenze specifiche e accesso controllato alle API tramite il sistema Guardian.

### 1.1. Obiettivi

*   **Routing Intelligente:** Assegnare automaticamente i task agli agenti più adatti in base alle competenze
*   **Specializzazione:** Permettere ad ogni agente di concentrarsi su un dominio specifico
*   **Sicurezza:** Controllare l'accesso alle API tramite permessi granulari (Guardian)
*   **Tracciabilità:** Mantenere uno storico completo di tutte le conversazioni e decisioni
*   **Scalabilità:** Supportare l'aggiunta di nuovi agenti senza modificare il codice esistente

---

## 2. Architettura

### 2.1. Componenti

```
┌─────────────────────────────────────────────────────────────┐
│                         MIO HUB Chat                        │
│                  (Interfaccia Utente)                       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   Orchestratore Backend                     │
│              (mihub-backend-rest/orchestrator)              │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Router    │→ │ LLM Module  │→ │  Guardian   │        │
│  │  (logica)   │  │(GPT/Gemini) │  │ (permessi)  │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │          Database (PostgreSQL)                      │   │
│  │  - agent_conversations                              │   │
│  │  - agent_messages                                   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┬───────────────┐
         ▼               ▼               ▼               ▼
    ┌────────┐      ┌────────┐      ┌────────┐      ┌────────┐
    │  mio   │      │  dev   │      │ manus  │      │ gemini │
    │ (GPT)  │      │ (GPT)  │      │_worker │      │_arch   │
    └────────┘      └────────┘      └────────┘      └────────┘
```

### 2.2. Repository

| Repository | Ruolo |
|------------|-------|
| **MIO-hub** | Configurazione agenti (`agents/permissions.json`), Guardian (`api/index.json`) |
| **mihub-backend-rest** | Backend orchestratore, moduli LLM, secrets, database |
| **dms-hub-app-new** | Frontend MIO HUB Chat (da implementare) |

---

## 3. Agenti Configurati

### 3.1. Tabella Agenti

| Agente | LLM | Competenze | Permessi Guardian |
|--------|-----|------------|-------------------|
| **mio** | GPT-5.1 | Coordinamento generale, decisioni strategiche, pianificazione | `allow` su orchestrator, tasks, logs |
| **dev** | GPT-5.1 | Sviluppo software, debugging, deploy, code review | `allow` su deploy, markets, GIS; `deny` su secrets |
| **manus_worker** | - | Esecuzione task operativi, raccolta dati, report | `allow` su logs, markets, GIS, vendors |
| **gemini_arch** | Gemini 3 Pro | Architettura sistema, design patterns, documentazione | `allow` su guardian, permissions, health |

### 3.2. Configurazione Agenti

**File:** `MIO-hub/agents/permissions.json`

```json
{
  "version": 3,
  "agents": [
    {
      "id": "mio",
      "name": "MIO - Coordinatore Generale",
      "role": "coordinator",
      "llm": {
        "provider": "openai",
        "model": "gpt-5.1-mini"
      },
      "capabilities": [
        "task_planning",
        "decision_making",
        "agent_coordination"
      ],
      "permissions": {
        "orchestrator": "allow",
        "tasks": "allow",
        "logs": "allow",
        "secrets": "deny"
      }
    },
    {
      "id": "dev",
      "name": "Dev - Sviluppatore",
      "role": "developer",
      "llm": {
        "provider": "openai",
        "model": "gpt-5.1-mini"
      },
      "capabilities": [
        "code_writing",
        "debugging",
        "deployment",
        "code_review"
      ],
      "permissions": {
        "deploy": "allow",
        "markets": "allow",
        "gis": "allow",
        "secrets": "deny"
      }
    },
    {
      "id": "manus_worker",
      "name": "Manus Worker - Esecutore",
      "role": "worker",
      "llm": null,
      "capabilities": [
        "data_collection",
        "report_generation",
        "task_execution"
      ],
      "permissions": {
        "logs": "allow",
        "markets": "allow",
        "gis": "allow",
        "vendors": "allow",
        "secrets": "deny"
      }
    },
    {
      "id": "gemini_arch",
      "name": "Gemini Architect - Architetto",
      "role": "architect",
      "llm": {
        "provider": "google",
        "model": "gemini-3-pro"
      },
      "capabilities": [
        "system_design",
        "architecture_review",
        "documentation"
      ],
      "permissions": {
        "guardian": "allow",
        "permissions": "allow",
        "health": "allow",
        "secrets": "deny"
      }
    }
  ]
}
```

---

## 4. Backend Orchestratore

### 4.1. Modulo `orchestrator.js`

**Path:** `mihub-backend-rest/modules/orchestrator.js`

**Funzioni principali:**

```javascript
// Seleziona l'agente appropriato per un task
function selectAgent(taskDescription) {
  // Logica di routing basata su keywords e competenze
  if (taskDescription.includes('deploy') || taskDescription.includes('bug')) {
    return 'dev';
  }
  if (taskDescription.includes('architettura') || taskDescription.includes('design')) {
    return 'gemini_arch';
  }
  if (taskDescription.includes('report') || taskDescription.includes('dati')) {
    return 'manus_worker';
  }
  return 'mio'; // Default: coordinatore
}

// Esegue un task con l'agente selezionato
async function executeTask(agent, taskDescription, context) {
  // 1. Crea conversazione nel database
  const conversationId = await createConversation(agent, taskDescription);
  
  // 2. Recupera configurazione agente
  const agentConfig = getAgentConfig(agent);
  
  // 3. Chiama LLM con prompt + contesto
  const response = await callLLM(agentConfig, taskDescription, context);
  
  // 4. Salva messaggi nel database
  await saveMessage(conversationId, 'user', taskDescription);
  await saveMessage(conversationId, 'assistant', response);
  
  return { conversationId, response };
}
```

### 4.2. Modulo `llm.js`

**Path:** `mihub-backend-rest/modules/llm.js`

**Integrazione LLM:**

```javascript
const { getSecret } = require('./secrets');
const OpenAI = require('openai');
const { GoogleGenerativeAI } = require('@google/generative-ai');

// Chiama OpenAI GPT-5.1
async function callOpenAI(model, messages) {
  const apiKey = await getSecret('openai_api_key');
  const openai = new OpenAI({ apiKey });
  
  const response = await openai.chat.completions.create({
    model,
    messages,
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return response.choices[0].message.content;
}

// Chiama Google Gemini 3 Pro
async function callGemini(model, prompt) {
  const apiKey = await getSecret('gemini_api_key');
  const genAI = new GoogleGenerativeAI(apiKey);
  const geminiModel = genAI.getGenerativeModel({ model });
  
  const result = await geminiModel.generateContent(prompt);
  return result.response.text();
}

// Router principale
async function callLLM(agentConfig, prompt, context) {
  const { provider, model } = agentConfig.llm;
  
  if (provider === 'openai') {
    return await callOpenAI(model, [
      { role: 'system', content: context },
      { role: 'user', content: prompt }
    ]);
  }
  
  if (provider === 'google') {
    return await callGemini(model, `${context}\n\n${prompt}`);
  }
  
  throw new Error(`Provider ${provider} non supportato`);
}
```

### 4.3. API Endpoint

**Endpoint:** `POST /api/orchestrator/run`

**Request:**
```json
{
  "task": "Analizza i log Guardian degli ultimi 7 giorni e genera un report",
  "agent": "mio",
  "context": {
    "user_id": "admin@comune.it",
    "project": "dms-hub"
  }
}
```

**Response:**
```json
{
  "success": true,
  "conversation_id": 42,
  "agent": "mio",
  "response": "Ho analizzato i log Guardian...",
  "metadata": {
    "model": "gpt-5.1-mini",
    "tokens": 1234,
    "duration_ms": 3500
  }
}
```

**Endpoint:** `GET /api/orchestrator/conversations/:id`

**Response:**
```json
{
  "success": true,
  "conversation": {
    "id": 42,
    "agent_name": "mio",
    "task_description": "Analizza i log Guardian...",
    "status": "completed",
    "created_at": "2025-11-24T12:00:00Z",
    "completed_at": "2025-11-24T12:05:00Z",
    "messages": [
      {
        "role": "user",
        "content": "Analizza i log Guardian...",
        "created_at": "2025-11-24T12:00:00Z"
      },
      {
        "role": "assistant",
        "content": "Ho analizzato i log...",
        "created_at": "2025-11-24T12:05:00Z"
      }
    ]
  }
}
```

---

## 5. Database

### 5.1. Tabella `agent_conversations`

**Schema:**
```sql
CREATE TABLE agent_conversations (
  id SERIAL PRIMARY KEY,
  agent_name VARCHAR(50) NOT NULL,
  task_description TEXT NOT NULL,
  status VARCHAR(50) DEFAULT 'active',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_agent_conversations_agent ON agent_conversations(agent_name);
CREATE INDEX idx_agent_conversations_status ON agent_conversations(status);
CREATE INDEX idx_agent_conversations_created ON agent_conversations(created_at DESC);
```

**Valori `status`:**
- `active` - Conversazione in corso
- `completed` - Conversazione completata con successo
- `failed` - Conversazione fallita
- `cancelled` - Conversazione cancellata dall'utente

### 5.2. Tabella `agent_messages`

**Schema:**
```sql
CREATE TABLE agent_messages (
  id SERIAL PRIMARY KEY,
  conversation_id INTEGER REFERENCES agent_conversations(id) ON DELETE CASCADE,
  role VARCHAR(20) NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_agent_messages_conversation ON agent_messages(conversation_id);
CREATE INDEX idx_agent_messages_created ON agent_messages(created_at DESC);
```

**Valori `role`:**
- `user` - Messaggio dall'utente
- `assistant` - Risposta dall'agente LLM
- `system` - Messaggio di sistema (es. errori, notifiche)

---

## 6. Integrazione Guardian

### 6.1. Controllo Permessi

Prima di eseguire qualsiasi operazione, l'orchestratore verifica i permessi dell'agente tramite Guardian.

**Flusso:**
```
1. Agente richiede accesso a endpoint (es. /api/markets)
2. Orchestratore consulta Guardian (agents/permissions.json)
3. Guardian verifica permesso per agente + endpoint
4. Se "allow" → procede
5. Se "deny" → ritorna errore 403
```

**Esempio:**
```javascript
const { checkPermission } = require('./guardian');

async function executeAPICall(agent, endpoint, method) {
  // Verifica permesso
  const allowed = await checkPermission(agent, endpoint, method);
  
  if (!allowed) {
    throw new Error(`Agente ${agent} non ha permesso per ${method} ${endpoint}`);
  }
  
  // Esegue chiamata API
  return await fetch(endpoint, { method });
}
```

### 6.2. Logging

Tutte le operazioni dell'orchestratore vengono loggate in `mio_agent_logs`:

```json
{
  "timestamp": "2025-11-24T12:00:00Z",
  "agent": "mio",
  "service_id": "orchestrator",
  "endpoint": "/api/orchestrator/run",
  "method": "POST",
  "status_code": 200,
  "risk": "medium",
  "success": true,
  "message": "Task eseguito con successo",
  "meta_json": {
    "conversation_id": 42,
    "task": "Analizza log Guardian",
    "tokens": 1234,
    "duration_ms": 3500
  }
}
```

---

## 7. Frontend MIO HUB Chat

### 7.1. Interfaccia Utente (Da Implementare)

**Componenti:**
- `MIOHubChat.tsx` - Chat principale con input utente
- `AgentSelector.tsx` - Selezione manuale agente (opzionale)
- `ConversationHistory.tsx` - Storico conversazioni
- `AgentStatus.tsx` - Stato agenti (online/offline)

**Flusso:**
```
1. Utente scrive task nella chat
2. Frontend chiama POST /api/orchestrator/run
3. Backend seleziona agente e esegue task
4. Frontend mostra risposta in tempo reale
5. Conversazione salvata nello storico
```

### 7.2. API Client

**File:** `dms-hub-app-new/client/src/utils/orchestratorAPI.ts`

```typescript
export async function runTask(task: string, agent?: string) {
  const response = await fetch(`${API_URL}/api/orchestrator/run`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ task, agent })
  });
  
  return await response.json();
}

export async function getConversation(id: number) {
  const response = await fetch(`${API_URL}/api/orchestrator/conversations/${id}`);
  return await response.json();
}
```

---

## 8. Stato Attuale e Prossimi Step

### 8.1. Stato Attuale

**✅ Completato:**
- Configurazione agenti in `agents/permissions.json`
- Backend orchestratore (`modules/orchestrator.js`)
- Modulo LLM con integrazione GPT-5.1 e Gemini 3 Pro
- Database schema (`agent_conversations`, `agent_messages`)
- API endpoints (`/api/orchestrator/run`, `/api/orchestrator/conversations/:id`)
- Integrazione Guardian per controllo permessi
- Sistema secrets per API keys

**⚠️ Disabilitato:**
- Orchestratore disabilitato (`ORCHESTRATOR_ENABLED=false`) a causa di problema con dipendenza `uuid`

**⏳ Da Implementare:**
- Frontend MIO HUB Chat
- Risoluzione problema `uuid`
- Riabilitazione orchestratore
- Testing end-to-end

### 8.2. Prossimi Step

1. **Risolvere problema uuid** - Identificare e fixare il problema con la dipendenza `uuid` che causa crash del backend
2. **Riabilitare orchestratore** - Impostare `ORCHESTRATOR_ENABLED=true` dopo fix
3. **Implementare frontend** - Creare interfaccia MIO HUB Chat in Dashboard PA
4. **Testing** - Testare tutti i flussi con i 4 agenti
5. **Monitoraggio** - Aggiungere metriche e dashboard per monitorare performance agenti
6. **Ottimizzazione** - Migliorare logica di routing basata su ML/AI
7. **Documentazione** - Creare guide per aggiungere nuovi agenti

---

## 9. Best Practices

### 9.1. Aggiungere un Nuovo Agente

1. **Aggiornare `agents/permissions.json`:**
   ```json
   {
     "id": "nuovo_agente",
     "name": "Nuovo Agente",
     "role": "specialist",
     "llm": {
       "provider": "openai",
       "model": "gpt-5.1-mini"
     },
     "capabilities": ["capability_1", "capability_2"],
     "permissions": {
       "endpoint_1": "allow",
       "endpoint_2": "deny"
     }
   }
   ```

2. **Aggiornare Guardian (`api/index.json`):**
   Aggiungere permessi per il nuovo agente su tutti gli endpoint rilevanti.

3. **Aggiornare logica routing (`orchestrator.js`):**
   ```javascript
   if (taskDescription.includes('keyword_nuovo_agente')) {
     return 'nuovo_agente';
   }
   ```

4. **Testare:**
   ```bash
   curl -X POST https://orchestratore.mio-hub.me/api/orchestrator/run \
     -H "Content-Type: application/json" \
     -d '{"task": "Task per nuovo agente", "agent": "nuovo_agente"}'
   ```

### 9.2. Sicurezza

- **MAI committare API keys** - Usare sempre il sistema secrets
- **Verificare permessi** - Ogni operazione deve passare da Guardian
- **Loggare tutto** - Ogni azione deve essere tracciata in `mio_agent_logs`
- **Limitare rate** - Implementare rate limiting per prevenire abusi
- **Validare input** - Sanitizzare tutti gli input utente prima di passarli agli LLM

### 9.3. Performance

- **Caching** - Cachare risposte frequenti per ridurre chiamate LLM
- **Timeout** - Impostare timeout per chiamate LLM (max 30s)
- **Retry** - Implementare retry automatico per errori temporanei
- **Monitoring** - Monitorare latenza e costi LLM

---

**Ultimo aggiornamento:** 24 Novembre 2025  
**Versione documento:** 1.0  
**Autore:** Manus AI
