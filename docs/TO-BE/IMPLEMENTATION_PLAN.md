# Implementation Plan

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Stato:** Ufficiale  
**Obiettivo:** Fornire un piano di implementazione pratico e dettagliato per migrare dal sistema AS-IS all'architettura TO-BE definita nel `MASTER-SYSTEM-DESIGN.md`.

---

## Fase 1: Pulizia DB e Fix Problemi Attuali

**Obiettivo:** Stabilizzare l'infrastruttura dati, eliminare le dipendenze obsolete e garantire che Drizzle sia allineato a PostgreSQL.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **1.1: Rimuovere `DATABASE_URL` da Frontend** | Basso | Nessuna | [ ] |
| - Rimuovere la variabile da `.env.production` e `.env.example` nel repo `dms-hub-app-new`. | | | |
| - Verificare e rimuovere la variabile dalle configurazioni di Vercel. | | | |
| **1.2: Validare Configurazione Drizzle** | Medio | Task 1.1 | [ ] |
| - Assicurarsi che Drizzle nel Core API sia configurato per usare solo PostgreSQL. | | | |
| - Verificare che non ci siano residui di configurazioni MySQL. | | | |
| **1.3: Applicare Migrazioni Iniziali** | Alto | Task 1.2 | [ ] |
| - Generare le migrazioni Drizzle dallo schema definito in `03-modello-dati-e-mapping.md`. | | | |
| - Applicare le migrazioni al DB PostgreSQL per creare la struttura delle tabelle. | | | |

---

## Fase 2: Allineamento Completo allo Schema Dati TO-BE

**Obiettivo:** Finalizzare il mapping dei dati e migrare i dati dal sistema legacy al nuovo schema PostgreSQL.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **2.1: Ottenere Accesso a DB Heroku** | Critico | **Decisione Utente** | [ ] |
| - Ottenere credenziali di accesso in sola lettura al DB PostgreSQL di Heroku. | | | |
| **2.2: Finalizzare Mapping Dati** | Alto | Task 2.1 | [ ] |
| - Analizzare lo schema reale di Heroku e aggiornare `03-modello-dati-e-mapping.md` con mapping campo-per-campo. | | | |
| **2.3: Sviluppare Script ETL** | Alto | Task 2.2 | [ ] |
| - Creare uno script per estrarre, trasformare e caricare i dati da Heroku al nuovo DB. | | | |
| **2.4: Eseguire Migrazione di Test** | Medio | Task 2.3 | [ ] |
| - Eseguire lo script ETL su un ambiente di staging per validare la correttezza dei dati. | | | |

---

## Fase 3: Implementazione DMS Core API Completa

**Obiettivo:** Sviluppare il "cervello" del sistema, esponendo tutte le funzionalità di business tramite API.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **3.1: Identificare Repo Backend** | Critico | **Decisione Utente** | [ ] |
| - Identificare il repository GitHub del backend tRPC (`orchestratore.mio-hub.me`). | | | |
| **3.2: Implementare Endpoint CRUD** | Alto | Fase 1, Task 3.1 | [ ] |
| - Sviluppare endpoint per `markets`, `stalls`, `vendors`, `concessions`, `attendances`. | | | |
| **3.3: Implementare Logica di Business** | Alto | Task 3.2 | [ ] |
| - Implementare la logica per la creazione di concessioni, gestione presenze e spuntisti. | | | |
| **3.4: Refactoring Frontend** | Alto | Task 3.3 | [ ] |
| - Assicurarsi che il frontend `dms-hub-app-new` utilizzi i nuovi endpoint del Core API per tutte le operazioni. | | | |

---

## Fase 4: Integrazione DMS Legacy su Heroku

**Obiettivo:** Gestire la coesistenza dei due sistemi durante la fase di transizione.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **4.1: Definire Strategia di Sincronizzazione** | Alto | Fase 2, Fase 3 | [ ] |
| - Decidere se la sincronizzazione sarà unidirezionale (Heroku → Nuovo DB) o bidirezionale. | | | |
| **4.2: Sviluppare Job di Sincronizzazione** | Alto | Task 4.1 | [ ] |
| - Creare un job schedulato (es. cron job) che esegua lo script ETL per mantenere i dati allineati. | | | |
| **4.3: Mettere Heroku in Read-Only** | Medio | Go-Live | [ ] |
| - Al momento del go-live definitivo, configurare il sistema Heroku in modalità di sola lettura. | | | |

---

## Fase 5: Integrazione Funzionalità Aggiuntive

**Obiettivo:** Arricchire il sistema con le funzionalità avanzate previste.

| Task | Impatto | Dipendenze | Stato |
|---|---|---|---|
| **5.1: Integrazione GIS** | Alto | Fase 3 | [ ] |
| - Progettare e implementare la visualizzazione delle mappe dei mercati nel frontend. | | | |
| - Sviluppare endpoint nel Core API per fornire dati geospaziali. | | | |
| **5.2: Integrazione CO₂** | Medio | Fase 3 | [ ] |
| - Definire come calcolare e tracciare le emissioni di CO₂. | | | |
| - Aggiungere i campi necessari al modello dati e implementare la logica nel Core API. | | | |
| **5.3: Integrazione Wallet** | Alto | Fase 3 | [ ] |
| - Progettare il sistema di gestione dei crediti/pagamenti. | | | |
| - Implementare le tabelle `wallets`, `transactions` e la logica di gestione nel Core API. | | | |
