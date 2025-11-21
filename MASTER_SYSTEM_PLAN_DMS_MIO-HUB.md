# MASTER SYSTEM PLAN: DMS / MIO-HUB

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Ultimo Aggiornamento:** 21 Novembre 2025 - 19:30 GMT+1  
**Stato:** Ufficiale  
**Obiettivo:** Questo documento è la **fonte di verità unica e ufficiale** per l'intero sistema DMS / MIO-HUB. Unifica visione, architettura, modello dati e piano di implementazione in un unico punto di riferimento per tutti gli stakeholder (umani e AI).

---

## 1. Visione & Obiettivi

### Visione di Business

Creare un sistema di gestione dei mercati ambulanti **moderno, centralizzato e data-driven**, in grado di:
- Semplificare le operazioni quotidiane di gestione delle presenze e delle concessioni.
- Offrire una **visione geospaziale** dei mercati tramite l'integrazione di mappe GIS (Pepe GIS).
- Abilitare nuove funzionalità a valore aggiunto come il monitoraggio delle emissioni di CO₂ e un sistema di wallet digitale per gli ambulanti.
- Integrare e infine sostituire il sistema legacy DMS su Heroku, garantendo continuità operativa.

### Visione Tecnica

L'architettura TO-BE mira a creare un sistema **scalabile, manutenibile e sicuro**, basato sui seguenti principi:

- **Single Source of Truth**: Un unico database PostgreSQL e un unico "cervello" (Core API) per tutta la business logic.
- **Separazione delle Responsabilità (SoC)**: Il frontend è responsabile solo della UI, il backend della logica di business.
- **API-First**: Tutte le interazioni con i dati avvengono tramite API ben definite e sicure.
- **Interoperabilità**: Il sistema legacy DMS su Heroku viene trattato come un sistema esterno da integrare durante la fase di transizione.

### Ruolo dei Componenti

| Componente | Ruolo | Hosting |
|---|---|---|
| **DMS Storico** | Sistema legacy da integrare e poi dismettere. Fonte dati iniziale. | Heroku |
| **DMS Core API** | Unico "cervello" del sistema. Gestisce tutta la business logic e l'accesso ai dati. | Hetzner |
| **Dashboard / MIO-HUB** | Interfaccia utente web per la gestione del sistema. | Vercel |
| **App Ambulanti** | Strumento per gli ambulanti per check-in e altre operazioni. | - |
| **Pepe GIS / Editor Mappe** | Motore per la creazione e visualizzazione delle mappe dei mercati. | - |

---

## 2. Architettura TO-BE (Sintesi)

Lo schema logico del sistema TO-BE è il seguente:

- **Frontend (Dashboard / MIO-HUB)**: Applicazione Next.js su Vercel che funge da interfaccia utente. **Non ha accesso diretto al database** e comunica esclusivamente con il Core API.
- **Core API su Hetzner**: Servizio Node.js/tRPC che espone tutta la business logic e funge da unico punto di accesso al database. È il **master dei dati e delle logiche**.
  - **AS-IS (Temporaneo)**: Backend REST (`/root/mihub-backend-rest/`) attivo su Hetzner porta 3001, esposto tramite Nginx su `orchestratore.mio-hub.me`. Usato come adapter temporaneo per implementare endpoint GIS.
  - **TO-BE (Target)**: Monorepo `mihub` con TypeScript + tRPC + Drizzle, da ripristinare dopo fix errori compilazione.
- **DB Postgres**: Unico database standard per tutto il sistema, ospitato su Hetzner/Neon.
- **DMS Legacy (Heroku)**: Sistema esterno trattato come fonte dati da cui migrare e con cui sincronizzarsi durante la transizione.
- **Pepe GIS**: Motore per la generazione delle piante dei mercati, i cui output (es. GeoJSON) vengono importati e gestiti dal Core API.

---

## 3. Modello Dati Chiave

Il modello dati su PostgreSQL è il cuore del sistema. Le tabelle principali includono:

