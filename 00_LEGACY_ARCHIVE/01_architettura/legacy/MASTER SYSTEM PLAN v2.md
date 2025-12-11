# MASTER SYSTEM PLAN v2 - DMS / MIO-HUB

**Stato:** APPROVED  
**Owner:** Chcndr  
**Executor principale:** Manus / ABACUS  
**Ultimo aggiornamento:** 2025-11-21

---

## 0. Obiettivo del piano

Mettere ordine DEFINITIVO in tutto l’ecosistema:

- DMS legacy su Heroku (gestionle storico)
- Nuovo backend su Hetzner (Node API)
- Database principale su Postgres (Neon)
- Dashboard DMS-HUB / MIO-HUB su Vercel
- Integrazione GIS (Pepe GIS / Editor v3)
- Futuro nuovo gestionale unificato

Con vincoli chiari:

- **Un solo DB “vero” per il nuovo mondo:** Postgres (Neon).
- **Il frontend non tocca mai il DB direttamente:** parla solo con la API.
- **DMS legacy viene copiato / integrato, non tenuto in parallelo all’infinito.**
- **Pepe GIS è PRIORITÀ #1 di implementazione.**

---

## 1. Mappa ad alto livello (TO-BE)

Componenti principali:

1. **DMS Core API (nuovo) – Hetzner**
   - Backend Node.js/TypeScript (apps/api).
   - ORM: Drizzle (configurato SOLO Postgres).
   - Connessione a Neon Postgres.
   - Espone le API per:
     - Mercati
     - Piazzole
     - Ambulanti
     - Concessioni
     - Presenze / Spunta
     - Modulo GIS (mappe mercati / piazzole)
     - CO₂ / Wallet / automazioni (moduli nuovi)

2. **Database principale – Postgres (Neon)**
   - Nome progetto: `dms-hub-production`.
   - È la **single source of truth** per il nuovo sistema.
   - Lo schema ufficiale è documentato in:
     - `docs/03 - Modello Dati e Mapping (TO-BE).md`

3. **Frontend – DMS-HUB / MIO-HUB – Vercel**
   - Next.js / React (apps/frontend).
   - Parla solo con `DMS Core API` via HTTP (REST/tRPC).
   - Mostra:
     - Dashboard gestionale
     - Sezioni mercati / piazzole / ambulanti
     - Nuova sezione “GIS / Pianta Mercato”
     - Moduli CO₂ / Wallet / automazioni

4. **DMS legacy – Heroku**
   - Gestionale storico.
   - Contiene:
     - anagrafiche ufficiali oggi (mercati, piazzole, ambulanti, concessioni)
     - logiche di presenze/spunta operative attuali
   - Nel nuovo piano:
     - è una **fonte dati di riferimento** da copiare / mappare
     - viene integrato tramite:
       - lettura schema DB
       - eventuali API se disponibili

5. **Pepe GIS / Editor V3**
   - Strumento che produce la **mappa digitale del mercato**:
     - piazzole come poligoni / punti
     - layer e attributi (codici piazzola, settori, ecc.)
   - Nel nuovo sistema:
     - viene agganciato a DMS Core API
     - i dati GIS finiscono in Postgres (Neon) come geometrie (GeoJSON / JSONB)
     - la dashboard mostra la pianta su mappa con i dati “vivi” di ambulanti/presenze.

---

## 2. Principi architetturali

1. **Un solo DB per il nuovo mondo:**  
   - Tutti i moduli nuovi usano **solo** Postgres (Neon).
   - PlanetScale/MySQL → da spegnere / eliminare dal codice.

2. **API-first:**  
   - Frontend e app parlano SOLO con la Core API.
   - Niente query dirette dal frontend al DB.

3. **Legacy isolato:**  
   - DMS Heroku è trattato come “sistema esterno”.
   - Lo usiamo per:
     - capire logiche
     - importare / sincronizzare dati
   - NON aggiungiamo logica nuova lì dentro.

4. **GIS di prima classe:**  
   - Le piazzole NON sono solo righe in tabella:
     - hanno geometria.
   - Ogni piazzola è un oggetto che vive sia:
     - nel DB (anagrafica)
     - nel GIS (poligono sulla mappa).
   - La chiave di collegamento è chiara: ID mercato, ID piazzola, codici.

---

## 3. FASE 1 – PEPE GIS INTEGRATION

**Stato:** IN CORSO

### 3.1 Obiettivo

Rendere operativa la **pianta digitale del mercato** direttamente in DMS-HUB / MIO-HUB:

- Carico la pianta dal Pepe GIS Editor v3.
- La vedo sulla dashboard come mappa interattiva.
- Ogni piazzola sulla mappa è collegata:
  - all’anagrafica piazzola
  - all’ambulante / concessione
  - alle presenze del giorno.

### 3.2 Modellazione dati piazzole (stalls)

**Stato:** COMPLETATO

La tabella `stalls` è stata estesa per includere i campi GIS:

- `orientation`: `REAL`
- `kind`: `VARCHAR(50)`
- `widthM`: `REAL`
- `depthM`: `REAL`
- `rawGeojson`: `JSONB`
- `legacyId`: `VARCHAR(255)`

### 3.3 Endpoint import v3

**Stato:** COMPLETATO

Il backend (`mihub/apps/api`) ora espone i seguenti endpoint nel `dmsHubRouter`:

- `importFromSlotEditor`: `POST /trpc/dmsHub.markets.importFromSlotEditor`
- `importAuto`: `POST /trpc/dmsHub.markets.importAuto`

### 3.4 Endpoint lettura mappa

**Stato:** COMPLETATO

L'endpoint `markets.getById` (`GET /trpc/dmsHub.markets.getById`) ora restituisce anche le piazzole (`stalls`) associate al mercato.

### 3.5 UI mappa mercato nel frontend

**Stato:** COMPLETATO

È stata creata la pagina `src/pages/dashboard/mercati.tsx` nel frontend (`dms-hub-app-new`) che:

- Mostra la lista dei mercati.
- Al click, visualizza una mappa interattiva (MapLibre GL) con le piazzole.
- Mostra un popup con i dettagli della piazzola al click.

---

## 4. Nuovo gestionale – logica presa dal DMS legacy

Dopo il GIS, il secondo pezzo grosso è **replicare la logica del gestionale DMS** nel nuovo Core.

### 4.1 Dominio minimo da copiare

- Mercati
- Piazzole
- Ambulanti
- Concessioni
- Presenze / Spunta
- Flussi di assegnazione spuntisti / gestione assenze

Riferimenti:
- `Analisi Funzionale Completa - Sistema Legacy DMS (Heroku).md`
- `03 - Modello Dati e Mapping (TO-BE).md`
- `DMS System Blueprint.md`
- `MASTER SYSTEM PLAN: DMS _ MIO-HUB.md` (versioni precedenti)

### 4.2 Strategia

1. Manus entra nel gestionale DMS su Heroku:
   - legge schema DB
   - legge codice (se accessibile)
   - documenta tabelle chiave e relazioni
