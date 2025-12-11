# Guida Operativa - Zapier Webhooks DMS Hub

## Versione
1.0.0 - 27 Novembre 2025

## Descrizione

Questa guida documenta l'integrazione Zapier per DMS Hub tramite webhook REST. Gli endpoint permettono di notificare eventi del sistema (nuove concessioni, cambi stato posteggi) a Zapier per automazioni esterne (email, Google Sheets, notifiche, ecc.).

---

## Endpoint Disponibili

### Base URL
```
https://mihub.157-90-29-66.nip.io
```

### 1. Nuova Concessione

**Endpoint:** `POST /api/hooks/zapier/new-concession`

**Descrizione:** Notifica a Zapier la creazione o aggiornamento di una concessione.

**Autenticazione:**
- Header: `x-zapier-api-key: YOUR_SECRET_KEY`
- Oppure: `Authorization: Bearer YOUR_SECRET_KEY`

**Payload JSON:**
```json
{
  "market_id": 1,
  "market_code": "GR001",
  "market_name": "Mercato Grosseto",
  "concession_id": 1234,
  "stall_number": 17,
  "stall_type": "fisso",
  "company_code": "V003",
  "company_name": "Alimentari Verdi",
  "state": "attiva",
  "valid_from": "2025-01-01",
  "valid_to": null,
  "source": "dms-ui"
}
```

**Campi Obbligatori:**
- `market_id` (number)
- `stall_number` (number/string)
- `company_name` (string)
- `state` (string)

**Risposta Successo (200):**
```json
{
  "status": "ok",
  "message": "Webhook received successfully",
  "event_type": "new-concession",
  "timestamp": "2025-11-27T10:15:00Z"
}
```

**Risposta Errore (400):**
```json
{
  "status": "error",
  "message": "Missing required fields: market_id, stall_number"
}
```

---

### 2. Cambio Stato Posteggio

**Endpoint:** `POST /api/hooks/zapier/stall-status-changed`

**Descrizione:** Notifica a Zapier i cambi di stato dei posteggi (libero → occupato, occupato → spunta, ecc.).

**Autenticazione:**
- Header: `x-zapier-api-key: YOUR_SECRET_KEY`
- Oppure: `Authorization: Bearer YOUR_SECRET_KEY`

**Payload JSON:**
```json
{
  "market_id": 1,
  "market_code": "GR001",
  "stall_number": 17,
  "old_status": "libero",
  "new_status": "occupato",
  "status_reason": "spunta",
  "company_code": "V003",
  "company_name": "Alimentari Verdi",
  "changed_at": "2025-11-27T10:15:00Z",
  "source": "dms-ui"
}
```

**Campi Obbligatori:**
- `market_id` (number)
- `stall_number` (number/string)
- `old_status` (string)
- `new_status` (string)

**Risposta Successo (200):**
```json
{
  "status": "ok",
  "message": "Webhook received successfully",
  "event_type": "stall-status-changed",
  "timestamp": "2025-11-27T10:15:00Z"
}
```

---

### 3. Test Endpoint

**Endpoint:** `GET /api/hooks/zapier/test`

**Descrizione:** Verifica che il webhook sia raggiungibile (nessuna autenticazione richiesta).

**Risposta (200):**
```json
{
  "status": "ok",
  "message": "Zapier webhook endpoint is reachable",
  "timestamp": "2025-11-27T10:15:00Z",
  "endpoints": [
    "POST /api/hooks/zapier/new-concession",
    "POST /api/hooks/zapier/stall-status-changed"
  ]
}
```

---

## Sicurezza

### API Key Condiviso

Gli endpoint webhook sono protetti da un API key condiviso configurato nel backend tramite variabile d'ambiente:

```bash
ZAPIER_WEBHOOK_SECRET=your-secret-key-here
```

**⚠️ IMPORTANTE:**
- Usa un secret complesso (almeno 32 caratteri casuali)
- Non committare il secret nei repository
- Condividi il secret solo con chi configura Zapier
- Ruota il secret periodicamente

### Generare un Secret Sicuro

