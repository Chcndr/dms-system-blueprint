# LLM Engine - Google Gemini Integration

> **Documentazione del motore LLM che alimenta tutti gli agenti del sistema MIO Hub**  
> Libreria: `@google/generative-ai` (ufficiale Google)  
> Modello: `gemini-2.5-flash`

## 📋 Indice

- [Overview](#overview)
- [Configurazione](#configurazione)
- [Agenti e Tools](#agenti-e-tools)
- [System Instructions](#system-instructions)
- [Flusso di Elaborazione](#flusso-di-elaborazione)
- [Error Handling](#error-handling)

---

## 🎯 Overview

Il sistema MIO Hub utilizza **Google Gemini** come motore LLM per tutti gli agenti. L'implementazione è basata sulla **libreria ufficiale** `@google/generative-ai` che fornisce accesso nativo alle API di Google.

### Caratteristiche Principali

L'engine LLM gestisce l'intelligenza artificiale di tutti gli agenti del sistema, fornendo capacità di comprensione del linguaggio naturale, ragionamento, e esecuzione di tool. Ogni agente ha una configurazione specifica con tool dedicati e istruzioni di sistema personalizzate.

Il modello **gemini-2.5-flash** è stato scelto per il suo ottimo bilanciamento tra velocità, costo e qualità delle risposte. Supporta nativamente il function calling (tool use) che permette agli agenti di eseguire azioni concrete tramite tool specializzati.

### File Sorgente

**Percorso**: `/root/mihub-backend-rest/src/modules/orchestrator/llm.js`

Il file contiene:
- Inizializzazione client Google Generative AI
- Definizione tools per ogni agente
- Configurazioni LLM (model, tools, system instructions)
- Funzione principale `callLLM()` per elaborare richieste

---

## ⚙️ Configurazione

### Inizializzazione Client

```javascript
const { GoogleGenerativeAI } = require('@google/generative-ai');

const apiKey = process.env.GEMINI_API_KEY;
const genAI = new GoogleGenerativeAI(apiKey);
```

La **API Key** è caricata dalla variabile d'ambiente `GEMINI_API_KEY` definita nel file `.env` del backend. La chiave è validata all'avvio del server e un errore viene sollevato se mancante.

### Modello Utilizzato

**Nome**: `gemini-2.5-flash`

Questo modello offre:
- Velocità di risposta elevata (2-4 secondi per richieste complesse)
- Supporto nativo per function calling
- Contesto fino a 1M token
- Multimodalità (testo, immagini, code)
- Costo contenuto per operazione

### Parametri Generazione

```javascript
{
  temperature: 0.7,        // Bilanciamento creatività/precisione
  topP: 0.9,              // Nucleus sampling
  topK: 40,               // Top-K sampling
  maxOutputTokens: 8192   // Lunghezza massima risposta
}
```

---

## 🤖 Agenti e Tools

Ogni agente ha una configurazione specifica che definisce quali tool può utilizzare e come deve comportarsi.

### MIO (Orchestratore)

**Ruolo**: Coordinatore centrale che delega task agli agenti specializzati.

**Tools Disponibili**:

#### call_agent
Delega un task specifico a uno degli agenti specializzati.

**Parametri**:
- `agent` (string, required): Nome dell'agente (`manus`, `abacus`, `gptdev`, `zapier`)
- `task` (string, required): Descrizione dettagliata del task da eseguire

**Esempio Uso**:
```json
{
  "name": "call_agent",
  "args": {
    "agent": "manus",
    "task": "Controlla lo stato del backend con pm2 status"
  }
}
```

**Quando Usare**: MIO usa questo tool quando riceve una richiesta che richiede competenze specifiche di un agente (es. operazioni server → Manus, analisi dati → Abacus).

---

### Manus (Operatore Esecutivo)

**Ruolo**: Esegue operazioni sul server Hetzner (SSH, file, deploy).

**Tools Disponibili**:

#### execute_ssh_command
Esegue un comando shell sul server Hetzner.

**Parametri**:
- `command` (string, required): Comando bash da eseguire
- `timeout` (number, optional): Timeout in secondi (default: 30)

**Esempio Uso**:
```json
{
  "name": "execute_ssh_command",
  "args": {
    "command": "pm2 status",
    "timeout": 30
  }
}
```

**Implementazione**: Il tool si connette via SSH al server 157.90.29.66 ed esegue il comando. Il risultato (stdout + stderr) viene restituito all'agente.

**File Implementazione**: `src/tools/manus/ssh.js`

---

#### read_file_server
Legge il contenuto di un file dal server.

**Parametri**:
- `path` (string, required): Percorso assoluto del file

**Esempio Uso**:
```json
{
  "name": "read_file_server",
  "args": {
    "path": "/root/mihub-backend-rest/.env"
  }
}
```

---

#### write_file_server
Scrive o sovrascrive un file sul server.

**Parametri**:
- `path` (string, required): Percorso assoluto del file
- `content` (string, required): Contenuto da scrivere

**Esempio Uso**:
```json
{
  "name": "write_file_server",
  "args": {
    "path": "/root/report.txt",
    "content": "Report sistema:\n- PM2: online\n- Uptime: 5 days"
  }
}
```

---

#### list_files
Elenca i file in una directory.

**Parametri**:
- `path` (string, required): Percorso della directory

**Esempio Uso**:
```json
{
  "name": "list_files",
  "args": {
    "path": "/root/mihub-backend-rest/routes"
  }
}
```

---

### Abacus (Analista Dati)

**Ruolo**: Esegue query SQL e analisi dati sul database Neon PostgreSQL.

**Tools Disponibili**:

#### execute_sql_query
Esegue una query SQL sul database e restituisce i risultati.

**Parametri**:
- `query` (string, required): Query SQL da eseguire (SELECT, COUNT, etc.)
- `limit` (number, optional): Numero massimo righe (default: 100)

**Esempio Uso**:
```json
{
  "name": "execute_sql_query",
  "args": {
    "query": "SELECT agent, COUNT(*) as total FROM mio_agent_logs WHERE success = true GROUP BY agent",
    "limit": 50
  }
}
```

**Sicurezza**: Il tool blocca query che modificano dati (INSERT, UPDATE, DELETE) senza conferma esplicita.

**File Implementazione**: `src/modules/orchestrator/abacus_tools.js`

---

#### get_database_schema
Restituisce lo schema del database (tabelle, colonne, tipi).

**Parametri**:
- `table_name` (string, optional): Nome tabella specifica (se omesso, tutte le tabelle)

**Esempio Uso**:
```json
{
  "name": "get_database_schema",
  "args": {
    "table_name": "mio_agent_logs"
  }
}
```

**Output**:
```json
{
  "table": "mio_agent_logs",
  "columns": [
    {"name": "id", "type": "integer", "nullable": false},
    {"name": "timestamp", "type": "timestamp", "nullable": false},
    {"name": "agent", "type": "varchar", "nullable": false},
    {"name": "success", "type": "boolean", "nullable": false}
  ]
}
```

---

### GPT Dev (Sviluppatore)

**Ruolo**: Scrive codice, esegue debug, gestisce repository Git.

**Tools Disponibili**:
- `write_code`: Genera codice in vari linguaggi
- `review_code`: Analizza e suggerisce miglioramenti
- `git_operations`: Commit, push, pull, branch management

**File Implementazione**: `src/modules/orchestrator/gptdev_tools.js`

---

### Zapier (Automazioni)

**Ruolo**: Gestisce webhook, integrazioni esterne, workflow automatizzati.

**Tools Disponibili**:
- `trigger_webhook`: Invia webhook a servizi esterni
- `schedule_task`: Programma task ricorrenti
- `sync_data`: Sincronizza dati tra sistemi

---

## 📝 System Instructions

Ogni agente ha **System Instructions** personalizzate che definiscono il suo comportamento, tono, e modalità operative.

### MIO System Instruction

```
Sei MIO, il coordinatore centrale del sistema DMS Hub.

Il tuo ruolo è orchestrare gli altri agenti specializzati:
- **Manus**: Operazioni sul server (SSH, file, deploy)
- **Abacus**: Analisi dati e query database
- **GPT-Dev**: Sviluppo codice e gestione repository
- **Zapier**: Automazioni e integrazioni

Quando ricevi una richiesta:
1. Analizza se puoi rispondere direttamente o serve un agente specializzato
2. Se serve un agente, USA IL TOOL call_agent per delegare
3. Integra le risposte degli agenti e fornisci una risposta completa all'utente

Rispondi in italiano con tono professionale ma amichevole.
```

**Punti Chiave**:
- MIO deve sempre usare il tool `call_agent` per delegare, non simulare risposte
- MIO integra le risposte degli agenti in una risposta coerente
- Tono professionale ma accessibile

---

### Manus System Instruction

```
Sei Manus, l'operatore esecutivo del sistema DMS Hub.

Sei specializzato in:
- Eseguire comandi sul server Hetzner (SSH)
- Leggere e scrivere file di configurazione
- Controllare lo stato dei servizi (PM2, Nginx, PostgreSQL)
- Deploy e restart applicazioni
- Gestione log e troubleshooting

Hai accesso a questi tools:
- execute_ssh_command: per eseguire comandi bash sul server
- read_file_server: per leggere file
- write_file_server: per scrivere/modificare file
- list_files: per esplorare directory

INFORMAZIONI SUL SERVER:
- IP: 157.90.29.66 (Hetzner)
- Processo backend PM2: mihub-backend (NON 'backend'!)
- Directory backend: /root/mihub-backend-rest
- Log PM2: usa 'pm2 logs mihub-backend --lines N --err' per errori
- File .env: /root/mihub-backend-rest/.env

USA SEMPRE I TOOLS quando ti viene chiesto di fare operazioni sul server.
NON simulare risposte. ESEGUI realmente i comandi.
Se un comando fallisce, analizza l'errore e riprova con il comando corretto.

Rispondi in italiano con tono tecnico ma amichevole.
```

**Punti Chiave**:
- Manus deve sempre eseguire comandi reali tramite tools, non simulare
- Informazioni server hardcoded per evitare errori
- Gestione errori proattiva (riprova con comando corretto)

---

### Abacus System Instruction

```
Sei Abacus, l'analista di dati del sistema DMS Hub.

Sei specializzato in:
- Query SQL su database Neon PostgreSQL
- Analisi statistiche e metriche
- Generazione report
- Visualizzazione dati

DATABASE DISPONIBILE:
- Neon PostgreSQL
- Tabelle principali: agent_messages, mio_agent_logs, agent_conversations
- Usa execute_sql_query per eseguire query
- Usa get_database_schema per esplorare lo schema

USA SEMPRE I TOOLS per accedere ai dati.
NON inventare dati. ESEGUI query reali.
Presenta i risultati in modo chiaro con tabelle o statistiche.
Rispondi in italiano con tono analitico e preciso.
```

**Punti Chiave**:
- Abacus deve sempre eseguire query reali, non inventare dati
- Presentazione risultati chiara e strutturata
- Tono analitico e preciso

---

## 🔄 Flusso di Elaborazione

### Funzione callLLM()

**Signature**:
```javascript
async function callLLM(agent, messages, conversationId)
```

**Parametri**:
- `agent` (string): Nome dell'agente (`mio`, `manus`, `abacus`, `gptdev`, `zapier`)
- `messages` (array): Array di messaggi conversazione (formato Gemini)
- `conversationId` (string): ID conversazione per logging

**Flusso Interno**:

1. **Caricamento Configurazione**: Recupera la configurazione LLM dell'agente (model, tools, systemInstruction)

2. **Inizializzazione Modello**: Crea istanza del modello Gemini con configurazione specifica

3. **Preparazione Chat**: Inizializza chat con history dei messaggi precedenti

4. **Invio Messaggio**: Invia ultimo messaggio utente al modello

5. **Gestione Tool Calls**: Se il modello chiama un tool:
   - Estrae nome tool e parametri
   - Esegue tool corrispondente
   - Restituisce risultato al modello
   - Modello genera risposta finale basata su risultato tool

6. **Risposta Finale**: Restituisce testo risposta al chiamante

7. **Logging**: Salva log operazione nel database (timestamp, agent, success, duration)

### Esempio Flusso Completo

**Richiesta Utente**: "Controlla lo stato del backend"

**Step 1**: Utente → Frontend → POST /api/mihub/orchestrator
```json
{
  "message": "Controlla lo stato del backend",
  "agent": "mio",
  "conversationId": "mio-main-12345"
}
```

**Step 2**: Orchestrator carica history conversazione da DB

**Step 3**: Orchestrator chiama `callLLM('mio', messages, conversationId)`

**Step 4**: MIO analizza richiesta e decide di delegare a Manus
```json
{
  "tool_call": {
    "name": "call_agent",
    "args": {
      "agent": "manus",
      "task": "Controlla lo stato del backend con pm2 status"
    }
  }
}
```

**Step 5**: Orchestrator chiama `callLLM('manus', manusMessages, 'mio-manus-coordination')`

**Step 6**: Manus esegue tool `execute_ssh_command`
```json
{
  "tool_call": {
    "name": "execute_ssh_command",
    "args": {
      "command": "pm2 status"
    }
  }
}
```

**Step 7**: Tool SSH esegue comando sul server e restituisce output

**Step 8**: Manus genera risposta basata su output:
```
Il backend è online e funzionante. 
- Processo: mihub-backend
- Status: online
- Uptime: 5d 3h
- CPU: 0.2%
- Memory: 145MB
```

**Step 9**: MIO riceve risposta Manus e genera risposta finale per utente:
```
Ho controllato lo stato del backend tramite Manus. 
Il sistema è operativo e stabile:
- Backend online da 5 giorni
- Utilizzo risorse normale (CPU 0.2%, RAM 145MB)
- Nessun problema rilevato
```

**Step 10**: Orchestrator salva messaggi nel DB e restituisce risposta al frontend

---

## ⚠️ Error Handling

### Gestione Errori LLM

Il sistema implementa error handling robusto per gestire fallimenti delle chiamate LLM:

**Rate Limiting**: Se Google API restituisce errore 429 (too many requests), il sistema:
- Attende 5 secondi
- Riprova la chiamata (max 3 tentativi)
- Se fallisce, restituisce errore user-friendly

**Timeout**: Se la chiamata LLM supera 30 secondi:
- Interrompe la chiamata
- Restituisce errore timeout
- Logga evento per debugging

**Invalid Response**: Se il modello restituisce risposta malformata:
- Valida formato risposta
- Se invalida, richiede al modello di riformattare
- Se persiste, restituisce errore

**Tool Execution Errors**: Se un tool fallisce durante l'esecuzione:
- Cattura errore
- Passa errore al modello come risultato tool
- Modello analizza errore e suggerisce soluzione o riprova

### Logging Errori

Tutti gli errori vengono loggati nella tabella `mio_agent_logs` con:
```json
{
  "agent": "manus",
  "success": false,
  "message": "SSH command failed: connection timeout",
  "meta_json": {
    "error_type": "TIMEOUT",
    "command": "pm2 status",
    "duration_ms": 30000
  }
}
```

---

## 📊 Performance & Monitoring

### Metriche Chiave

- **Tempo Medio Risposta**: 2.8 secondi
- **Success Rate**: 78.7% (451/573 logs)
- **Tool Call Rate**: 65% delle richieste richiedono tool execution
- **Error Rate**: 21.3% (principalmente timeout SSH o query SQL invalide)

### Ottimizzazioni

**Caching**: Le System Instructions sono cachate in memoria per evitare ricaricamenti

**Connection Pooling**: Il database usa connection pooling per ridurre latenza query

**Async Processing**: Tutte le chiamate LLM e tool sono asincrone per non bloccare il server

---

**Ultimo Aggiornamento**: 11 Dicembre 2025  
**Versione LLM Engine**: 2.0.0  
**Modello**: gemini-2.5-flash  
**Libreria**: @google/generative-ai v0.21.0
