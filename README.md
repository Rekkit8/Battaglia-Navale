
# ⚓ Battaglia Navale — Documentazione del Progetto

Gioco multiplayer a turni basato su connessione TCP, con interfaccia grafica Tkinter, effetti sonori sintetizzati e salvataggio delle statistiche in JSON.

---

## Struttura del Progetto

```
battaglia_navale/
├── server.py       # Server TCP: coordina la partita tra due client
├── client.py       # Client GUI: interfaccia grafica Tkinter "Naval War Room"
├── game_logic.py   # Logica di gioco pura (navi, colpi, vittoria)
└── statistiche.json  # Generato automaticamente al termine di ogni partita
```

---

## Avvio

### 1 — Avviare il server
```bash
python server.py
```
Il server si mette in ascolto su `0.0.0.0:50007` e attende due giocatori.

### 2 — Avviare i client (su due terminali o macchine diverse)
```bash
python client.py
```
Dalla schermata di login inserire nome, indirizzo IP del server e porta, poi premere **CONNETTI**.

> Per una partita in locale entrambi i client usano `127.0.0.1:50007`.

---

## Architettura

### `game_logic.py` — Logica pura
Modulo indipendente dalla rete che definisce le strutture dati e le regole di gioco.

| Elemento | Descrizione |
|---|---|
| `Griglia` | Classe che rappresenta la griglia 10×10 di un giocatore |
| `ACQUA / NAVE / COLPITO / MANCATO` | Costanti per lo stato di ogni cella |
| `verifica_colpo()` | Applica un colpo e restituisce `acqua`, `colpito` o `gia_colpito` |
| `is_affondata()` | Flood-fill che controlla se la nave colpita è completamente affondata |
| `tutte_affondate()` | Controlla la condizione di vittoria |
| `FLOTTA` | Lista delle navi standard: Portaerei (5), Corazzata (4), 2× Incrociatore (3), 3× Cacciatorpediniere (2) |

---

### `server.py` — Server TCP multi-thread

Il server accetta esattamente **due connessioni** e avvia un thread per ciascun giocatore.

**Fasi della partita:**

1. **Registrazione** — ricezione del nome di ogni giocatore
2. **Posizionamento** — ricezione della griglia (`richiesta_griglia` → `griglia`)
3. **Loop di gioco** — gestione dei messaggi `colpo` e `chat` fino alla condizione di vittoria o disconnessione

**Stato condiviso (`StatoPartita`):**

```python
stato.conn       # socket dei due giocatori [0, 1]
stato.nomi       # nomi registrati
stato.griglie    # oggetti Griglia di ciascun giocatore
stato.turno      # indice del giocatore corrente (0 o 1)
stato.lock       # threading.Lock per accesso esclusivo allo stato
```

**Gestione dei turni:** dopo ogni colpo in acqua il turno passa all'avversario. Un colpo a segno (anche con affondamento) mantiene il turno al giocatore corrente.

**Statistiche:** al termine della partita `aggiorna_stats()` incrementa vittorie/sconfitte e salva in `statistiche.json`.

---

### `client.py` — Client GUI Tkinter

L'interfaccia è suddivisa in tre schermate, con transizioni in dissolvenza (`fade_transition`).

| Schermata | Classe/Funzione | Descrizione |
|---|---|---|
| Login | `_build_login()` | Inserimento nome, IP, porta; accesso alla classifica |
| Posizionamento | `_build_placement()` | Drag & drop navi sulla griglia, auto-posizionamento, rotazione |
| Partita | `_build_game()` | Due griglie affiancate, log eventi, chat in tempo reale |

**Componenti grafici principali:**

- `LogoCanvas` — logo animato con onde, silhouette nave, radar rotante ed effetto glitch sul titolo
- `GridCanvas` — griglia tag-based con shimmer dell'acqua, sistema di particelle per esplosioni e spruzzi, anteprima posizionamento navi
- `StatusBar` — barra animata con luci lampeggianti e testo di stato
- `StatsPanel` — contatori in tempo reale di colpi, acqua e navi affondate

**Audio sintetizzato:** tutti i suoni (`explosion`, `splash`, `sunk`, `win`, `lose`, ecc.) sono generati via PCM 16-bit puro, senza dipendenze esterne. La riproduzione avviene in thread separati tramite `winsound` (Windows) o `aplay` (Linux).

