# ⚓ Battaglia Navale Multiplayer

Progetto sviluppato in Python come applicazione client-server per il gioco della Battaglia Navale in rete locale tramite protocollo TCP.

L'applicazione permette a due giocatori di sfidarsi in tempo reale, con gestione completa della partita, chat integrata, statistiche persistenti e interfaccia grafica realizzata con Tkinter.

---

## Caratteristiche principali

- Comunicazione client-server tramite socket TCP
- Interfaccia grafica avanzata sviluppata con Tkinter
- Posizionamento manuale o automatico della flotta
- Gestione dei turni e controllo delle regole di gioco
- Sistema di chat in tempo reale tra i giocatori
- Salvataggio automatico delle statistiche in formato JSON
- Rilevamento delle disconnessioni
- Effetti grafici, animazioni e audio integrati
- Classifica persistente dei giocatori

---

## Struttura del progetto

text battaglia_navale/ │ ├── server.py ├── client_gui.py ├── game_logic.py ├── statistiche.json └── README.md 

### server.py
Gestisce la connessione dei client, i turni di gioco, il controllo delle regole e il salvataggio delle statistiche.

### client_gui.py
Interfaccia grafica del giocatore con gestione della griglia, effetti visivi, chat e interazione con il server.

### game_logic.py
Contiene la logica condivisa del gioco: gestione delle navi, validazione dei colpi, affondamenti e condizioni di vittoria.

---

## Tecnologie utilizzate

- Python 3.10+
- Socket TCP
- Threading
- JSON
- Tkinter

Nessuna libreria esterna è necessaria.

---

## Avvio del progetto

### Avvio del server

bash python server.py 

### Avvio del client

bash python client_gui.py 

Se il server viene eseguito su un altro dispositivo della rete, è sufficiente modificare l'indirizzo IP del server nel client.

---

## Funzionamento

1. Connessione dei due giocatori al server.
2. Inserimento del nome utente.
3. Posizionamento della flotta.
4. Avvio della partita.
5. Alternanza dei turni di attacco.
6. Vittoria del giocatore che affonda tutte le navi avversarie.

---

## Statistiche

Al termine di ogni partita vengono aggiornate automaticamente:

- Vittorie
- Sconfitte
- Partite giocate

I dati vengono salvati nel file statistiche.json.

---

## Aspetti tecnici

Il progetto segue una chiara separazione tra:

- Logica di gioco
- Interfaccia grafica
- Comunicazione di rete

Il server utilizza thread separati per la gestione simultanea dei giocatori e meccanismi di sincronizzazione per proteggere lo stato condiviso della partita.

La scelta del protocollo TCP garantisce affidabilità, ordine dei messaggi e integrità della comunicazione tra client e server.

---

## Note per il colloquio orale

- **game_logic.py** è completamente separato dalla rete → rispetta la separazione tra logica e comunicazione
- Il server usa **threading**: un thread per giocatore + threading.Lock per accesso sicuro allo stato condiviso
- La connessione TCP garantisce **ordine** e **affidabilità** dei pacchetti (a differenza di UDP)
- La **disconnessione** viene rilevata quando recv() restituisce None o lancia un'eccezione

---

## Autore

Progetto realizzato per il corso di Telecomunicazioni e Sistemi (TEPSIT) come applicazione distribuita client-server in Python.
