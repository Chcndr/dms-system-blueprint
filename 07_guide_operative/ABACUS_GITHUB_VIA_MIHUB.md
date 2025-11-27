# Abacus → GitHub via MIHUB

## 1. Contesto

A partire dal 27 Novembre 2025, l'agente **Abacus** non accede più direttamente alle API di GitHub. Tutte le operazioni di lettura e scrittura su repository GitHub passano ora attraverso il backend **MIHUB**, che funge da proxy centralizzato.

Questa architettura garantisce:
- **Sicurezza:** Il token GitHub è memorizzato in modo sicuro nel sistema di secrets del DMS Hub, non hardcodato in Abacus.
- **Tracciabilità:** Tutte le chiamate sono loggate dal Guardian con categoria `abacus_github`.
- **Controllo:** È possibile monitorare e limitare l'accesso di Abacus ai repository tramite configurazione centralizzata.

## 2. Token GitHub

Il token utilizzato è il **"GitHub Personal Access Token"** configurato nella sezione **"Configurazioni → API & Agent Tokens"** del DMS Hub.

Il modulo `githubProxy` nel backend MIHUB legge questo token dal secret store e lo utilizza per autenticare tutte le richieste alle API di GitHub.

## 3. Endpoint Disponibili

Abacus ha accesso ai seguenti endpoint REST esposti dal backend MIHUB:

### 3.1 Lista File/Directory

**Endpoint:** `POST /api/abacus/github/list`

**Descrizione:** Lista file e directory in un repository GitHub.

**Body:**
```json
{
  "owner": "string",
  "repo": "string",
  "path": "string" (opzionale, default: root)
}
```

**Risposta:**
```json
{
  "success": true,
  "data": [
    {
      "name": "README.md",
      "path": "README.md",
      "type": "file",
      "size": 1234,
      "sha": "abc123...",
      "url": "https://api.github.com/repos/..."
    },
    ...
  ]
}
```

### 3.2 Leggi Contenuto File

**Endpoint:** `POST /api/abacus/github/get`

**Descrizione:** Legge il contenuto di un file da un repository GitHub. Il contenuto viene automaticamente decodificato da base64.

**Body:**
```json
{
  "owner": "string",
  "repo": "string",
  "path": "string"
}
```

**Risposta:**
```json
{
  "success": true,
  "data": {
    "name": "README.md",
    "path": "README.md",
    "type": "file",
    "size": 1234,
    "sha": "abc123...",
    "content": "base64_encoded_content",
    "decodedContent": "actual_file_content_as_string"
  }
}
```

### 3.3 Crea/Aggiorna File

**Endpoint:** `POST /api/abacus/github/update`

**Descrizione:** Crea un nuovo file o aggiorna un file esistente in un repository GitHub.

**Body:**
```json
{
  "owner": "string",
  "repo": "string",
  "path": "string",
  "content": "string",
  "message": "string",
  "sha": "string" (opzionale, richiesto per aggiornamenti)
}
```

**Nota:** Per aggiornare un file esistente, è necessario fornire il `sha` del file corrente. Questo può essere ottenuto tramite l'endpoint `/get`.

**Risposta:**
```json
{
  "success": true,
  "data": {
    "content": {
      "name": "README.md",
      "path": "README.md",
      "sha": "new_sha...",
      "size": 1234
    },
    "commit": {
      "sha": "commit_sha...",
      "message": "commit_message"
    }
  }
}
```

## 4. Repository Accessibili

Abacus può accedere ai seguenti repository tramite il proxy MIHUB:

| Repository | Permessi | Note |
| :--- | :--- | :--- |
| `Chcndr/dms-system-blueprint` | Lettura: ✅ / Scrittura: ✅ (solo `/reports/abacus/`) | Repository del Blueprint di sistema |
| `Chcndr/dms-hub-app-new` | Lettura: ✅ / Scrittura: ❌ | Repository frontend Dashboard PA |
| `Chcndr/mihub-backend-rest` | Lettura: ✅ / Scrittura: ❌ | Repository backend MIHUB |
| `Chcndr/MIO-hub` | Lettura: ✅ / Scrittura: ❌ | Repository catalogo API |

**Nota:** I permessi di scrittura sono limitati per garantire la sicurezza. Abacus può scrivere report nella cartella `/reports/abacus/` del Blueprint, ma non può modificare codice o configurazioni.

## 5. Logging e Guardian

Tutte le chiamate agli endpoint `/api/abacus/github/*` sono tracciate dal Guardian con:
- **Categoria:** `abacus_github`
- **Log Completo:** Request body, response status, errori

I log sono visibili nella sezione **"Logs → Guardian Logs"** del Dashboard PA.

## 6. Gestione Errori

Il modulo `githubProxy` gestisce i seguenti errori:

| Errore | Codice HTTP | Messaggio |
| :--- | :--- | :--- |
| Repository/File non trovato | 404 | `GitHub repository or path not found: {owner}/{repo}/{path}` |
| Token non valido | 401 | `GitHub API authentication failed. Check token validity.` |
| Permessi insufficienti | 403 | `GitHub API access denied. Check token permissions.` |
| Conflitto (file modificato) | 409 | `GitHub file conflict. Fetch the latest SHA and try again.` |
| Richiesta non valida | 422 | `GitHub API validation error: {message}` |

## 7. Esempio di Utilizzo

### Leggere un file dal Blueprint

```bash
curl -X POST https://mihub.157-90-29-66.nip.io/api/abacus/github/get \
  -H "Content-Type: application/json" \
  -d '{
    "owner": "Chcndr",
    "repo": "dms-system-blueprint",
    "path": "README.md"
  }'
```

### Creare un report

```bash
curl -X POST https://mihub.157-90-29-66.nip.io/api/abacus/github/update \
  -H "Content-Type: application/json" \
  -d '{
    "owner": "Chcndr",
    "repo": "dms-system-blueprint",
    "path": "reports/abacus/20251127_test_report.md",
    "content": "# Test Report\n\nThis is a test report.",
    "message": "docs: add test report from Abacus"
  }'
```

## 8. Riferimenti

- **Modulo Backend:** `mihub-backend-rest/src/modules/githubProxy.js`
- **Route Backend:** `mihub-backend-rest/routes/abacusGithub.js`
- **Catalogo API:** `MIO-hub/api/index.json` (endpoint `abacus.github.*`)
