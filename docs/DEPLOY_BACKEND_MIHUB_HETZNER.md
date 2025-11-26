# Procedura Deploy Backend MIHUB su Hetzner

**Autore:** Manus AI  
**Data Creazione:** 26 Novembre 2025  
**Ultimo Aggiornamento:** 26 Novembre 2025  
**Stato:** Documento Ufficiale  
**Repo:** `Chcndr/mihub-backend-rest`

---

## 🎯 Obiettivo

Questo documento definisce la procedura standard per deployare il backend MIHUB REST su server Hetzner in produzione.

**⚠️ IMPORTANTE:** Questo deploy NON viene fatto dall'utente finale, ma da un operatore tecnico (umano o agente) con accesso SSH al server.

---

## 👥 Ruoli e Responsabilità

### Chi può fare deploy

✅ **Operatori autorizzati:**
- Agenti tecnici con accesso SSH reale al server Hetzner
- Operatori umani con credenziali SSH documentate nel blueprint

❌ **Chi NON può fare deploy:**
- Utenti finali del sistema
- Agenti in sandbox isolato senza accesso SSH esterno

### Regole per gli agenti

**Se hai accesso SSH reale:**
- ✅ Segui esattamente questa procedura
- ✅ Documenta l'esito nel blueprint

**Se NON hai accesso SSH dal tuo ambiente:**
- ❌ NON provare ripetutamente SSH che va in timeout
- ❌ NON dire "devi fare tu il deploy" all'utente in modo generico
- ✅ Genera un blocco di comandi "ready to run" per un operatore umano
- ✅ Aggiorna il blueprint spiegando cosa va eseguito
- ✅ Passa direttamente ai test/logica una volta confermato il deploy

---

## 🖥️ Parametri Backend MIHUB

### Server Hetzner

```
Provider: Hetzner Cloud
IP: 157.90.29.66
Hostname: orchestratore.mio-hub.me
Domain: https://mihub.157-90-29-66.nip.io
OS: Ubuntu 22.04 LTS
```

### Accesso SSH

**Metodo 1: SSH Key (RACCOMANDATO)**
```bash
ssh -i ~/.ssh/hetzner_server_key root@157.90.29.66
```

**Metodo 2: Password (Backup)**
```bash
ssh root@157.90.29.66
# Password: tkFxkgnT4ieh
```

### Progetto Backend

```
Path: /root/mihub-backend-rest
Repo: https://github.com/Chcndr/mihub-backend-rest.git
Branch: master
Process Manager: PM2
Nome Processo: mihub-backend
Porta Interna: 3001
Porta Pubblica: 443 (HTTPS via Nginx)
```

---

## 📋 Procedura Standard Deploy

### 1. Connessione SSH

```bash
# Usa chiave SSH (raccomandato)
ssh -i ~/.ssh/hetzner_server_key root@157.90.29.66

# Oppure con password
ssh root@157.90.29.66
```

### 2. Navigazione Progetto

```bash
cd /root/mihub-backend-rest
```

### 3. Verifica Stato Git

```bash
# Verifica branch corrente e modifiche locali
git status

# Verifica commit remoto disponibile
git fetch
git log --oneline origin/master -5
```

### 4. Pull Ultime Modifiche

```bash
# Pull dal branch master
git pull origin master

# Verifica commit attuale
git log --oneline -1
```

**Output atteso:**
```
Already up to date.
oppure
Updating abc1234..def5678
Fast-forward
 routes/markets.js | 1 +
 1 file changed, 1 insertion(+)
```

### 5. Installazione Dipendenze (se necessario)

```bash
# Solo se package.json è stato modificato
npm install --production
```

**⚠️ Nota:** Skippa questo step se non ci sono nuove dipendenze.

### 6. Restart Backend

```bash
# Restart processo PM2
pm2 restart mihub-backend

# Verifica stato
pm2 status
```

**Output atteso:**
```
┌─────┬──────────────────┬─────────┬─────────┬──────────┐
│ id  │ name             │ status  │ restart │ uptime   │
├─────┼──────────────────┼─────────┼─────────┼──────────┤
│ 0   │ mihub-backend    │ online  │ 15      │ 2s       │
└─────┴──────────────────┴─────────┴─────────┴──────────┘
```

### 7. Verifica Logs (opzionale)

```bash
# Ultimi 50 log
pm2 logs mihub-backend --lines 50

# Monitoring real-time
pm2 monit
```

---

## 🧪 Test Post-Deploy

### Test 1: Health Check

```bash
curl "https://mihub.157-90-29-66.nip.io/api/health"
```

**Output atteso:**
```json
{
  "success": true,
  "message": "Server is running"
}
```

### Test 2: Endpoint Specifico (esempio: stalls)

```bash
# Verifica conteggio posteggi Grosseto
curl "https://mihub.157-90-29-66.nip.io/api/markets/1/stalls" | jq '.data | length'
```

