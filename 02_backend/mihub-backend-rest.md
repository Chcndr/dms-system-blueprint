# MIHUB Backend REST - Repository e Deployment

**Data Creazione:** 24 Novembre 2025  
**Versione:** 3.0.0  
**Repository:** https://github.com/Chcndr/mihub-backend-rest  
**Ambiente:** Produzione (Hetzner VPS)

---

## 1. Informazioni Repository

### 1.1. Dettagli GitHub

*   **Nome:** `mihub-backend-rest`
*   **Owner:** `Chcndr`
*   **URL:** https://github.com/Chcndr/mihub-backend-rest
*   **Visibilità:** Privato
*   **Branch principale:** `master`
*   **Descrizione:** Backend REST API per DMS MIO-HUB - Gestione Mercati, Guardian, Logging, Sistema Multi-Agente e Gestione Secrets

### 1.2. Separazione Repository

Il sistema DMS Hub è composto da **3 repository separati**:

| Repository | Scopo | Ambiente | URL |
|------------|-------|----------|-----|
| **mihub-backend-rest** | Backend REST API | Hetzner VPS | https://orchestratore.mio-hub.me |
| **dms-hub-app-new** | Dashboard PA (Frontend) | Vercel | https://dms-hub-app-new.vercel.app |
| **MIO-hub** | Config multi-agente, Guardian, Blueprint | GitHub | https://github.com/Chcndr/MIO-hub |

**Importante:** NON integrare il codice backend in MIO-hub o dms-hub-app-new. Mantenere i 3 repository separati.

---

## 2. Deployment su Hetzner

### 2.1. Path sul Server

```bash
/root/mihub-backend-rest
```

### 2.2. Configurazione Ambiente

**File:** `/root/mihub-backend-rest/.env`

```env
# Server Configuration
PORT=3001
NODE_ENV=production
BASE_URL=https://mihub.157-90-29-66.nip.io

# Database Configuration (PostgreSQL)
DB_HOST=localhost
DB_PORT=5432
DB_NAME=dms_hub
DB_USER=postgres
DB_PASSWORD=<password_reale>

# CORS Configuration
CORS_ORIGINS=https://dms-hub-app-new.vercel.app,https://dms-cittadini.vercel.app,http://localhost:5173

# Vercel Configuration (for deploy endpoint)
VERCEL_TOKEN=<token_vercel>
VERCEL_TEAM_ID=<team_id>

# Guardian Configuration
LOG_FILE=/root/MIO-hub/logs/api-guardian.log

# Secrets System Configuration
SECRETS_MASTER_KEY=<chiave_master_per_cifratura_AES256>

# Orchestrator Configuration
ORCHESTRATOR_ENABLED=false
```

**⚠️ IMPORTANTE - Secrets Master Key:**
La `SECRETS_MASTER_KEY` è la chiave master usata per cifrare/decifrare tutti i token salvati nel database. **NON DEVE MAI** essere committata su GitHub. Se persa, tutti i token salvati diventano irrecuperabili.

### 2.3. Processo PM2

**Nome processo:** `mihub-backend`

**Comandi:**
```bash
# Start
pm2 start index.js --name mihub-backend

# Restart
pm2 restart mihub-backend

# Logs
pm2 logs mihub-backend

# Status
pm2 status
```

### 2.4. Nginx Proxy

**Dominio:** https://orchestratore.mio-hub.me

**Configurazione:** `/etc/nginx/sites-available/orchestratore.mio-hub.me.conf`

