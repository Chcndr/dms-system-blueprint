# DMS System Blueprint

Questo repository contiene la documentazione ufficiale, completa e sempre aggiornata dell'ecosistema **DMS (Digital Market System) e MIO-HUB**.

---

## Struttura della Documentazione

La documentazione è organizzata in directory tematiche per facilitare la consultazione:

| Directory | Contenuto |
| :--- | :--- |
| **`01_architettura`** | Panoramica architetturale, piani di sistema, analisi AS-IS/TO-BE. |
| **`02_backend`** | Documentazione API tRPC, modelli dati, logica di business. |
| **`03_frontend`** | Componenti UI/UX, flussi utente, integrazione frontend-backend. |
| **`04_database`** | Schema completo del database (39 tabelle), migrazioni, relazioni. |
| **`05_gis`** | Integrazione con Pepe GIS, gestione mappe, layer e dati geografici. |
| **`06_mobilita`** | Architettura del Centro Mobilità, integrazione TPER, provider scalabili. |
| **`07_guide_operative`** | Guide pratiche per setup, deployment, gestione API keys e webhook. |

### Documenti Principali

- **[MASTER_SYSTEM_PLAN.md](01_architettura/MASTER_SYSTEM_PLAN.md)**: Il documento principale che descrive la visione, gli obiettivi e la roadmap completa del sistema.
- **Database Schema**: Schema completo del database con 39 tabelle (Drizzle/Postgres) - Documentato nel codice `drizzle/schema.ts`
- **[API Documentation](02_backend/)**: Raccolta di tutte le documentazioni delle API tRPC.
- **[Architettura Centro Mobilità](06_mobilita/architettura-centro-mobilita.md)**: Design scalabile per l'integrazione dei trasporti pubblici.
- **[Standard Mappe GIS](05_gis/standard-mappe-gis.md)**: Componenti ufficiali e deprecati per le mappe GIS.
- **[Gestione Mercati e Mappa GIS](05_gis/gestione-mercati-e-mappa.md)**: Sincronizzazione tabella, mappa, popup e vetrina.

---

## 🚀 Playbook di Deploy e Guide Operative

Per tutte le procedure di deploy, troubleshooting e gestione del sistema, fare riferimento alle seguenti guide:

- **[Playbook Ufficiale di Deploy](./07_guide_operative/PLAYBOOK_DEPLOY.md)**: La guida completa e ufficiale per il deploy di frontend, backend e database.
- **[Procedura Deploy Backend (Hetzner)](07_guide_operative/DEPLOY_BACKEND_HETZNER.md)**
- **[Procedura Deploy Frontend (Vercel)](07_guide_operative/DEPLOY_FRONTEND_VERCEL.md)**
- **[Troubleshooting: "Endpoint not available"](07_guide_operative/TROUBLESHOOTING_ENDPOINT_NOT_AVAILABLE.md)**

---

## Come Contribuire

Per proporre modifiche, creare una Pull Request. Ogni modifica deve essere approvata prima del merge su `main`.

---

*Questo repository è la fonte unica di verità per la documentazione del sistema DMS Hub.*

- **[MIO: Permessi e Flusso di Lavoro](./07_guide_operative/MIO_PERMESSI_E_FLUSSO_DI_LAVORO.md)**: Come l'agente AI MIO interagisce con il sistema.
