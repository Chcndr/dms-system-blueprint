# Implementation Plan

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Stato:** Ufficiale  
**Obiettivo:** Fornire un piano di implementazione pratico e dettagliato per migrare dal sistema AS-IS all'architettura TO-BE, con priorità all'integrazione GIS.

---

## Fase 0: "Vertical Slice" GIS FIRST

**Obiettivo:** Realizzare una prima fetta verticale funzionante che integri la pianta del mercato (Mappa GIS) nella Dashboard, collegandola a un modello dati minimo.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **0.1: Definire Modello Dati Minimo** | Alto | Nessuna | [ ] |
| - Creare tabelle `markets`, `stalls`, `vendors` in PostgreSQL. | | | |
| - La tabella `stalls` deve includere `market_id`, `stall_number` e `gis_feature_id`. | | | |
| **0.2: Sviluppare Importer per Pianta Mercato** | Alto | Task 0.1 | [ ] |
| - Analizzare il formato di output dell'editor v3 (GeoJSON/SHP). | | | |
| - Creare un importer nel backend (Hetzner) per leggere il file e popolare le tabelle `markets` e `stalls`. | | | |
| **0.3: Sviluppare API Core per GIS** | Alto | Task 0.2 | [ ] |
| - Esporre endpoint: `GET /markets`, `GET /markets/:id/map`, `GET /markets/:id/stalls`, `GET /stalls/:id`. | | | |
| **0.4: Sviluppare Vista Mappa in Dashboard** | Alto | Task 0.3 | [ ] |
| - Creare la pagina "Mappa GIS" nella Dashboard DMS-HUB. | | | |
| - Implementare selettore mercato e visualizzazione mappa con layer piazzole. | | | |
| - Al click su una piazzola, mostrare un pannello con dati mercato, posteggio e ambulante (anche fittizi). | | | |
| **0.5: Creare Stub Sezione "Gestionale"** | Medio | Task 0.4 | [ ] |
| - Nel pannello laterale, creare uno stub per la sezione "Gestionale" collegata a mercato, posteggio e ambulante. | | | |

---

## Fase 1: Pulizia DB e Fix Problemi Attuali

**Obiettivo:** Stabilizzare l'infrastruttura dati, eliminare le dipendenze obsolete e garantire che Drizzle sia allineato a PostgreSQL.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **1.1: Rimuovere `DATABASE_URL` da Frontend** | Basso | Nessuna | [ ] |
| **1.2: Validare Configurazione Drizzle** | Medio | Task 1.1 | [ ] |
| **1.3: Applicare Migrazioni Iniziali** | Alto | Task 1.2 | [ ] |

---

## Fase 2: Allineamento Completo allo Schema Dati TO-BE

**Obiettivo:** Finalizzare il mapping dei dati e migrare i dati dal sistema legacy al nuovo schema PostgreSQL.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **2.1: Ottenere Accesso a DB Heroku** | Critico | **Decisione Utente** | [ ] |
| **2.2: Finalizzare Mapping Dati** | Alto | Task 2.1 | [ ] |
| **2.3: Sviluppare Script ETL** | Alto | Task 2.2 | [ ] |
| **2.4: Eseguire Migrazione di Test** | Medio | Task 2.3 | [ ] |

---

## Fase 3: Implementazione DMS Core API Completa

**Obiettivo:** Sviluppare il "cervello" del sistema, esponendo tutte le funzionalità di business tramite API.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **3.1: Identificare Repo Backend** | Critico | **Decisione Utente** | [ ] |
| **3.2: Implementare Endpoint CRUD** | Alto | Fase 1, Task 3.1 | [ ] |
| **3.3: Implementare Logica di Business** | Alto | Task 3.2 | [ ] |
| **3.4: Refactoring Frontend** | Alto | Task 3.3 | [ ] |

---

## Fase 4: Integrazione DMS Legacy su Heroku

**Obiettivo:** Gestire la coesistenza dei due sistemi durante la fase di transizione.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **4.1: Definire Strategia di Sincronizzazione** | Alto | Fase 2, Fase 3 | [ ] |
| **4.2: Sviluppare Job di Sincronizzazione** | Alto | Task 4.1 | [ ] |
| **4.3: Mettere Heroku in Read-Only** | Medio | Go-Live | [ ] |

---

## Fase 5: Integrazione Funzionalità Aggiuntive

**Obiettivo:** Arricchire il sistema con le funzionalità avanzate previste.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **5.1: Integrazione GIS (Completa)** | Alto | Fase 0, Fase 3 | [ ] |
| **5.2: Integrazione CO₂** | Medio | Fase 3 | [ ] |
| **5.3: Integrazione Wallet** | Alto | Fase 3 | [ ] |
