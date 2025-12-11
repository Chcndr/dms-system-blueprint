# FIX – Gemini Error Handling (MIO / gemini_arch)

## 1. Contesto

- **Componenti Coinvolti:** MIO (agente orchestratore), `gemini_arch` (agente di analisi architetturale).
- **Codice Interessato:** Il modulo `src/modules/orchestrator/llm.js` nel repository `mihub-backend-rest`, responsabile delle chiamate ai modelli LLM, incluso Gemini.

## 2. Problema Precedente

- **Causa:** Il codice accedeva direttamente a `response.data.candidates[0].content.parts[0].text` senza verificare la validità della risposta dell'API di Gemini.
- **Errore in Produzione:** `Cannot read properties of undefined (reading '0')`.
- **Effetto:** Questo errore mascherava la causa reale del fallimento della chiamata, come ad esempio:
  - **429 Too Many Requests:** Limite di richieste superato.
  - **403 Forbidden:** Chiave API non valida o non autorizzata.
  - **400 Bad Request:** Richiesta malformata.
  - Risposta vuota a causa di filtri di sicurezza o altri problemi.

## 3. Nuovo Comportamento Richiesto

- **Verifiche Obbligatorie:** Prima di processare la risposta, il codice deve sempre controllare che:
  1. La risposta non contenga un oggetto `error`.
  2. Lo status HTTP della risposta sia `200 OK`.
  3. L'array `candidates` esista e abbia una lunghezza maggiore di zero.
- **Gestione degli Errori:** Se una di queste condizioni non è soddisfatta, il sistema deve:
  1. Loggare la risposta completa di Gemini per il debug.
  2. Restituire un messaggio di errore chiaro e leggibile all'agente MIO e, di conseguenza, all'utente (es. "Gemini 429 Too Many Requests", "Gemini API key invalid", etc.).
  3. **Mai più** generare un'eccezione non gestita a causa dell'accesso a `candidates[0]`.

- **Esempio Pseudo-Codice (TypeScript):**

```typescript
async function callGeminiAPI(prompt: string): Promise<string> {
  try {
    const response = await axios.post(...);

    // 1. Check for HTTP errors (axios handles this, but good to be explicit)
    if (response.status !== 200) {
      console.error('Gemini API returned non-200 status:', response.status, response.data);
      throw new Error(`Gemini API error: ${response.status}`);
    }

    const data = response.data;

    // 2. Check for API-level errors in the response body
    if (data.error) {
      console.error('Gemini API error response:', data.error);
      throw new Error(`Gemini API error: ${data.error.message}`);
    }

    // 3. Check for empty or invalid candidates
    if (!data.candidates || data.candidates.length === 0) {
      console.error('Gemini API returned no candidates:', data);
      throw new Error('Gemini API returned no candidates (check quota or safety filters).');
    }

    // 4. Securely extract the content
    const content = data.candidates[0]?.content?.parts[0]?.text;
    if (!content) {
      console.error('Gemini API candidate has no content:', data.candidates[0]);
      throw new Error('Gemini API returned a candidate with no text content.');
    }

    return content;

  } catch (error) {
    // Log and re-throw a user-friendly error
    console.error('Failed to call Gemini:', error);
    throw new Error(`MIO failed to communicate with Gemini: ${error.message}`);
  }
}
```

## 4. Procedura di Test

1.  **Test Chiamata Normale:**
    -   Inviare una richiesta standard a MIO (es. "analizza il blueprint").
    -   **Risultato Atteso:** MIO risponde correttamente.
2.  **Test con Chiave Sbagliata:**
    -   Modificare temporaneamente la chiave API di Gemini con una non valida.
    -   **Risultato Atteso:** L'UI mostra un messaggio di errore chiaro, come "Gemini API access denied (403). Check API key configuration."
3.  **Test con Quota Esaurita (Simulata):**
    -   Se possibile, simulare una risposta 429.
    -   **Risultato Atteso:** L'UI mostra un messaggio chiaro, come "Gemini API rate limit exceeded (429). Please try again later.", senza eccezioni JavaScript.

## 5. Note Operative

- **Standard per Nuove Integrazioni:** Ogni nuova integrazione con un modello LLM deve seguire queste regole di gestione degli errori.
- **Riferimento Incrociato:** Per comprendere il flusso di lavoro completo di MIO e i suoi permessi, fare riferimento al documento [MIO: Permessi e Flusso di Lavoro](./MIO_PERMESSI_E_FLUSSO_DI_LAVORO.md).
