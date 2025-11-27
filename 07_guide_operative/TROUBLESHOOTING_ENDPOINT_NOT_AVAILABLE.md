# Troubleshooting: "This endpoint is not available"

**Data:** 2025-11-27
**Autore:** Manus AI

Questa guida spiega come risolvere l'errore "This endpoint is not available. Check api/index.json for available endpoints." che si verifica quando si tenta di chiamare un endpoint dal frontend.

## Causa Principale

L'errore indica che il **Guardian API** sta bloccando la richiesta perché l'endpoint non è registrato nel file di configurazione `api/index.json` del repository [Chcndr/MIO-hub](https://github.com/Chcndr/MIO-hub).

Anche se l'endpoint è implementato e funzionante nel backend (`mihub-backend-rest`), il Guardian non lo riconosce e lo blocca per motivi di sicurezza.

## Procedura di Risoluzione

1. **Clonare il repository MIO-hub**
   ```bash
   gh repo clone Chcndr/MIO-hub /path/to/MIO-hub
   ```

2. **Aprire il file `api/index.json`**

3. **Aggiungere il nuovo endpoint**
   - Individuare la sezione corretta dove inserire l'endpoint (es. "Mercati", "Imprese", "Concessioni").
   - Aggiungere un nuovo oggetto JSON con la configurazione dell'endpoint, seguendo la struttura degli endpoint esistenti.

   **Esempio (GET /api/markets/:marketId/companies):**
   ```json
   {
     "id": "companies.listByMarket",
     "method": "GET",
     "path": "/api/markets/:marketId/companies",
     "category": "Imprese",
     "description": "Lista imprese registrate per mercato",
     "risk_level": "low",
     "require_auth": true,
     "enabled": true,
     "test": {
       "enabled": true,
       "default_params": {"marketId": 1},
       "expected_status": 200
     },
     "implemented": "rest",
     "implementation_note": "REST endpoint implemented in mihub-backend-rest",
     "guardian_log": true,
     "log_category": "markets_companies"
   }
   ```

4. **Commit e push delle modifiche**
   ```bash
   git add api/index.json
   git commit -m "feat: add new endpoint to API catalog"
   git push origin master
   ```

5. **Attendere l'aggiornamento della cache CDN**
   - Potrebbero essere necessari circa 5 minuti prima che le modifiche siano visibili nel Dashboard PA.

6. **Verificare nel Dashboard PA**
   - Navigare in "Integrazioni e API" → "API Dashboard".
   - Verificare che il nuovo endpoint sia presente e che il badge "Implemented" sia corretto.

7. **Testare nuovamente la chiamata dal frontend**
   - L'errore "endpoint not available" non dovrebbe più comparire.
