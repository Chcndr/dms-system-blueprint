# 06 - Roadmap Tecnica (TO-BE)

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Stato:** Ufficiale  
**Obiettivo:** Fornire una checklist tecnica con step concreti e spuntabili per migrare dal sistema AS-IS all'architettura TO-BE definita nel `MASTER-SYSTEM-DESIGN.md`.

---

## Fase 1: Pulizia e Allineamento Database

**Obiettivo:** Standardizzare tutto su PostgreSQL, eliminare le dipendenze da PlanetScale/MySQL e allineare Drizzle al nuovo schema.

- [ ] **1.1: Setup Database PostgreSQL Target**
    - [ ] Verificare la configurazione dell'istanza PostgreSQL su Hetzner/Neon.
    - [ ] Creare un database dedicato per il nuovo sistema DMS-HUB.
    - [ ] Configurare accessi sicuri per il DMS Core API.

- [ ] **1.2: Eliminazione Dipendenze da PlanetScale/MySQL**
    - [ ] Eseguire una ricerca (`grep`) nel codebase del frontend (Vercel) e del backend per trovare tutte le stringhe di connessione e le configurazioni relative a PlanetScale o MySQL.
    - [ ] Rimuovere tutte le configurazioni obsolete.
    - [ ] Identificare e refattorizzare eventuali parti di codice (specialmente nel frontend Next.js) che potrebbero ancora tentare di connettersi direttamente a un database MySQL.

- [ ] **1.3: Allineamento Drizzle ORM**
    - [ ] Configurare Drizzle nel DMS Core API per puntare esclusivamente al nuovo database PostgreSQL.
    - [ ] Generare le migrazioni Drizzle a partire dallo schema definito nel file `03-modello-dati-e-mapping.md`.
    - [ ] Applicare le migrazioni al database PostgreSQL target per creare la struttura delle tabelle.

---

## Fase 2: Analisi e Mapping Completo del Gestionale DMS (Heroku)

**Obiettivo:** Ottenere una comprensione completa dello schema e dei dati del sistema legacy per finalizzare il piano di migrazione.

- [ ] **2.1: Accesso e Analisi Database Heroku**
    - [ ] Ottenere le credenziali di accesso in sola lettura per il database PostgreSQL del gestionale DMS su Heroku.
    - [ ] Connettersi al database e documentare lo schema reale (nomi tabelle, nomi colonne, tipi di dati, relazioni).
    - [ ] Eseguire query esplorative per comprendere la struttura dei dati e le relazioni non evidenti dall'analisi funzionale.

- [ ] **2.2: Recupero File di Riferimento**
    - [ ] **(Cruciale)** Recuperare il file `Anagrafiche_API_DMS.xlsx`.
    - [ ] Analizzare il file per identificare ID stabili, anagrafiche e regole di business.

- [ ] **2.3: Finalizzazione Mapping Dati**
    - [ ] Aggiornare il documento `docs/TO-BE/03-modello-dati-e-mapping.md` con una mappatura campo-per-campo dettagliata, includendo trasformazioni di tipo e regole di normalizzazione.

- [ ] **2.4: Sviluppo Script di Migrazione (ETL)**
    - [ ] Creare uno script (es. Node.js o Python) che:
        - [ ] Si connetta sia al database di Heroku (sorgente) che al nuovo PostgreSQL (target).
        - [ ] Estraia i dati dalle tabelle sorgente.
        - [ ] Li trasformi secondo le regole di mapping definite.
        - [ ] Li carichi nelle tabelle target del nuovo database.
    - [ ] Eseguire una migrazione di test su un ambiente di staging.

---

## Fase 3: Allineamento Codice al Progetto Finale

**Obiettivo:** Rendere il DMS Core API il cervello del sistema e disaccoppiare completamente il frontend dall'accesso diretto ai dati.

- [ ] **3.1: Implementazione DMS Core API (Hetzner)**
    - [ ] Sviluppare o completare gli endpoint tRPC per le funzionalità CRUD (Create, Read, Update, Delete) di base:
        - [ ] `markets` (mercati)
        - [ ] `stalls` (piazzole)
        - [ ] `vendors` (ambulanti)
        - [ ] `concessions` (concessioni)
        - [ ] `attendances` (presenze)
    - [ ] Implementare la logica di business principale (es. creazione concessione, gestione spunta) all'interno dei servizi del Core API.
    - [ ] Implementare un meccanismo di autenticazione e autorizzazione per proteggere gli endpoint.

- [ ] **3.2: Refactoring Frontend (Vercel)**
    - [ ] Rimuovere qualsiasi codice nel frontend Next.js che acceda direttamente a un database.
    - [ ] Refattorizzare tutti i componenti e le pagine per utilizzare il client tRPC per comunicare con il DMS Core API.
    - [ ] Assicurarsi che il frontend sia responsabile solo della presentazione dei dati e della gestione dello stato della UI.

- [ ] **3.3: Preparazione Integrazioni Future**
    - [ ] Definire e documentare gli endpoint nel Core API che verranno utilizzati dall'**App Mobile DMS**.
    - [ ] Progettare gli endpoint nel Core API che forniranno i dati geospaziali delle piazzole al futuro servizio **GIS**.

---

## Fase 4: Validazione e Go-Live

**Obiettivo:** Validare il nuovo sistema e pianificare il passaggio definitivo.

- [ ] **4.1: Test End-to-End**
    - [ ] Eseguire test completi del flusso "giornata tipo di mercato" utilizzando il nuovo frontend, Core API e database.
    - [ ] Validare che i dati migrati siano corretti e consistenti.

- [ ] **4.2: Go-Live**
    - [ ] Pianificare una finestra di manutenzione per la migrazione finale dei dati.
    - [ ] Eseguire lo script ETL definitivo.
    - [ ] Aggiornare i record DNS per puntare al nuovo sistema.
    - [ ] Mettere il sistema legacy DMS su Heroku in modalità di sola lettura come archivio storico.
