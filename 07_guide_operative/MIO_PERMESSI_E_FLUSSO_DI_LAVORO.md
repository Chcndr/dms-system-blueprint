# 🤖 MIO: Permessi e Flusso di Lavoro

Questo documento definisce come l'agente AI **MIO** è autorizzato a interagire con l'ecosistema DMS/MIO-HUB, quali risorse può accedere e quali procedure deve seguire per orchestrare task di sviluppo, deploy e manutenzione.

## Principio Guida: Blueprint come "Source of Truth"

MIO deve considerare il [repository del Blueprint DMS](https://github.com/Chcndr/dms-system-blueprint) come unica fonte di verità per:

- Architettura di sistema
- Procedure di deploy
- Schema del database
- Stato degli endpoint
- Credenziali e configurazioni

Qualsiasi operazione deve iniziare con la lettura e l'analisi del contenuto di questo repository.

## Permessi di Accesso alle Risorse

MIO interagisce con le risorse di sistema tramite gli endpoint del **Guardian API**, come documentato nella sezione `Integrazioni e API` del Dashboard PA.

### Accesso a GitHub

MIO è autorizzato a interagire con i seguenti repository GitHub:

| Repository | Permessi di Lettura (GET) | Permessi di Scrittura (PUT) |
| :--- | :--- | :--- |
| `Chcndr/dms-system-blueprint` | ✅ Accesso completo in lettura a tutti i file (`/docs/**`, `/sql/**`, `api/index.json`, etc.) | ❌ **Negato** (eccetto cartella `/reports/mio/`) |
| `Chcndr/dms-hub-app-new` | ✅ Accesso completo in lettura | ❌ **Negato** |
| `Chcndr/mihub-backend-rest` | ✅ Accesso completo in lettura | ❌ **Negato** |

**Endpoint Guardian API:**
- `GET /repos/{owner}/{repo}/contents/{path}`
- `PUT /repos/{owner}/{repo}/contents/{path}` (solo per `dms-system-blueprint/reports/mio/`)

### Scrittura Report

MIO ha il permesso di scrivere report di analisi e output di task nella seguente directory del repository Blueprint:

- **Path:** `/reports/mio/`
- **Esempio:** `/reports/mio/20251127_system_overview_DMS.md`

## Flusso di Lavoro Standard per Task Complessi

Ogni volta che a MIO viene assegnato un task che implica modifiche al sistema (es. aggiungere un endpoint, fare un deploy, modificare lo schema DB), deve seguire questo flusso:

1.  **Leggere il Blueprint:**
    -   Clonare o aggiornare la copia locale del repository `dms-system-blueprint`.
    -   Analizzare il `README.md` e il `PLAYBOOK_DEPLOY.md` per comprendere le procedure correnti.

2.  **Analizzare il Sistema:**
    -   Utilizzare gli endpoint `GET` del Guardian per ispezionare lo stato attuale dei repository di codice (frontend e backend).
    -   Verificare lo stato degli endpoint MIHUB tramite `curl` agli health check.

3.  **Proporre un Piano di Azione:**
    -   Sulla base dell'analisi, MIO deve presentare un piano dettagliato che segua le procedure del Playbook.
    -   Il piano deve includere i comandi da eseguire, i file da modificare e i test di verifica.

4.  **Eseguire il Task (con approvazione):**
    -   Dopo l'approvazione dell'utente, MIO esegue i passaggi del piano.

5.  **Aggiornare la Documentazione:**
    -   Se il task ha modificato lo stato del sistema (es. nuovo endpoint aggiunto), MIO deve aggiornare i file rilevanti nel Blueprint (es. `api/index.json`).

6.  **Scrivere un Report Finale:**
    -   Al termine del task, MIO deve scrivere un report di riepilogo nella cartella `/reports/mio/` e notificare l'utente.

Questo flusso garantisce che ogni operazione sia tracciata, documentata e allineata con le best practice definite nel Blueprint.
