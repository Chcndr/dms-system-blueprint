# API Map - Backend MIO Hub

> **Mappa completa degli endpoint API attivi sul backend Node.js**  
> Server: `157.90.29.66:3000` | Base URL: `https://api.mio-hub.me`

## 📋 Indice

- [Orchestrazione](#orchestrazione)
- [Conversazioni & Chat](#conversazioni--chat)
- [Logging & Guardian](#logging--guardian)
- [Agenti Specializzati](#agenti-specializzati)
- [Business Logic](#business-logic)
- [Amministrazione](#amministrazione)
- [Integrazioni](#integrazioni)

---

## 🎯 Orchestrazione

### POST /api/mihub/orchestrator

**Endpoint principale** per l'orchestrazione multi-agente. Riceve richieste dall'utente, le analizza tramite LLM Gemini, decide quale agente attivare, e restituisce la risposta.

**Request Body:**
```json
{
  "message": "Analizza i log degli ultimi 7 giorni",
  "agent": "mio",
  "conversationId": "mio-main-12345",
  "context": {
    "userId": "user123",
    "sessionId": "sess456"
  }
}
```

**Response:**
```json
{
  "success": true,
  "response": "Ho analizzato i log...",
  "agent": "mio",
  "delegatedTo": "abacus",
  "executionTime": 3214
}
```

**Flusso Interno:**
1. Riceve messaggio utente
2. Carica contesto conversazione dal database
3. Chiama LLM Gemini con tools disponibili
4. Se LLM chiama `call_agent`, delega ad agente specializzato
5. Salva messaggi nel database
6. Restituisce risposta finale

**File Sorgente:** `routes/orchestrator.js`

---

## 💬 Conversazioni & Chat

### GET /api/mihub/chats

Recupera la lista delle conversazioni attive con metadati.

**Query Parameters:**
- `agent` (optional): Filtra per agente specifico
- `status` (optional): Filtra per stato (active, archived)
- `limit` (optional): Numero massimo risultati (default: 50)

**Response:**
```json
{
  "success": true,
  "conversations": [
    {
      "id": "conv_123",
      "conversation_id": "mio-manus-coordination",
      "agent": "manus",
      "status": "active",
      "last_message": "Deploy completato",
      "created_at": "2025-12-11T02:30:00Z",
      "updated_at": "2025-12-11T03:15:00Z"
    }
  ]
}
```

**File Sorgente:** `routes/chats.js`

---

### POST /api/mihub/chats

Crea una nuova conversazione.

**Request Body:**
```json
{
  "conversation_id": "manus-single",
  "agent": "manus",
  "initialMessage": "Ciao Manus, ho bisogno di aiuto"
}
```

**Response:**
```json
{
  "success": true,
  "conversation": {
    "id": "conv_456",
    "conversation_id": "manus-single",
    "agent": "manus",
    "status": "active",
    "created_at": "2025-12-11T04:00:00Z"
  }
}
```

**File Sorgente:** `routes/chats.js`

---

### GET /api/mihub/chats/:id

Recupera tutti i messaggi di una conversazione specifica.

**Path Parameters:**
- `id`: ID della conversazione

**Response:**
```json
{
  "success": true,
  "conversation": {
    "id": "conv_123",
    "conversation_id": "mio-manus-coordination",
    "agent": "manus"
  },
  "messages": [
    {
      "id": "msg_1",
      "role": "user",
      "content": "Fai il deploy del backend",
      "created_at": "2025-12-11T02:30:00Z"
    },
    {
      "id": "msg_2",
      "role": "assistant",
      "content": "Deploy in corso...",
      "created_at": "2025-12-11T02:30:15Z"
    }
  ]
}
```

**File Sorgente:** `routes/chats.js`

---

## 📊 Logging & Guardian

### GET /api/guardian/logs

**NUOVO** - Recupera gli ultimi log degli agenti per la dashboard. Supporta filtri avanzati.

**Query Parameters:**
- `agent` (optional): Filtra per agente (mio, manus, abacus, gptdev, zapier)
- `limit` (optional): Numero massimo risultati (default: 50, max: 100)
- `from` (optional): Data inizio (ISO 8601)
- `to` (optional): Data fine (ISO 8601)

**Response:**
```json
[
  {
    "timestamp": "2025-12-11T02:20:28.030Z",
    "agent": "abacus",
    "serviceId": "mihub.orchestrator.run",
    "path": "/api/mihub/orchestrator",
    "method": "POST",
    "statusCode": 200,
    "status": "allowed",
    "reason": null,
    "response_time_ms": 3178,
    "risk": "medium"
  }
]
```

**File Sorgente:** `routes/guardian.js` (linea 204-289)

---

### GET /api/logs/getLogs

Recupera tutti i logs del sistema con filtri avanzati.

**Query Parameters:**
- `agent` (optional): Filtra per agente
- `serviceId` (optional): Filtra per service ID
- `success` (optional): Filtra per successo (true/false)
- `limit` (optional): Numero massimo risultati (default: 100)
- `from` (optional): Data inizio
- `to` (optional): Data fine

**Response:**
```json
{
  "success": true,
  "logs": [
    {
      "id": 1234,
      "timestamp": "2025-12-11T02:20:28.030Z",
      "agent": "mio",
      "service_id": "mihub.orchestrator.run",
      "endpoint": "/api/mihub/orchestrator",
      "method": "POST",
      "status_code": 200,
      "risk": "low",
      "success": true,
      "message": "Orchestrator: auto mode, agent: mio",
      "meta_json": {
        "durationMs": 3178,
        "conversationId": "mio-main"
      }
    }
  ],
  "total": 573
}
```

**File Sorgente:** `routes/logs.js`

---

### POST /api/logs/createLog

Crea un nuovo log entry.

**Request Body:**
```json
{
  "agent": "manus",
  "serviceId": "mihub.ssh.execute",
  "endpoint": "/api/tools/ssh",
  "method": "POST",
  "statusCode": 200,
  "risk": "medium",
  "success": true,
  "message": "SSH command executed successfully",
  "meta": {
    "command": "pm2 status",
    "durationMs": 450
  }
}
```

**Response:**
```json
{
  "success": true,
  "logId": 1235
}
```

**File Sorgente:** `routes/logs.js`

---

### GET /api/logs/stats

Restituisce statistiche aggregate sui logs.

**Query Parameters:**
- `from` (optional): Data inizio
- `to` (optional): Data fine
- `agent` (optional): Filtra per agente

**Response:**
```json
{
  "success": true,
  "stats": {
    "total": 573,
    "success": 451,
    "error": 122,
    "warning": 0,
    "byAgent": {
      "mio": 245,
      "manus": 128,
      "abacus": 95,
      "gptdev": 67,
      "zapier": 38
    },
    "avgResponseTime": 2847
  }
}
```

**File Sorgente:** `routes/logs.js`

---

### GET /api/mihub/logs

**NUOVO** - Recupera tutti i logs del sistema per il tab "Tutti i Log" nella dashboard.

**Query Parameters:**
- `limit` (optional): Numero massimo risultati (default: 100)
- `agent` (optional): Filtra per agente
- `from` (optional): Data inizio
- `to` (optional): Data fine

**Response:** Stesso formato di `/api/logs/getLogs`

**File Sorgente:** `routes/mihub.js` (aggiunto 11 Dicembre 2025)

---

### GET /api/guardian/health

Health check del sistema Guardian.

**Response:**
```json
{
  "success": true,
  "status": "healthy",
  "timestamp": "2025-12-11T04:00:00Z",
  "database": {
    "connected": true,
    "logCount": 573
  },
  "version": "2.0.0"
}
```

**File Sorgente:** `routes/guardian.js`

---

## 🤖 Agenti Specializzati

### POST /api/gptdev/execute

Esegue task di sviluppo tramite agente GPT Dev.

**Request Body:**
```json
{
  "task": "Crea una funzione JavaScript per validare email",
  "context": {
    "language": "javascript",
    "framework": "node"
  }
}
```

**File Sorgente:** `routes/gptdev.js`

---

### POST /api/abacus/query

Esegue query SQL o analisi dati tramite Abacus.

**Request Body:**
```json
{
  "query": "SELECT COUNT(*) FROM mio_agent_logs WHERE success = true",
  "database": "neondb"
}
```

**File Sorgente:** `routes/abacusSql.js`

---

### POST /api/abacus/github

Interagisce con GitHub tramite Abacus.

**Request Body:**
```json
{
  "action": "list_repos",
  "owner": "Chcndr"
}
```

**File Sorgente:** `routes/abacusGithub.js`

---

### POST /api/tools/manus/ssh

Esegue comandi SSH tramite Manus.

**Request Body:**
```json
{
  "command": "pm2 status",
  "timeout": 30
}
```

**File Sorgente:** `routes/toolsManus.js`, `src/tools/manus/ssh.js`

---

## 🏢 Business Logic

### GET /api/imprese

Recupera la lista delle imprese registrate.

**Query Parameters:**
- `limit` (optional): Numero massimo risultati
- `offset` (optional): Offset per paginazione
- `search` (optional): Ricerca per nome/ragione sociale

**Response:**
```json
{
  "success": true,
  "imprese": [
    {
      "id": 1,
      "ragione_sociale": "Impresa Test SRL",
      "partita_iva": "12345678901",
      "codice_fiscale": "TSTCMP85M01H501Z",
      "indirizzo": "Via Roma 1",
      "citta": "Roma",
      "cap": "00100",
      "created_at": "2025-01-15T10:00:00Z"
    }
  ],
  "total": 150
}
```

**File Sorgente:** `routes/imprese.js`

---

### GET /api/imprese/:id

Recupera dettagli di un'impresa specifica.

**Path Parameters:**
- `id`: ID dell'impresa

**File Sorgente:** `routes/imprese.js`

---

### GET /api/qualificazioni

Recupera le qualificazioni disponibili per le imprese.

**Response:**
```json
{
  "success": true,
  "qualificazioni": [
    {
      "id": 1,
      "codice": "QUAL_001",
      "descrizione": "Commercio alimentare",
      "categoria": "alimentare"
    }
  ]
}
```

**File Sorgente:** `routes/qualificazioni.js`

---

### GET /api/markets

Recupera la lista dei mercati.

**Response:**
```json
{
  "success": true,
  "markets": [
    {
      "id": 1,
      "nome": "Mercato Centrale",
      "indirizzo": "Piazza del Mercato",
      "citta": "Roma",
      "giorni_apertura": ["lunedì", "mercoledì", "sabato"],
      "orario_apertura": "07:00",
      "orario_chiusura": "14:00",
      "numero_posteggi": 150
    }
  ]
}
```

**File Sorgente:** `routes/markets.js`

---

### GET /api/gis/markets

Recupera dati GIS dei mercati (coordinate, geometrie).

**File Sorgente:** `routes/gis.js`

---

## 🔧 Amministrazione

### POST /api/admin/migrate-pdnd

Esegue migrazione dati PDND.

**File Sorgente:** `routes/migratePDND.js`

---

### POST /api/admin/deploy

Trigger deploy automatico.

**File Sorgente:** `routes/adminDeploy.js`

---

### GET /api/admin/secrets

Gestisce secrets di sistema.

**File Sorgente:** `routes/adminSecrets.js`

---

## 🔗 Integrazioni

### POST /api/webhook

Riceve webhook generici.

**File Sorgente:** `routes/webhook.js`

---

### POST /api/webhooks/:service

Riceve webhook da servizi specifici (Zapier, GitHub, ecc.).

**File Sorgente:** `routes/webhooks.js`

---

### GET /api/integrations

Lista integrazioni attive.

**File Sorgente:** `routes/integrations.js`

---

## 📝 Note Implementazione

### Autenticazione

Attualmente il sistema non implementa autenticazione JWT. Gli endpoint sono protetti tramite:
- Rate limiting (100 req/min per IP)
- CORS configurato per domini whitelisted
- API Keys per integrazioni esterne

### Error Handling

Tutti gli endpoint seguono questo formato per gli errori:

```json
{
  "success": false,
  "error": "Error type",
  "message": "Detailed error message",
  "code": "ERROR_CODE"
}
```

### Logging

Ogni chiamata API viene loggata automaticamente nella tabella `mio_agent_logs` con:
- Timestamp
- Agent chiamante
- Endpoint
- Status code
- Tempo di risposta
- Metadati aggiuntivi

---

**Ultimo Aggiornamento**: 11 Dicembre 2025  
**Versione API**: 2.0.0  
**Maintainer**: Team MIO Hub


---

## 🔄 Gestione Concessioni (Subingresso)

### POST /api/concessions/:id/associa-posteggio

**NUOVO (03 Gennaio 2026)** - Endpoint per completare il trasferimento di un posteggio durante un subingresso. Esegue in transazione atomica:

1. Disassocia il posteggio dal cedente
2. Chiude la concessione del cedente (stato → CESSATA)
3. Associa il posteggio al subentrante
4. Trasferisce il wallet dal cedente al subentrante
5. Attiva la nuova concessione (stato → ATTIVA)

**Path Parameters:**
- `id`: ID della concessione di subingresso (deve essere in stato DA_ASSOCIARE)

**Request Body:** Nessuno (tutti i dati sono già nella concessione)

**Response Success:**
```json
{
  "success": true,
  "data": {
    "id": 123,
    "tipo_concessione": "subingresso",
    "stato": "ATTIVA",
    "stall_id": 456,
    "market_id": 1,
    "impresa_id": 789
  },
  "message": "Posteggio associato con successo. Trasferimento completato."
}
```

**Response Error (concessione non trovata):**
```json
{
  "success": false,
  "error": "Concessione non trovata"
}
```

**Response Error (tipo errato):**
```json
{
  "success": false,
  "error": "Questa operazione è valida solo per concessioni di tipo subingresso"
}
```

**Response Error (stato errato):**
```json
{
  "success": false,
  "error": "La concessione è già stata associata o non è in stato DA_ASSOCIARE"
}
```

**Logica Interna:**
- Verifica che la concessione sia di tipo `subingresso` e stato `DA_ASSOCIARE`
- Cerca e chiude eventuali concessioni attive del cedente per lo stesso posteggio
- Aggiorna la tabella `stalls` con il nuovo `vendor_id` e `impresa_id`
- Trasferisce il saldo del wallet CONCESSION dal cedente al subentrante
- Aggiorna lo stato della concessione a `ATTIVA`

**File Sorgente:** `routes/concessions.js` (linee 734-917)

**Frontend:** Il pulsante "Associa Posteggio" è visibile nel tab "Aggiorna Posteggi" della modale dettaglio concessione, solo per concessioni di tipo subingresso in stato DA_ASSOCIARE.

---