```bash
# Linux/Mac
openssl rand -hex 32

# Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

---

## Esempi curl

### Test Connettività

```bash
curl -X GET https://mihub.157-90-29-66.nip.io/api/hooks/zapier/test
```

### Nuova Concessione

```bash
curl -X POST https://mihub.157-90-29-66.nip.io/api/hooks/zapier/new-concession \
  -H "Content-Type: application/json" \
  -H "x-zapier-api-key: YOUR_SECRET_KEY" \
  -d '{
    "market_id": 1,
    "market_code": "GR001",
    "market_name": "Mercato Grosseto",
    "concession_id": 1234,
    "stall_number": 17,
    "stall_type": "fisso",
    "company_code": "V003",
    "company_name": "Alimentari Verdi",
    "state": "attiva",
    "valid_from": "2025-01-01",
    "source": "dms-ui"
  }'
```

### Cambio Stato Posteggio

```bash
curl -X POST https://mihub.157-90-29-66.nip.io/api/hooks/zapier/stall-status-changed \
  -H "Content-Type: application/json" \
  -H "x-zapier-api-key: YOUR_SECRET_KEY" \
  -d '{
    "market_id": 1,
    "market_code": "GR001",
    "stall_number": 17,
    "old_status": "libero",
    "new_status": "occupato",
    "status_reason": "spunta",
    "company_code": "V003",
    "company_name": "Alimentari Verdi",
    "changed_at": "2025-11-27T10:15:00Z",
    "source": "dms-ui"
  }'
```

---

## Test con Postman

### 1. Importa Collection

Crea una nuova collection "DMS Zapier Webhooks" con le seguenti request:

#### Request 1: Test Endpoint
- **Method:** GET
- **URL:** `https://mihub.157-90-29-66.nip.io/api/hooks/zapier/test`
- **Headers:** Nessuno

#### Request 2: New Concession
- **Method:** POST
- **URL:** `https://mihub.157-90-29-66.nip.io/api/hooks/zapier/new-concession`
- **Headers:**
  - `Content-Type: application/json`
  - `x-zapier-api-key: YOUR_SECRET_KEY`
- **Body (raw JSON):** Vedi esempio sopra

#### Request 3: Stall Status Changed
- **Method:** POST
- **URL:** `https://mihub.157-90-29-66.nip.io/api/hooks/zapier/stall-status-changed`
- **Headers:**
  - `Content-Type: application/json`
  - `x-zapier-api-key: YOUR_SECRET_KEY`
- **Body (raw JSON):** Vedi esempio sopra

### 2. Variabili d'Ambiente

Crea un environment "DMS Production" con:
- `base_url`: `https://mihub.157-90-29-66.nip.io`
- `zapier_api_key`: `YOUR_SECRET_KEY`

Usa `{{base_url}}` e `{{zapier_api_key}}` nelle request.

---

## Configurazione Zapier

### Zap 1: Nuova Concessione → Email + Google Sheets

**Trigger:**
1. Scegli "Webhooks by Zapier"
2. Seleziona "Catch Hook"
3. Copia l'URL generato da Zapier (es: `https://hooks.zapier.com/hooks/catch/123456/abcdef/`)

**Filter (opzionale):**
- Esegui solo se `market_code` = `GR001` (Grosseto)

**Action 1 - Email:**
1. Scegli "Gmail" o "Email by Zapier"
2. To: `suap@comune.grosseto.it`
3. Subject: `[DMS] Nuova concessione {{stall_number}} – {{company_name}}`
4. Body:
```
Nuova concessione registrata:

Mercato: {{market_name}} ({{market_code}})
Posteggio: {{stall_number}} ({{stall_type}})
Impresa: {{company_name}} ({{company_code}})
Stato: {{state}}
Validità: dal {{valid_from}} al {{valid_to}}
Fonte: {{source}}

Data evento: {{timestamp}}
```

**Action 2 - Google Sheets:**
1. Scegli "Google Sheets"
2. Action: "Create Spreadsheet Row"
3. Spreadsheet: "Concessioni DMS"
4. Worksheet: "Sheet1"
5. Colonne:
   - Data Evento: `{{timestamp}}`
   - Mercato: `{{market_name}}`
   - Codice Mercato: `{{market_code}}`
   - Posteggio: `{{stall_number}}`
   - Tipo: `{{stall_type}}`
   - Impresa: `{{company_name}}`
   - Codice Impresa: `{{company_code}}`
   - Stato: `{{state}}`
   - Valida Dal: `{{valid_from}}`
   - Valida Al: `{{valid_to}}`
   - Fonte: `{{source}}`

**Test:**
Usa curl per inviare un webhook di test e verifica che:
1. Email arrivi correttamente
2. Riga venga aggiunta a Google Sheets

---

### Zap 2: Cambio Stato Posteggio → Notifica Slack

