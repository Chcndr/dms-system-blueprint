# ✅ FIX: 3 Problemi Critici (Novembre 2025)

**Data:** 27 Novembre 2025  
**Autore:** Manus AI

## 1. Problema: Endpoint `/api/admin/secrets` bloccato

- **Sintomo:** L'UI del Dashboard PA non poteva salvare i token perché il Guardian bloccava le chiamate.
- **Causa:** Gli endpoint non erano registrati in `api/index.json`.
- **Soluzione:** Aggiunti 3 endpoint (GET, POST, DELETE) al catalogo API.
- **Commit:** `d6c64ca` su `MIO-hub`

## 2. Problema: UI non salva token nel database

- **Sintomo:** Anche dopo aver sbloccato l'endpoint, il salvataggio falliva.
- **Causa:** Il Guardian legge `api/index.json` da GitHub e la cache CDN non era aggiornata.
- **Soluzione:** Atteso 5 minuti per la propagazione della cache.
- **Stato:** ✅ RISOLTO

## 3. Problema: `git pull` fallisce su Hetzner

- **Sintomo:** Il server non poteva aggiornare il codice da GitHub.
- **Causa:** Git non aveva credenziali configurate.
- **Soluzione:** Configurato Git per usare il token del CLI via `~/.git-credentials`.
- **Stato:** ✅ RISOLTO

## Bonus: GitHub Proxy non trovava token

- **Sintomo:** Endpoint GitHub Proxy falliva con errore 401.
- **Causa:** Parametro `scope` mancante nella chiamata `getSecret()`.
- **Soluzione:** Aggiunto `scope: 'repo'` alla chiamata.
- **Commit:** `8be3171` su `mihub-backend-rest`

---

**Tutti i problemi sono stati risolti e documentati.**
