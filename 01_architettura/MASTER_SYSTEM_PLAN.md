# MASTER SYSTEM PLAN - DMS HUB + MIO HUB

**Ultima Modifica:** 7 Dicembre 2025  
**Versione:** 3.1 (Fix Bug Critici)

---

## Indice

1. [Architettura Generale e Domini](#1-architettura-generale-e-domini)
2. [Schema Database](#2-schema-database)
3. [Flusso Messaggi](#3-flusso-messaggi)
4. [Componenti Sistema](#4-componenti-sistema)
5. [Integrazioni](#5-integrazioni)
6. [Sistema Mappe GIS](#6-sistema-mappe-gis)
7. [Deployment](#7-deployment)
8. [Sicurezza](#8-sicurezza)
9. [Monitoring](#9-monitoring)
10. [Roadmap](#10-roadmap)

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

---

## 3. Flusso Messaggi

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
