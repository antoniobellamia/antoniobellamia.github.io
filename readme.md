# 🚀 Documentazione Tecnica: Web 2D Shooter Game

## 1. Panoramica del Progetto
Il progetto è un videogioco sparatutto 2D basato su browser, sviluppato interamente in HTML, CSS e Vanilla JavaScript. Il giocatore controlla una navicella/cannone posizionata in basso, con l'obiettivo di distruggere un bersaglio mobile in alto, gestendo un numero limitato di munizioni ed evitando una barriera protettiva.

---

## 2. Funzionalità Principali (Features)

* **Sistema di Controllo Ibrido:** Supporto sia per tastiera fisica (`W`, `A`, `S`, `D`, `Frecce`, `Spazio`, `E`) sia per controlli touch a schermo (che possono essere disattivati all'avvio).
* **Gestione Munizioni:** Il giocatore inizia con 5 colpi. Sparando senza munizioni si va in **Game Over**.
* **Meccanica di Ricarica (Reload):** Un blocco mobile verde scorre in alto. Colpirlo fornisce +5 munizioni. Se le munizioni scendono a 1 il blocco diventa rosso; se superano 15, diventa blu e sblocca il Super Colpo.
* **Super Colpo (Super Shot):** Attivabile con il tasto `E` (o l'icona del fulmine) solo quando si hanno >= 15 munizioni. Il colpo diventa dorato, attraversa la barriera, infligge 2 danni al bersaglio, ma riduce drasticamente le munizioni rimaste (divise per 3).
* **Barriera Dinamica:** Un ostacolo marrone si muove orizzontalmente bloccando i proiettili standard.
* **Sistema del Bersaglio (Boss):** Il bersaglio si muove orizzontalmente, ha una barra della vita (7 HP) e rigenera 1 HP ogni 20 secondi.
* **Tracking delle Statistiche:** Il gioco calcola il tempo impiegato, i colpi sparati e le abilità Super utilizzate, mostrandoli a fine partita (sia in caso di vittoria che di sconfitta).

---

## 3. Dizionario delle Variabili (Variabili Globali)

### Elementi del DOM
| Variabile | Tipo | Descrizione |
| :--- | :--- | :--- |
| `boxT` | *HTMLElement* | Riferimento al bersaglio (`#target`). |
| `boxG` | *HTMLElement* | Riferimento al giocatore/cannone (`#gun`). |
| `reload` | *HTMLElement* | Riferimento al blocco mobile per ricaricare le munizioni (`#reload`). |
| `vitaBar` | *HTMLElement* | Riferimento alla barra verde della vita del bersaglio (`#vita`). |
| `barriera` | *HTMLElement* | Riferimento all'ostacolo mobile (`#barrier`). |
| `leftbtn`, `rightbtn`, `firebtn`, `superbtn`| *HTMLElement* | Riferimenti ai pulsanti touch a schermo. |
| `body` | *HTMLElement* | Riferimento al corpo della pagina (usato per l'animazione dello sfondo). |

### Variabili di Posizionamento e Dimensione
| Variabile | Tipo | Descrizione |
| :--- | :--- | :--- |
| `inc` | *Number* | Valore fisso (30px) usato per l'incremento di movimento dei blocchi. |
| `posLeftT` | *Number* | Posizione attuale sull'asse X del bersaglio. |
| `posLeftG` | *Number* | Posizione attuale sull'asse X del cannone del giocatore (inizializzata al centro). |
| `dim` | *Number* | Larghezza fisica in pixel (200px) del bersaglio. |
| `dimG` | *Number* | Larghezza fisica in pixel (300px) del giocatore. |
| `posLeftR` | *Number* | Posizione sull'asse X del blocco di ricarica. |
| `TBottom` | *Number* | Posizione sull'asse Y (dal basso) della base del bersaglio. |
| `barrieraWidth` | *Number* | Larghezza della barriera (1/4 della larghezza della finestra). |
| `barrieraBottom` | *Number* | Posizione Y della barriera (30px sotto il bersaglio). |
| `barrieraLeft` | *Number* | Posizione X attuale della barriera. |

### Variabili di Stato e Logica di Gioco
| Variabile | Tipo | Descrizione |
| :--- | :--- | :--- |
| `isRight` | *Boolean* | Flag di direzione per il bersaglio (true = va a destra, false = va a sinistra). |
| `reloadIsRight` | *Boolean* | Flag di direzione per il blocco di ricarica. |
| `barrierIsRight` | *Boolean* | Flag di direzione per la barriera. |
| `vita` | *Number* | Punti vita attuali del bersaglio (Inizia a 7). |
| `proiettili` | *Number* | Numero attuale di proiettili del giocatore (Inizia a 5). |
| `superP` | *Boolean* | Flag che indica se il colpo attualmente in canna è un "Super Colpo". |

### Contatori e Intervalli (Timers)
| Variabile | Tipo | Descrizione |
| :--- | :--- | :--- |
| `timer` | *Number* | Contatore dei secondi trascorsi dall'inizio del gioco. |
| `colpiSparati` | *Number* | Statistica: numero totale di proiettili esplosi. |
| `superUtilizzate` | *Number* | Statistica: quante volte è stata attivata l'abilità Super. |
| `boxTInterval` | *Interval* | Loop che muove il bersaglio ogni 100ms. |
| `timerInterval` | *Interval* | Loop che incrementa la variabile `timer` ogni secondo (1000ms). |
| `scrollSpeed` | *Number* | Velocità (1px) per lo scorrimento del background tramite JS. |

---

## 4. Dizionario delle Funzioni e degli Eventi

### Funzioni Autonome
* **`scrollBackground()`:** Chiamata ogni 50ms, fa scorrere verticalmente l'immagine di sfondo modificando `backgroundPositionY`. Si sovrappone parzialmente all'animazione CSS `@keyframes scrollBackground`.
* **`triggerKeyEvent(key)`:** Genera e simula "artificialmente" la pressione di un tasto della tastiera. Viene utilizzata per far funzionare i pulsanti touch a schermo riciclando la logica degli eventi da tastiera.
* **`sposta()`:** Controlla e aggiorna la posizione orizzontale del bersaglio (`boxT`). Rimbalza automaticamente quando tocca i bordi dello schermo (basandosi su `window.innerWidth`).
* **`spostarefill()`:** Controlla e aggiorna la posizione orizzontale del blocco di ricarica (`reload`). Intervallo: 1.5 secondi (1500ms).
* **`spostaBarriera()`:** Controlla e aggiorna la posizione orizzontale della barriera (`barrier`). Intervallo: 150ms.

### Intervalli Anonimi (SetInterval)
* **Rigenerazione Salute:** Ogni 20 secondi (20.000ms), se la `vita` del bersaglio è compresa tra 1 e 6, recupera 1 HP e la barra visiva si aggiorna.

### Event Listeners (Input)
* **`touchstart` sui controlli su schermo:** Mappa il tocco dell'utente su bottoni virtuali invocando `triggerKeyEvent` con il relativo pulsante fisico mappato nell'oggetto `keyMap` (`ArrowLeft`, `ArrowRight`, `ArrowUp`, `e`).
* **`keydown` (Il Loop Centrale del Giocatore):** Gestisce tutte le azioni dell'utente.
    * **Movimento:** Tasti `D` / `ArrowRight` (Destra) e `A` / `ArrowLeft` (Sinistra). Muovono la navicella aggiornando `posLeftG`.
    * **Armamento Super:** Tasto `E`. Se i proiettili sono >= 15, imposta `superP = true` e colora il blocco reload di viola.
    * **Fuoco (Sparare):** Tasto `Spazio` o `ArrowUp`.
        1. Genera un elemento `div.proj` nel DOM.
        2. Scala le munizioni (o manda in Game Over se vuote). Se è attivo un Super Colpo, l'estetica del proiettile cambia (altezza 50px, colore oro) e i proiettili vengono ridotti a 1/3 del totale.
        3. Crea un intervallo indipendente `proiettileInterval` che muove il proiettile verso l'alto di 10px ogni 10ms.

### Sistema di Collisioni (All'interno del fuoco)
Durante il viaggio del proiettile, ad ogni scatto (10ms), il codice esegue dei calcoli di intersezione tra la posizione del proiettile (`posBottom`, `posLeftP`) e gli altri oggetti:
1.  **Hit Barriera:** Se le coordinate coincidono e `!superP` (non è un super colpo), il proiettile viene distrutto (`proiettile.remove()`) e l'intervallo si ferma.
2.  **Hit Bersaglio:** Se le coordinate coincidono con il target:
    * Sottrae 1 HP (o 2 HP se `superP`).
    * Se l'HP scende a 0 o meno: Applica le texture grafiche dell'esplosione, ferma tutti gli intervalli, cambia lo sfondo del body e mostra l'Alert di Vittoria con le statistiche, poi ricarica la pagina.
3.  **Fuori Bordo (Miss):** Se il proiettile va oltre il bersaglio senza colpire, cambia colore in `aliceblue` e si disattiva il flag `superP`.
4.  **Hit Ricarica:** Se colpisce il blocco delle munizioni, il proiettile si distrugge e aggiunge +5 a `proiettili`.
5.  **Check Colore Munizioni:** Aggiorna il colore del riquadro numerico (Rosso <= 1, Verde standard, Blu >= 15) in tempo reale in base all'esito del colpo.
