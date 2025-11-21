# Analisi Sistema Legacy DMS (Heroku)

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**URL Sistema:** https://lapsy-dms.herokuapp.com  
**Credenziali:** checchi@me.com / Dms2022!

---

## Panoramica Generale

Il sistema DMS legacy è un gestionale web-based ospitato su Heroku, sviluppato con il framework Lapsy. Il sistema è organizzato in sezioni principali accessibili tramite un menu laterale.

### Sezioni Principali

Il menu laterale presenta le seguenti voci:

1. **Amministrazione** - Gestione clienti, login utente e colonnine
2. **Ambulanti** - Gestione degli operatori commerciali
3. **Comuni** - Gestione dei comuni
4. **Mercati** - Gestione dei mercati
5. **Spuntisti** - Gestione degli spuntisti

---

## Sezione: Amministrazione

### Pannello Clienti

La sezione Amministrazione mostra tre pannelli principali:

#### 1. Clienti
- **Campi**: ID, Ragione Sociale
- **Dati presenti**: Cliente "Demo" (ID: 1)
- **Azioni**: Nuovo, Modifica, Cancella

#### 2. Login utente
- **Campi**: ID, Email, Password, Nome, Cognome, Titolo, Ruolo
- **Azioni**: Nuovo, Salva, Cancella
- **Note**: Tabella vuota al momento dell'accesso

#### 3. Colonnine
- **Campi**: ID, Descrizione, GUID, Latitudine, Longitudine
- **Azioni**: Nuovo, Salva, Cancella
- **Note**: Tabella vuota al momento dell'accesso

---

## Osservazioni Tecniche

- Il sistema utilizza un framework chiamato "Lapsy" (visibile dal logo in alto a destra)
- L'interfaccia è basata su tabelle (datatable) con ID univoci
- Ogni sezione ha pulsanti standard: Nuovo, Modifica/Salva, Cancella
- Il sistema richiede la selezione di un cliente per lavorare (campo di ricerca in alto)
- L'interfaccia è in italiano

---

## Prossimi Passi

1. Esplorare la sezione "Ambulanti" per documentare la gestione degli operatori
2. Esplorare la sezione "Mercati" per documentare la gestione dei mercati e delle piazzole
3. Esplorare la sezione "Comuni" per capire la relazione con i mercati
4. Esplorare la sezione "Spuntisti" per comprendere questa funzionalità
5. Documentare i campi e le relazioni tra le varie entità
