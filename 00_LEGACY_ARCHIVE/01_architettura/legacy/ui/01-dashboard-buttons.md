# 📋 Inventario Completo Tasti Dashboard PA - DMS HUB

**Data Analisi:** 22 Novembre 2025  
**Repository:** `Chcndr/dms-hub-app-new`  
**File Analizzato:** `client/src/pages/DashboardPA.tsx`  
**Componenti Mappa Identificati:** 4

---

## 🎯 Panoramica

La Dashboard PA contiene **24 sezioni principali** accessibili tramite tasti nella home.
Ogni tasto attiva un tab specifico con contenuti e funzionalità dedicate.

### Componenti Mappa Identificati

| Componente | Tipo | Stato | Usa API Backend |
|------------|------|-------|-----------------|
| `MarketMapComponent.tsx` | Mappa GIS Mercati | ✅ **CERTIFICATO** | ✅ Sì (`orchestratore.mio-hub.me`) |
| `GISMap.tsx` | Mappa Generica Marker | ⚠️ Da valutare | ❌ No (dati via props) |
| `MobilityMap.tsx` | Mappa Mobilità | ⚠️ Da valutare | ❓ Da verificare |
| `Map.tsx` | Wrapper Google Maps | 📚 Documentazione | N/A |

---

## 📊 Elenco Completo Tasti (24 Sezioni)

### 1. Overview
- **Nome Visibile:** "Overview"
- **Icona:** `BarChart3`
- **Colore:** Teal (`#14b8a6`)
- **Dove:** Home → Sezioni Dashboard (prima posizione)
- **Rotta/Tab:** `activeTab === 'overview'`
- **API/Backend:** `trpc.analytics.overview.useQuery()`
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Dashboard principale con KPI e statistiche generali

---