```nginx
server {
    listen 443 ssl http2;
    server_name orchestratore.mio-hub.me;

    ssl_certificate /etc/letsencrypt/live/orchestratore.mio-hub.me/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/orchestratore.mio-hub.me/privkey.pem;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## 3. Struttura del Progetto

```
mihub-backend-rest/
├── config/
│   └── database.js          # Configurazione PostgreSQL
├── modules/
│   ├── secrets.js           # Sistema gestione secrets (AES-256-GCM)
│   ├── llm.js               # Integrazione LLM (GPT-5.1, Gemini 3 Pro)
│   └── orchestrator.js      # Logica orchestratore multi-agente
├── routes/
│   ├── logs.js              # 5 endpoint logging
│   ├── guardian.js          # 3 endpoint Guardian
│   ├── mihub.js             # 3 endpoint Tasks/Deploy
│   ├── orchestrator.js      # 2 endpoint orchestratore
│   ├── admin.js             # 3 endpoint secrets (admin only)
│   ├── markets.js           # API Gestione Mercati
│   ├── stalls.js            # API Posteggi
│   ├── vendors.js           # API Operatori
│   ├── concessions.js       # API Concessioni
│   └── gis.js               # API GIS
├── index.js                 # Entry point
├── package.json             # Dipendenze
├── .env.example             # Template configurazione
└── README.md                # Documentazione
```

---

## 4. API Endpoints

### 4.1. Logging (`/api/logs/*`)

| Endpoint | Metodo | Descrizione |
|----------|--------|-------------|
| `/api/logs/initSchema` | POST | Crea tabella `mio_agent_logs` |
| `/api/logs/createLog` | POST | Scrive un log nel database |
| `/api/logs/getLogs` | GET | Legge log con filtri |
| `/api/logs/stats` | GET | Statistiche aggregate |
| `/api/logs/clear` | DELETE | Svuota log (solo dev) |

### 4.2. Guardian (`/api/guardian/*`)

| Endpoint | Metodo | Descrizione |
|----------|--------|-------------|
| `/api/guardian/debug/testEndpoint` | POST | Testa endpoint e logga risultato |
| `/api/guardian/health` | GET | Health check Guardian |
| `/api/guardian/permissions` | GET | Ritorna permessi agenti |

### 4.3. MIHUB Tasks/Deploy (`/api/mihub/*`)

| Endpoint | Metodo | Descrizione |
|----------|--------|-------------|
| `/api/mihub/tasks` | GET | Lista task con filtri |
| `/api/mihub/tasks` | POST | Crea nuovo task |
| `/api/mihub/deploy/vercel` | POST | Trigger deploy Vercel |

### 4.4. Orchestratore Multi-Agente (`/api/orchestrator/*`)

| Endpoint | Metodo | Descrizione |
|----------|--------|-------------|
| `/api/orchestrator/run` | POST | Esegue task con routing agenti |
| `/api/orchestrator/conversations/:id` | GET | Recupera conversazione agente |

**Stato:** Implementato ma disabilitato (`ORCHESTRATOR_ENABLED=false`) fino a risoluzione problemi dipendenze.

### 4.5. Secrets Management (`/admin/secrets/*`)

| Endpoint | Metodo | Descrizione |
|----------|--------|-------------|
| `/admin/secrets` | POST | Salva un secret cifrato |
| `/admin/secrets` | GET | Lista tutti i secrets (solo nomi) |
| `/admin/secrets/:name` | DELETE | Elimina un secret |

**⚠️ Sicurezza:** Questi endpoint sono protetti da Guardian con `deny` per tutti gli agenti LLM. Solo admin umani possono accedere.

**Tipi di secrets supportati:**
- `openai_api_key` - OpenAI API Key
- `gemini_api_key` - Google Gemini API Key
- `anthropic_api_key` - Anthropic Claude API Key
- `github_token` - GitHub Personal Access Token
- `vercel_token` - Vercel Deploy Token

### 4.6. Gestione Mercati (`/api/markets/*`, `/api/stalls/*`, `/api/vendors/*`, `/api/concessions/*`)

Endpoint per la gestione completa di mercati, posteggi, operatori e concessioni.

### 4.7. GIS (`/api/gis/*`)

Endpoint per il sistema GIS (mappe, geometrie, marker).

---

## 5. Database PostgreSQL

### 5.1. Tabella `mio_agent_logs`

**Schema:**
```sql
CREATE TABLE mio_agent_logs (
  id SERIAL PRIMARY KEY,
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  agent VARCHAR(50) NOT NULL,
  service_id VARCHAR(100),
  endpoint VARCHAR(255) NOT NULL,
  method VARCHAR(10) NOT NULL,
  status_code INTEGER,
  risk VARCHAR(20),
  success BOOLEAN DEFAULT true,
  message TEXT,
  meta_json JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_mio_agent_logs_agent ON mio_agent_logs(agent);
CREATE INDEX idx_mio_agent_logs_service_id ON mio_agent_logs(service_id);
CREATE INDEX idx_mio_agent_logs_timestamp ON mio_agent_logs(timestamp DESC);
CREATE INDEX idx_mio_agent_logs_success ON mio_agent_logs(success);
```

**Inizializzazione:**
```bash
curl -X POST https://orchestratore.mio-hub.me/api/logs/initSchema
```

### 5.2. Tabella `mihub_tasks`

**Schema:**
```sql
CREATE TABLE mihub_tasks (
  id SERIAL PRIMARY KEY,
  project VARCHAR(100) NOT NULL,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  type VARCHAR(50) DEFAULT 'code-change',
  status VARCHAR(50) DEFAULT 'pending',
  priority VARCHAR(20) DEFAULT 'medium',
  assigned_agent VARCHAR(50),
  created_by VARCHAR(50),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP WITH TIME ZONE,
  meta_json JSONB
);

CREATE INDEX idx_mihub_tasks_project ON mihub_tasks(project);
CREATE INDEX idx_mihub_tasks_status ON mihub_tasks(status);
CREATE INDEX idx_mihub_tasks_assigned_agent ON mihub_tasks(assigned_agent);
```

**Auto-creazione:** La tabella si crea automaticamente al primo POST `/api/mihub/tasks`.

### 5.3. Tabella `secrets`

**Schema:**
```sql
CREATE TABLE secrets (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) UNIQUE NOT NULL,
  encrypted_value TEXT NOT NULL,
  iv TEXT NOT NULL,
  salt TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_secrets_name ON secrets(name);
```

**Sicurezza:**
- Valori cifrati con AES-256-GCM
- IV (Initialization Vector) unico per ogni secret
- Salt per derivazione chiave con PBKDF2
- Master key mai salvata nel database

### 5.4. Tabella `agent_conversations`

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
```

### 5.5. Tabella `agent_messages`

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
```

---

## 6. Sistema Secrets ("Cassetto Token")

### 6.1. Architettura

Il sistema secrets implementa un **vault cifrato** per salvare API key e token in modo sicuro nel database PostgreSQL.

**Componenti:**
1. **Modulo `secrets.js`:** Logica di cifratura/decifratura
2. **Route `/admin/secrets`:** API REST per gestione secrets
3. **Tabella `secrets`:** Storage cifrato nel database
4. **Guardian:** Protezione con `deny` su `admin.secrets.*` per agenti LLM

### 6.2. Cifratura

**Algoritmo:** AES-256-GCM (Galois/Counter Mode)
**Derivazione chiave:** PBKDF2 con 100.000 iterazioni
**IV:** Generato casualmente per ogni secret (16 bytes)
**Salt:** Generato casualmente per ogni secret (32 bytes)

**Flusso di cifratura:**
```
Master Key + Salt → PBKDF2 → Derived Key
Derived Key + IV + Plaintext → AES-256-GCM → Ciphertext + Auth Tag
```

**Flusso di decifratura:**
```
Master Key + Salt → PBKDF2 → Derived Key
Derived Key + IV + Ciphertext + Auth Tag → AES-256-GCM → Plaintext
```

### 6.3. API Usage

**Salvare un secret:**
```bash
curl -X POST https://orchestratore.mio-hub.me/admin/secrets \
  -H "Content-Type: application/json" \
  -d '{
    "name": "openai_api_key",
    "value": "sk-proj-..."
  }'
```

**Listare secrets:**
```bash
curl https://orchestratore.mio-hub.me/admin/secrets
```

**Risposta:**
```json
{
  "success": true,
  "secrets": [
    {
      "name": "openai_api_key",
      "last4": "abcd",
      "created_at": "2025-11-24T..."
    }
  ]
}
```

**Eliminare un secret:**
```bash
curl -X DELETE https://orchestratore.mio-hub.me/admin/secrets/openai_api_key
```

### 6.4. Integrazione con LLM

Il modulo `llm.js` usa automaticamente i secrets salvati:

```javascript
const { getSecret } = require('./secrets');

async function callOpenAI(prompt) {
  const apiKey = await getSecret('openai_api_key');
  // ... usa apiKey per chiamare OpenAI
}
```

---

## 7. Orchestratore Multi-Agente

### 7.1. Architettura

L'orchestratore implementa un sistema di **routing intelligente** che assegna task agli agenti appropriati in base alle loro competenze.

**Componenti:**
1. **Modulo `orchestrator.js`:** Logica di routing e gestione conversazioni
2. **Route `/api/orchestrator/run`:** Endpoint principale
3. **Tabelle `agent_conversations` e `agent_messages`:** Storage conversazioni
4. **Modulo `llm.js`:** Integrazione con GPT-5.1 e Gemini 3 Pro

### 7.2. Agenti Configurati

| Agente | LLM | Competenze | Permessi Guardian |
|--------|-----|------------|-------------------|
| **mio** | GPT-5.1 | Coordinamento, decisioni strategiche | `allow` su orchestrator, tasks |
| **dev** | GPT-5.1 | Sviluppo, debugging, deploy | `allow` su deploy, `deny` su secrets |
| **manus_worker** | - | Esecuzione task operativi | `allow` su logs, markets, GIS |
| **gemini_arch** | Gemini 3 Pro | Architettura, design system | `allow` su guardian, permissions |

### 7.3. Flusso di Esecuzione

```
1. POST /api/orchestrator/run con task description
2. Orchestratore analizza task e seleziona agente
3. Crea conversazione in agent_conversations
4. Chiama LLM con prompt + contesto agente
5. Salva messaggi in agent_messages
6. Ritorna risposta agente
```

### 7.4. API Usage

**Eseguire un task:**
```bash
curl -X POST https://orchestratore.mio-hub.me/api/orchestrator/run \
  -H "Content-Type: application/json" \
  -d '{
    "task": "Analizza i log Guardian degli ultimi 7 giorni",
    "agent": "mio"
  }'
```

**Risposta:**
```json
{
  "success": true,
  "conversation_id": 42,
  "agent": "mio",
  "response": "Ho analizzato i log...",
  "metadata": {
    "model": "gpt-5.1-mini",
    "tokens": 1234
  }
}
```

**Recuperare conversazione:**
```bash
curl https://orchestratore.mio-hub.me/api/orchestrator/conversations/42
```

### 7.5. Stato Attuale

**⚠️ DISABILITATO:** L'orchestratore è implementato ma disabilitato (`ORCHESTRATOR_ENABLED=false`) a causa di un problema con la dipendenza `uuid` che causava crash del backend.

**Prossimo step:** Risolvere il problema `uuid` e riabilitare l'orchestratore.

---

## 8. Flusso di Deploy

### 8.1. Workflow Standard

```bash
# 1. Sviluppo locale
cd /home/ubuntu/mihub-backend-rest
# ... modifiche al codice ...

# 2. Commit e push su GitHub
git add -A
git commit -m "feat: descrizione modifiche"
git push origin master

# 3. Deploy su Hetzner (SSH)
ssh root@157.90.29.66
cd /root/mihub-backend-rest
git pull origin master
npm install  # se ci sono nuove dipendenze
pm2 restart mihub-backend

# 4. Verifica
curl https://orchestratore.mio-hub.me/health
pm2 logs mihub-backend
```

### 8.2. Rollback

```bash
# Su Hetzner
cd /root/mihub-backend-rest
git log --oneline  # trova commit precedente
git reset --hard <commit_hash>
pm2 restart mihub-backend
```

---

## 9. Sicurezza

### 9.1. Controllo Permessi

Il backend implementa il controllo permessi basato su `agents/permissions.json`:

*   **Header richiesto:** `x-agent-id: <agent_name>`
*   **Validazione:** Ogni endpoint verifica i permessi dell'agente
*   **Conferma richiesta:** Azioni ad alto rischio richiedono header `x-confirm-action: true`

**Esempio:**
```bash
# Deploy (solo dev/mio con conferma)
curl -X POST https://orchestratore.mio-hub.me/api/mihub/deploy/vercel \
  -H "x-agent-id: dev" \
  -H "x-confirm-action: true" \
  -H "Content-Type: application/json" \
  -d '{"project": "dms-hub-app-new", "branch": "master"}'
```

### 9.2. Protezione Secrets

**Guardian Configuration (`MIO-hub/api/index.json`):**
```json
{
  "id": "admin.secrets.save",
  "name": "Salva Secret Cifrato",
  "endpoint": "/admin/secrets",
  "method": "POST",
  "risk": "critical",
  "permissions": {
    "mio": "deny",
    "dev": "deny",
    "manus_worker": "deny",
    "gemini_arch": "deny"
  }
}
```

**Tutti gli endpoint `/admin/secrets/*` sono protetti con `deny` per tutti gli agenti LLM.**

### 9.3. Rate Limiting

*   **Limite:** 500 richieste per 15 minuti (aumentato da 100)
*   **Implementazione:** `express-rate-limit`

### 9.4. CORS

*   **Origini consentite:** Dashboard PA, App Cittadini, App Operatore, localhost
*   **Credentials:** Abilitati
*   **Headers custom:** `x-agent-id`, `x-confirm-action`

---

## 10. Monitoraggio

### 10.1. Health Check

```bash
curl https://orchestratore.mio-hub.me/health
```

**Risposta:**
```json
{
  "status": "ok",
  "timestamp": "2025-11-24T...",
  "service": "mihub-backend-rest",
  "version": "3.0.0",
  "uptime": 123456,
  "features": {
    "gis": true,
    "markets": true,
    "guardian": true,
    "logging": true,
    "tasks": true,
    "secrets": true,
    "orchestrator": false
  }
}
```

### 10.2. Logs PM2

```bash
pm2 logs mihub-backend --lines 100
```

### 10.3. Database Logs

```bash
curl https://orchestratore.mio-hub.me/api/logs/stats
```

---

## 11. Troubleshooting

### 11.1. Backend non risponde

```bash
# Verifica processo
pm2 status

# Restart
pm2 restart mihub-backend

# Logs
pm2 logs mihub-backend --err
```

### 11.2. Errore database

```bash
# Verifica connessione PostgreSQL
psql -U postgres -d dms_hub -c "SELECT COUNT(*) FROM mio_agent_logs;"

# Ricrea schema
curl -X POST https://orchestratore.mio-hub.me/api/logs/initSchema
```

### 11.3. Errore CORS

Verifica che il frontend usi `VITE_API_URL=https://orchestratore.mio-hub.me` e che l'origine sia in `CORS_ORIGINS` nel `.env`.

### 11.4. Errore Secrets

```bash
# Verifica che SECRETS_MASTER_KEY sia configurata
ssh root@157.90.29.66
cd /root/mihub-backend-rest
grep SECRETS_MASTER_KEY .env

# Testa cifratura/decifratura
curl -X POST https://orchestratore.mio-hub.me/admin/secrets \
  -H "Content-Type: application/json" \
  -d '{"name": "test_key", "value": "test_value"}'

curl https://orchestratore.mio-hub.me/admin/secrets
```

---

## 12. Prossimi Step

1.  ✅ **Sistema Secrets** - Completato
2.  ✅ **Backend Orchestratore** - Completato (ma disabilitato)
3.  ⏳ **Risolvere problema uuid** - In corso
4.  ⏳ **Riabilitare orchestratore** - Dopo fix uuid
5.  ⏳ **Frontend API Tokens Page** - Da reintegrare con branch di feature
6.  🔜 **Integrare orchestratore in MIO HUB chat** - Fase successiva
7.  🔜 **Configurare backup database** - Cron job
8.  🔜 **Implementare monitoring** - Sentry, LogRocket
9.  🔜 **Aggiungere autenticazione JWT** - Sicurezza avanzata

---

**Ultimo aggiornamento:** 24 Novembre 2025  
**Versione documento:** 3.0  
**Autore:** Manus AI
