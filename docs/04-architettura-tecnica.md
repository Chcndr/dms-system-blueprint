# 04. Architettura Tecnica

Questo documento descrive l'architettura del software, sia per il backend che per il frontend.

## Backend (MIO-Hub Orchestrator)

Il backend è un'applicazione Node.js che utilizza Express come web server e **tRPC** per definire e esporre le API in modo typesafe. La logica di business è suddivisa in router, ciascuno dedicato a un dominio funzionale specifico.

### 1. Struttura dei Router

Il router principale (`server/routers.ts`) aggrega i seguenti sub-router:

- `dmsHubRouter`: Gestione delle entità del dominio DMS (mercati, posteggi, prenotazioni, pagamenti).
- `mihubRouter`: Gestione degli agenti AI, delle conversazioni e dei task.
- `integrationsRouter`: Gestione di webhook e integrazioni con sistemi esterni.
- `eventBus`: Un semplice bus per la gestione di eventi di sistema.

### 2. Endpoint API (Procedure tRPC)

Di seguito la lista delle procedure tRPC esposte, che costituiscono il Core API del sistema.

#### 2.1. `dmsHubRouter`

| Procedura | Descrizione |
|---|---|
| `getMarkets` | Ottiene la lista di tutti i mercati. |
| `getMarketById` | Ottiene un mercato specifico. |
| `createMarket` | Crea un nuovo mercato. |
| `updateMarket` | Aggiorna un mercato esistente. |
| `deleteMarket` | Elimina un mercato. |
| `getStallsByMarket` | Ottiene i posteggi di un mercato. |
| `createStall` | Crea un nuovo posteggio. |
| `updateStall` | Aggiorna un posteggio. |
| `deleteStall` | Elimina un posteggio. |
| `getBookingsByMarket` | Ottiene le prenotazioni di un mercato. |
| `createBooking` | Crea una nuova prenotazione. |
| `cancelBooking` | Annulla una prenotazione. |

#### 2.2. `mihubRouter`

| Procedura | Descrizione |
|---|---|
| `getConversations` | Ottiene le conversazioni di un utente. |
| `createConversation` | Crea una nuova conversazione. |
| `getTasks` | Ottiene i task di un utente. |
| `createTask` | Crea un nuovo task. |
| `orchestrator` | Invia un messaggio all'orchestratore degli agenti AI. |

## Frontend (Dashboard PA)

Il frontend è un'applicazione single-page costruita con **React** e **Vite**, che garantisce un'esperienza utente moderna e reattiva.

### 1. Struttura dei Componenti

I componenti riutilizzabili sono organizzati nella directory `client/src/components` e includono elementi chiave come:

- `Dashboard`: Il layout principale dell'applicazione.
- `Chat`: L'interfaccia per interagire con gli agenti AI.
- `Map`: Il componente per la visualizzazione e l'interazione con le mappe GIS.
- `Table`: Un componente generico per la visualizzazione tabellare dei dati.

### 2. Pagine e Routing

Il routing è gestito da `react-router-dom`. Le pagine principali includono:

- `/`: Homepage
- `/dashboard-pa`: La dashboard principale per la Pubblica Amministrazione.
- `/login`: La pagina di autenticazione.

### 3. State Management

Lo stato globale dell'applicazione è gestito tramite una combinazione di:

- **React Context**: Utilizzato per informazioni globali come lo stato dell'utente e le configurazioni dell'applicazione.
- **tRPC React Query**: Sfruttato per il data fetching, il caching e la sincronizzazione dello stato con il backend in modo efficiente e typesafe.
