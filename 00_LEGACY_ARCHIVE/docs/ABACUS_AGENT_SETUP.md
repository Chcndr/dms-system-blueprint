# ABACUS - Agente Analisi Dati

## 📋 Panoramica

**ABACUS** è l'agente specializzato nell'analisi dati e nella lettura di repository del sistema DMS Hub. Fornisce capacità di analisi architetturale, lettura di blueprint e accesso ai repository GitHub.

## 🎯 Stato Attuale

**Data aggiornamento:** 27 Novembre 2025

### ✅ Funzionalità Operative

1. **Chat UI Multi-Agente**
   - Input chat funzionante
   - Selezione agente Abacus operativa
   - Invio e ricezione messaggi correttamente implementati
   - Loading indicator attivo

2. **Backend GitHub Proxy**
   - Endpoint `/api/abacus/github/list` - Lista file repository ✅
   - Endpoint `/api/abacus/github/get` - Leggi contenuto file ✅
   - Endpoint `/api/abacus/github/update` - Aggiorna/crea file ✅
   - Token GitHub configurato correttamente nei secrets ✅

3. **Agente Orchestratore**
   - Abacus risponde tramite `gemini_arch`
   - Conversazioni persistenti con `conversationId`
   - Routing messaggi funzionante

### ⚠️ Limitazioni Attuali

**Integrazione GitHub Proxy**
- Gli endpoint GitHub Proxy sono operativi ma **non ancora integrati** nell'agente
- Abacus richiede copy-paste manuale invece di leggere direttamente dai repository
- Serve configurare il prompt di sistema per usare gli endpoint disponibili

## 🔧 Architettura

### Frontend (dms-hub-app-new)

**File:** `client/src/pages/DashboardPA.tsx`

**Stati Abacus:**
```typescript
const [abacusMessages, setAbacusMessages] = useState<Array<{ role: 'user' | 'assistant' | 'system'; text: string; agent?: string }>>([]);
const [abacusInputValue, setAbacusInputValue] = useState('');
const [abacusLoading, setAbacusLoading] = useState(false);
const [abacusError, setAbacusError] = useState<string | null>(null);
const [abacusConversationId, setAbacusConversationId] = useState<string | null>(null);
```

**Handler:**
```typescript
const handleSendAbacus = async () => {
  // Invia messaggio all'orchestratore
  const response = await callOrchestrator({
    mode: 'auto',
    conversationId: abacusConversationId,
    message: text,
    meta: { source: 'dashboard_abacus', agent: 'abacus' },
  });
};
```

### Backend (mihub-backend-rest)

**File:** `routes/abacus.js`

**Endpoint disponibili:**

#### 1. POST /api/abacus/github/list
Lista file e directory in un repository.

**Request:**
```json
{
  "owner": "Chcndr",
  "repo": "dms-system-blueprint",
  "path": ""
}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "name": "README.md",
      "path": "README.md",
      "type": "file",
      "size": 3503,
      "sha": "ee6c9a9...",
      "url": "https://api.github.com/...",
      "html_url": "https://github.com/...",
      "download_url": "https://raw.githubusercontent.com/..."
    }
  ]
}
```

#### 2. POST /api/abacus/github/get
Legge il contenuto di un file dal repository.

**Request:**
```json
{
  "owner": "Chcndr",
  "repo": "dms-system-blueprint",
  "path": "README.md"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "name": "README.md",
    "path": "README.md",
    "sha": "ee6c9a9...",
    "size": 3503,
    "content": "IyBETVMgU3lzdGVtIEJsdWVwcmludA...",
    "encoding": "base64",
    "decodedContent": "# DMS System Blueprint\n\n..."
  }
}
```

#### 3. POST /api/abacus/github/update
Crea o aggiorna un file nel repository.

**Request:**
```json
{
  "owner": "Chcndr",
  "repo": "dms-system-blueprint",
  "path": "reports/new_report.md",
  "content": "# New Report\n\nContent here...",
  "message": "Add new report",
  "sha": "optional_sha_for_update"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "commit": {
      "sha": "abc123...",
      "message": "Add new report"
    },
    "content": {
      "name": "new_report.md",
      "path": "reports/new_report.md",
      "sha": "def456..."
    }
  }
}
```

## 🔐 Configurazione Token GitHub

**Percorso:** Impostazioni → API & Token → GitHub Personal Access Token

**Permessi richiesti:**
- `repo` - Accesso completo ai repository privati
  - `repo:status` - Accesso allo stato dei commit
  - `repo_deployment` - Accesso ai deployment
  - `public_repo` - Accesso ai repository pubblici
  - `repo:invite` - Accesso agli inviti ai repository

