# API Orchestrator – MIO HUB

**Endpoint:** `/api/mihub/orchestrator`  
**Metodo:** `POST`  
**Autenticazione:** Nessuna (gestita da Vercel)

---

## 🎯 Scopo

Questo endpoint è il punto di ingresso unico per tutte le comunicazioni dal frontend MIO Hub verso il backend. Gestisce sia le conversazioni con l'orchestratore centrale (MIO) sia le conversazioni dirette con gli agenti specializzati (Manus, Abacus, Zapier, GPT Dev).

---

## 📦 Request Body

Il body della richiesta varia in base alla modalità (`mode`) utilizzata.

### Modalità `auto` (Chat Principale MIO)

Utilizzata per inviare messaggi all'orchestratore centrale, che decide autonomamente a quale agente inoltrare la richiesta.

```json
{
  "message": "string",
  "mode": "auto",
  "conversationId": "string | null"
}
```

| Campo | Tipo | Descrizione | Obbligatorio |
| :--- | :--- | :--- | :--- |
| `message` | `string` | Testo del messaggio inviato dall'utente. | ✅ Sì |
| `mode` | `string` | Deve essere `"auto"`. | ✅ Sì |
| `conversationId` | `string` | ID della conversazione esistente. Se `null`, ne viene creata una nuova. | ❌ No |

### Modalità `manual` (Chat Singola Agente)

Utilizzata per inviare messaggi diretti a un agente specifico, bypassando l'orchestrazione automatica.

```json
{
  "message": "string",
  "mode": "manual",
  "targetAgent": "string",
  "conversationId": "string | null"
}
```

| Campo | Tipo | Descrizione | Obbligatorio |
| :--- | :--- | :--- | :--- |
| `message` | `string` | Testo del messaggio inviato dall'utente. | ✅ Sì |
| `mode` | `string` | Deve essere `"manual"`. | ✅ Sì |
| `targetAgent` | `string` | Nome dell'agente target (`gptdev`, `manus`, `abacus`, `zapier`). | ✅ Sì |
| `conversationId` | `string` | ID della conversazione esistente. Se `null`, ne viene creata una nuova. | ❌ No |

---

## 📤 Response Body

### Risposta di Successo (`200 OK`)

```json
{
  "success": true,
  "agent": "string",
  "conversationId": "string",
  "message": "string",
  "internalTraces": [],
  "error": null
}
```

| Campo | Tipo | Descrizione |
| :--- | :--- | :--- |
| `success` | `boolean` | `true` se la richiesta è andata a buon fine. |
| `agent` | `string` | Nome dell'agente che ha risposto (`mio` o uno degli agenti specializzati). |
| `conversationId` | `string` | ID della conversazione, restituito sempre per garantire la persistenza. |
| `message` | `string` | Testo della risposta generata dall'agente. |
| `internalTraces` | `array` | Log interni per il debug (usato dalla vista 4 agenti). |
| `error` | `null` | `null` in caso di successo. |

### Risposta di Errore (`400`, `404`, `500`)

```json
{
  "success": false,
  "agent": "string",
  "conversationId": "string | null",
  "message": null,
  "error": {
    "type": "string",
    "provider": "string | null",
    "statusCode": "number",
    "message": "string"
  }
}
```

| Campo | Tipo | Descrizione |
| :--- | :--- | :--- |
| `success` | `boolean` | `false` in caso di errore. |
| `error.type` | `string` | Tipo di errore (es. `validation_error`, `not_found`). |
| `error.statusCode` | `number` | Codice di stato HTTP dell'errore (es. `400`, `404`). |
| `error.message` | `string` | Messaggio di errore descrittivo. |

---

## 🧪 Esempi di Chiamata `cURL`

### Creare una nuova conversazione con MIO

```bash
curl -X POST https://orchestratore.mio-hub.me/api/mihub/orchestrator \
-H "Content-Type: application/json" \
-d '{
  "mode": "auto",
  "message": "ping"
}'
```

### Continuare una conversazione esistente

```bash
curl -X POST https://orchestratore.mio-hub.me/api/mihub/orchestrator \
-H "Content-Type: application/json" \
-d '{
  "mode": "auto",
  "message": "Come stai?",
  "conversationId": "141e8fca-d512-4457-a20c-e2ff2258d3f5"
}'
```

### Inviare un messaggio diretto a Manus

```bash
curl -X POST https://orchestratore.mio-hub.me/api/mihub/orchestrator \
-H "Content-Type: application/json" \
-d '{
  "mode": "manual",
  "targetAgent": "manus",
  "message": "Crea un file di test"
}'
```
