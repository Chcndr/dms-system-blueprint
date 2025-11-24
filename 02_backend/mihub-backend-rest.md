# MIHUB Backend REST - Repository e Deployment

**Data Creazione:** 24 Novembre 2025  
**Versione:** 2.0.0  
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
*   **Descrizione:** Backend REST API per DMS MIO-HUB - Gestione Mercati, Guardian, Logging e Sistema Multi-Agente

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
```

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
├── routes/
│   ├── logs.js              # 5 endpoint logging
│   ├── guardian.js          # 3 endpoint Guardian
│   ├── mihub.js             # 3 endpoint Tasks/Deploy
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

### 4.4. Gestione Mercati (`/api/markets/*`, `/api/stalls/*`, `/api/vendors/*`, `/api/concessions/*`)

Endpoint per la gestione completa di mercati, posteggi, operatori e concessioni.

### 4.5. GIS (`/api/gis/*`)

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

---

## 6. Flusso di Deploy

### 6.1. Workflow Standard

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

### 6.2. Rollback

```bash
# Su Hetzner
cd /root/mihub-backend-rest
git log --oneline  # trova commit precedente
git reset --hard <commit_hash>
pm2 restart mihub-backend
```

---

## 7. Sicurezza

### 7.1. Controllo Permessi

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

### 7.2. Rate Limiting

*   **Limite:** 100 richieste per 15 minuti
*   **Implementazione:** `express-rate-limit`

### 7.3. CORS

*   **Origini consentite:** Dashboard PA, App Cittadini, App Operatore, localhost
*   **Credentials:** Abilitati
*   **Headers custom:** `x-agent-id`, `x-confirm-action`

---

## 8. Monitoraggio

### 8.1. Health Check

```bash
curl https://orchestratore.mio-hub.me/health
```

**Risposta:**
```json
{
  "status": "ok",
  "timestamp": "2025-11-24T...",
  "service": "mihub-backend-rest",
  "version": "2.0.0",
  "uptime": 123456,
  "features": {
    "gis": true,
    "markets": true,
    "guardian": true,
    "logging": true,
    "tasks": true
  }
}
```

### 8.2. Logs PM2

```bash
pm2 logs mihub-backend --lines 100
```

### 8.3. Database Logs

```bash
curl https://orchestratore.mio-hub.me/api/logs/stats
```

---

## 9. Troubleshooting

### 9.1. Backend non risponde

```bash
# Verifica processo
pm2 status

# Restart
pm2 restart mihub-backend

# Logs
pm2 logs mihub-backend --err
```

### 9.2. Errore database

```bash
# Verifica connessione PostgreSQL
psql -U postgres -d dms_hub -c "SELECT COUNT(*) FROM mio_agent_logs;"

# Ricrea schema
curl -X POST https://orchestratore.mio-hub.me/api/logs/initSchema
```

### 9.3. Errore CORS

Verifica che il frontend usi `VITE_API_URL=https://orchestratore.mio-hub.me` e che l'origine sia in `CORS_ORIGINS` nel `.env`.

---

## 10. Prossimi Step

1.  **Configurare .env su Hetzner** con credenziali reali
2.  **Inizializzare database** con `/api/logs/initSchema`
3.  **Testare tutti gli endpoint** dalla Dashboard PA
4.  **Configurare backup database** (cron job)
5.  **Implementare monitoring** (Sentry, LogRocket)
6.  **Aggiungere autenticazione JWT** per sicurezza avanzata

---

**Ultimo aggiornamento:** 24 Novembre 2025  
**Versione documento:** 1.0
