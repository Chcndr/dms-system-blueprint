# API Integrazioni e Log & Debug

**Data:** 22 Novembre 2024  
**Autore:** Manus AI Agent  
**Versione:** 1.0  
**Backend:** `mihub-backend-rest` (Hetzner)

---

## 1. API Integrazioni

Queste API gestiscono le integrazioni esterne (DMS Legacy, Pepe GIS, TPER, etc.) e permettono di testare endpoint e monitorare lo stato delle connessioni.

### 1.1 Test Endpoint

**Endpoint:** `POST /api/integrations/test-endpoint`

**Descrizione:** Testa la connettività con un endpoint esterno.

**Request Body:**
```json
{
  "name": "DMS Legacy",
  "url": "https://dms-legacy.herokuapp.com/api/health",
  "method": "GET",
  "headers": {
    "Authorization": "Bearer xxx"
  }
}
```

**Response:**
```json
{
  "success": true,
  "status": 200,
  "responseTime": 245,
  "data": { ... }
}
```

### 1.2 Lista Integrazioni

**Endpoint:** `GET /api/integrations/list`

**Descrizione:** Restituisce la lista di tutte le integrazioni configurate.

**Response:**
```json
[
  {
    "id": 1,
    "name": "DMS Legacy",
    "type": "dms-legacy",
    "status": "active",
    "lastSync": "2024-11-22T14:30:00Z"
  },
  {
    "id": 2,
    "name": "Pepe GIS",
    "type": "pepe-gis",
    "status": "active",
    "lastSync": "2024-11-22T14:25:00Z"
  }
]
```

### 1.3 Stato Integrazione

**Endpoint:** `GET /api/integrations/:id/status`

**Descrizione:** Restituisce lo stato dettagliato di un'integrazione.

**Response:**
```json
{
  "id": 1,
  "name": "DMS Legacy",
  "status": "active",
  "lastSync": "2024-11-22T14:30:00Z",
  "lastError": null,
  "metrics": {
    "totalRequests": 1234,
    "successRate": 98.5,
    "avgResponseTime": 320
  }
}
```

---

## 2. API Log & Debug

Queste API gestiscono la scrittura e la lettura dei log di sistema, utili per il debugging e il monitoraggio.

### 2.1 Schema Tabella `system_logs`

```sql
CREATE TABLE system_logs (
  id SERIAL PRIMARY KEY,
  timestamp TIMESTAMP DEFAULT NOW() NOT NULL,
  level VARCHAR(20) NOT NULL, -- info, warning, error, debug
  source VARCHAR(100) NOT NULL, -- api, integration, cron, etc.
  message TEXT NOT NULL,
  metadata JSONB, -- Dati aggiuntivi (user_id, request_id, etc.)
  created_at TIMESTAMP DEFAULT NOW() NOT NULL
);

CREATE INDEX idx_system_logs_timestamp ON system_logs(timestamp DESC);
CREATE INDEX idx_system_logs_level ON system_logs(level);
CREATE INDEX idx_system_logs_source ON system_logs(source);
```

### 2.2 Scrivi Log

**Endpoint:** `POST /api/logs/write`

**Descrizione:** Scrive un nuovo log nel sistema.

**Request Body:**
```json
{
  "level": "error",
  "source": "integration-dms-legacy",
  "message": "Failed to sync data from DMS Legacy",
  "metadata": {
    "error": "Connection timeout",
    "url": "https://dms-legacy.herokuapp.com/api/sync"
  }
}
```

**Response:**
```json
{
  "success": true,
  "logId": 12345
}
```

### 2.3 Lista Log

**Endpoint:** `GET /api/logs/list`

**Query Parameters:**
- `level` (optional): Filtra per livello (info, warning, error, debug)
- `source` (optional): Filtra per sorgente
- `startDate` (optional): Data inizio (ISO 8601)
- `endDate` (optional): Data fine (ISO 8601)
- `limit` (optional): Numero massimo di risultati (default: 100)
- `offset` (optional): Offset per paginazione (default: 0)

**Response:**
```json
{
  "logs": [
    {
      "id": 12345,
      "timestamp": "2024-11-22T14:30:00Z",
      "level": "error",
      "source": "integration-dms-legacy",
      "message": "Failed to sync data from DMS Legacy",
      "metadata": {
        "error": "Connection timeout",
        "url": "https://dms-legacy.herokuapp.com/api/sync"
      }
    }
  ],
  "total": 1,
  "limit": 100,
  "offset": 0
}
```

### 2.4 Statistiche Log

**Endpoint:** `GET /api/logs/stats`

**Query Parameters:**
- `startDate` (optional): Data inizio
- `endDate` (optional): Data fine

**Response:**
```json
{
  "totalLogs": 5432,
  "byLevel": {
    "info": 3200,
    "warning": 1500,
    "error": 650,
    "debug": 82
  },
  "bySource": {
    "api": 2100,
    "integration": 1800,
    "cron": 1532
  },
  "errorRate": 11.9
}
```

---

## 3. Implementazione Backend

### 3.1 Struttura File

```
mihub-backend-rest/
├── routes/
│   ├── integrations.js
│   └── logs.js
├── services/
│   ├── integrationService.js
│   └── logService.js
└── models/
    ├── Integration.js
    └── SystemLog.js
```

### 3.2 Esempio Codice (Express.js)

**routes/logs.js:**
```javascript
const express = require('express');
const router = express.Router();
const logService = require('../services/logService');

router.post('/write', async (req, res) => {
  try {
    const { level, source, message, metadata } = req.body;
    const logId = await logService.writeLog({ level, source, message, metadata });
    res.json({ success: true, logId });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

router.get('/list', async (req, res) => {
  try {
    const { level, source, startDate, endDate, limit, offset } = req.query;
    const result = await logService.listLogs({ level, source, startDate, endDate, limit, offset });
    res.json(result);
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

module.exports = router;
```

---

## 4. Note Implementative

- **Autenticazione**: Tutte le API devono essere protette con autenticazione (JWT o session).
- **Rate Limiting**: Implementare rate limiting per prevenire abusi.
- **Logging Automatico**: Tutte le richieste API devono scrivere automaticamente un log.
- **Retention Policy**: I log più vecchi di 90 giorni possono essere archiviati o eliminati.
