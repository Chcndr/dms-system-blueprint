# API Pepe GIS

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**Obiettivo:** Documentare le API operative per l'integrazione Pepe GIS.

---

## 1. Endpoint Principali

### 1.1. Importazione Mappa

- **Endpoint:** `POST /trpc/dmsHub.markets.importFromSlotEditor`
- **Descrizione:** Importa una pianta del mercato dall'Editor v3.
- **Input:**
  - `marketId`: ID del mercato
  - `slotEditorData`: Oggetto JSON generato dall'Editor v3
- **Output:**
  - `success`: `true` o `false`
  - `message`: Messaggio di stato
  - `stallsCreated`: Numero di piazzole create

### 1.2. Lettura Mappa

- **Endpoint:** `GET /trpc/dmsHub.markets.getById`
- **Descrizione:** Legge i dettagli di un mercato, inclusa la mappa GIS.
- **Input:**
  - `marketId`: ID del mercato
- **Output:**
  - `market`: Dati del mercato
  - `geometry`: Geometrie del mercato (bounding box, aree custom)
  - `stalls`: Array di piazzole con dati GIS
  - `markers`: Array di marker custom
  - `areas`: Array di aree custom

---

## 2. Collegamento Dati

Per collegare le piazzole agli ambulanti, concessioni e presenze, si usano gli ID delle tabelle correlate:

- **`stalls.id`:** ID univoco della piazzola.
- **`concessions.stall_id`:** Collega una concessione a una piazzola.
- **`vendor_presences.stall_id`:** Collega una presenza a una piazzola.

L'endpoint `markets.getById` può essere esteso per includere queste informazioni, oppure si possono creare endpoint dedicati per ottenere i dettagli di una singola piazzola con tutti i dati collegati.

---

## 3. Esempio di Utilizzo (Frontend)

1. **Caricamento Mappa:**
   - L'utente seleziona un mercato dalla lista.
   - Il frontend chiama `GET /trpc/dmsHub.markets.getById` per ottenere i dati del mercato e le piazzole.
   - La mappa viene renderizzata con MapLibre GL, posizionando le piazzole in base alle coordinate `lat`/`lng`.

2. **Dettaglio Piazzola:**
   - L'utente clicca su una piazzola sulla mappa.
   - Il frontend mostra un popup con i dettagli della piazzola (numero, stato, dimensioni).
   - Se necessario, il frontend può chiamare un altro endpoint (es. `GET /trpc/dmsHub.stalls.getDetails`) per ottenere i dati dell'ambulante, della concessione e delle presenze associate a quella piazzola.
