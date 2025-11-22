# Report Stato Lavori - 21 Novembre 2025

**Autore:** Manus AI

---

## 1. Cosa è già pronto in produzione

- **Backend Core API (Hetzner):** L'API è operativa e serve il frontend.
- **Database (PostgreSQL su Neon):** Il database è attivo e contiene lo schema aggiornato.
- **Frontend (Vercel):** La Dashboard è online e comunica con il backend.

## 2. Cosa è pronto solo a livello di codice

- **Integrazione Pepe GIS:**
  - **Schema DB:** Lo schema per le mappe GIS è definito e le migrazioni Drizzle sono pronte.
  - **API:** Gli endpoint per l'importazione e la lettura delle mappe sono implementati nel backend.
  - **UI:** La pagina per la visualizzazione delle mappe è implementata nel frontend.

## 3. Cosa manca ancora

Per avere la pianta GIS caricabile e le piazzole collegate a mercati, ambulanti e concessioni, mancano i seguenti passi:

1. **Applicare le migrazioni Drizzle al DB di produzione:**
   - Eseguire `pnpm db:push` sul server Hetzner per aggiornare lo schema del database.

2. **Deploy del backend e frontend aggiornati:**
   - Eseguire il deploy delle ultime versioni dei repository `mihub` e `dms-hub-app-new` su Hetzner e Vercel.

3. **Caricare i dati del mercato di Grosseto Esperando:**
   - Usare l'endpoint `POST /trpc/dmsHub.markets.importFromSlotEditor` per caricare il file `editor-v3-sample.json` nel database.

4. **Collegare i dati:**
   - Una volta migrati i dati dal DMS legacy, sarà necessario eseguire degli script per collegare le piazzole (`stalls`) agli ambulanti (`vendors`) e alle concessioni (`concessions`) usando i rispettivi ID.
