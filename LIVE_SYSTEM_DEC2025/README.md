# MIO HUB - SISTEMA LIVE (Dicembre 2025)

> **Documentazione del sistema MIO Hub funzionante e operativo**  
> Questa documentazione riflette esattamente il codice in produzione sul server Hetzner e Vercel.

## 📋 Indice

1. **[01_ARCHITECTURE](./01_ARCHITECTURE/)** - Architettura sistema "8 Isole"
2. **[02_BACKEND_CORE](./02_BACKEND_CORE/)** - Backend Node.js, API, LLM Engine, Tools
3. **[03_DATABASE_SCHEMA](./03_DATABASE_SCHEMA/)** - Schema PostgreSQL Neon
4. **[04_FRONTEND_DASHBOARD](./04_FRONTEND_DASHBOARD/)** - Dashboard PA Vercel

---

## 🎯 Panoramica Sistema

Il **MIO Hub** è un sistema multi-agente orchestrato che coordina diversi agenti AI specializzati per gestire il Digital Market System (DMS). Il sistema è basato su un'architettura "8 Isole" che separa le conversazioni dirette utente-agente dalle conversazioni di coordinamento backstage.

### Componenti Principali

Il sistema è composto da tre componenti principali che lavorano in sinergia:

**Backend Node.js** ospitato su server Hetzner (157.90.29.66) gestisce l'orchestrazione degli agenti, l'elaborazione delle richieste tramite LLM Google Gemini, e l'esecuzione dei tool specializzati. Il backend espone oltre 30 endpoint API REST per gestire conversazioni, logging, integrazioni e business logic.

**Database PostgreSQL Neon** centralizza tutti i dati del sistema in cloud, includendo messaggi delle conversazioni, logs degli agenti, dati business (mercati, imprese, qualificazioni) e configurazioni. Il database utilizza uno schema ottimizzato per gestire le 8 isole conversazionali tramite il campo `conversation_id`.

**Frontend Dashboard PA** deployato su Vercel fornisce l'interfaccia utente per interagire con gli agenti, visualizzare logs e statistiche, gestire mercati e imprese. La dashboard supporta due modalità di visualizzazione: Vista Singola (conversazione diretta con un agente) e Vista 4 Agenti (MIO coordina backstage).

### Agenti Attivi

Il sistema coordina cinque agenti AI specializzati, ciascuno con competenze specifiche:

**MIO** è l'orchestratore principale che coordina gli altri agenti, gestisce le richieste complesse dividendole in subtask, e mantiene il contesto conversazionale. MIO decide quale agente attivare in base alla natura della richiesta.

**Manus** è l'esecutivo generale che gestisce operazioni di sistema, deploy, configurazioni server, e operazioni SSH. È l'agente più versatile per task operativi.

**Abacus** è specializzato in analisi dati, query SQL, integrazioni GitHub, e generazione report. Fornisce insights analitici e statistiche.

**GPT Dev** è il developer agent che scrive codice, esegue debug, gestisce repository Git, e implementa nuove funzionalità.

**Zapier** gestisce automazioni, webhook, integrazioni con servizi esterni, e workflow automatizzati.

---

## 🏗️ Architettura "8 Isole"

L'architettura del sistema è organizzata in 8 "isole" conversazionali indipendenti, ognuna con un `conversation_id` univoco che permette di separare e tracciare le diverse tipologie di interazione.

### Isole Vista Singola (User → Agent Diretto)

Nelle conversazioni Vista Singola, l'utente interagisce direttamente con un singolo agente senza intermediazione di MIO. Ogni isola ha un `conversation_id` nel formato `{agent}-single`:

- **Island 1**: `manus-single` - User ↔ Manus
- **Island 2**: `abacus-single` - User ↔ Abacus  
- **Island 3**: `zapier-single` - User ↔ Zapier
- **Island 4**: `gptdev-single` - User ↔ GPT Dev

### Isole Vista 4 Agenti (MIO Coordina Backstage)

Nelle conversazioni Vista 4 Agenti, MIO orchestra gli agenti backstage per completare task complessi. L'utente vede solo MIO, ma dietro le quinte MIO coordina gli altri agenti. Ogni coordinamento ha un `conversation_id` nel formato `mio-{agent}-coordination`:

- **Island 5**: `mio-manus-coordination` - MIO ↔ Manus (backstage)
- **Island 6**: `mio-abacus-coordination` - MIO ↔ Abacus (backstage)
- **Island 7**: `mio-zapier-coordination` - MIO ↔ Zapier (backstage)
- **Island 8**: `mio-gptdev-coordination` - MIO ↔ GPT Dev (backstage)

### Database Condiviso

Tutte le 8 isole salvano i messaggi nella stessa tabella `agent_messages` del database PostgreSQL Neon. Il campo `conversation_id` permette di filtrare e separare le conversazioni. Questo approccio centralizzato semplifica il logging, l'analisi e il debug del sistema.

