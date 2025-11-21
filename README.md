# DMS System Blueprint

Questo repository contiene la mappa ufficiale dell'architettura e della documentazione tecnica del sistema **DMS / DMS-HUB / MIO-HUB**.

## Scopo

Questo blueprint serve come **punto di verità ufficiale** per tutti gli sviluppi futuri, sia per sviluppatori umani che per agenti AI. Ogni nuova analisi, decisione architetturale o sviluppo deve partire da qui e essere documentato qui.

## Struttura

La documentazione è organizzata nella directory `docs/`:

- `00-overview.md`: Panoramica generale e visione futura (TO-BE)
- `01-sistemi-e-ambienti.md`: Heroku, Vercel, Hetzner, DB, app, ecc.
- `02-dominio-funzionale.md`: Mercati, piazzole, ambulanti, concessioni, presenze, CO₂, wallet, ecc.
- `03-modello-dati-e-mapping.md`: Schema target Postgres + mapping dal gestionale DMS e da `Anagrafiche_API_DMS.xlsx`
- `04-architettura-tecnica.md`: Frontend, backend/Core API, pattern di integrazione
- `05-integrazioni-esterne.md`: Gestionale DMS su Heroku, app ambulanti, GIS, ecc.
- `06-roadmap.md`: Step tecnici in ordine (pulizia DB, migrazione a Postgres, integrazione gestionale DMS, GIS, ecc.)

La directory `references/` contiene materiali di riferimento, come analisi storiche e anagrafiche.