**Output atteso:** Numero corretto di posteggi (es. 160 dopo allineamento GIS)

### Test 3: Verifica Commit Deployato

```bash
# Sul server Hetzner
cd /root/mihub-backend-rest
git log --oneline -1
```

Confronta con commit pushato su GitHub.

---

## 🚨 Troubleshooting

### Problema: Git pull fallisce (autenticazione)

**Sintomo:**
```
fatal: could not read Password for 'https://github.com'
```

**Soluzione:**
```bash
# Cambia remote URL a HTTPS pubblico (per repo pubblici)
git remote set-url origin https://github.com/Chcndr/mihub-backend-rest.git

# Oppure usa SSH (se chiave configurata)
git remote set-url origin git@github.com:Chcndr/mihub-backend-rest.git
```

### Problema: PM2 processo non trovato

**Sintomo:**
```
[PM2] Process mihub-backend not found
```

**Soluzione:**
```bash
# Lista tutti i processi
pm2 list

# Se nome diverso, usa quello corretto
pm2 restart <nome_processo_corretto>

# Se processo non esiste, avvialo
pm2 start index.js --name mihub-backend
pm2 save
```

### Problema: Porta 3001 già in uso

**Sintomo:**
```
Error: listen EADDRINUSE: address already in use :::3001
```

**Soluzione:**
```bash
# Trova processo che usa porta 3001
lsof -i :3001

# Kill processo
kill -9 <PID>

# Restart PM2
pm2 restart mihub-backend
```

### Problema: Nginx non raggiungibile

**Sintomo:**
```
curl: (7) Failed to connect to mihub.157-90-29-66.nip.io port 443
```

**Soluzione:**
```bash
# Verifica stato Nginx
systemctl status nginx

# Restart Nginx
systemctl restart nginx

# Verifica configurazione
nginx -t
```

---

## 📊 Checklist Deploy

Prima del deploy:
- [ ] Commit pushato su GitHub branch `master`
- [ ] Credenziali SSH disponibili
- [ ] Backup database (se necessario)

Durante il deploy:
- [ ] SSH connesso
- [ ] `git pull` completato
- [ ] `npm install` (se necessario)
- [ ] `pm2 restart` eseguito
- [ ] `pm2 status` mostra "online"

Dopo il deploy:
- [ ] Health check API OK
- [ ] Endpoint specifici testati
- [ ] Logs PM2 senza errori
- [ ] UI produzione funzionante
- [ ] Documentazione blueprint aggiornata

---

## 📝 Log Deploy

### Template Log Deploy

```markdown
## Deploy [DATA]

**Commit:** [HASH] - [MESSAGGIO]
**Operatore:** [NOME/AGENTE]
**Durata:** [MINUTI]

**Modifiche:**
- [FILE 1]: [DESCRIZIONE]
- [FILE 2]: [DESCRIZIONE]

**Test:**
- ✅ Health check OK
- ✅ Endpoint X: [RISULTATO]
- ✅ UI produzione: [RISULTATO]

**Note:**
[EVENTUALI NOTE O PROBLEMI]
```

### Esempio Log Deploy

```markdown
## Deploy 26 Novembre 2025

**Commit:** 798669b - "feat: filter stalls API by is_active for markets"
**Operatore:** Manus AI
**Durata:** 5 minuti

**Modifiche:**
- routes/markets.js: Aggiunto filtro `is_active = true` alla query stalls

**Test:**
- ✅ Health check OK
- ✅ Endpoint /api/markets/1/stalls: 160 posteggi (era 182)
- ✅ Posteggio 109 rimosso correttamente
- ✅ UI produzione: tabella Grosseto allineata a GIS

**Note:**
Allineamento stalls Grosseto completato. GIS come master della numerazione.
```

---

## 🔗 Riferimenti

**Documentazione Correlata:**
- `01_architettura/legacy/root_legacy/CREDENZIALI_E_ACCESSI.md` - Credenziali SSH e database
- `02_backend/mihub-backend-rest.md` - Architettura backend REST
- `docs/grosseto_stalls_alignment.md` - Allineamento posteggi Grosseto (esempio)

**Repository:**
- GitHub: https://github.com/Chcndr/mihub-backend-rest
- Branch principale: `master`

**Server:**
- Hetzner Cloud: https://console.hetzner.cloud
- Domain: https://mihub.157-90-29-66.nip.io

---

## ⚠️ Note Importanti

1. **Mai deployare direttamente in produzione senza test locali** (se possibile)
2. **Sempre verificare logs PM2 dopo restart** per errori runtime
3. **Backup database prima di modifiche schema** (se applicabile)
4. **Documentare sempre l'esito del deploy** nel blueprint
5. **Non lasciare processi zombie** dopo deploy fallito

---

**Versione Documento:** 1.0  
**Prossima Revisione:** Quando cambia infrastruttura o procedura
