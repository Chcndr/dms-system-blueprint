# MASTER PLAN DMS / MIO-HUB

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Stato:** Ufficiale  
**Obiettivo:** Questo documento è la **fonte di verità unica e ufficiale** per l'intero sistema DMS / MIO-HUB. Unisce visione, architettura, modello dati e piano di implementazione in un unico punto di riferimento per tutti gli stakeholder (umani e AI).

---

## 1. Visione Generale

### Visione di Business

Creare un sistema di gestione dei mercati ambulanti **moderno, centralizzato e data-driven**, in grado di:
- Semplificare le operazioni quotidiane di gestione delle presenze e delle concessioni.
- Offrire una **visione geospaziale** dei mercati tramite l'integrazione di mappe GIS.
- Abilitare nuove funzionalità a valore aggiunto come il monitoraggio delle emissioni di CO₂ e un sistema di wallet digitale per gli ambulanti.
- Integrare e infine sostituire il sistema legacy DMS su Heroku, garantendo continuità operativa.

### Visione Tecnica

L'architettura TO-BE mira a creare un sistema **scalabile, manutenibile e sicuro**, basato sui seguenti principi:

- **Single Source of Truth**: Un unico database PostgreSQL e un unico "cervello" (Core API) per tutta la business logic.
- **Separazione delle Responsabilità (SoC)**: Il frontend è responsabile solo della UI, il backend della logica di business.
- **API-First**: Tutte le interazioni con i dati avvengono tramite API ben definite e sicure.
- **Interoperabilità**: Il sistema legacy DMS su Heroku viene trattato come un sistema esterno da integrare durante la fase di transizione.

---

## 2. Architettura AS-IS vs TO-BE

### Architettura AS-IS (Stato Attuale)

- **Frontend (Vercel)**: `dms-hub-app-new`, basato su Next.js, comunica con un backend tRPC.
- **Backend (Hetzner)**: `orchestratore.mio-hub.me`, un servizio tRPC la cui codebase non è stata ancora identificata.
- **Database**: Frammentato tra PostgreSQL (configurazione Drizzle) e residui di configurazione MySQL/PlanetScale.
- **Sistema Legacy (Heroku)**: Gestionale DMS funzionante ma isolato, con un proprio database PostgreSQL.

### Architettura TO-BE (Progetto Finale)

| Componente | Decisione | Motivazione |
|---|---|---|
| **Database** | **PostgreSQL Unico** | Standardizzazione, performance e scalabilità. |
| **Backend** | **DMS Core API su Hetzner** | Centralizzazione di tutta la business logic. |
| **Frontend** | **Next.js su Vercel** | Responsabile solo della UI, comunica esclusivamente tramite il DMS Core API. |
| **Sistema Legacy** | **DMS su Heroku come sistema esterno** | Fonte dati da integrare e poi dismettere. |
| **Mappe GIS** | **Layer di visualizzazione sui dati** | Integrazione di piante mercato dall'editor v3, visualizzate come layer nel frontend. |

---

## 3. Modello Dati Principale (PostgreSQL)

Lo schema target è progettato per essere normalizzato e scalabile. Le tabelle principali sono:

- **`markets`**: Anagrafica dei mercati.
- **`stalls`**: Anagrafica delle piazzole, con riferimento al mercato (`market_id`) e all'identificativo GIS (`gis_feature_id`).
- **`vendors`**: Anagrafica degli ambulanti.
- **`concessions`**: Gestione delle concessioni, legando `stalls` e `vendors`.
- **`attendances`**: Rilevamento delle presenze giornaliere.

*Per il dettaglio completo dello schema e del mapping dal sistema legacy, fare riferimento al documento `docs/TO-BE/03-modello-dati-e-mapping.md`.* 

---

## 4. Integrazione GIS (Mappa GIS + Editor v3) - PRIORITARIA

Questa è la prima "vertical slice" da implementare. L'obiettivo è visualizzare la pianta di un mercato all'interno della Dashboard, con interattività di base.

**Flusso di lavoro:**
1.  **Importazione**: Un file della pianta del mercato (es. GeoJSON) generato dall'editor v3 viene importato dal backend.
2.  **Elaborazione**: L'importer crea i record per il mercato e le relative piazzole nel database PostgreSQL, salvando le geometrie e gli ID.
3.  **Esposizione via API**: Il Core API espone endpoint per recuperare i dati della mappa e delle piazzole.
4.  **Visualizzazione in Dashboard**: La Dashboard consuma le API per visualizzare la mappa e permettere all'utente di cliccare sulle piazzole per vederne i dettagli.

---

## 5. Copia/Integrazione Gestionale DMS (Heroku)

Subito dopo la Fase 0, l'obiettivo è replicare le funzionalità del gestionale legacy nel nuovo sistema, collegandole alla vista GIS.

**Flusso di lavoro:**
1.  **Analisi Completa**: Analizzare lo schema del DB di Heroku e i flussi funzionali (anagrafiche, presenze, spunta).
2.  **Replicare Funzionalità**: Implementare la logica di business nel Core API.
3.  **Collegare alla Dashboard**: Integrare queste funzionalità nella sezione "Gestionale" della Dashboard, in modo che siano accessibili dal pannello laterale che si apre al click su una piazzola.

---

## 6. Roadmap Completa per Fasi

*Il piano dettagliato con task, impatto e dipendenze è nel file `docs/TO-BE/IMPLEMENTATION_PLAN.md`. Di seguito un riepilogo delle fasi.* 

- **Fase 0: "Vertical Slice" GIS FIRST** (PRIORITARIA)
- **Fase 1: Pulizia DB e Fix Problemi Attuali**
- **Fase 2: Allineamento Completo allo Schema Dati TO-BE**
- **Fase 3: Implementazione DMS Core API Completa**
- **Fase 4: Integrazione DMS Legacy su Heroku**
- **Fase 5: Integrazione Funzionalità Aggiuntive (CO₂, Wallet, ecc.)**

---

## 7. Rischi e Decisioni Aperte

| Rischio / Blocco | Azione Richiesta (Utente) | Impatto |
|---|---|---|
| **Accesso al DB di Heroku** | Fornire credenziali di accesso in sola lettura al DB PostgreSQL di Heroku. | **Alto**. Blocca la Fase 2 (migrazione dati) e l'analisi completa per la Fase 5. |
| **Identificazione Repository Backend** | Indicare quale repository GitHub contiene il codice del backend tRPC (`orchestratore.mio-hub.me`). | **Alto**. Blocca la Fase 3 (sviluppo Core API) e quindi tutte le fasi successive. |
| **Formato Output Editor v3** | Fornire un file di esempio (GeoJSON/SHP) generato dall'editor v3. | **Medio**. Necessario per avviare la Fase 0 (sviluppo importer). |