**Trigger:**
1. "Webhooks by Zapier" → "Catch Hook"
2. Copia URL generato

**Filter:**
- Esegui solo se `new_status` = `spunta` (notifica solo le spunte)

**Action - Slack:**
1. Scegli "Slack"
2. Action: "Send Channel Message"
3. Channel: `#dms-alerts`
4. Message:
```
🚨 Posteggio spuntato!

Mercato: {{market_name}} ({{market_code}})
Posteggio: {{stall_number}}
Impresa: {{company_name}}
Motivo: {{status_reason}}
Cambio: {{old_status}} → {{new_status}}
Data: {{changed_at}}
```

---

## Logging

Tutti gli eventi webhook vengono loggati nella tabella `zapier_webhook_logs`:

```sql
SELECT 
  id,
  event_type,
  payload->>'market_code' AS market,
  payload->>'stall_number' AS stall,
  outcome,
  error_message,
  created_at
FROM zapier_webhook_logs
ORDER BY created_at DESC
LIMIT 50;
```

**Campi:**
- `event_type`: `new-concession` o `stall-status-changed`
- `payload`: JSON completo ricevuto
- `outcome`: `OK` o `ERROR`
- `error_message`: Messaggio di errore se `outcome = ERROR`
- `created_at`: Timestamp evento

---

## Troubleshooting

### Errore 401 Unauthorized

**Causa:** API key mancante o errato

**Soluzione:**
1. Verifica che l'header `x-zapier-api-key` sia presente
2. Verifica che il valore corrisponda a `ZAPIER_WEBHOOK_SECRET` nel backend
3. Controlla che non ci siano spazi o caratteri invisibili

### Errore 400 Missing Required Fields

**Causa:** Payload incompleto

**Soluzione:**
1. Verifica che tutti i campi obbligatori siano presenti
2. Controlla i tipi di dato (es: `market_id` deve essere number)
3. Usa il payload di esempio come riferimento

### Webhook Non Ricevuto da Zapier

**Causa:** URL Zapier non configurato o errato

**Soluzione:**
1. Verifica che l'URL Zapier sia stato copiato correttamente
2. Testa l'endpoint con curl per verificare che funzioni
3. Controlla i log di Zapier per vedere se il webhook è arrivato
4. Verifica che il Zap sia attivo (non in pausa)

### Logging Non Funziona

**Causa:** Tabella `zapier_webhook_logs` non creata

**Soluzione:**
```bash
ssh root@157.90.29.66
cd /root/mihub-backend-rest
psql -U postgres -d dms_hub -f migrations/007_create_zapier_webhook_logs.sql
```

---

## Monitoraggio

### Query Utili

**Eventi ultimi 7 giorni:**
```sql
SELECT 
  DATE(created_at) AS date,
  event_type,
  outcome,
  COUNT(*) AS count
FROM zapier_webhook_logs
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY DATE(created_at), event_type, outcome
ORDER BY date DESC, event_type;
```

**Errori recenti:**
```sql
SELECT 
  event_type,
  payload,
  error_message,
  created_at
FROM zapier_webhook_logs
WHERE outcome = 'ERROR'
ORDER BY created_at DESC
LIMIT 20;
```

**Statistiche per mercato:**
```sql
SELECT 
  payload->>'market_code' AS market,
  event_type,
  COUNT(*) AS count
FROM zapier_webhook_logs
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY payload->>'market_code', event_type
ORDER BY count DESC;
```

---

## Riferimenti

- **Repository Backend:** `Chcndr/mihub-backend-rest`
- **File Route:** `routes/webhooks.js`
- **Migration SQL:** `migrations/007_create_zapier_webhook_logs.sql`
- **API Index:** `dms-system-blueprint/api/index.json`
- **Zapier Documentation:** https://zapier.com/help/doc/how-get-started-webhooks-zapier

---

## Changelog

### v1.0.0 - 27 Novembre 2025
- ✅ Implementati endpoint `/api/hooks/zapier/new-concession` e `/api/hooks/zapier/stall-status-changed`
- ✅ Aggiunto logging in tabella `zapier_webhook_logs`
- ✅ Implementata autenticazione con API key condiviso
- ✅ Creata documentazione completa con esempi curl e Postman
- ✅ Configurazione Zap di esempio (Email + Google Sheets)

---

**Autore:** Manus (Agente Esecutivo)  
**Data:** 27 Novembre 2025  
**Versione:** 1.0.0
