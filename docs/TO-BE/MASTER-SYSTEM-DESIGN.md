# MASTER SYSTEM DESIGN (TO-BE)

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Stato:** Ufficiale  
**Obiettivo:** Definire l'architettura finale (TO-BE) del sistema DMS, che funge da guida per tutti gli sviluppi futuri.

---

## 1. Visione e Principi Guida

L'architettura TO-BE mira a creare un sistema **centralizzato, moderno e scalabile**, superando la frammentazione attuale. I principi chiave sono:

- **Single Source of Truth**: Un unico database e un unico "cervello" (Core API) per la business logic.
- **Separazione delle Responsabilità (SoC)**: Il frontend è responsabile solo della UI, il backend della logica di business.
- **API-First**: Tutte le interazioni con i dati avvengono tramite API ben definite.
- **Interoperabilità**: Il sistema legacy DMS su Heroku viene trattato come un sistema esterno da integrare, non come il centro del sistema.

---

## 2. Decisioni Architetturali Non Negoziabili

Le seguenti decisioni sono state prese e non sono soggette a modifiche:

| Componente | Decisione | Motivazione |
|---|---|---|
| **Database** | **PostgreSQL Unico** | Standardizzazione, performance e scalabilità. Abbandono totale di PlanetScale/MySQL. |
| **Backend** | **DMS Core API su Hetzner** | Centralizzazione di tutta la business logic in un unico servizio. Sarà l'unico "cervello" del sistema. |
| **Frontend** | **Next.js su Vercel** | Responsabile solo della UI. Comunica **esclusivamente** tramite il DMS Core API (via tRPC/REST/GraphQL). **Nessun accesso diretto al database.** |
| **Sistema Legacy** | **DMS su Heroku come sistema esterno** | Trattato come una fonte dati/operativa da integrare durante la transizione, ma non è più il centro del sistema. |
| **App Mobile** | **Integrazione con DMS Core API** | L'app per gli ambulanti parlerà direttamente con il nuovo Core API, non più con il gestionale legacy. |
| **GIS / Mappe** | **Layer di visualizzazione sui dati** | Le mappe saranno un layer che visualizza i dati (piazzole, mercati) presenti nel DB centrale, agganciati a ID stabili. |

---

## 3. Architettura dei Componenti

![Diagramma Architettura TO-BE](architecture_diagram_tobe.png)  
*(Nota: il diagramma verrà generato e aggiunto in una fase successiva)*

### 3.1 DMS Core API (Hetzner)
- **Stack**: Node.js, tRPC (o REST/GraphQL se necessario).
- **Responsabilità**:
  - Gestione di tutta la logica di business (creazione concessioni, gestione presenze, ecc.).
  - Unico punto di accesso ai dati del database PostgreSQL.
  - Esposizione di endpoint sicuri per frontend e app mobile.
  - Integrazione con il sistema legacy DMS su Heroku per la sincronizzazione dei dati durante la fase di transizione.

### 3.2 Frontend DMS-HUB (Vercel)
- **Stack**: Next.js, React.
- **Responsabilità**:
  - Fornire l'interfaccia utente amministrativa per la gestione di mercati, ambulanti, concessioni, ecc.
  - Eseguire chiamate al DMS Core API per leggere e scrivere dati.
  - **NON deve contenere logica di business o accessi diretti al database.**

### 3.3 Database (PostgreSQL su Hetzner/Neon)
- **Modello**: Schema relazionale normalizzato.
- **Responsabilità**: Unica fonte di verità per tutti i dati del sistema (mercati, piazzole, ambulanti, concessioni, presenze, utenti, wallet, ecc.).
- **Accesso**: Consentito solo dal DMS Core API.

### 3.4 Integrazioni
- **DMS Legacy (Heroku)**: Il Core API si connetterà al database di Heroku (in sola lettura) per migrare i dati e garantire l'operatività durante la transizione.
- **App Mobile DMS**: Si connetterà agli endpoint del Core API per leggere dati (es. concessioni) e scrivere dati (es. presenze).
- **GIS**: Un servizio di mappe (es. Mapbox, Leaflet) verrà integrato nel frontend e utilizzerà i dati forniti dal Core API per visualizzare le piante dei mercati.

---

## 4. Flusso Dati: Giornata Tipo di Mercato

1.  **Setup Mattutino**:
    - Un operatore comunale accede al **Frontend DMS-HUB**.
    - Il frontend chiama il **Core API** per ottenere la lista delle concessioni e delle presenze attese per il mercato odierno.
    - Il **Core API** legge i dati dal **Database PostgreSQL**.

2.  **Arrivo Ambulante (con App)**:
    - L'ambulante arriva al mercato e fa il check-in tramite l'**App Mobile DMS**.
    - L'app invia una richiesta di registrazione presenza al **Core API**.
    - Il **Core API** valida la concessione e registra la presenza nel **Database PostgreSQL**.

3.  **Gestione Spuntisti (senza App)**:
    - L'operatore comunale, tramite il **Frontend DMS-HUB**, visualizza le piazzole libere (dati forniti dal **Core API**).
    - Assegna una piazzola a uno spuntista, registrando la presenza tramite un'azione sul frontend.
    - Il frontend invia la richiesta al **Core API**, che aggiorna il **Database PostgreSQL**.

4.  **Controllo e Chiusura**:
    - Durante il giorno, gli operatori possono verificare le presenze in tempo reale tramite il **Frontend DMS-HUB**.
    - A fine giornata, il sistema consolida i dati di presenza, pronti per la reportistica e la fatturazione.

---

## 5. Conclusione

Questo Master System Design definisce una chiara separazione dei compiti e centralizza la logica di business, creando un'architettura robusta, manutenibile e pronta per future evoluzioni. Ogni sviluppo dovrà aderire strettamente a questo progetto per garantire la coerenza e la qualità del sistema finale.