- **`markets`**: Anagrafica dei mercati.
- **`stalls`**: Anagrafica delle piazzole, con riferimento al mercato (`market_id`) e all'identificativo GIS (`gis_feature_id`).
- **`vendors`**: Anagrafica degli ambulanti.
- **`concessions`**: Gestione delle concessioni, legando `stalls` e `vendors`.
- **`attendances`**: Rilevamento delle presenze giornaliere e gestione della spunta.
- **`co2_emissions`**, **`wallets`**, **`automations`**: Tabelle per le funzionalità avanzate.

---

## 4. Integrazioni Principali

- **DMS Legacy (Heroku)**: Integrazione in sola lettura per la migrazione dei dati e la sincronizzazione durante la fase di transizione.
- **App DMS Ambulanti**: Si connetterà direttamente al Core API per leggere dati (es. concessioni) e scrivere dati (es. presenze).
- **Pepe GIS / Editor v3**: ✅ **IMPLEMENTATO** - L'output dell'editor (piante dei mercati in formato GeoJSON) viene servito tramite endpoint REST `/api/gis/market-map`. Visualizzazione mappa interattiva disponibile su `/market-gis`.

---

## 5. Roadmap di Implementazione

### Fase 0 – Base Tecnica Pulita (Obbligatoria)

**Obiettivo:** Stabilizzare l'infrastruttura, standardizzare PostgreSQL e garantire che il frontend comunichi solo tramite API.

- [ ] Standardizzare tutto su PostgreSQL.
- [ ] Sistemare Core API su Hetzner con Drizzle configurato SOLO per Postgres.
- [ ] Eliminare accessi diretti a PlanetScale / MySQL dal frontend (Vercel).
- [ ] Verificare che la Dashboard parli solo con il Core API.

### Fase 1 – Integrazione Pepe GIS come Primo Caso d'Uso Forte

**Obiettivo:** Avere nella Dashboard una mappa GIS funzionante che carica la pianta del mercato e mostra i dati principali.

**Stato:** ✅ **IMPLEMENTATO IN PRODUZIONE** (Backend REST temporaneo)

- [x] **Endpoint Backend**: `GET /api/gis/market-map` - ✅ **PRODUZIONE** - https://orchestratore.mio-hub.me/api/gis/market-map
- [x] **Pagina Mappa**: `/market-gis` - Visualizzazione interattiva con Leaflet (Branch: `feature/gis-market-map-v1`)
- [x] **Componenti**: Contorno mercato (Polygon) + Piazzole (Circle) + Popup dettagli
- [ ] Definire ID standard condivisi tra Postgres, Pepe GIS e DMS Legacy
- [ ] Sviluppare endpoint CRUD completo per mercati, piazzole, ambulanti
- [ ] Sviluppare importer per scrittura dati Editor v3 nel database
- [ ] Implementare pannelli laterali interattivi nella mappa
- [ ] Creare sezione "Gestionale" collegata ai dati della mappa

**Dettagli Implementazione:**
- **Backend**: Router Express `gisRouter.ts` con validazione formato Editor v3
- **Frontend**: Componente React `MarketGISPage.tsx` con react-leaflet
- **Dati**: File JSON di esempio con 3 piazzole (pronto per 160 piazzole Grosseto)
- **Test**: ✅ Build OK, endpoint funzionante, visualizzazione corretta

### Fase 2 – Copia e Allineamento del Gestionale DMS (Heroku)

**Obiettivo:** Replicare la logica di business del gestionale legacy nel nuovo sistema.

- [ ] Analizzare a fondo lo schema DB e le logiche di business di Heroku.
- [ ] Completare il mapping campo-per-campo nel modello dati TO-BE.
- [ ] Replicare nel Core API la logica gestionale critica (presenze, spunta, assenze).

### Fase 3 – Nuovo Gestionale dentro la Dashboard / MIO-HUB

**Obiettivo:** Sostituire le funzionalità del gestionale storico con nuove schermate nella Dashboard.

- [ ] Sviluppare le interfacce per la gestione di mercati, piazzole, ambulanti, concessioni e presenze.
- [ ] Basare tutte le funzionalità sul Core API + Postgres + Pepe GIS.

### Fase 4 – Feature Avanzate e Automazioni

**Obiettivo:** Implementare le funzionalità a valore aggiunto.

- [ ] Sviluppare i moduli per CO₂, TCC, wallet e automazioni, appoggiandosi all'architettura unificata.
