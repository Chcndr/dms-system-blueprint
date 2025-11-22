# Backend Ufficiale vs Legacy

**Data:** 22 Novembre 2024  
**Autore:** Manus AI Agent  
**Versione:** 1.0

---

## 1. Backend Ufficiale (In Produzione)

- **Nome:** `mihub-backend-rest`
- **Hosting:** Server Hetzner (157.90.29.66)
- **Tecnologia:** Node.js + Express + PM2
- **Database:** PostgreSQL (Neon)
- **Dominio:** `orchestratore.mio-hub.me` (via Nginx reverse proxy)
- **Porta:** 3001
- **Codice:** Non versionato su Git, presente in `/root/mihub-backend-rest/`

**Questo è l'unico backend da usare per tutti i nuovi sviluppi.**

## 2. Backend Legacy (Deprecato)

- **Nome:** Monorepo `mihub`
- **Repository:** `Chcndr/mihub`
- **Tecnologia:** TypeScript + tRPC + Drizzle ORM
- **Stato:** **DEPRECATO** - Non usare per nuovi sviluppi.

**Problemi Noti:**
- **Errori di build:** Il progetto non compila a causa di errori TypeScript in Drizzle ORM (vedi `docs/ERRORI-LEGACY-DRIZZLE.md`).
- **Non deployato:** A causa degli errori di build, non è mai stato deployato in produzione.

**Utilizzo:**
- **Solo come riferimento storico:** Il codice può essere consultato per capire la logica di business implementata, ma non deve essere modificato o eseguito.
- **Non correggere gli errori:** Non investire tempo nel fixare gli errori di Drizzle, in quanto il progetto è deprecato.
