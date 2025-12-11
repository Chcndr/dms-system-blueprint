# 02. Dominio Funzionale

Questo documento descrive i concetti chiave e le entità che compongono il dominio funzionale del sistema DMS-HUB.

## Entità Principali

Il dominio ruota attorno alla gestione dei mercati, degli operatori, delle transazioni economiche e degli impatti di sostenibilità.

### 1. Mercati (`markets`)

L'entità **Mercato** rappresenta un'area fisica dove si svolge l'attività di vendita. Ogni mercato è geolocalizzato e ha attributi specifici come indirizzo e orari di apertura.

### 2. Banchi/Negozi (`shops`)

Un **Banco** (o negozio) è un'unità di vendita individuale che opera all'interno di un mercato. Ogni banco è associato a un mercato specifico e ha una propria anagrafica, che include la categoria merceologica e le certificazioni.

### 3. Utenti (`users`)

Gli **Utenti** sono i cittadini che interagiscono con il sistema, principalmente attraverso l'app mobile. Possono effettuare check-in, guadagnare e spendere crediti di carbonio (TCC).

### 4. Transazioni (`transactions`)

Le **Transazioni** tracciano ogni movimento di **Token Carbon Credit (TCC)**. I TCC vengono guadagnati (`earn`) tramite comportamenti virtuosi (es. check-in al mercato con mezzi sostenibili) e spesi (`spend`) presso i banchi aderenti.

### 5. Check-in (`checkins`)

Il **Check-in** è l'azione con cui un utente registra la propria presenza in un mercato. Questa azione è fondamentale per il calcolo del risparmio di CO₂ e per l'assegnazione dei TCC.

## Concetti Chiave

### Token Carbon Credit (TCC)

I TCC sono un incentivo digitale per promuovere comportamenti sostenibili. Il loro valore e le modalità di assegnazione sono definiti nella tabella `carbon_credits_config`. Gli utenti accumulano TCC nel proprio **Wallet** (gestito nella tabella `extended_users`) e possono spenderli come sconti presso i negozi del mercato.

### Sostenibilità e CO₂

Il sistema calcola il risparmio di CO₂ generato dagli utenti che scelgono mezzi di trasporto sostenibili per recarsi al mercato. Questo calcolo si basa sui dati raccolti durante il check-in e contribuisce al punteggio di sostenibilità dell'utente.

### Concessioni e Presenze

La gestione delle **concessioni** (diritto di un operatore a occupare una piazzola) e delle **presenze** (registrazione giornaliera degli operatori al mercato) sono funzionalità chiave del gestionale DMS legacy. La loro implementazione nel nuovo sistema è una priorità della roadmap e richiederà un'analisi dettagliata del sistema esistente su Heroku.
