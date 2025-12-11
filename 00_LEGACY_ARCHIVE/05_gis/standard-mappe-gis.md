# Standard Mappe GIS - Dashboard PA

**Data:** 25 Novembre 2025  
**Autore:** Manus AI Agent  
**Versione:** 2.0

---

## 1. Architettura Mappe (AS-IS)

La Dashboard PA utilizza **tre componenti mappa** standardizzati, ognuno con uno scopo specifico. Questa documentazione riflette lo stato **reale in produzione**.

### 1.1. GISMap (Componente Base)

**File:** `client/src/components/GISMap.tsx`

**Caratteristiche:**
- Libreria: Leaflet + OpenStreetMap
- Componente base generico e riutilizzabile
- Props: `center`, `zoom`, `markers`, `polygons`, `polylines`, etc.
- **NON deprecato** - è il componente base usato da altri componenti

**Uso:**
- Usato da `MarketMapComponent`
- Usato da altri componenti mappe future

**⚠️ VINCOLO:**
- NON toccare MAI questo componente
- È SACRO e funzionante

### 1.2. MarketMapComponent (Mappa Mercati)

**File:** `client/src/components/MarketMapComponent.tsx`

**Caratteristiche:**
- Usa `GISMap` come base
- Gestione posteggi mercati
- Colori per stato:
  - `free`: Verde (`#10b981`)
  - `occupied`: Rosso (`#ef4444`)
  - `reserved`: Arancione (`#f59e0b`)
- Dati VERI da database

**Props:**
- `mapData`: GeoJSON con geometrie posteggi
- `stallsData`: Dati aggiornati dal database (stato, operatore, etc.)

**Endpoint tRPC:**
- `dmsHub.markets.getMapData`
- `dmsHub.stalls.listByMarket`

**Uso:**
- ✅ Gestione Mercati → Mercato Grosseto → Posteggi (SACRO)
- ✅ Gestione HUB Negozi (interno)
- ⚠️ MapPage (placeholder, mappa vera da aggiungere)

**⚠️ VINCOLO:**
- NON toccare MAI questo componente
- È SACRO e funzionante
- Se si rompe → ROLLBACK immediato

### 1.3. MobilityMap (Mappa Mobilità)

**File:** `client/src/components/MobilityMap.tsx`

**Caratteristiche:**
- Usa `GISMap` come base
- Gestione trasporti pubblici
- Fermate, parcheggi, percorsi

**Props:**
- `stops`: Array di fermate/parcheggi

**Endpoint tRPC:**
- `mobility.list`

**Uso:**
- Centro Mobilità

### 1.4. Map.tsx (Google Maps)

**File:** `client/src/components/Map.tsx`

**Caratteristiche:**
- Google Maps
- **NON usato** in produzione
- Non causa conflitti

**Stato:**
- Presente nel codice ma non utilizzato

---

## 2. Utilizzo nella Dashboard PA

### 2.1. Mappe VERE (Dati Database)

| Sezione | Componente | Stato |
|---|---|---|
| Gestione Mercati → Grosseto → Posteggi | `MarketMapComponent` | ✅ SACRO - Funzionante |
| Gestione HUB Negozi | `MarketMapComponent` | ✅ Funzionante |
| Centro Mobilità | `MobilityMap` | ✅ Funzionante |
| MapPage (pagina Mappa) | Placeholder | ⚠️ Mappa vera da aggiungere |

### 2.2. Mappe MOCK (RIMOSSE)

**Stato:** ✅ Tutte rimosse (4/4)

| Sezione | Stato | Commit |
|---|---|---|
| Overview (card "Mappa Interattiva") | ✅ Rimossa | `c7b1c76` |
| Tab Mercati (mappa mock) | ✅ Rimossa | `c7b1c76` |
| Segnalazioni Civiche (mappa mock) | ✅ Rimossa | `c7b1c76` |
| Controlli e Sanzioni (mappa mock) | ✅ Rimossa | `c7b1c76` |

**Motivo rimozione:**
- Confondevano gli utenti
- Dati finti non utili

---

## 3. Mappa Grosseto (SACRA)

### 3.1. Ubicazione

**Path:** Dashboard PA → Gestione Mercati → Mercato Grosseto → Posteggi

### 3.2. Caratteristiche

- 160 posteggi colorati
- Leaflet + OpenStreetMap
- Dati VERI da database
- Funzionante e stabile

### 3.3. Componenti Coinvolti

**⚠️ NON TOCCARE MAI:**
1. `GestioneMercati.tsx` - Componente Gestione Mercati
2. `MarketMapComponent.tsx` - Componente mappa mercati
3. `GISMap.tsx` - Componente base Leaflet

