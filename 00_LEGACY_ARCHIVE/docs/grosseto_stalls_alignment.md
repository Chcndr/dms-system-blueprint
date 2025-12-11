# Allineamento Posteggi Mercato Grosseto

**Autore:** Manus AI  
**Data Allineamento:** 26 Novembre 2025  
**Mercato:** Grosseto (GR001)  
**Database:** Neon PostgreSQL  
**GIS:** GeoJSON market-map

---

## 🎯 Obiettivo

Allineare la numerazione dei posteggi del Mercato Grosseto tra database e GIS, utilizzando il **GIS come master** della numerazione.

**Problema iniziale:**
- Database: 160 posteggi (numeri 1-160 sequenziali)
- GIS: 160 features (numeri 1-185 non sequenziali, con buchi)
- Disallineamento: 22 numeri presenti solo in DB, 22 numeri presenti solo in GIS
- Conseguenza: Click su posteggio in mappa non corrispondeva alla tabella UI

---

## 📊 Analisi Pre-Allineamento

### Database (Neon)

**Tabella:** `stalls`  
**Market ID:** 1 (Grosseto)  
**Totale record:** 160

**Numeri posteggi DB:**
```
1-160 (sequenziali, senza buchi)
```

### GIS (GeoJSON)

**Endpoint:** `/api/gis/market-map?market_id=1`  
**Totale features:** 160

**Numeri posteggi GIS:**
```
1-79, 93, 96-105, 110-116, 118-127, 130-160, 161-179, 182-183, 185
(non sequenziali, con buchi)
```

### Diff Identificato

**22 numeri SOLO nel DB (mancanti in GIS):**
```
80-92, 94-95, 106-109, 117, 128-129
```

**22 numeri SOLO nel GIS (mancanti in DB):**
```
161-179, 182-183, 185
```

**Caso critico:** Posteggio 109
- Presente nel DB (status="riservato")
- NON presente nel GIS (manca poligono)
- Risultato: visibile in tabella UI, NON visibile in mappa

---

## 🔧 Soluzione Implementata

### Fase 1: Aggiunta Colonna `is_active`

```sql
ALTER TABLE stalls 
ADD COLUMN is_active boolean DEFAULT true;
```

**Scopo:** Distinguere posteggi attivi (presenti in GIS) da posteggi storici/revocati (solo DB).

### Fase 2: Disattivazione Posteggi Storici

```sql
UPDATE stalls
SET is_active = false
WHERE market_id = 1
  AND number IN (
    '80', '81', '82', '83', '84', '85', '86', '87', '88', '89',
    '90', '91', '92', '94', '95', '106', '107', '108', '109',
    '117', '128', '129'
  );
```

**Risultato:** 22 posteggi disattivati

### Fase 3: Attivazione Posteggi GIS Esistenti

```sql
UPDATE stalls
SET is_active = true
WHERE market_id = 1
  AND number IN (
    -- Numeri 1-79, 93, 96-105, 110-116, 118-127, 130-160
    -- (lista completa nel file alignment_stalls.sql)
  );
```

**Risultato:** 138 posteggi attivati

### Fase 4: Creazione Stub Posteggi GIS-Only

```sql
INSERT INTO stalls (
  market_id, number, status, type, is_active,
  created_at, updated_at
)
VALUES
  (1, '161', 'libero', 'libero', true, NOW(), NOW()),
  (1, '162', 'libero', 'libero', true, NOW(), NOW()),
  -- ... (22 record totali)
  (1, '185', 'libero', 'libero', true, NOW(), NOW());
```

**Risultato:** 22 posteggi stub creati

### Fase 5: Modifica API Backend

**File:** `/root/mihub-backend-rest/routes/markets.js`  
**Commit:** `798669b`

**Modifica query SQL:**
```diff
SELECT s.*
FROM stalls s
WHERE s.market_id = $1
+  AND (s.is_active = true OR s.is_active IS NULL)
ORDER BY s.number ASC
```

**Scopo:** Filtrare solo posteggi attivi nelle risposte API.

---

## ✅ Risultato Post-Allineamento

### Database

**Totale record:** 182 (160 originali + 22 stub)

**Posteggi attivi (`is_active=true`):** 160
- Numeri: 1-79, 93, 96-105, 110-116, 118-127, 130-160, 161-179, 182-183, 185

**Posteggi storici (`is_active=false`):** 22
- Numeri: 80-92, 94-95, 106-109, 117, 128-129

### API

**Endpoint:** `/api/markets/1/stalls`

**Risposta:**
```json
{
  "success": true,
  "data": [ /* 160 posteggi */ ]
}
```

**Conteggio:** 160 ✅  
**Numeri presenti:** Match perfetto con GIS ✅  
**Posteggio 109:** NON presente ✅

### UI Dashboard PA

**URL:** https://dms-hub-app-new.vercel.app/dashboard-pa

**Gestione Mercati → Grosseto:**
- Posteggi Totali: 160 ✅
- Occupati: 99
- Liberi: 61
- Riservati: 0

**Tabella Posteggi:**
- 160 righe visibili
- Numerazione allineata con GIS
- Posteggio 109 NON presente

**Mappa GIS:**
- 160 poligoni visibili
- Click su poligono → corrispondenza corretta con tabella

---

## 📝 Numeri Posteggi Attivi (GIS Master)

### Range 1-79
```
1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
21, 22, 23, 24, 25, 26, 27, 28, 29, 30,
31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
41, 42, 43, 44, 45, 46, 47, 48, 49, 50,
51, 52, 53, 54, 55, 56, 57, 58, 59, 60,
61, 62, 63, 64, 65, 66, 67, 68, 69, 70,
71, 72, 73, 74, 75, 76, 77, 78, 79
```

