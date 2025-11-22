# Implementation Plan - Checklist Operativa

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Obiettivo:** Fornire una checklist operativa per l'implementazione del nuovo sistema DMS-HUB.

---

## FASE 1: Pepe GIS Integration (PRIORITÀ ASSOLUTA)

**Obiettivo:** Rendere operativa la pianta digitale del mercato nella Dashboard.

| Task | Repository | Cartella / File | Output Atteso |
|---|---|---|---|
| **1.1. Definizione Schema DB** | `dms-system-blueprint` | `docs/Schema DB Pepe GIS.md` | Schema DB completo e documentato. |
| **1.2. Migrazioni Drizzle** | `mihub` | `apps/api/drizzle/` | Migrazioni Drizzle generate e pronte per il push. |
| **1.3. Documentazione API** | `dms-system-blueprint` | `docs/API Pepe GIS.md` | API documentate con esempi di utilizzo. |
| **1.4. Implementazione UI** | `dms-hub-app-new` | `src/pages/dashboard/mercati.tsx` | Pagina funzionante con mappa interattiva. |

---

## FASE 2: Allineamento DMS Legacy

**Obiettivo:** Copiare la logica e i dati del gestionale DMS legacy nel nuovo Core API.

| Task | Repository | Cartella / File | Output Atteso |
|---|---|---|---|
| **2.1. Analisi DB Heroku** | - | - | Credenziali di accesso al DB Heroku. |
| **2.2. Mapping Dati** | `dms-system-blueprint` | `docs/03 - Modello Dati e Mapping (TO-BE).md` | Mapping completo tra DB legacy e nuovo. |
| **2.3. Sviluppo API** | `mihub` | `apps/api/server/dmsHubRouter.ts` | Endpoint per replicare le funzionalità legacy. |
| **2.4. Migrazione Dati** | - | - | Script di migrazione dati da Heroku a Neon. |

---

## FASE 3: Nuovo Gestionale in Dashboard

**Obiettivo:** Creare le interfacce per la gestione di mercati, piazzole, ambulanti, concessioni e presenze nella nuova Dashboard.

| Task | Repository | Cartella / File | Output Atteso |
|---|---|---|---|
| **3.1. UI Gestione Mercati** | `dms-hub-app-new` | `src/pages/dashboard/mercati/` | CRUD completo per la gestione dei mercati. |
| **3.2. UI Gestione Piazzole** | `dms-hub-app-new` | `src/pages/dashboard/piazzole/` | CRUD completo per la gestione delle piazzole. |
| **3.3. UI Gestione Ambulanti** | `dms-hub-app-new` | `src/pages/dashboard/ambulanti/` | CRUD completo per la gestione degli ambulanti. |
| **3.4. UI Gestione Concessioni** | `dms-hub-app-new` | `src/pages/dashboard/concessioni/` | CRUD completo per la gestione delle concessioni. |
| **3.5. UI Gestione Presenze** | `dms-hub-app-new` | `src/pages/dashboard/presenze/` | Interfaccia per la gestione delle presenze giornaliere. |

---

## FASE 4: Feature Avanzate

**Obiettivo:** Integrare le funzionalità avanzate di CO₂, wallet e automazioni.

| Task | Repository | Cartella / File | Output Atteso |
|---|---|---|---|
| **4.1. Modulo CO₂** | `mihub` / `dms-hub-app-new` | - | API e UI per il calcolo e la visualizzazione dell'impronta di carbonio. |
| **4.2. Modulo Wallet** | `mihub` / `dms-hub-app-new` | - | API e UI per la gestione del wallet TCC. |
| **4.3. Modulo Automazioni** | `mihub` | - | API per l'esecuzione di task automatici (es. notifiche, report). |