### 3.4. Protocollo Test

**OBBLIGATORIO dopo ogni deploy:**

1. Aprire Dashboard PA → Gestione Mercati → Mercato Grosseto → Posteggi
2. Verificare che la mappa sia visibile con 160 posteggi colorati
3. Se mappa rotta → ROLLBACK immediato

**Causa rottura:**
- Handler MIO senza check `if (mioLoading) return;` causa re-render multipli
- Re-render multipli rompono inizializzazione Leaflet
- Documentato in: `/home/ubuntu/ANALISI_ROTTURA_MAPPA_GIS.md`

---

## 4. MapPage (Rifatta)

### 4.1. Stato

**File:** `client/src/pages/MapPage.tsx`

**Caratteristiche:**
- ✅ Rifatta completamente con design Dashboard PA
- ✅ NO barra laterale mobile
- ✅ Campo ricerca: Comune o Mercato
- ✅ Layout pulito e responsive
- ⚠️ Placeholder per mappa (NO mappa vera ancora)
- ✅ Icone e UI coerenti

### 4.2. Uso Futuro

- Preview strumenti Dashboard Negozianti/Ambulanti
- Mappa GIS vera da aggiungere dopo

---

## 5. Ispezione Codice Mappe

### 5.1. Documento

**File:** `/home/ubuntu/ISPEZIONE_COMPLETA_CODICE_MAPPE.md`

### 5.2. Risultato

- ✅ Codebase pulito
- ✅ Nessun conflitto critico
- ✅ Import corretti
- ✅ Nessun codice morto
- ✅ Componenti ben separati

### 5.3. Componenti Analizzati

1. `GISMap.tsx` - Base Leaflet (SACRO)
2. `MarketMapComponent.tsx` - Mappa mercati (SACRO)
3. `MobilityMap.tsx` - Mappa mobilità
4. `Map.tsx` - Google Maps (non usato)
5. `MapPage.tsx` - Pagina Mappa (rifatta)

---

## 6. Best Practices

### 6.1. Modifiche Mappe

**⚠️ REGOLE FERREE:**

1. **NON toccare MAI:**
   - `GISMap.tsx`
   - `MarketMapComponent.tsx`
   - `GestioneMercati.tsx`

2. **Test OBBLIGATORIO:**
   - Mappa Grosseto PRIMA di qualsiasi altra cosa
   - Se rotta → ROLLBACK immediato

3. **Handler MIO:**
   - Aggiungere `if (mioLoading) return;` per evitare re-render
   - Aggiungere `disabled={mioLoading}` al bottone

4. **Deploy:**
   - Sempre via Git (commit → push → pull → pm2 restart)
   - MAI modifiche manuali

### 6.2. Aggiunta Nuove Mappe

**Procedura:**

1. Usare `GISMap` come base
2. Creare nuovo componente specifico (es. `NewMapComponent.tsx`)
3. Passare dati come props
4. Testare in isolamento
5. Testare mappa Grosseto dopo integrazione
6. Se mappa Grosseto rotta → ROLLBACK

---

## 7. Problemi Noti

### 7.1. Rottura Mappa GIS

**Problema:**
- Handler MIO senza check `if (mioLoading) return;` causa re-render multipli
- Re-render multipli rompono inizializzazione Leaflet

**Soluzione:**
- Aggiungere check loading all'inizio di handler
- Disabilitare bottone durante loading

**Documentazione:**
- `/home/ubuntu/ANALISI_ROTTURA_MAPPA_GIS.md`

### 7.2. Widget DMS AI Assistant

**Problema:**
- Widget sotto altri elementi (z-index basso)

**Soluzione:**
- ✅ z-index aumentato a 9999

**Nota:**
- Widget NON usa orchestratore (API esterna separata)

---

## 8. Roadmap Mappe

### Fase 1 (Corrente) ✅ Completata

- ✅ Mappa Grosseto funzionante (SACRA)
- ✅ Mappe mock rimosse (4/4)
- ✅ MapPage rifatta con design Dashboard PA
- ✅ Ispezione codice completata

### Fase 2 (Prossima)

- Aggiungere mappa GIS vera a MapPage
- Integrare ricerca Comune/Mercato con backend
- Preview strumenti Dashboard Negozianti/Ambulanti

### Fase 3 (Futura)

- Mappe interattive con tool calling
- Filtri avanzati
- Export dati mappa

---

**Autore:** Manus AI Agent  
**Ultimo aggiornamento:** 25 Novembre 2025 - Mappe mock rimosse, MapPage rifatta, codebase pulito
