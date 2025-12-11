# Guida Deploy e Test: GitHub Proxy per Abacus

**Data:** 27 Novembre 2025  
**Autore:** Manus AI  
**Commit Backend:** `4644613`  
**Commit API Catalog:** `f0738cc`

---

## 1. Contesto

Questa guida descrive la procedura completa per il deploy e il test del nuovo modulo **GitHub Proxy** nel backend MIHUB. Questo modulo centralizza l'accesso a GitHub per l'agente Abacus, utilizzando un token sicuro e fornendo logging centralizzato.

**Stato Codice:** Pronto e pushato su GitHub.
**Stato Deploy:** **NON ESEGUITO.** Da fare manualmente seguendo questa guida.

---

## 2. Procedura di Deploy (Operatore Umano)

L'operatore con accesso SSH al server Hetzner deve eseguire i seguenti comandi.

### 2.1 Accesso al Server

```bash
ssh root@157.90.29.66
```

### 2.2 Aggiornamento del Codice

Navigare alla directory del progetto e aggiornare il codice dal repository GitHub.

```bash
cd /root/mihub-backend-rest
git pull origin master
```

### 2.3 Installazione Dipendenze

Verificare se `package.json` è stato modificato. In caso affermativo, installare le nuove dipendenze.

```bash
npm install
```

### 2.4 Riavvio del Servizio

Riavviare il processo del backend MIHUB con PM2 per applicare le modifiche.

```bash
pm2 restart mihub-backend-rest
```

### 2.5 Verifica dei Log

Controllare i log di PM2 per assicurarsi che il servizio sia ripartito senza errori. Verificare la presenza del nuovo modulo nel log di avvio.

```bash
pm2 logs mihub-backend-rest --lines 50
```

**Output Atteso (tra gli altri):**
```
🔗 Abacus GitHub Proxy: Enabled
```

---

## 3. Procedura di Test (Operatore Umano)

Dopo il deploy, eseguire i seguenti test per verificare il corretto funzionamento del GitHub Proxy.

### 3.1 Test degli Endpoint con `curl`

Eseguire i seguenti comandi `curl` per testare i 3 nuovi endpoint.

#### 3.1.1 Test `list`

**Comando:**
```bash
curl -X POST https://mihub.157-90-29-66.nip.io/api/abacus/github/list \
  -H "Content-Type: application/json" \
  -d '{
    "owner": "Chcndr",
    "repo": "dms-system-blueprint",
    "path": "07_guide_operative"
  }'
```

**Risultato Atteso:** Un JSON con `"success": true` e un array di file/directory nella `data`.

#### 3.1.2 Test `get`

**Comando:**
```bash
curl -X POST https://mihub.157-90-29-66.nip.io/api/abacus/github/get \
  -H "Content-Type: application/json" \
  -d '{
    "owner": "Chcndr",
    "repo": "dms-system-blueprint",
    "path": "README.md"
  }'
```

**Risultato Atteso:** Un JSON con `"success": true` e un oggetto `data` contenente i dettagli del file, incluso `decodedContent` con il testo del README.

#### 3.1.3 Test `update` (Creazione File)

**Comando:**
```bash
curl -X POST https://mihub.157-90-29-66.nip.io/api/abacus/github/update \
  -H "Content-Type: application/json" \
  -d '{
    "owner": "Chcndr",
    "repo": "dms-system-blueprint",
    "path": "reports/abacus/20251127_deploy_test.md",
    "content": "# Deploy Test\n\nTest di creazione file via GitHub Proxy completato con successo.",
    "message": "test: create report file via Abacus GitHub proxy"
  }'
```

**Risultato Atteso:** Un JSON con `"success": true` e i dettagli del commit. Verificare che il file sia stato creato su GitHub.

### 3.2 Verifica Guardian Logs

1. Accedere al **Dashboard PA** → **Logs** → **Guardian Logs**.
2. Filtrare per la categoria `abacus_github`.
3. **Risultato Atteso:** Le 3 chiamate `curl` precedenti devono essere state loggate correttamente, con request body e response status.

### 3.3 Verifica API Dashboard

1. Accedere al **Dashboard PA** → **Integrazioni e API** → **API Dashboard**.
2. **Risultato Atteso:** Deve essere presente la nuova categoria **"Abacus GitHub"** con i 3 nuovi endpoint (`/list`, `/get`, `/update`).

---

## 4. Cosa Fare in Caso di Problemi

- **Errore 500 dagli endpoint:** Controllare i log di PM2 (`pm2 logs mihub-backend-rest`) per errori interni.
- **Errore 401/403:** Verificare che il **"GitHub Personal Access Token"** sia configurato correttamente in "Configurazioni → API & Agent Tokens" e che abbia i permessi necessari.
- **Errore 404:** Assicurarsi che `owner`, `repo` e `path` siano corretti.
- **Endpoint non visibili in API Dashboard:** Verificare che il commit `f0738cc` sia presente nel repository `MIO-hub` e che la cache del browser sia pulita.

---

**Fine della Guida**