**Rete:** il client usa un thread dedicato `_listen_loop()` per ricevere i messaggi del server in modo non bloccante, delegando l'aggiornamento della GUI al thread principale via `root.after()`.

---

## Protocollo di Comunicazione

Tutti i messaggi sono dizionari JSON terminati da `\n`, inviati su TCP.

| Tipo | Direzione | Campi principali |
|---|---|---|
| `nome` | Client → Server | `nome` |
| `ok` | Server → Client | `messaggio` |
| `avversario` | Server → Client | `nome` |
| `richiesta_griglia` | Server → Client | — |
| `griglia` | Client → Server | `celle` (lista 10×10) |
| `inizio` | Server → Client | `turno`, `messaggio` |
| `colpo` | Client → Server | `riga`, `col` |
| `risultato_colpo` | Server → Client (broadcast) | `riga`, `col`, `esito`, `tiratore`, `nave`\* |
| `turno` | Server → Client (broadcast) | `giocatore` |
| `fine_partita` | Server → Client (broadcast) | `vincitore`, `messaggio` |
| `chat` | Bidirezionale | `testo` / `mittente`, `ora` |
| `disconnessione` | Server → Client | `messaggio` |

\* Il campo `nave` è presente solo quando `esito == "affondato"` e contiene la lista di coordinate della nave.

---

## Dipendenze

| Libreria | Uso |
|---|---|
| `tkinter` | Interfaccia grafica (stdlib) |
| `socket` | Comunicazione TCP (stdlib) |
| `threading` | Concorrenza server e client (stdlib) |
| `json` | Protocollo e statistiche (stdlib) |
| `wave`, `struct`, `io` | Sintesi audio PCM (stdlib) |
| `winsound` *(opzionale)* | Riproduzione audio su Windows |
| `aplay` *(opzionale)* | Riproduzione audio su Linux |

Nessuna dipendenza esterna: il progetto gira con la sola stdlib di Python 3.10+.

---

## Note per il Colloquio Orale

### Separazione delle responsabilità
`game_logic.py` è completamente indipendente dalla rete e non importa alcun modulo di comunicazione. Questo rispetta il principio di separazione tra logica di dominio e infrastruttura: il modulo può essere testato in isolamento, riusato da altri frontend (es. una versione a riga di comando) e modificato senza toccare il codice di rete.

### Threading nel server
Il server usa un modello **un thread per giocatore**: ogni connessione è gestita da un thread indipendente (`gestisci_giocatore`), il che permette di leggere da entrambe le socket in parallelo senza bloccare il server in attesa. Lo stato condiviso (`StatoPartita`) è protetto da un `threading.Lock` che garantisce l'accesso esclusivo alle sezioni critiche (verifica del turno, aggiornamento della griglia, controllo della vittoria), prevenendo race condition.

### TCP vs UDP
La scelta del protocollo TCP (SOCK_STREAM) garantisce:
- **Ordine** dei pacchetti — i messaggi arrivano nella stessa sequenza in cui sono stati inviati, fondamentale per la coerenza dei turni
- **Affidabilità** — nessun pacchetto viene silenziosamente perso (diversamente da UDP/SOCK_DGRAM)
- **Stream orientato alla connessione** — il server rileva immediatamente se un client si disconnette

### Rilevamento della disconnessione
Quando un client si disconnette improvvisamente (crash, chiusura finestra), la chiamata `conn.recv()` restituisce `b""` (buffer vuoto) oppure solleva un'eccezione. In entrambi i casi la funzione `ricevi()` restituisce `None`, che `gestisci_giocatore()` interpreta come segnale di disconnessione e delega a `gestisci_disconnessione()` per notificare l'avversario e chiudere la sessione in modo pulito.

### Aggiornamento thread-safe della GUI
In Tkinter solo il thread principale può modificare i widget. Il thread di ascolto (`_listen_loop`) usa `root.after(0, callback)` per postare ogni aggiornamento nella coda degli eventi di Tkinter, evitando race condition sull'interfaccia grafica.

### Flood-fill in `is_affondata()`
Dopo ogni colpo a segno, la funzione esplora ricorsivamente le celle adiacenti (su/giù/sinistra/destra) per trovare tutti i segmenti della nave. Se nessun segmento è ancora `NAVE` (intatto), la nave è affondata. Questo approccio funziona correttamente per navi di qualsiasi forma lineare senza dover memorizzare esplicitamente la posizione delle navi al momento del piazzamento.
