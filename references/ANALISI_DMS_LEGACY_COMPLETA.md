# Analisi Funzionale Completa - Sistema Legacy DMS (Heroku)

**Autore:** Manus AI  
**Data:** 21 Novembre 2025  
**URL Sistema:** https://lapsy-dms.herokuapp.com  
**Credenziali:** checchi@me.com / Dms2022!  
**Obiettivo:** Documentare in modo esaustivo il sistema legacy per guidare la ricostruzione delle sue funzionalità nella nuova Dashboard DMS-HUB, garantire l'interoperabilità e pianificare la futura sostituzione.

---

## Sommario Esecutivo

Questo documento fornisce un'analisi funzionale dettagliata del sistema legacy DMS ospitato su Heroku. L'analisi è stata condotta attraverso l'esplorazione di tutte le sezioni dell'interfaccia utente in modalità di sola lettura. L'obiettivo è mappare tutte le entità, le relazioni, i campi e i workflow per creare una specifica chiara per lo sviluppo del nuovo sistema DMS-HUB.

Il sistema è multi-tenant, basato su un cliente "Demo", e presenta 5 sezioni principali:
1.  **Amministrazione**: Gestione clienti, utenti e colonnine.
2.  **Ambulanti**: Anagrafica ambulanti, gestione documenti e utenti dell'app mobile.
3.  **Comuni**: Anagrafica dei comuni (attualmente vuota).
4.  **Mercati**: Sezione più complessa, per la gestione di mercati, programmazione, piazzole, concessioni e presenze.
5.  **Spuntisti**: Gestione degli ambulanti occasionali.

Di seguito, ogni sezione è analizzata in dettaglio.

---

## 1. SEZIONE: AMMINISTRAZIONE

![Screenshot Amministrazione](screenshot_amministrazione.webp)

La sezione Amministrazione è la dashboard principale e contiene tre pannelli per la gestione delle entità di base del sistema.

### 1.1 Pannello "Clienti"
- **Descrizione**: Gestisce i clienti del sistema (tenant).
- **Azioni**: Nuovo, Modifica, Cancella.
- **Campi**: `ID`, `Ragione Sociale`.
- **Dati di Esempio**: ID 1, Ragione Sociale "Demo".

### 1.2 Pannello "Login utente"
- **Descrizione**: Gestisce gli utenti che possono accedere al pannello di amministrazione.
- **Azioni**: Nuovo, Salva, Cancella.
- **Campi**: `ID`, `Email`, `Password`, `Nome`, `Cognome`, `Titolo`, `Ruolo`.
- **Note**: La tabella è vuota, suggerendo che la gestione utenti potrebbe essere centralizzata o non utilizzata per il cliente Demo.

### 1.3 Pannello "Colonnine"
- **Descrizione**: Gestisce punti di rilevamento geolocalizzati, probabilmente per il check-in/check-out.
- **Azioni**: Nuovo, Salva, Cancella.
- **Campi**: `ID`, `Descrizione`, `GUID`, `Latitudine`, `Longitudine`.
- **Note**: La tabella è vuota.

---

## 2. SEZIONE: AMBULANTI

![Screenshot Ambulanti](screenshot_ambulanti.webp)

Questa sezione gestisce l'anagrafica completa degli operatori commerciali (ambulanti).

### 2.1 Pannello "Ambulanti"
- **Descrizione**: Lista principale degli ambulanti registrati.
- **Azioni**: Nuovo, Modifica, Cancella.
- **Campi**: `ID`, `Ragione Sociale`.
- **Dati di Esempio**: "Ambulante 1-7", "INTIM 8 DI CHECCHI ANDREA", "Lapsy srl".

### 2.2 Pannello "Documenti"
- **Descrizione**: Gestisce i documenti associati a un ambulante selezionato (es. licenze, DURC).
- **Azioni**: Salva, Cancella, Carica file.
- **Campi**: `ID`, `Tipo`, `Nome File`, `Tipo File`, `Valido dal`, `Valido al`.
- **Note**: La funzione "Carica file" suggerisce un sistema di upload documentale.

### 2.3 Pannello "Utenti APP"
- **Descrizione**: Gestisce gli account per l'applicazione mobile associati a un ambulante.
- **Azioni**: New, Modify, Delete.
- **Campi**: `ID`, `Email`, `Titolo`, `Nome`, `Cognome`, `Ruolo`.
- **Dati di Esempio**: Utenti con email `checchi101@me.com` a `checchi108@me.com` e ruolo "AMB".

---

## 3. SEZIONE: COMUNI

![Screenshot Comuni](screenshot_comuni.webp)

- **Descrizione**: Dovrebbe gestire l'anagrafica dei comuni in cui si svolgono i mercati.
- **Azioni**: Nuovo, Modifica, Cancella.
- **Campi**: `ID`, `Ragione Sociale`, `Nome`, `Cognome`.
- **Note**: La tabella è vuota. I campi "Ragione Sociale", "Nome" e "Cognome" sembrano inappropriati per un comune, potrebbe essere un riuso di un template di pannello.

