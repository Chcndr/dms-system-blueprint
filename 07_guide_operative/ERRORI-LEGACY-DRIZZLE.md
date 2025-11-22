# Errori Legacy Drizzle ORM

**Data:** 22 Novembre 2024  
**Autore:** Manus AI Agent  
**Versione:** 1.0

---

## 1. Descrizione del Problema

Il monorepo legacy `mihub` (`Chcndr/mihub`) presenta errori di compilazione TypeScript legati a **Drizzle ORM versione 0.44.7**.

**Questo repository è DEPRECATO. Non investire tempo nel fixare questi errori.**

### Errori Specifici

```
node_modules/.pnpm/drizzle-orm@0.44.7_mysql2@3.15.3_pg@8.16.3_postgres@3.4.7/node_modules/drizzle-orm/gel-core/columns/duration.d.ts(1,31): 
error TS2307: Cannot find module 'gel' or its corresponding type declarations.

node_modules/.pnpm/drizzle-orm@0.44.7_mysql2@3.15.3_pg@8.16.3_postgres@3.4.7/node_modules/drizzle-orm/gel-core/query-builders/query.d.ts(23,22): 
error TS2420: Class 'GelRelationalQuery<TResult>' incorrectly implements interface 'SQLWrapper'.
  Property 'getSQL' is missing in type 'GelRelationalQuery<TResult>' but required in type 'SQLWrapper'.
```

### Causa

- **Dipendenze incompatibili** nel pacchetto Drizzle ORM 0.44.7
- **Modulo 'gel' non trovato** (problema di packaging di Drizzle)
- **Incompatibilità TypeScript**

### Impatto

- ❌ **Build del backend fallisce**
- ❌ **Build del client fallisce**

---

## 2. Soluzioni (Solo per Riferimento Storico)

Queste soluzioni sono documentate solo per completezza. **NON implementarle.**

- **Aggiornare Drizzle ORM:** L'aggiornamento potrebbe risolvere il problema, ma richiederebbe una migrazione significativa del codice.
- **Downgrade Drizzle ORM:** Un downgrade a una versione precedente stabile potrebbe funzionare, ma non è garantito.
- **Fix Temporaneo:** Skippare il type checking durante la build.

---

## 3. Conclusione

**Il monorepo `mihub` è da considerarsi un archivio storico.**

- ✅ **NON correggere** gli errori Drizzle.
- ✅ **NON usare** questo repository per nuovi sviluppi.
- ✅ **USARE** il backend REST `mihub-backend-rest` su Hetzner per tutte le nuove funzionalità.
