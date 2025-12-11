# Report Analisi Sistema DMS/MIO-HUB

**Data:** 27 Novembre 2025  
**Autore:** Manus AI (per MIO Agent)  
**Versione:** 1.0

---

## Executive Summary

Il sistema DMS (Digital Market System) e MIO-HUB costituisce un ecosistema complesso per la gestione digitale dei mercati ambulanti su scala nazionale. L'architettura è distribuita su più piattaforme (Vercel, Hetzner, Neon) e integra componenti frontend, backend REST, database PostgreSQL e servizi esterni (GIS, PDND, SSO).

Questo report fornisce una panoramica strutturata dei componenti principali, dei flussi dati, degli endpoint critici e dei punti di attenzione per la manutenzione e l'evoluzione del sistema.

---

## 1. Componenti Sistema

Il sistema è organizzato secondo un'architettura distribuita che separa le responsabilità tra diversi layer tecnologici.

### 1.1 Frontend (Vercel)

| Proprietà | Valore |
| :--- | :--- |
| **Repository** | [Chcndr/dms-hub-app-new](https://github.com/Chcndr/dms-hub-app-new) |
| **Piattaforma** | Vercel |
| **Framework** | Next.js + React + tRPC |
| **URL Produzione** | https://dms-hub-app-new.vercel.app |
| **Branch Deploy** | `master` |

**Funzionalità Principali:**
- Dashboard PA per monitoraggio mercati, posteggi, imprese e concessioni
- Sezione Integrazioni e API Dashboard per visualizzazione endpoint
- Logs e Guardian Logs per tracciamento chiamate
- Gestione Mercati con mappa GIS integrata

### 1.2 Backend REST (Hetzner)

| Proprietà | Valore |
| :--- | :--- |
| **Repository** | [Chcndr/mihub-backend-rest](https://github.com/Chcndr/mihub-backend-rest) |
| **Piattaforma** | Hetzner VPS |
| **URL Base** | https://mihub.157-90-29-66.nip.io/api |
| **Process Manager** | PM2 |
| **Server** | 157.90.29.66 (SSH root@157.90.29.66) |

**Endpoint Critici:**
- `GET /api/gis/health` - Health check modulo GIS ✅
- `GET /api/dmsHub/markets/list` - Lista mercati
- `GET /api/markets/:marketId/companies` - Lista imprese per mercato ✅
- `POST /api/markets/:marketId/companies` - Crea nuova impresa ✅
- `PUT /api/markets/companies/:companyId` - Aggiorna impresa ✅
- `GET /api/markets/:marketId/concessions` - Lista concessioni ✅
- `POST /api/markets/:marketId/concessions` - Crea nuova concessione ✅
- `PUT /api/markets/concessions/:concessionId` - Aggiorna concessione ✅

### 1.3 Database (Neon PostgreSQL)

| Proprietà | Valore |
| :--- | :--- |
| **Provider** | Neon (Serverless Postgres) |
| **Schema** | 39 tabelle (Drizzle ORM) |
| **Tabelle Principali** | `markets`, `stalls`, `vendors`, `concessions`, `dms_companies`, `dms_suap_instances` |

**Migrazioni:** Gestite tramite file `.sql` versionati nel Blueprint (cartella `/sql`).

### 1.4 Orchestratore Multi-Agente

| Proprietà | Valore |
| :--- | :--- |
| **URL** | https://orchestratore.mio-hub.me/api/mihub/orchestrator |
| **Agenti** | MIO (coordinatore), Abacus, Guardian, Pepe |
| **Stato** | Fase 1 - Vista singola MIO attiva |

---

## 2. Flussi Dati Principali

Il sistema gestisce diversi flussi di dati che attraversano i componenti descritti.

### 2.1 Gestione Mercati e Posteggi

**Flusso:**
1. L'utente accede al Dashboard PA → Gestione Mercati.
2. Il frontend chiama `GET /api/dmsHub/markets/list` per ottenere la lista dei mercati.
3. Per ogni mercato, il frontend carica i posteggi tramite `GET /api/markets/:marketId/stalls`.
4. I posteggi vengono visualizzati su mappa GIS (integrazione Pepe GIS).

### 2.2 Gestione Imprese e Concessioni

**Flusso:**
1. L'utente apre il tab "Imprese / Concessioni" per un mercato specifico.
2. Il frontend chiama `GET /api/markets/:marketId/companies` per caricare le imprese registrate.
3. L'utente può creare una nuova impresa tramite modale, che invia `POST /api/markets/:marketId/companies`.
4. Il backend MIHUB salva i dati nella tabella `vendors` del database Neon.
5. Le concessioni seguono un flusso analogo con endpoint `/concessions`.

### 2.3 Logging e Guardian

**Flusso:**
1. Ogni chiamata agli endpoint MIHUB viene intercettata dal middleware di logging.
2. I log vengono salvati e resi disponibili nella sezione "Logs → Guardian Logs" del Dashboard PA.
3. Gli endpoint sono registrati in `api/index.json` per essere visibili in "Integrazioni e API → API Dashboard".

---

## 3. Endpoint Critici e Stato Implementazione

La tabella seguente riassume lo stato degli endpoint principali del sistema.

| Endpoint | Metodo | Categoria | Stato | Note |
| :--- | :--- | :--- | :--- | :--- |
| `/api/gis/health` | GET | GIS | ✅ Implemented | Health check funzionante |
| `/api/dmsHub/markets/list` | GET | Mercati | ✅ Implemented | Restituisce lista mercati |
| `/api/markets/:marketId/stalls` | GET | Posteggi | ✅ Implemented | Lista posteggi per mercato |
| `/api/markets/:marketId/companies` | GET | Imprese | ✅ Implemented | Lista imprese per mercato |
| `/api/markets/:marketId/companies` | POST | Imprese | ✅ Implemented | Crea nuova impresa |
| `/api/markets/companies/:companyId` | PUT | Imprese | ✅ Implemented | Aggiorna impresa |
| `/api/markets/:marketId/concessions` | GET | Concessioni | ✅ Implemented | Lista concessioni per mercato |
| `/api/markets/:marketId/concessions` | POST | Concessioni | ✅ Implemented | Crea nuova concessione |
| `/api/markets/concessions/:concessionId` | PUT | Concessioni | ✅ Implemented | Aggiorna concessione |

**Nota:** Tutti gli endpoint Imprese/Concessioni sono stati recentemente aggiunti e registrati in `api/index.json` (commit `0139a07`).

---

## 4. Punti Scoperti e Azioni Necessarie

Sulla base dell'analisi del Blueprint e dello stato attuale del sistema, sono emersi i seguenti punti di attenzione.

### 4.1 Endpoint Non Ancora Registrati in `api/index.json`

**Problema:** Alcuni endpoint implementati nel backend MIHUB potrebbero non essere ancora registrati in `api/index.json`, causando l'errore "endpoint not available" nel Guardian.

**Azione:** Verificare sistematicamente tutti gli endpoint in `mihub-backend-rest/routes/` e confrontarli con `api/index.json`. Aggiungere eventuali endpoint mancanti.

### 4.2 Migrazioni DB da Eseguire

**Problema:** Non è chiaro se tutte le migrazioni SQL documentate nel Blueprint siano state eseguite sul database Neon di produzione.

**Azione:** Creare una checklist delle migrazioni SQL nella cartella `/sql` e verificare quali sono state applicate. Documentare lo stato nel Blueprint.

### 4.3 Task di Deploy da Automatizzare

**Problema:** Attualmente il deploy del backend Hetzner richiede accesso SSH manuale e restart di PM2.

**Azione:** Valutare l'implementazione di un workflow CI/CD (es. GitHub Actions) per automatizzare il deploy su Hetzner dopo ogni merge su `master`.

### 4.4 Handler MIO da Sistemare

**Problema:** Il MASTER_SYSTEM_PLAN segnala che manca il check `if (mioLoading) return;` nell'handler MIO del frontend.

**Azione:** Verificare il componente React che gestisce la chat MIO e aggiungere il controllo per evitare chiamate duplicate durante il caricamento.

### 4.5 Documentazione Permessi MIO

**Problema:** Non è chiaro quali repository e risorse MIO può accedere e modificare.

**Azione:** ✅ **Completato** - Creato il documento `MIO_PERMESSI_E_FLUSSO_DI_LAVORO.md` che definisce i permessi di lettura/scrittura per MIO.

---

## 5. Raccomandazioni

Per garantire la stabilità e la manutenibilità del sistema, si raccomanda di:

1. **Seguire sempre il Playbook di Deploy:** Ogni modifica al codice deve seguire le procedure definite in `PLAYBOOK_DEPLOY.md`.
2. **Aggiornare `api/index.json` per ogni nuovo endpoint:** Questo garantisce che Guardian e API Dashboard siano sempre allineati.
3. **Versionare le migrazioni SQL:** Ogni modifica allo schema DB deve essere tracciata con un file `.sql` datato.
4. **Testare gli endpoint dopo ogni deploy:** Utilizzare i comandi `curl` documentati nel Playbook per verificare il corretto funzionamento.
5. **Documentare ogni task completato:** MIO deve scrivere un report finale nella cartella `/reports/mio/` per ogni task significativo.

---

## 6. Conclusioni

Il sistema DMS/MIO-HUB è in uno stato funzionante e ben documentato. Le recenti aggiunte degli endpoint Imprese/Concessioni hanno migliorato la completezza del sistema. Con l'adozione del Playbook di Deploy e della guida Permessi MIO, il sistema è ora pronto per essere gestito in modo più strutturato e automatizzato.

Le azioni prioritarie da intraprendere sono:
- Verificare e completare la registrazione di tutti gli endpoint in `api/index.json`.
- Documentare lo stato delle migrazioni SQL.
- Valutare l'automazione del deploy backend.

---

**Fine del Report**