### 2. Clienti
- **Nome Visibile:** "Clienti"
- **Icona:** `Users`
- **Colore:** Teal (`#14b8a6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'users'`
- **API/Backend:** `trpc.users.analytics.useQuery()`
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Analisi utenti e crescita

---

### 3. Mercati
- **Nome Visibile:** "Mercati"
- **Icona:** `Store`
- **Colore:** Teal (`#14b8a6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'markets'`
- **API/Backend:** `trpc.analytics.markets.useQuery()`
- **Componenti Mappa:** ✅ **`<GISMap>`** (riga 1035)
- **Dati Mappa:** Props statiche (non da API)
- **Note:** ⚠️ **MAPPA DA SOSTITUIRE** con `MarketMapComponent`

**Dettagli Mappa Attuale:**
```tsx
<GISMap
  center={[42.7634, 11.1139]}
  zoom={13}
  markers={[
    {
      id: 1,
      position: [42.7634, 11.1139],
      type: 'market',
      title: 'Mercato Centrale Grosseto',
      description: '12.500 visitatori/mese'
    }
  ]}
  height="400px"
/>
```

**Azione Richiesta:** Sostituire con `MarketMapComponent` che legge dati da API

---

### 4. Prodotti
- **Nome Visibile:** "Prodotti"
- **Icona:** `ShoppingCart`
- **Colore:** Teal (`#14b8a6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'products'`
- **API/Backend:** Mock data (nessuna API)
- **Componenti Mappa:** ✅ **`<GISMap>`** (riga 1146)
- **Dati Mappa:** Props statiche
- **Note:** ⚠️ **MAPPA DA SOSTITUIRE** - Mostra distribuzione prodotti per area

**Dettagli Mappa Attuale:**
```tsx
<GISMap
  center={[42.7634, 11.1139]}
  zoom={12}
  markers={[
    // Marker statici per categorie prodotti
  ]}
  height="400px"
/>
```

**Azione Richiesta:** Valutare se serve mappa o sostituire con visualizzazione alternativa

---

### 5. Sostenibilità
- **Nome Visibile:** "Sostenibilità"
- **Icona:** `Leaf`
- **Colore:** Verde (`#10b981`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'sustainability'`
- **API/Backend:** `trpc.sustainability.metrics.useQuery()`
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Metriche ambientali e CO2

---

### 6. TPAS
- **Nome Visibile:** "TPAS"
- **Icona:** `Package`
- **Colore:** Viola (`#8b5cf6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'tpas'`
- **API/Backend:** Mock data
- **Componenti Mappa:** ✅ **`<GISMap>`** (riga 2656)
- **Dati Mappa:** Props statiche
- **Note:** ⚠️ **MAPPA DA VALUTARE** - Sistema TPAS (Tessera Prepagata Alimentare Sostenibile)

**Dettagli Mappa Attuale:**
```tsx
<GISMap
  center={[42.7634, 11.1139]}
  zoom={13}
  markers={[
    // Marker punti vendita TPAS
  ]}
  height="400px"
/>
```

**Azione Richiesta:** Valutare se mantenere o sostituire

---

### 7. Carbon Credits
- **Nome Visibile:** "Carbon Credits"
- **Icona:** `Coins`
- **Colore:** Arancione (`#f59e0b`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'carboncredits'`
- **API/Backend:** Mock data
- **Componenti Mappa:** ✅ **`<GISMap>`** (riga 2830)
- **Dati Mappa:** Props statiche
- **Note:** ⚠️ **MAPPA DA VALUTARE** - Distribuzione crediti carbonio

**Dettagli Mappa Attuale:**
```tsx
<GISMap
  center={[42.7634, 11.1139]}
  zoom={12}
  markers={[
    // Marker progetti ecocrediti
  ]}
  height="400px"
/>
```

**Azione Richiesta:** Valutare se mantenere o sostituire

---

### 8. Real-time
- **Nome Visibile:** "Real-time"
- **Icona:** `Activity`
- **Colore:** Rosso (`#ef4444`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'realtime'`
- **API/Backend:** Mock data
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Monitoraggio real-time attività

---

### 9. Logs
- **Nome Visibile:** "Logs"
- **Icona:** `FileText`
- **Colore:** Cyan (`#06b6d4`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'logs'`
- **API/Backend:** `trpc.logs.system.useQuery()`
- **Componenti Mappa:** ❌ Nessuno
- **Note:** System logs e Guardian logs (2 sub-tab)

---

### 10. Agente AI
- **Nome Visibile:** "Agente AI"
- **Icona:** `Bot`
- **Colore:** Viola (`#8b5cf6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'ai'`
- **API/Backend:** `useOrchestrator()`, `useAgentConversation()`
- **Componenti Mappa:** ❌ Nessuno
- **Componente Dedicato:** `<MIOAgent />` (riga 3179)
- **Note:** Chat con MIO Agent, integrato con orchestratore

---

### 11. Sicurezza
- **Nome Visibile:** "Sicurezza"
- **Icona:** `Shield`
- **Colore:** Rosso (`#ef4444`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'security'`
- **API/Backend:** Mock data
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Monitoraggio sicurezza e accessi

---

### 12. Debug
- **Nome Visibile:** "Debug"
- **Icona:** `Bug`
- **Colore:** Grigio (`#6b7280`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'debug'`
- **API/Backend:** Mock data
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Strumenti debug e diagnostica

---

### 13. Qualificazione
- **Nome Visibile:** "Qualificazione"
- **Icona:** `Award`
- **Colore:** Teal (`#14b8a6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'qualification'`
- **API/Backend:** `trpc.businesses.list.useQuery()`
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Gestione qualificazione imprese

---

### 14. Segnalazioni & IoT
- **Nome Visibile:** "Segnalazioni & IoT"
- **Icona:** `AlertTriangle`
- **Colore:** Arancione (`#f59e0b`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'civic'`
- **API/Backend:** `trpc.civicReports.list.useQuery()`
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Segnalazioni civiche e sensori IoT

---

### 15. Utenti Imprese
- **Nome Visibile:** "Utenti Imprese"
- **Icona:** `Building2`
- **Colore:** Blu (`#3b82f6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'businesses'`
- **API/Backend:** `trpc.businesses.list.useQuery()`
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Gestione utenti business

---

### 16. Controlli/Sanzioni
- **Nome Visibile:** "Controlli/Sanzioni"
- **Icona:** `ClipboardCheck`
- **Colore:** Rosso (`#ef4444`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'inspections'`
- **API/Backend:** `trpc.inspections.list.useQuery()`
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Sistema controlli e sanzioni

---

### 17. Notifiche
- **Nome Visibile:** "Notifiche"
- **Icona:** `Bell`
- **Colore:** Viola (`#8b5cf6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'notifications'`
- **API/Backend:** `trpc.notifications.list.useQuery()`
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Centro notifiche

---

### 18. Centro Mobilità
- **Nome Visibile:** "Centro Mobilità"
- **Icona:** `Navigation`
- **Colore:** Blu (`#3b82f6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'mobility'`
- **API/Backend:** `trpc.mobility.list.useQuery()`
- **Componenti Mappa:** ✅ **`<MobilityMap>`** (riga 3077)
- **Dati Mappa:** Props da API
- **Note:** ✅ **MAPPA DEDICATA** - Componente specifico per mobilità

**Dettagli Mappa Attuale:**
```tsx
<MobilityMap
  routes={realData.mobilityData}
  center={[42.7634, 11.1139]}
  zoom={13}
/>
```

**Azione Richiesta:** Verificare se `MobilityMap` è aggiornato o serve refactoring

---

### 19. Report
- **Nome Visibile:** "Report"
- **Icona:** `FileBarChart`
- **Colore:** Verde (`#10b981`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'reports'`
- **API/Backend:** Mock data
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Generazione report e analytics

---

### 20. Integrazioni
- **Nome Visibile:** "Integrazioni"
- **Icona:** `Plug`
- **Colore:** Viola (`#8b5cf6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'integrations'`
- **API/Backend:** Nessuno (componente dedicato)
- **Componenti Mappa:** ❌ Nessuno
- **Componente Dedicato:** `<Integrazioni />` (riga 3322)
- **Note:** Gestione integrazioni esterne (Zapier, MIO, Abacus)

---

### 21. Impostazioni
- **Nome Visibile:** "Impostazioni"
- **Icona:** `Settings`
- **Colore:** Grigio (`#6b7280`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'settings'`
- **API/Backend:** Mock data
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Configurazioni sistema

---

### 22. Gestione Mercati ✅ GOLDEN VERSION
- **Nome Visibile:** "Gestione Mercati"
- **Icona:** `Store`
- **Colore:** Teal (`#14b8a6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'gestione-mercati'`
- **API/Backend:** 
  - `https://orchestratore.mio-hub.me/api/markets`
  - `https://orchestratore.mio-hub.me/api/stalls`
  - `https://orchestratore.mio-hub.me/api/vendors`
  - `https://orchestratore.mio-hub.me/api/concessions`
  - `https://orchestratore.mio-hub.me/api/gis/market-map`
- **Componenti Mappa:** ✅ **`<MarketMapComponent>`** (nel componente `GestioneMercati`)
- **Componente Dedicato:** `<GestioneMercati />` (riga 3346)
- **Note:** ✅ **VERSIONE DI RIFERIMENTO - NON MODIFICARE**

**Struttura:**
- Tab 1: Anagrafica mercato
- Tab 2: Posteggi (tabella + mappa GIS integrata)
- Tab 3: Imprese / Concessioni

**Caratteristiche Mappa:**
- ✅ Numeri scalabili con zoom
- ✅ Numeri trasparenti (no sfondo bianco)
- ✅ Colori dinamici dal database
- ✅ Popup informativi con dati DB
- ✅ Collegamento bidirezionale tabella ↔ mappa
- ✅ 4 layer maps selezionabili
- ✅ 160 posteggi numerati

---

### 23. Documentazione
- **Nome Visibile:** "Documentazione"
- **Icona:** `FileText`
- **Colore:** Blu (`#3b82f6`)
- **Dove:** Home → Sezioni Dashboard
- **Rotta/Tab:** `activeTab === 'docs'`
- **API/Backend:** Nessuno
- **Componenti Mappa:** ❌ Nessuno
- **Note:** Documentazione sistema e guide

---

### 24. (MANCANTE) Gestione HUB / Negozi / Servizi
- **Nome Visibile:** "Gestione HUB, Negozi e Servizi" (da aggiungere)
- **Icona:** `Building2` (proposta)
- **Colore:** Teal (`#14b8a6`) (proposta)
- **Dove:** Home → Sezioni Dashboard (da aggiungere)
- **Rotta/Tab:** `activeTab === 'hub-negozi'` (da creare)
- **API/Backend:** Da implementare
- **Componenti Mappa:** Da valutare
- **Note:** ⚠️ **DA IMPLEMENTARE** - Nuova sezione richiesta

---

## 🗺️ Riepilogo Componenti Mappa

### Mappe Attualmente Usate

| Sezione | Componente | Dati | Stato | Azione |
|---------|------------|------|-------|--------|
| Mercati (tab) | `<GISMap>` | Props statiche | ⚠️ Vecchia | ✅ Sostituire con `MarketMapComponent` |
| Prodotti (tab) | `<GISMap>` | Props statiche | ⚠️ Vecchia | ⚠️ Valutare se serve |
| TPAS (tab) | `<GISMap>` | Props statiche | ⚠️ Vecchia | ⚠️ Valutare se serve |
| Carbon Credits (tab) | `<GISMap>` | Props statiche | ⚠️ Vecchia | ⚠️ Valutare se serve |
| Centro Mobilità (tab) | `<MobilityMap>` | API | ✅ Dedicata | ✅ Verificare aggiornamento |
| **Gestione Mercati** | `<MarketMapComponent>` | **API** | ✅ **CERTIFICATA** | ✅ **GOLDEN VERSION** |

### Componenti Mappa Disponibili

| File | Tipo | Usa API | Stato | Note |
|------|------|---------|-------|------|
| `MarketMapComponent.tsx` | Mappa GIS Mercati | ✅ Sì | ✅ **CERTIFICATO** | **Componente di riferimento** |
| `GISMap.tsx` | Mappa Generica | ❌ No (props) | ⚠️ Da deprecare | Usata in 4 tab con dati statici |
| `MobilityMap.tsx` | Mappa Mobilità | ✅ Sì | ⚠️ Da verificare | Componente dedicato mobilità |
| `Map.tsx` | Wrapper Google Maps | N/A | 📚 Docs | Solo documentazione/template |

---

## 📝 Piano Azioni Suggerite

### Priorità Alta

1. **Sostituire `<GISMap>` nel tab "Mercati"**
   - Componente: `<MarketMapComponent>`
   - Dati: API `orchestratore.mio-hub.me/api/gis/market-map`
   - Beneficio: Dati real-time dal database invece di props statiche

### Priorità Media

2. **Valutare mappe in tab Prodotti, TPAS, Carbon Credits**
   - Verificare se servono realmente mappe
   - Se sì, creare componenti dedicati con API
   - Se no, sostituire con visualizzazioni alternative (grafici, tabelle)

3. **Verificare e aggiornare `MobilityMap`**
   - Controllare se usa API aggiornate
   - Verificare se segue pattern `MarketMapComponent`
   - Documentare uso corretto

### Priorità Bassa

4. **Deprecare `GISMap.tsx`**
   - Dopo aver sostituito tutti gli usi
   - Creare migration guide
   - Rimuovere dal codebase

---

## 🚫 Vincoli e Regole

### NON Modificare

- ✅ **Gestione Mercati** (tab completo) - Versione di riferimento
- ✅ **`MarketMapComponent.tsx`** - Componente certificato
- ✅ **Backend API** - No breaking changes senza documentazione

### Modifiche Permesse (con PR)

- ⚠️ Sostituzione `<GISMap>` con `<MarketMapComponent>` in altri tab
- ⚠️ Aggiunta nuovo tab "Gestione HUB / Negozi / Servizi"
- ⚠️ Refactoring `MobilityMap` (se necessario)

### Processo Modifiche

1. Creare branch dedicata
2. Implementare modifiche
3. Testare localmente
4. Creare Pull Request con descrizione dettagliata
5. Review e merge

---

## 📚 Riferimenti

- **Blueprint GIS:** `dms-system-blueprint/GIS_SYSTEM_UPDATE_21NOV2025.md`
- **Componente Certificato:** `client/src/components/MarketMapComponent.tsx`
- **Documentazione Componente:** `client/src/components/MarketMapComponent.README.md`
- **Dashboard PA:** `client/src/pages/DashboardPA.tsx`

---

**Ultima Revisione:** 22 Novembre 2025  
**Prossimo Aggiornamento:** Dopo implementazione nuovo tab HUB/Negozi/Servizi