---

## 4. SEZIONE: MERCATI

Questa è la sezione più articolata e rappresenta il cuore funzionale del sistema. È organizzata in 5 tab.

### 4.1 Tab "Gestione"

![Screenshot Mercati - Gestione](screenshot_mercati_gestione.webp)

- **Pannello "Mercati"**: Lista dei mercati.
  - **Campi**: `ID`, `Descrizione`, `Città`, `Dal`, `Al`.
  - **Dati di Esempio**: "Cervia Demo" e "Test Bologna".
- **Pannello "Programmazione"**: Definisce la ricorrenza del mercato selezionato.
  - **Campi**: `Cadenza`, `Gg sett.`, `Gg mese`, `Mese`, `Ordinale`, `Ora inizio`, `Ora fine`.
  - **Dati di Esempio** (per Cervia Demo): Cadenza "Giornaliero", Ora inizio "00:30", Ora fine "23:50".

### 4.2 Tab "Piazzole"
- **Descrizione**: Gestisce le piazzole (posteggi) del mercato selezionato.
- **Pannello "Piazzole"**: Lista delle piazzole.
  - **Campi**: `Numero`.
  - **Dati di Esempio** (per Cervia Demo): `F001P001`, `F001P002`, etc.
  - **Azioni**: "Carica piazzole" (suggerisce importazione massiva da file).

### 4.3 Tab "Concessioni"

![Screenshot Mercati - Concessioni](screenshot_mercati_concessioni.webp)

- **Descrizione**: Assegna le piazzole agli ambulanti per un determinato periodo.
- **Pannello "Concessioni"**: Lista delle concessioni attive.
  - **Campi**: `ID`, `Mercato`, `Ambulante`, `Piazzola`, `Dal`, `Al`, `Stato`.
  - **Dati di Esempio**: Mostra ambulanti (es. "M14 F001P001") assegnati a piazzole di "Cervia Demo" con stato "ATTIVA".
- **Pannello "Locazioni"**: Probabilmente per concessioni temporanee o subaffitti.
  - **Campi**: `ID`, `Ambulante`, `Valida dal`, `Valida al`, `Importo`, `Stato`.
  - **Note**: La tabella è vuota.

### 4.4 Tab "Presenze"

![Screenshot Mercati - Presenze](screenshot_mercati_presenze.webp)

- **Descrizione**: Registra le presenze giornaliere degli ambulanti al mercato.
- **Azioni**: Selettore di data e pulsante "Aggiorna".
- **Campi Tabella**: `Ambulante`, `Piazzola`, `Registrazione`, `Ingresso`, `Uscita`, `Prezzo`, `Spazzatura`, `Presenza`.
- **Note**: La tabella è vuota per la data odierna. Questa funzionalità è cruciale per il controllo operativo.

### 4.5 Tab "Mappe"
- **Descrizione**: Tab presente ma non attivo. Probabilmente una funzionalità non implementata o disabilitata, che avrebbe dovuto mostrare una mappa grafica del mercato e delle piazzole.

---

## 5. SEZIONE: SPUNTISTI

![Screenshot Spuntisti](screenshot_spuntisti.webp)

- **Descrizione**: Gestisce gli ambulanti "spuntisti", ovvero operatori occasionali che partecipano al mercato per occupare posteggi liberi.
- **Azioni**: Nuovo, Salva, Cancella.
- **Campi**: `Ambulante`, `Valido dal`, `Valido al`, `Importo`, `Stato`.
- **Dati di Esempio**: Diversi "Spunta" e "Ambulante" con stato "ATTIVO" e importo di € 500,00.

---

## Conclusioni e Raccomandazioni

L'analisi ha rivelato un sistema gestionale completo per i mercati ambulanti, con una chiara struttura master-detail e workflow definiti. La sezione **Mercati** è il fulcro dell'applicazione e richiede la massima attenzione durante la re-implementazione.

**Gap Identificati:**
1.  **File `Anagrafiche_API_DMS.xlsx` Mancante**: Cruciale per il mapping del database.
2.  **Funzionalità "Mappe"**: Non implementata nel sistema legacy, rappresenta un'opportunità di miglioramento nel nuovo sistema.
3.  **Sezione "Comuni"**: Da chiarire lo scopo e la struttura dati corretta.

**Prossimi Passi Raccomandati:**
1.  **Ottenere il file `Anagrafiche_API_DMS.xlsx`** per completare il mapping dei dati.
2.  **Progettare il nuovo schema dati** basandosi sull'analisi funzionale e sul file di mapping.
3.  **Sviluppare le sezioni del DMS-HUB** seguendo l'ordine di priorità: Mercati, Ambulanti, Spuntisti, Amministrazione, Comuni.
4.  **Progettare e implementare la funzionalità "Mappe"** come un miglioramento chiave, offrendo una visualizzazione grafica interattiva dei mercati.
