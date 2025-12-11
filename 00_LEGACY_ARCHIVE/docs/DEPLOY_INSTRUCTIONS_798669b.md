# Istruzioni Deploy Commit 798669b

**Data:** 26 Novembre 2025  
**Commit:** `798669b` - "feat: filter stalls API by is_active for markets"  
**Repo:** `Chcndr/mihub-backend-rest`  
**Server:** Hetzner 157.90.29.66

---

## 🎯 Obiettivo

Deployare commit `798669b` che aggiunge filtro `is_active = true` all'endpoint `/api/markets/:id/stalls`.

**Risultato atteso:**
- API `/api/markets/1/stalls` restituisce 160 posteggi (invece di 182)
- Posteggio 109 NON più visibile (disattivato con `is_active=false`)

---

## 📋 Istruzioni Ready-to-Run

### Step 1: Connetti SSH a Hetzner

```bash
ssh root@157.90.29.66
# Password: tkFxkgnT4ieh
# Oppure con chiave: ssh -i ~/.ssh/hetzner_server_key root@157.90.29.66
```

### Step 2: Naviga al progetto

```bash
cd /root/mihub-backend-rest
```

### Step 3: Verifica stato Git

```bash
git status
git log --oneline -1
```

**Output attuale atteso:**
```
HEAD detached at 63d4b86
oppure
On branch master
Your branch is behind 'origin/master' by 1 commit
```

### Step 4: Fetch e Pull (con autenticazione)

**⚠️ PROBLEMA:** Il repo è privato e serve autenticazione GitHub.

**Opzione A: Usa GitHub Personal Access Token**

```bash
# Genera token su https://github.com/settings/tokens (scope: repo)
# Poi:
git pull https://YOUR_GITHUB_TOKEN@github.com/Chcndr/mihub-backend-rest.git master
```

**Opzione B: Configura SSH key su server**

```bash
# Genera chiave SSH sul server (se non esiste)
ssh-keygen -t ed25519 -C "hetzner-deploy@mihub" -f ~/.ssh/id_ed25519 -N ""

# Mostra chiave pubblica
cat ~/.ssh/id_ed25519.pub

# Copia output e aggiungilo su GitHub:
# https://github.com/Chcndr/mihub-backend-rest/settings/keys
# → Add deploy key → Paste → Save

# Poi pull
git pull origin master
```

**Opzione C: Download manuale file modificato**

```bash
# Scarica solo il file modificato
curl -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3.raw" \
  -o routes/markets.js \
  https://api.github.com/repos/Chcndr/mihub-backend-rest/contents/routes/markets.js?ref=798669b

# Verifica modifica
grep "is_active" routes/markets.js
```

### Step 5: Verifica commit deployato

```bash
git log --oneline -1
```

**Output atteso:**
```
798669b feat: filter stalls API by is_active for markets
```

### Step 6: Restart backend

```bash
pm2 restart mihub-backend
pm2 status
```

**Output atteso:**
```
┌─────┬──────────────────┬─────────┬─────────┬──────────┐
│ id  │ name             │ status  │ restart │ uptime   │
├─────┼──────────────────┼─────────┼─────────┼──────────┤
│ 0   │ mihub-backend    │ online  │ 16      │ 2s       │
└─────┴──────────────────┴─────────┴─────────┴──────────┘
```

### Step 7: Test API

```bash
# Test 1: Conteggio posteggi
curl "https://mihub.157-90-29-66.nip.io/api/markets/1/stalls" | jq '.data | length'
```

**Output atteso:** `160` (era 182 prima del deploy)

```bash
# Test 2: Verifica posteggio 109 NON presente
curl "https://mihub.157-90-29-66.nip.io/api/markets/1/stalls" | jq '.data[] | select(.number == "109")'
```

**Output atteso:** Nessun risultato (vuoto)

```bash
# Test 3: Verifica numeri presenti
curl "https://mihub.157-90-29-66.nip.io/api/markets/1/stalls" | jq '.data[].number' | sort -n | head -20
```

**Output atteso:** Lista numeri GIS (1, 2, 3, ..., 79, 93, 96, ...)

---

## ✅ Checklist Deploy

- [ ] SSH connesso a Hetzner
- [ ] Navigato a `/root/mihub-backend-rest`
- [ ] Git pull completato (commit 798669b)
- [ ] PM2 restart eseguito
- [ ] PM2 status mostra "online"
- [ ] API test 1: 160 posteggi ✅
- [ ] API test 2: posteggio 109 assente ✅
- [ ] API test 3: numeri GIS corretti ✅

---

## 📝 Log Deploy

**Quando completato, aggiorna questo file con:**

```markdown
## Deploy Completato

**Data:** [DATA E ORA]
**Operatore:** [NOME]
**Durata:** [MINUTI]

**Commit deployato:** 798669b ✅

**Test:**
- ✅ API /api/markets/1/stalls: 160 posteggi
- ✅ Posteggio 109: non presente
- ✅ PM2 status: online

**Note:**
[EVENTUALI NOTE]
```

---

## 🚨 Troubleshooting

### Problema: Git pull richiede autenticazione

**Soluzione rapida:** Usa Opzione C (download manuale file)

### Problema: PM2 restart fallisce

```bash
pm2 logs mihub-backend --lines 50
```

Verifica errori syntax nel file `routes/markets.js`.

### Problema: API restituisce ancora 182 posteggi

**Causa:** Deploy non completato o cache.

**Soluzione:**
```bash
# Hard restart PM2
pm2 delete mihub-backend
pm2 start /root/mihub-backend-rest/index.js --name mihub-backend
pm2 save

# Verifica logs
pm2 logs mihub-backend
```

---

## 📎 Allegati

**File modificato:**
- `routes/markets.js` (riga 140)

**Modifica applicata:**
```diff
WHERE s.market_id = $1
+  AND (s.is_active = true OR s.is_active IS NULL)
ORDER BY s.number ASC
```

**Riferimenti:**
- Commit GitHub: https://github.com/Chcndr/mihub-backend-rest/commit/798669b
- Procedura deploy: `docs/DEPLOY_BACKEND_MIHUB_HETZNER.md`
- Allineamento Grosseto: `docs/grosseto_stalls_alignment.md` (da creare dopo deploy)

---

**Stato:** ⏳ In attesa di esecuzione  
**Prossimo step:** Eseguire deploy seguendo queste istruzioni