**Storage:**
- Token salvato cifrato (AES-256-GCM) nel database
- Recuperato tramite `secrets.getSecret('GITHUB_PERSONAL_ACCESS_TOKEN', 'repo')`

## 🧪 Test Endpoint

### Test Lista File
```bash
curl -s -X POST "https://mihub.157-90-29-66.nip.io/api/abacus/github/list" \
  -H "Content-Type: application/json" \
  -d '{"owner":"Chcndr","repo":"dms-system-blueprint","path":""}' | jq '.'
```

### Test Lettura File
```bash
curl -s -X POST "https://mihub.157-90-29-66.nip.io/api/abacus/github/get" \
  -H "Content-Type: application/json" \
  -d '{"owner":"Chcndr","repo":"dms-system-blueprint","path":"README.md"}' | jq '.success, .data.name, .data.size'
```

### Test Aggiornamento File
```bash
curl -s -X POST "https://mihub.157-90-29-66.nip.io/api/abacus/github/update" \
  -H "Content-Type: application/json" \
  -d '{
    "owner":"Chcndr",
    "repo":"dms-system-blueprint",
    "path":"test.md",
    "content":"# Test\n\nThis is a test file.",
    "message":"Test update from Abacus"
  }' | jq '.'
```

## 📝 TODO - Prossimi Passi

### 1. Integrare GitHub Proxy nell'Agente

**File da modificare:** Configurazione agente `gemini_arch` nell'orchestratore

**Aggiungere al prompt di sistema:**
```
Hai accesso ai seguenti endpoint per leggere repository GitHub:

1. Lista file: POST /api/abacus/github/list
   Body: {"owner":"Chcndr","repo":"nome-repo","path":"percorso/opzionale"}

2. Leggi file: POST /api/abacus/github/get
   Body: {"owner":"Chcndr","repo":"nome-repo","path":"percorso/file.md"}

3. Aggiorna file: POST /api/abacus/github/update
   Body: {"owner":"Chcndr","repo":"nome-repo","path":"file.md","content":"contenuto","message":"commit message"}

Quando l'utente chiede di leggere un file da un repository, usa questi endpoint invece di chiedere il copy-paste.
```

### 2. Aggiungere Tool Calling

Configurare l'agente per usare function calling con gli endpoint GitHub:

```typescript
const tools = [
  {
    name: "github_list_files",
    description: "Lista file e directory in un repository GitHub",
    parameters: {
      type: "object",
      properties: {
        owner: { type: "string", description: "Owner del repository (es. Chcndr)" },
        repo: { type: "string", description: "Nome del repository" },
        path: { type: "string", description: "Percorso opzionale (default: root)" }
      },
      required: ["owner", "repo"]
    }
  },
  {
    name: "github_read_file",
    description: "Legge il contenuto di un file da un repository GitHub",
    parameters: {
      type: "object",
      properties: {
        owner: { type: "string" },
        repo: { type: "string" },
        path: { type: "string", description: "Percorso completo del file" }
      },
      required: ["owner", "repo", "path"]
    }
  }
];
```

### 3. Test End-to-End

Dopo l'integrazione, testare con:
```
Utente: "Abacus, leggi il file README.md del repository dms-system-blueprint"

Risposta attesa:
- Abacus chiama POST /api/abacus/github/get
- Riceve il contenuto del file
- Analizza e risponde con un summary
```

## 📊 Metriche

**Endpoint GitHub Proxy:**
- ✅ Uptime: 100%
- ✅ Response time: < 500ms
- ✅ Success rate: 100% (test manuali)

**Chat UI:**
- ✅ Input funzionante
- ✅ Messaggi inviati/ricevuti
- ✅ Loading indicator

## 🔗 Link Utili

- Dashboard PA: https://dms-hub-app-new.vercel.app/dashboard-pa
- Backend API: https://mihub.157-90-29-66.nip.io
- Repository Frontend: https://github.com/Chcndr/dms-hub-app-new
- Repository Backend: https://github.com/Chcndr/mihub-backend-rest
- Repository Blueprint: https://github.com/Chcndr/dms-system-blueprint

## 📞 Supporto

Per problemi o domande su Abacus:
1. Verificare che il token GitHub sia configurato correttamente
2. Testare gli endpoint manualmente con curl
3. Controllare i log Guardian per errori di autenticazione
4. Verificare che l'agente gemini_arch sia attivo nell'orchestratore
