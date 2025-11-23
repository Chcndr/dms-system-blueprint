# Architettura Modulo Guardian

**Data:** 23 Novembre 2025

**Autore:** Manus AI

## 1. Introduzione

Il modulo **Guardian** è un sistema centralizzato di monitoraggio, logging e debug per l'applicazione DMS Hub. È stato introdotto per ripristinare e migliorare le sezioni **Debug / Log / Integrazioni** della Dashboard PA, fornendo un controllo completo sugli errori, le performance e l'utilizzo delle API.

### 1.1. Obiettivi

*   **Inventario API Centralizzato:** Fornire un elenco dinamico e sempre aggiornato di tutti gli endpoint API disponibili nel sistema (tRPC e REST legacy).
*   **Logging Unificato:** Centralizzare i log provenienti da tutte le componenti dell'applicazione (DMS Hub, MIHUB, Agenti AI) in un unico punto accessibile.
*   **Debug Facilitato:** Offrire strumenti per il debug e il test degli endpoint direttamente dalla Dashboard PA.
*   **Monitoraggio Proattivo:** Identificare e risolvere rapidamente errori, colli di bottiglia e anomalie.

## 2. Architettura Generale

Il modulo Guardian si compone di due parti principali:

1.  **Backend (MIHUB):** Un insieme di servizi e un router tRPC dedicati che espongono gli endpoint per l'inventario API, i log e il debug.
2.  **Frontend (Dashboard PA):** Una serie di componenti React che si collegano agli endpoint del backend per visualizzare i dati e fornire le funzionalità di interazione.

### 2.1. Stack Tecnologico

*   **Backend:** Node.js, tRPC, Drizzle ORM, TypeScript
*   **Frontend:** React, TypeScript, Vite, tRPC Client, Recharts

## 3. Componenti Backend

Il backend del modulo Guardian è implementato all'interno del server MIHUB (`dms-hub-app-new/server`).

### 3.1. `apiInventoryService.ts`

Questo servizio è responsabile della creazione e gestione dell'inventario API.

*   **`getAPIInventory()`:** Funzione che ritorna un array di oggetti `APIEndpoint`, rappresentanti tutti gli endpoint disponibili nel sistema. L'inventario è attualmente statico ma è progettato per essere generato dinamicamente in futuro.
*   **`APIEndpoint` (interfaccia):** Definisce la struttura di un endpoint, includendo `id`, `path`, `method`, `description`, `category`, `status`, ecc.

### 3.2. `apiLogsService.ts`

Questo servizio gestisce il logging centralizzato.

*   **`addLog()`:** Aggiunge un nuovo log al sistema. I log sono salvati in memoria (per la versione attuale) e su console.
*   **`getLogs()`:** Ritorna i log filtrati per livello, applicazione o tipo.
*   **`getLogStats()`:** Fornisce statistiche aggregate sui log.
*   **`APILog` (interfaccia):** Definisce la struttura di un log, includendo `id`, `timestamp`, `level`, `app`, `type`, `message`, ecc.

### 3.3. `guardianRouter.ts`

Router tRPC che espone gli endpoint del modulo Guardian.

| Endpoint                        | Metodo | Descrizione                                      |
| ------------------------------- | ------ | ------------------------------------------------ |
| `guardian.integrations`         | `GET`  | Ritorna l'inventario API completo.               |
| `guardian.logs`                 | `GET`  | Ritorna i log centralizzati con filtri.          |
| `guardian.debug.testEndpoint`   | `POST` | Proxy per testare un endpoint API.               |
| `guardian.logs.init`            | `POST` | Inizializza log di demo per la visualizzazione.  |
| `guardian.stats`                | `GET`  | Ritorna statistiche aggregate su API e log.      |

## 4. Componenti Frontend

Il frontend del modulo Guardian è integrato nella Dashboard PA (`dms-hub-app-new/client`).

### 4.1. `GuardianIntegrations.tsx`

Componente che visualizza l'inventario API.

*   **Funzionalità:**
    *   Si collega a `trpc.guardian.integrations` per ottenere l'elenco degli endpoint.
    *   Visualizza gli endpoint raggruppati per categoria.
    *   Permette di filtrare per categoria e stato.
    *   Mostra statistiche aggregate (totale API, attive, con auth, ecc.).
    *   Fornisce una sezione per testare gli endpoint tramite `trpc.guardian.testEndpoint`.
*   **Sostituisce:** Il precedente componente `APIDashboardV2.tsx` che usava dati statici.

### 4.2. `GuardianLogsSection.tsx`

Componente per la visualizzazione dei log.

*   **Funzionalità:**
    *   Si collega a `trpc.guardian.logs` per ottenere i log.
    *   Mostra i log in tab separati: "Tutti i Log", "System Logs", "Guardian Logs".
    *   Visualizza statistiche sui log (totale, info, warning, error).
    *   Fornisce una formattazione chiara e badge colorati per i livelli di log.
*   **Sostituisce:** La precedente implementazione `LogsSection` che leggeva da un file statico su GitHub.

### 4.3. `GuardianDebugSection.tsx`

Componente per il monitoraggio degli errori.

*   **Funzionalità:**
    *   Si collega a `trpc.guardian.logs` filtrando per `level: 'error'`. 
    *   Mostra un elenco dettagliato degli errori e dei warning.
    *   Fornisce suggerimenti automatici per la risoluzione dei problemi (`getSuggestion()`).
    *   Visualizza statistiche sugli errori e warning.
*   **Sostituisce:** Il precedente componente `GuardianDebugStats` che leggeva da un file statico.

## 5. Flusso Dati

1.  Il backend (server MIHUB) viene avviato e i router tRPC, incluso `guardianRouter`, vengono registrati.
2.  L'utente accede alla Dashboard PA e naviga nelle sezioni **Integrazioni**, **Log** o **Debug**.
3.  I componenti frontend (`GuardianIntegrations`, `GuardianLogsSection`, `GuardianDebugSection`) effettuano una chiamata tRPC agli endpoint Guardian corrispondenti.
4.  Il `guardianRouter` nel backend riceve la richiesta e invoca i servizi `apiInventoryService` o `apiLogsService`.
5.  I servizi recuperano i dati (dall'inventario statico o dai log in memoria) e li ritornano al router.
6.  Il router invia i dati al frontend.
7.  I componenti React renderizzano i dati, mostrando l'inventario API, i log o le informazioni di debug.

## 6. Conclusioni

Il modulo Guardian fornisce una soluzione robusta e centralizzata per il monitoraggio e il debug dell'applicazione DMS Hub. Sostituendo le precedenti implementazioni basate su dati statici e file esterni, Guardian garantisce che le informazioni siano sempre aggiornate e provenienti direttamente dal backend, offrendo un controllo completo e in tempo reale sullo stato del sistema.
