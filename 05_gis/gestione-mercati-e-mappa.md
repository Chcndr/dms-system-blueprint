# Gestione Mercati e Mappa GIS

## 1. Architettura e Fonte di Verità

L'architettura del sistema di gestione mercati e mappa GIS si basa su una **singola fonte di verità**: il database PostgreSQL. Tutti i dati dinamici (stato posteggi, operatori, concessioni) sono gestiti dal backend REST e riflessi in tempo reale su tutti i componenti frontend.

- **Backend**: `mihub-backend-rest` (Hetzner)
- **Frontend**: `dms-hub-app-new`
- **Database**: PostgreSQL

## 2. Modello Dati

Le tabelle principali coinvolte sono:

- `markets`: Anagrafica mercati
- `stalls`: Anagrafica posteggi
- `vendors`: Anagrafica operatori
- `concessions`: Gestione concessioni
- `market_geometry`: Geometrie GeoJSON delle mappe

### Tabella `stalls`

| Campo | Tipo | Descrizione |
|---|---|---|
| `id` | `integer` | ID univoco posteggio |
| `market_id` | `integer` | ID mercato di appartenenza |
| `number` | `varchar` | Numero posteggio |
| `status` | `enum` | Stato posteggio (`free`, `occupied`, `reserved`, `booked`, `maintenance`) |
| `category` | `varchar` | Categoria merceologica |
| `gis_slot_id` | `varchar` | ID univoco nel GeoJSON per collegamento |
| `notes` | `text` | Note aggiuntive |

## 3. API Backend

Il `marketsRouter` espone le seguenti API:

- **`listStalls(marketId)`**: Restituisce la lista completa dei posteggi per un mercato, con dati operatore e concessione.
- **`updateStall(id, updates)`**: Aggiorna lo stato, la categoria e le note di un posteggio.
- **`getEnrichedMapData(marketId)`**: Restituisce il GeoJSON della mappa arricchito con i dati dei posteggi dal database.

## 4. Componenti Frontend

- **`GestioneMercati.tsx`**: Componente principale con tab per anagrafica, mappa, operatori e concessioni.
- **`PosteggiTab.tsx`**: Tabella posteggi con pallini colorati, select per cambio stato e sincronizzazione con la mappa.
- **`MarketMapComponent.tsx`**: Mappa GIS che visualizza i posteggi con colori basati sullo stato e popup migliorato con pulsante "Visita Vetrina".

## 5. Sincronizzazione Dati

1. L'utente cambia lo stato di un posteggio nella tabella del `PosteggiTab`.
2. Viene chiamata l'API `updateStall` per aggiornare il database.
3. Al successo, la tabella viene ricaricata e la mappa aggiorna i colori e i dati del popup.

## 6. Testing

Per testare il sistema:

1. **Backend**: Verificare che le API `listStalls`, `updateStall` e `getEnrichedMapData` funzionino correttamente.
2. **Frontend**:
   - Aprire la Dashboard PA e andare in "Gestione Mercati".
   - Selezionare un mercato e verificare che la tabella posteggi e la mappa siano caricate.
   - Cambiare lo stato di un posteggio dalla tabella e verificare che il colore sulla mappa si aggiorni.
   - Cliccare su un posteggio sulla mappa e verificare che il popup mostri i dati corretti e il pulsante "Visita Vetrina" funzioni.
   - Cliccare su una riga della tabella e verificare che la mappa si sposti e apra il popup corretto.
