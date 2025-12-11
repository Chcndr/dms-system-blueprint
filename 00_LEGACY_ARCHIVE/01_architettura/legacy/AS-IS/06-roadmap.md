_# 06. Roadmap

Questo documento definisce i passi strategici per la modernizzazione e l'evoluzione del sistema DMS-HUB.

## Fasi Principali

La roadmap è suddivisa in fasi, da affrontare in ordine sequenziale per garantire una transizione controllata e sicura.

### Fase 1: Stabilizzazione e Pulizia Tecnica

L'obiettivo di questa fase è risolvere i problemi noti che affliggono l'ambiente di produzione e creare una base stabile per gli sviluppi futuri.

- **Azione 1.1: Risoluzione Problema Cache Vercel**
  - **Descrizione**: Creare un nuovo progetto Vercel per il frontend (`dms-hub-app-new`) per eliminare i problemi di caching che impediscono il deploy di nuove versioni.
  - **Stato**: Completato.

- **Azione 1.2: Allineamento Drizzle ORM a PostgreSQL**
  - **Descrizione**: Rimuovere dal codice ogni riferimento a tipi e funzioni specifiche di MySQL e correggere gli errori di tipo risultanti dalla migrazione a PostgreSQL.
  - **Stato**: Completato (commit `d717837`).

- **Azione 1.3: Unificazione Configurazioni**
  - **Descrizione**: Allineare le configurazioni di build (`tsconfig.json`, `vite.config.ts`, ecc.) tra l'ambiente di sviluppo locale e l'ambiente di produzione su Vercel per eliminare gli errori di build.
  - **Stato**: Da avviare.

### Fase 2: Analisi e Mapping del Sistema Legacy

Questa fase è dedicata alla comprensione approfondita del gestionale DMS su Heroku per pianificarne la migrazione.

- **Azione 2.1: Documentazione Funzionale DMS**
  - **Descrizione**: Accedere al sistema `lapsy-dms` in modalità **READ-ONLY** e documentare in dettaglio ogni schermata, campo e workflow. L'output sarà un documento che descrive il comportamento funzionale del sistema.
  - **Stato**: Da avviare.

- **Azione 2.2: Mapping Dati DMS -> DMS-HUB**
  - **Descrizione**: Analizzare il file `Anagrafiche_API_DMS.xlsx` e il database MySQL legacy per creare un mapping completo tra le tabelle e i campi del vecchio sistema e il nuovo schema PostgreSQL.
  - **Stato**: Da avviare.

### Fase 3: Sviluppo Funzionalità Mancanti

Una volta completata l'analisi, si procederà con lo sviluppo delle funzionalità necessarie per raggiungere la parità con il sistema legacy.

- **Azione 3.1: Implementazione Gestione Concessioni e Presenze**
  - **Descrizione**: Sviluppare nel DMS-HUB i moduli per la gestione delle concessioni e la registrazione delle presenze, basandosi sull'analisi del sistema legacy.
  - **Stato**: Da avviare.

- **Azione 3.2: Integrazione Editor GIS**
  - **Descrizione**: Integrare un componente editor GIS nel frontend per la gestione delle piante dei mercati.
  - **Stato**: Da avviare.

### Fase 4: Migrazione e Dismissione

L'ultima fase prevede la migrazione dei dati e la progressiva dismissione del sistema legacy.

- **Azione 4.1: Sviluppo Script di Migrazione**
  - **Descrizione**: Creare script per migrare i dati storici dal database MySQL del DMS legacy al nuovo database PostgreSQL.
  - **Stato**: Da avviare.

- **Azione 4.2: Dismissione Sistema Heroku**
  - **Descrizione**: Una volta che tutte le funzionalità e i dati saranno migrati e validati, il sistema su Heroku potrà essere dismesso.
  - **Stato**: Da avviare.