### Range 93-160 (con buchi)
```
93, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105,
110, 111, 112, 113, 114, 115, 116,
118, 119, 120, 121, 122, 123, 124, 125, 126, 127,
130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140,
141, 142, 143, 144, 145, 146, 147, 148, 149, 150,
151, 152, 153, 154, 155, 156, 157, 158, 159, 160
```

### Range 161-185 (GIS-only)
```
161, 162, 163, 164, 165, 166, 167, 168, 169, 170,
171, 172, 173, 174, 175, 176, 177, 178, 179,
182, 183, 185
```

**Totale:** 160 posteggi attivi

---

## 📝 Numeri Posteggi Storici (Disattivati)

### Range 80-92
```
80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92
```

### Altri
```
94, 95, 106, 107, 108, 109, 117, 128, 129
```

**Totale:** 22 posteggi storici

**Motivo disattivazione:** Numeri non presenti nel GIS (poligoni mancanti o rinumerati).

---

## 🧪 Test di Verifica

### Test 1: Conteggio API

```bash
curl "https://mihub.157-90-29-66.nip.io/api/markets/1/stalls" | jq '.data | length'
```

**Risultato:** `160` ✅

### Test 2: Posteggio 109 Assente

```bash
curl "https://mihub.157-90-29-66.nip.io/api/markets/1/stalls" | jq '.data[] | select(.number == "109")'
```

**Risultato:** Nessun output (vuoto) ✅

### Test 3: Numeri Storici Assenti

```bash
curl "https://mihub.157-90-29-66.nip.io/api/markets/1/stalls" | jq -r '.data[].number' | grep -E "^(80|81|...|109|117|128|129)$"
```

**Risultato:** Nessun match ✅

### Test 4: Numeri GIS-Only Presenti

```bash
curl "https://mihub.157-90-29-66.nip.io/api/markets/1/stalls" | jq -r '.data[].number' | grep -E "^(161|162|...|185)$" | wc -l
```

**Risultato:** `22` ✅

### Test 5: UI Dashboard PA

**Navigazione:**
1. https://dms-hub-app-new.vercel.app/dashboard-pa
2. Gestione Mercati → Grosseto → Posteggi

**Verifica:**
- Posteggi Totali: 160 ✅
- Tabella: 160 righe ✅
- Mappa: 160 poligoni ✅

---

## 📚 File Correlati

**Script SQL:**
- `/root/alignment_stalls.sql` - Script completo eseguito su Neon

**Report:**
- `/root/report_diff_stalls.md` - Analisi diff DB vs GIS
- `/root/alignment_summary.md` - Riepilogo insiemi
- `/root/db_stalls_numbers.txt` - Numeri DB (160)
- `/root/gis_stalls_numbers.txt` - Numeri GIS (160)
- `/root/diff_stalls_only_db.txt` - Numeri solo DB (22)
- `/root/diff_stalls_only_gis.txt` - Numeri solo GIS (22)

**Deploy:**
- `docs/DEPLOY_BACKEND_MIHUB_HETZNER.md` - Procedura deploy ufficiale
- `docs/DEPLOY_INSTRUCTIONS_798669b.md` - Istruzioni deploy commit specifico

**Commit:**
- GitHub: https://github.com/Chcndr/mihub-backend-rest/commit/798669b

---

## 🔄 Manutenzione Futura

### Aggiunta Nuovo Posteggio

**Se il numero è già nel GIS:**
1. Verificare che il record esista nel DB con `is_active=true`
2. Se manca, fare INSERT con `is_active=true`

**Se il numero NON è nel GIS:**
1. Chiedere a Pepe di aggiungere il poligono nel GIS
2. Dopo aggiornamento GIS, fare INSERT nel DB con `is_active=true`

### Rimozione Posteggio

**NON cancellare mai record dal DB!**

1. Impostare `is_active=false`
2. Aggiungere nota in campo `notes` con motivo e data

```sql
UPDATE stalls
SET is_active = false,
    notes = 'Revocato il 2025-11-26 - motivo: ...',
    updated_at = NOW()
WHERE market_id = 1 AND number = 'XXX';
```

### Rigenerazione GIS

Se Pepe fornisce nuovo GIS con numerazione diversa:

1. Esportare numeri GIS: `jq '.features[].properties.number' new_gis.json`
2. Confrontare con DB: `SELECT number FROM stalls WHERE market_id=1 AND is_active=true`
3. Eseguire script allineamento (come fatto oggi)
4. Testare API e UI

---

## ⚠️ Note Importanti

1. **GIS è il master:** La numerazione dei posteggi segue sempre il GIS fornito da Pepe
2. **is_active è la fonte di verità:** Solo posteggi con `is_active=true` sono visibili in UI
3. **Mai cancellare record:** Usare sempre `is_active=false` per disattivare
4. **API filtra automaticamente:** Il backend filtra `is_active=true`, non serve modificare frontend
5. **Backup prima di modifiche:** Sempre fare backup DB prima di UPDATE/INSERT massivi

---

## 📊 Statistiche Finali

| Metrica | Valore |
|---------|--------|
| Posteggi attivi | 160 |
| Posteggi storici | 22 |
| Totale record DB | 182 |
| Features GIS | 160 |
| Allineamento DB-GIS | 100% ✅ |
| Allineamento API-UI | 100% ✅ |

---

**Versione Documento:** 1.0  
**Prossima Revisione:** Quando viene aggiornato il GIS o la numerazione posteggi
