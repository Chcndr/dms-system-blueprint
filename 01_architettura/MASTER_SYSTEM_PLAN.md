# MASTER SYSTEM PLAN - DMS HUB + MIO HUB

**Ultima Modifica:** 7 Dicembre 2025  
**Versione:** 3.2 (Sistema Rating Imprese PDND)

---

## Indice

1. [Architettura Generale e Domini](#1-architettura-generale-e-domini)
2. [Schema Database](#2-schema-database)
3. [Sistema Rating Imprese PDND](#3-sistema-rating-imprese-pdnd)
4. [Flusso Messaggi](#4-flusso-messaggi)
5. [Componenti Sistema](#5-componenti-sistema)
6. [Integrazioni](#6-integrazioni)
7. [Sistema Mappe GIS](#7-sistema-mappe-gis)
8. [Deployment](#8-deployment)
9. [Sicurezza](#9-sicurezza)
10. [Monitoring](#10-monitoring)
11. [Roadmap](#11-roadmap)

---

## 1. Architettura Generale e Domini

### 1.1. Architettura Logica

Il sistema DMS HUB è composto da:

- **Frontend Vercel:** Dashboard PA (React + Vite + TypeScript)
- **Backend Hetzner:** REST API per orchestratore multi-agente (Node.js + Express)
- **LLM Council Hetzner:** Servizio per confronto LLM (Python/FastAPI + React)
- **Database:** PostgreSQL (Neon)
- **Storage:** File system locale + GitHub logs

### 1.2. Architettura Domini `mio-hub.me` (TO-BE)

Questa sezione definisce l'architettura ufficiale dei domini e funge da "legge" per tutte le configurazioni future. Tutti i servizi devono essere esposti tramite HTTPS.

| Dominio | Servizio | Piattaforma | Porta Locale | Note |
| :--- | :--- | :--- | :--- | :--- |
| **`app.mio-hub.me`** | Frontend Dashboard PA | Vercel | N/A | Dominio principale per l'interfaccia utente. |
| **`api.mio-hub.me`** | Backend MIO Hub | Hetzner (Nginx) | `3000` | Endpoint REST per l'orchestratore e la logica di business. |
| **`council.mio-hub.me`**| Frontend LLM Council | Hetzner (Nginx) | `8002` | Interfaccia utente per il Concilio AI. |
| **`council-api.mio-hub.me`**| Backend LLM Council | Hetzner (Nginx) | `8001` | API di supporto per il Concilio AI. |
| `www.mio-hub.me` | Redirect | DNS Provider | N/A | Redirect verso `app.mio-hub.me`. |
| `mio-hub.me` | Redirect | DNS Provider | N/A | Redirect verso `app.mio-hub.me`. |

---

## 2. Schema Database

### 2.1. Tabella `agents` (Ripristinata)

La tabella `agents` è stata ripristinata per risolvere il bug "Unknown agent".

```sql
CREATE TABLE agents (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    role VARCHAR(100),
    description TEXT,
    endpoint VARCHAR(255),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2.2. Tabelle Messaggi

- **`conversations`**: Storico delle conversazioni
- **`conversations_messages`**: Messaggi delle conversazioni (usato per lo storico)
- **`agent_messages`**: Log dei messaggi per la vista 4 quadranti

### 2.3. Tabelle Sistema Rating Imprese PDND

**Implementato:** 7 Dicembre 2025 (FASE 1 & 2 MASTER PLAN)

#### Tabella `imprese` (Espansa)

Anagrafica Master con campi PDND completi (23 colonne totali):

```sql
ALTER TABLE imprese 
  RENAME COLUMN ragione_sociale TO denominazione;

ALTER TABLE imprese ADD COLUMN:
  numero_rea VARCHAR(20),
  cciaa_sigla VARCHAR(5),
  forma_giuridica VARCHAR(100),      -- SRL, SNC, DI, etc.
  stato_impresa VARCHAR(50),         -- ATTIVA, CESSATA, IN LIQUIDAZIONE
  indirizzo_via VARCHAR(255),
  indirizzo_civico VARCHAR(20),
  indirizzo_cap VARCHAR(10),
  indirizzo_provincia VARCHAR(5),
  indirizzo_stato VARCHAR(50) DEFAULT 'Italia',
  pec VARCHAR(255),                  -- OBBLIGATORIO per PA
  codice_ateco VARCHAR(20),
  descrizione_ateco TEXT,
  data_iscrizione_ri DATE,
  flag_impresa_artigiana BOOLEAN DEFAULT FALSE;
```

#### Tabella `dms_durc_snapshots`

Storico verifiche DURC (Documento Unico Regolarità Contributiva):

```sql
CREATE TABLE dms_durc_snapshots (
  id SERIAL PRIMARY KEY,
  impresa_id INTEGER REFERENCES imprese(id) ON DELETE CASCADE,
  protocollo_inps VARCHAR(100),
  esito_regolarita VARCHAR(50) NOT NULL,  -- REGOLARE / IRREGOLARE
  data_scadenza DATE NOT NULL,            -- Cruciale per semaforo
  data_verifica TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  raw_response JSONB                      -- Payload PDND originale
);

CREATE INDEX idx_durc_impresa ON dms_durc_snapshots(impresa_id);
CREATE INDEX idx_durc_scadenza ON dms_durc_snapshots(data_scadenza);
```

#### Tabella `dms_suap_instances`

Istanze SUAP (Sportello Unico Attività Produttive):

```sql
CREATE TABLE dms_suap_instances (
  id SERIAL PRIMARY KEY,
  impresa_id INTEGER REFERENCES imprese(id) ON DELETE CASCADE,
  market_id INTEGER,                      -- Link a mercati
  stall_id INTEGER,                       -- Link a posteggi
  cui VARCHAR(100),                       -- Codice Univoco Istanza
  tipo_pratica VARCHAR(100),              -- APERTURA, SUBINGRESSO, RINNOVO
  stato_pratica VARCHAR(50),              -- ACCOLTA, SOSPESA, RIGETTATA
  data_scadenza_concessione DATE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_suap_impresa ON dms_suap_instances(impresa_id);
CREATE INDEX idx_suap_market ON dms_suap_instances(market_id);
```

---

## 3. Sistema Rating Imprese PDND

**Implementato:** 7 Dicembre 2025  
**Versione:** 1.0 (FASE 1 & 2 completate)

### 3.1. Architettura Funzionale

Il sistema distingue due aree:

1. **Gestione Mercati > Tab Imprese:** Anagrafica Master (Data Entry)
2. **Dashboard > Qualificazioni:** Motore di Rating (Consultazione)

### 3.2. Endpoint Backend

#### Imprese API (`routes/imprese.js`)

| Metodo | Endpoint | Descrizione |
|--------|----------|-------------|
| GET | `/api/imprese` | Lista imprese con filtri (status, comune, settore) |
| GET | `/api/imprese/:id` | Dettaglio impresa (23 campi PDND) |
| POST | `/api/imprese` | Crea nuova impresa |
| PUT | `/api/imprese/:id` | Aggiorna impresa |
| DELETE | `/api/imprese/:id` | Elimina impresa |
| **GET** | **`/api/imprese/:id/rating`** | **🚦 Calcola semaforo conformità** |

#### Qualificazioni API (`routes/qualificazioni.js`)

| Metodo | Endpoint | Descrizione |
|--------|----------|-------------|
| GET | `/api/qualificazioni` | Lista tutte le qualificazioni |
| GET | `/api/qualificazioni/impresa/:id` | Qualificazioni per impresa |
| GET | `/api/qualificazioni/durc/:id` | Storico DURC per impresa |
| POST | `/api/qualificazioni/durc` | Salva snapshot DURC |
| GET | `/api/qualificazioni/suap/:id` | Istanze SUAP per impresa |
| POST | `/api/qualificazioni/suap` | Crea istanza SUAP |
| PUT | `/api/qualificazioni/suap/:id` | Aggiorna istanza SUAP |

### 3.3. Logica Semaforo Rating

**Endpoint:** `GET /api/imprese/:id/rating`

#### Algoritmo di Calcolo

**🔴 ROSSO (Non Conforme - Blocco Operativo):**
- `stato_impresa` ≠ 'ATTIVA' (CESSATA, IN LIQUIDAZIONE)
- DURC `esito_regolarita` = 'IRREGOLARE'
- DURC scaduto (`data_scadenza` < oggi)
- Concessione scaduta

**🟡 GIALLO (Azione Richiesta - Warning):**
- DURC in scadenza (< 30 giorni)
- PEC mancante o non valida
- Concessione in scadenza (< 30 giorni)
- DURC non presente

**🟢 VERDE (Conforme - OK):**
- `stato_impresa` = 'ATTIVA'
- DURC regolare e valido (> 30 giorni)
- PEC presente
- Concessioni attive

#### Risposta JSON

```json
{
  "success": true,
  "data": {
    "impresa_id": 1,
    "denominazione": "Alimentari Rossi & C.",
    "rating": {
      "status": "GIALLO",
      "score": 50,
      "label": "Azione Richiesta"
    },
    "checks": {
      "stato_impresa": { "value": "ATTIVA", "valid": true },
      "durc": {
        "esito": "REGOLARE",
        "scadenza": "2025-02-01",
        "valid": false
      },
      "pec": { "value": "info@rossi.pec.it", "valid": true },
      "concessioni": { "count": 2, "attive": 2 }
    },
    "warnings": [
      {
        "type": "durc",
        "message": "DURC in scadenza tra 25 giorni",
        "severity": "warning",
        "data_scadenza": "2025-02-01"
      }
    ],
    "errors": []
  }
}
```

### 3.4. Migration Database

**Endpoint:** `POST /api/admin/migrate-pdnd`  
**File:** `routes/migratePDND.js`

**Funzione:**
- Esegue migration SQL sul database Neon
- Idempotente (usa `IF NOT EXISTS` e `ADD COLUMN IF NOT EXISTS`)
- Registrato in MIO-hub/api/index.json
- Monitorato in sezioni Logs e Debug

**Esecuzione:**
```bash
curl -X POST http://157.90.29.66:3000/api/admin/migrate-pdnd
```

---

## 4. Flusso Messaggi

### 3.1. Fix Completo Ecosistema (07/12/2025)

**Problema:** Un errore critico `saveAgentLog is not defined` impediva il salvataggio dei log degli agenti, causando un errore 500 in tutte le chat. Inoltre, gli agenti MIO, Manus, Zapier e GPT Dev non erano funzionanti a causa di problemi di configurazione LLM e gestione dei tool calls.

**Causa:**
1.  **`saveAgentLog`:** Errore di sintassi in `routes/orchestrator.js` (commento JSDoc non chiuso e parentesi graffa extra).
2.  **Agenti non funzionanti:** Mancata gestione dei `tool_calls` nelle risposte di Gemini, nomi agenti non allineati e configurazioni mancanti nel file `src/modules/orchestrator/llm.js`.

**Soluzione:**
1.  Correzione della sintassi in `orchestrator.js`.
2.  Modifica di `llm.js` per gestire correttamente i `tool_calls`.
3.  Aggiornamento di `orchestrator.js` per processare la nuova struttura di risposta.
4.  Aggiunta di un alias per `gptdev` → `dev`.
5.  Allineamento dei nomi degli agenti in `validAgents`.

**Stato:** ✅ **RISOLTO**. Tutti gli agenti sono ora operativi e il sistema è stabile.

### 3.2. Gestione Messaggi

Per risolvere il bug dei messaggi duplicati, il backend ora salva i messaggi solo in due tabelle:

1. **`conversations_messages`**: per lo storico della conversazione
2. **`agent_messages`**: per la vista 4 quadranti

Il salvataggio nella tabella `chats` è stato rimosso.

---

(Il resto del documento rimane invariato)
