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
- **[Database Schema](04_database/schema.md)**: Descrizione dettagliata di tutte le 39 tabelle del database Drizzle/Postgres.
- **[API Documentation](02_backend/)**: Raccolta di tutte le documentazioni delle API tRPC.
- **[Architettura Centro Mobilità](06_mobilita/architettura-centro-mobilita.md)**: Design scalabile per l'integrazione dei trasporti pubblici.
- **[Standard Mappe GIS](05_gis/standard-mappe-gis.md)**: Componenti ufficiali e deprecati per le mappe GIS.

---

## Come Contribuire

Per proporre modifiche, creare una Pull Request. Ogni modifica deve essere approvata prima del merge su `main`.

---

*Questo repository è la fonte unica di verità per la documentazione del sistema DMS Hub.*