---

## 🔗 Endpoint API Principali

Il backend espone numerosi endpoint REST organizzati per funzionalità. Gli endpoint più critici per il funzionamento del sistema sono:

### Orchestrazione

**POST /api/mihub/orchestrator** è l'endpoint principale che riceve le richieste dell'utente, le analizza tramite LLM, decide quale agente attivare, e restituisce la risposta. Questo endpoint gestisce tutto il flusso orchestrato.

### Conversazioni

**GET /api/mihub/chats** restituisce la lista delle conversazioni attive con metadati (agent, status, ultimo messaggio).

**POST /api/mihub/chats** crea una nuova conversazione specificando l'agente e il `conversation_id`.

**GET /api/mihub/chats/:id** recupera tutti i messaggi di una specifica conversazione per visualizzarli nella dashboard.

### Logging & Guardian

**GET /api/guardian/logs** recupera gli ultimi 50 log degli agenti per la sezione "Attività Agenti Recente (Guardian)" nella dashboard. Supporta filtri per agent, date range.

**GET /api/logs/getLogs** recupera tutti i logs del sistema con filtri avanzati (agent, serviceId, success, date range).

**POST /api/logs/createLog** crea un nuovo log entry quando un agente esegue un'azione.

**GET /api/logs/stats** restituisce statistiche aggregate sui logs (totale, successi, errori, warning).

### Business Logic

**GET /api/imprese** restituisce la lista delle imprese registrate nel sistema.

**GET /api/qualificazioni** restituisce le qualificazioni disponibili per le imprese.

**GET /api/markets** gestisce i dati dei mercati (posizioni, orari, posteggi).

---

## 📊 Statistiche Sistema (11 Dicembre 2025)

Il sistema è operativo e gestisce un volume significativo di operazioni giornaliere. Le statistiche attuali mostrano:

**Database**: 573 logs totali registrati, di cui 451 operazioni completate con successo (78.7%) e 122 errori (21.3%). Il sistema traccia attività di 11 agenti unici.

**Performance**: Il tempo medio di risposta dell'orchestratore è di circa 3 secondi per richieste complesse, con picchi fino a 5 secondi per operazioni che richiedono coordinamento multi-agente.

**Uptime**: Il backend Node.js è gestito da PM2 con auto-restart, garantendo alta disponibilità. Il database Neon offre 99.9% di uptime garantito.

---

## 🚀 Quick Start per AI Agents

Se sei un AI Agent (come MIO) e devi capire come funziona il sistema, segui questo percorso:

1. **Leggi [01_ARCHITECTURE/system-overview.md](./01_ARCHITECTURE/system-overview.md)** per comprendere l'architettura "8 Isole" e il flusso delle richieste.

2. **Studia [02_BACKEND_CORE/api-map.md](./02_BACKEND_CORE/api-map.md)** per conoscere tutti gli endpoint API disponibili e come chiamarli.

3. **Esamina [02_BACKEND_CORE/llm-engine.md](./02_BACKEND_CORE/llm-engine.md)** per capire come funziona l'LLM Engine (Google Gemini) e come vengono processate le richieste.

4. **Consulta [02_BACKEND_CORE/tools-system.md](./02_BACKEND_CORE/tools-system.md)** per vedere quali tool puoi usare (SSH, Browser, SQL, GitHub).

5. **Verifica [03_DATABASE_SCHEMA/schema.md](./03_DATABASE_SCHEMA/schema.md)** per conoscere la struttura del database e le tabelle disponibili.

---

## 📝 Documentazione Aggiuntiva

### Guide Operative

Per operazioni specifiche consulta le guide nella cartella legacy `00_LEGACY_ARCHIVE/07_guide_operative/`:
- Deploy backend su Hetzner
- Deploy frontend su Vercel  
- Troubleshooting errori comuni
- Configurazione agenti

### Credenziali & Accessi

Le credenziali di accesso ai servizi (Database, Server, API Keys) sono documentate in `00_LEGACY_ARCHIVE/CREDENTIALS.md` e `00_LEGACY_ARCHIVE/CREDENZIALIEACCESSI-DMS_MIO-HUB.md`.

**⚠️ ATTENZIONE**: Le credenziali sono sensibili e non devono essere condivise pubblicamente.

---

## 🔄 Aggiornamenti

Questa documentazione viene aggiornata ogni volta che il sistema subisce modifiche significative. L'ultimo aggiornamento è stato effettuato l'**11 Dicembre 2025** nell'ambito dell'**Operazione "Specchio Reale"**.

Per segnalare discrepanze tra questa documentazione e il sistema reale, contattare il team di sviluppo.

---

**Versione Sistema**: 2.0.0  
**Data Documentazione**: 11 Dicembre 2025  
**Autore**: Manus AI Agent  
**Repository**: [Chcndr/dms-system-blueprint](https://github.com/Chcndr/dms-system-blueprint)
