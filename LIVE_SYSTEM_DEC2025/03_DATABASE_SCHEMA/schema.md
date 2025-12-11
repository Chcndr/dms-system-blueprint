# Database Schema - Neon PostgreSQL

> **Schema completo del database MIO Hub**  
> Database: Neon PostgreSQL (Cloud)  
> Host: `ep-bold-silence-a2p3zcfr.eu-central-1.aws.neon.tech`

## 📋 Indice

- [Connection Info](#connection-info)
- [Tabelle Conversazioni](#tabelle-conversazioni)
- [Tabelle Logging](#tabelle-logging)
- [Tabelle Business](#tabelle-business)
- [Relazioni](#relazioni)
- [Indici e Performance](#indici-e-performance)

---

## 🔗 Connection Info

### Connection String

```
postgresql://neondb_owner:npg_lYG6JQ5Krtsi@ep-bold-silence-a2p3zcfr.eu-central-1.aws.neon.tech/neondb?sslmode=require
```

**⚠️ ATTENZIONE**: Questa connection string è sensibile e non deve essere esposta pubblicamente.

### Configurazione Backend

**File**: `/root/mihub-backend-rest/config/database.js`

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false },
  max: 20,              // Max connections nel pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 10000
});
```

---

## 💬 Tabelle Conversazioni

### agent_messages

Tabella principale che contiene **tutti i messaggi** delle 8 isole conversazionali.

**Struttura**:
```sql
CREATE TABLE agent_messages (
  id SERIAL PRIMARY KEY,
  conversation_id VARCHAR(255) NOT NULL,
  agent VARCHAR(50) NOT NULL,
  role VARCHAR(20) NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  meta_json JSONB
);
```

**Colonne**:

- **id** (integer, PK): ID univoco messaggio
- **conversation_id** (varchar): ID conversazione (es. `mio-manus-coordination`, `abacus-single`)
- **agent** (varchar): Nome agente (`mio`, `manus`, `abacus`, `gptdev`, `zapier`)
- **role** (varchar): Ruolo messaggio (`user`, `assistant`, `system`)
- **content** (text): Contenuto testuale del messaggio
- **created_at** (timestamp): Data/ora creazione messaggio
- **meta_json** (jsonb): Metadati aggiuntivi (tool calls, execution time, etc.)

**Esempi Dati**:

```sql
-- Messaggio utente in Vista Singola
INSERT INTO agent_messages (conversation_id, agent, role, content) VALUES
('manus-single', 'manus', 'user', 'Controlla lo stato del backend');

-- Risposta agente
INSERT INTO agent_messages (conversation_id, agent, role, content) VALUES
('manus-single', 'manus', 'assistant', 'Il backend è online e funzionante...');

-- Messaggio coordinamento backstage
INSERT INTO agent_messages (conversation_id, agent, role, content) VALUES
('mio-abacus-coordination', 'abacus', 'system', 'Analizza i log degli ultimi 7 giorni');
```

**Indici**:
```sql
CREATE INDEX idx_agent_messages_conversation ON agent_messages(conversation_id);
CREATE INDEX idx_agent_messages_agent ON agent_messages(agent);
CREATE INDEX idx_agent_messages_created ON agent_messages(created_at DESC);
```

**Query Comuni**:

```sql
-- Recupera tutti i messaggi di una conversazione
SELECT * FROM agent_messages 
WHERE conversation_id = 'mio-manus-coordination' 
ORDER BY created_at ASC;

-- Conta messaggi per agente
SELECT agent, COUNT(*) as total 
FROM agent_messages 
GROUP BY agent;

-- Ultimi 50 messaggi
SELECT * FROM agent_messages 
ORDER BY created_at DESC 
LIMIT 50;
```

---

### agent_conversations

Tabella che traccia le conversazioni attive e il loro stato.

**Struttura**:
```sql
CREATE TABLE agent_conversations (
  id SERIAL PRIMARY KEY,
  conversation_id VARCHAR(255) UNIQUE NOT NULL,
  agent VARCHAR(50) NOT NULL,
  status VARCHAR(20) DEFAULT 'active',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  meta_json JSONB
);
```

**Colonne**:

- **id** (integer, PK): ID univoco conversazione
- **conversation_id** (varchar, unique): ID conversazione univoco
- **agent** (varchar): Agente principale della conversazione
- **status** (varchar): Stato (`active`, `archived`, `paused`)
- **created_at** (timestamp): Data/ora creazione
- **updated_at** (timestamp): Data/ora ultimo aggiornamento
- **meta_json** (jsonb): Metadati (user_id, session_id, context)

**Esempi Dati**:

```sql
INSERT INTO agent_conversations (conversation_id, agent, status) VALUES
('mio-main-12345', 'mio', 'active'),
('manus-single', 'manus', 'active'),
('mio-abacus-coordination', 'abacus', 'active');
```

**Trigger Update Timestamp**:
```sql
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_agent_conversations_updated_at 
BEFORE UPDATE ON agent_conversations 
FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

---

## 📊 Tabelle Logging

### mio_agent_logs

**Tabella principale per il logging Guardian** di tutte le operazioni degli agenti.

**Struttura**:
```sql
CREATE TABLE mio_agent_logs (
  id SERIAL PRIMARY KEY,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  agent VARCHAR(50) NOT NULL,
  service_id VARCHAR(100),
  endpoint VARCHAR(255),
  method VARCHAR(10),
  status_code INTEGER,
  risk VARCHAR(20),
  success BOOLEAN NOT NULL,
  message TEXT,
  meta_json JSONB
);
```

**Colonne**:

- **id** (integer, PK): ID univoco log
- **timestamp** (timestamp): Data/ora operazione
- **agent** (varchar): Agente che ha eseguito l'operazione
- **service_id** (varchar): ID servizio (es. `mihub.orchestrator.run`)
- **endpoint** (varchar): Endpoint API chiamato
- **method** (varchar): Metodo HTTP (`GET`, `POST`, etc.)
- **status_code** (integer): HTTP status code (200, 500, etc.)
- **risk** (varchar): Livello rischio (`low`, `medium`, `high`)
- **success** (boolean): Operazione riuscita (true/false)
- **message** (text): Messaggio descrittivo
- **meta_json** (jsonb): Metadati (durationMs, conversationId, errorDetails)

**Esempi Dati**:

```sql
INSERT INTO mio_agent_logs 
(agent, service_id, endpoint, method, status_code, risk, success, message, meta_json) 
VALUES
('mio', 'mihub.orchestrator.run', '/api/mihub/orchestrator', 'POST', 200, 'low', true, 
 'Orchestrator: auto mode, agent: mio', 
 '{"durationMs": 3178, "conversationId": "mio-main"}'),
 
('manus', 'mihub.ssh.execute', '/api/tools/ssh', 'POST', 200, 'medium', true,
 'SSH command executed: pm2 status',
 '{"command": "pm2 status", "durationMs": 450}'),
 
('abacus', 'mihub.sql.query', '/api/abacus/query', 'POST', 500, 'medium', false,
 'SQL query failed: syntax error',
 '{"query": "SELECT * FROM invalid_table", "error": "relation does not exist"}');
```

**Indici**:
```sql
CREATE INDEX idx_mio_agent_logs_agent ON mio_agent_logs(agent);
CREATE INDEX idx_mio_agent_logs_timestamp ON mio_agent_logs(timestamp DESC);
CREATE INDEX idx_mio_agent_logs_success ON mio_agent_logs(success);
CREATE INDEX idx_mio_agent_logs_service ON mio_agent_logs(service_id);
```

**Query Statistiche**:

```sql
-- Totale logs per agente
SELECT agent, COUNT(*) as total, 
       SUM(CASE WHEN success THEN 1 ELSE 0 END) as successes,
       SUM(CASE WHEN NOT success THEN 1 ELSE 0 END) as errors
FROM mio_agent_logs
GROUP BY agent;

-- Tempo medio risposta per endpoint
SELECT endpoint, 
       AVG((meta_json->>'durationMs')::int) as avg_duration_ms,
       COUNT(*) as total_calls
FROM mio_agent_logs
WHERE meta_json->>'durationMs' IS NOT NULL
GROUP BY endpoint
ORDER BY avg_duration_ms DESC;

-- Logs ultimi 7 giorni
SELECT * FROM mio_agent_logs
WHERE timestamp >= NOW() - INTERVAL '7 days'
ORDER BY timestamp DESC;
```

**Statistiche Attuali** (11 Dicembre 2025):
- Totale logs: **573**
- Successi: **451** (78.7%)
- Errori: **122** (21.3%)
- Agenti unici: **11**

---

### system_logs

Logs di sistema (non agenti).

**Struttura**:
```sql
CREATE TABLE system_logs (
  id SERIAL PRIMARY KEY,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  level VARCHAR(20),
  component VARCHAR(100),
  message TEXT,
  meta_json JSONB
);
```

**Livelli**: `info`, `warning`, `error`, `critical`

---

### audit_logs

Logs di audit per operazioni sensibili.

**Struttura**:
```sql
CREATE TABLE audit_logs (
  id SERIAL PRIMARY KEY,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  user_id VARCHAR(100),
  action VARCHAR(100),
  resource VARCHAR(255),
  details TEXT,
  ip_address VARCHAR(45)
);
```

---

### webhook_logs

Logs webhook ricevuti/inviati.

**Struttura**:
```sql
CREATE TABLE webhook_logs (
  id SERIAL PRIMARY KEY,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  direction VARCHAR(10),
  service VARCHAR(100),
  endpoint VARCHAR(255),
  payload JSONB,
  response JSONB,
  status_code INTEGER
);
```

---

## 🏢 Tabelle Business

### markets

Dati dei mercati rionali.

**Struttura**:
```sql
CREATE TABLE markets (
  id SERIAL PRIMARY KEY,
  nome VARCHAR(255) NOT NULL,
  indirizzo VARCHAR(255),
  citta VARCHAR(100),
  cap VARCHAR(10),
  giorni_apertura VARCHAR[],
  orario_apertura TIME,
  orario_chiusura TIME,
  numero_posteggi INTEGER,
  coordinate POINT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### imprese

Imprese registrate nel sistema.

**Struttura**:
```sql
CREATE TABLE imprese (
  id SERIAL PRIMARY KEY,
  ragione_sociale VARCHAR(255) NOT NULL,
  partita_iva VARCHAR(11) UNIQUE,
  codice_fiscale VARCHAR(16),
  indirizzo VARCHAR(255),
  citta VARCHAR(100),
  cap VARCHAR(10),
  telefono VARCHAR(20),
  email VARCHAR(255),
  pec VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### qualificazioni

Qualificazioni imprese.

**Struttura**:
```sql
CREATE TABLE qualificazioni (
  id SERIAL PRIMARY KEY,
  codice VARCHAR(50) UNIQUE NOT NULL,
  descrizione VARCHAR(255),
  categoria VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### vendors

Venditori ambulanti.

**Struttura**:
```sql
CREATE TABLE vendors (
  id SERIAL PRIMARY KEY,
  impresa_id INTEGER REFERENCES imprese(id),
  nome VARCHAR(100),
  cognome VARCHAR(100),
  codice_fiscale VARCHAR(16),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### stalls

Posteggi mercati.

**Struttura**:
```sql
CREATE TABLE stalls (
  id SERIAL PRIMARY KEY,
  market_id INTEGER REFERENCES markets(id),
  numero_posteggio VARCHAR(20),
  superficie_mq DECIMAL(6,2),
  tipo VARCHAR(50),
  coordinate POINT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### concessions

Concessioni posteggi.

**Struttura**:
```sql
CREATE TABLE concessions (
  id SERIAL PRIMARY KEY,
  stall_id INTEGER REFERENCES stalls(id),
  vendor_id INTEGER REFERENCES vendors(id),
  data_inizio DATE,
  data_fine DATE,
  stato VARCHAR(20),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🔗 Relazioni

### Diagramma ER (Conversazioni)

```
agent_conversations (1) ----< (N) agent_messages
  |
  └─ conversation_id (FK in agent_messages)
```

### Diagramma ER (Business)

```
markets (1) ----< (N) stalls
imprese (1) ----< (N) vendors
stalls (1) ----< (N) concessions
vendors (1) ----< (N) concessions
```

---

## ⚡ Indici e Performance

### Indici Principali

**Conversazioni**:
```sql
CREATE INDEX idx_agent_messages_conversation ON agent_messages(conversation_id);
CREATE INDEX idx_agent_messages_created ON agent_messages(created_at DESC);
CREATE INDEX idx_agent_conversations_agent ON agent_conversations(agent);
```

**Logging**:
```sql
CREATE INDEX idx_mio_agent_logs_timestamp ON mio_agent_logs(timestamp DESC);
CREATE INDEX idx_mio_agent_logs_agent ON mio_agent_logs(agent);
CREATE INDEX idx_mio_agent_logs_success ON mio_agent_logs(success);
```

**Business**:
```sql
CREATE INDEX idx_imprese_piva ON imprese(partita_iva);
CREATE INDEX idx_vendors_impresa ON vendors(impresa_id);
CREATE INDEX idx_stalls_market ON stalls(market_id);
```

### Ottimizzazioni

**JSONB Indexes** per query su meta_json:
```sql
CREATE INDEX idx_agent_messages_meta ON agent_messages USING GIN (meta_json);
CREATE INDEX idx_mio_agent_logs_meta ON mio_agent_logs USING GIN (meta_json);
```

**Partial Indexes** per query frequenti:
```sql
-- Solo messaggi recenti (ultimi 30 giorni)
CREATE INDEX idx_agent_messages_recent 
ON agent_messages(created_at DESC) 
WHERE created_at >= NOW() - INTERVAL '30 days';

-- Solo logs con errori
CREATE INDEX idx_mio_agent_logs_errors 
ON mio_agent_logs(timestamp DESC) 
WHERE success = false;
```

### Connection Pooling

Il backend usa **pg Pool** con:
- Max connections: 20
- Idle timeout: 30s
- Connection timeout: 10s

Questo garantisce performance ottimali anche sotto carico elevato.

---

## 🔄 Backup & Maintenance

### Backup Automatici

Neon PostgreSQL esegue backup automatici:
- **Point-in-time recovery**: Ultimi 7 giorni
- **Snapshot giornalieri**: Retention 30 giorni
- **Replica geografica**: EU-Central-1 (Frankfurt)

### Maintenance

**Vacuum automatico**: Attivo per tutte le tabelle

**Analyze automatico**: Aggiorna statistiche query optimizer

**Monitoring**: Neon Dashboard fornisce metriche real-time su:
- Query performance
- Connection count
- Storage usage
- CPU/Memory usage

---

## 📝 Query Utili

### Conversazioni Attive

```sql
SELECT c.conversation_id, c.agent, c.status,
       COUNT(m.id) as message_count,
       MAX(m.created_at) as last_message_at
FROM agent_conversations c
LEFT JOIN agent_messages m ON c.conversation_id = m.conversation_id
WHERE c.status = 'active'
GROUP BY c.id, c.conversation_id, c.agent, c.status
ORDER BY last_message_at DESC;
```

### Statistiche Agenti

```sql
SELECT agent,
       COUNT(*) as total_operations,
       SUM(CASE WHEN success THEN 1 ELSE 0 END) as successes,
       SUM(CASE WHEN NOT success THEN 1 ELSE 0 END) as errors,
       ROUND(AVG((meta_json->>'durationMs')::numeric), 2) as avg_duration_ms
FROM mio_agent_logs
WHERE timestamp >= NOW() - INTERVAL '7 days'
GROUP BY agent
ORDER BY total_operations DESC;
```

### Logs con Errori

```sql
SELECT timestamp, agent, endpoint, message, meta_json
FROM mio_agent_logs
WHERE success = false
  AND timestamp >= NOW() - INTERVAL '24 hours'
ORDER BY timestamp DESC;
```

---

**Ultimo Aggiornamento**: 11 Dicembre 2025  
**Database Version**: PostgreSQL 15.3  
**Provider**: Neon (Serverless PostgreSQL)  
**Region**: EU-Central-1 (Frankfurt)
