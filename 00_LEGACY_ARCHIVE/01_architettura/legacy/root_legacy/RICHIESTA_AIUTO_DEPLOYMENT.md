# RICHIESTA AIUTO - Deployment Endpoint Mappa GIS

**Data:** 21 Novembre 2025  
**Priorità:** ALTA  
**Fase:** Phase 1 - Integrazione Mappa GIS (Editor v3)

---

## SITUAZIONE ATTUALE

### Obiettivo
Completare l'import dei dati della Mappa GIS (160 piazzole del mercato "Grosseto Esperando") dal file JSON generato da Editor v3 nel database di produzione.

### Problema Riscontrato
L'endpoint API per l'importazione dei dati GIS **risulta non implementato** quando testato dalla Dashboard:

```
POST /api/dmsHub/markets/importFromSlotEditor
Response: { "success": false, "error": "endpoint is not defined" }
```

---

## ARCHITETTURA SCOPERTA

### Sistema in Produzione (Hetzner 157.90.29.66)

Ho scoperto che esistono **DUE backend separati** sul server:

#### 1. Backend REST (ATTIVO in produzione)
- **Path:** `/root/mihub-backend-rest/`
- **Processo:** PM2 (PID 49672)
- **File principale:** `index.js`
- **Tecnologia:** Node.js REST API (non tRPC)
- **Stato:** ✅ RUNNING (da 19 Nov)

#### 2. Backend tRPC (Repository GitHub)
- **Path:** `/root/mihub/`
- **Repository:** https://github.com/Chcndr/mihub
- **Tecnologia:** Node.js + tRPC
- **Stato:** ❌ NON DEPLOYATO
- **Note:** Contiene l'endpoint `importFromSlotEditor` ma NON è in produzione

---

## CODICE DELL'ENDPOINT (Presente ma non deployato)

L'endpoint esiste nel repository `/root/mihub/apps/api/server/dmsHubRouter.ts`:

```typescript
markets: router({
  importFromSlotEditor: publicProcedure
    .input(z.object({
      marketName: z.string(),
      city: z.string(),
      address: z.string(),
      slotEditorData: z.object({
        container: z.any(),
        center: z.object({ lat: z.number(), lng: z.number() }),
        stalls: z.array(z.object({
          number: z.string(),
          lat: z.number(),
          lng: z.number(),
          areaMq: z.number().optional(),
          category: z.string().optional(),
        })),
        // ... altri campi
      }),
    }))
    .mutation(async ({ input }) => {
      // Logica di import nel database
    }),
})
```

**Registrazione router:** Il router `dmsHub` è correttamente registrato in `routers.ts` alla linea 138.

---

## TENTATIVI DI RISOLUZIONE

### 1. Aggiornamento Codice ✅
- Aggiornato `/root/mihub` da commit `f167c60` a `aacd9e8` (5 commit avanti)
- Comando: `git reset --hard origin/master`

### 2. Rebuild Docker Container ❌
- Tentato rebuild del container Docker
- **Fallito** con errori TypeScript:
  - `error TS2488: Type 'NeonHttpQueryResult<never>' must have a '[Symbol.iterator]()' method`
  - `error TS2307: Cannot find module 'nanoid'`
  - Altri errori di tipo in `dmsHubRouter.ts`, `mihubRouter.ts`, `storage.ts`

### 3. Rollback ✅
- Fatto rollback a commit `f167c60` (versione stabile)
- Ma questo commit NON ha l'endpoint `importFromSlotEditor`

---

## DATI PRONTI PER L'IMPORT

Ho già estratto i dati JSON dal PDF di Editor v3:

**File:** `/home/ubuntu/grosseto_esperando_market_data.json`

**Contenuto:**
- 160 piazzole con coordinate geografiche
- Mercato: "Grosseto Esperando"
- Formato: Compatibile con l'endpoint `importFromSlotEditor`

---

## DOMANDE PER RISOLVERE

### 1. Quale backend dovrebbe gestire l'endpoint?
- Il backend REST in `/root/mihub-backend-rest/` (attualmente in produzione)?
- Il backend tRPC in `/root/mihub/` (non deployato)?

### 2. Come deployare il backend tRPC?
- Serve sostituire completamente il backend REST?
- Oppure i due backend devono coesistere?
- Come configurare Nginx per il routing?

### 3. Come risolvere gli errori TypeScript?
Il codice aggiornato ha errori di compilazione. Opzioni:
- Fixare gli errori nel codice
- Usare una versione stabile precedente
- Implementare l'endpoint nel backend REST esistente

### 4. Strategia di deployment corretta?
Secondo il blueprint:
- Core API su Hetzner (Node.js/tRPC)
- PostgreSQL su Neon (già configurato)
- Frontend su Vercel (già deployato)

Ma in produzione c'è un backend REST, non tRPC!

---

## CREDENZIALI E ACCESSI

### Database Neon PostgreSQL
```
Host: ep-bold-silence-adftsojg-pooler.c-2.us-east-1.aws.neon.tech
Database: neondb
User: neondb_owner
Password: npg_lYG6JQ5Krtsi
```

### Server Hetzner
```
IP: 157.90.29.66
User: root
SSH Key: /home/ubuntu/.ssh/hetzner_server_key
```

### Repository GitHub
```
Repo: Chcndr/mihub
Branch: master
Ultimo commit: aacd9e8 (con endpoint importFromSlotEditor)
```

---

## SOLUZIONE PROPOSTA

### Opzione A: Deploy Backend tRPC (Allineato al blueprint)
1. Fixare errori TypeScript nel codice
2. Configurare build e deployment
3. Sostituire backend REST con backend tRPC
4. Aggiornare configurazione Nginx

### Opzione B: Implementare endpoint nel backend REST (Quick fix)
1. Aggiungere endpoint `/api/dmsHub/markets/importFromSlotEditor` nel backend REST
2. Riutilizzare logica dal backend tRPC
3. Restart PM2
4. Testare import

### Opzione C: Chiedere supporto sviluppatore
- Serve qualcuno che conosca l'architettura completa
- Capire quale backend è quello "ufficiale"
- Pianificare migrazione corretta

---

## PROSSIMI PASSI RICHIESTI

1. **Chiarire architettura**: Quale backend è quello corretto secondo il progetto?
2. **Strategia deployment**: Come deployare correttamente il backend tRPC?
3. **Fix errori**: Come risolvere gli errori TypeScript nel codice aggiornato?
4. **Import dati**: Una volta risolto, importare i 160 piazzole nel database

---

## CONTATTI

**Repository Blueprint:** https://github.com/Chcndr/dms-system-blueprint  
**Dashboard:** https://dms-hub-app-new.vercel.app/dashboard-pa  
**Backend API:** https://orchestratore.mio-hub.me (attualmente backend REST)

---

**Stato:** ⏸️ BLOCCATO - Richiede intervento per chiarire architettura e strategia deployment
