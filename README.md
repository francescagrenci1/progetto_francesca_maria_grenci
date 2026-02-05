# VERIFICA DI UN ENCODING FAME
## Descrizione generale
In chemioinformartica, con "encoding" si intende la trascrizione di molecole sottoforma di stringa, al pari di quanto accade con le formule molecolari. 
In particolare, in questo progetto viene utilizzato un tipo di encoding detto "FAME", che segue delle regole di implementazione ben precise.
Il progetto consiste nello scrivere una funzione che verifichi la validità dell'encoding FAME di una determinata molecola data, analizzando sia la sua correttezza sintattica sia quella relativa alla valenza dei singoli atomi e legami, escludendo la presenza di legami aromatici.

### Regole dell'encoding FAME
Per la realizzazione di questo progetto, sono state seguite le seguenti regole di encoding FAME:
- regole sintattiche: all'interno dell'encoding, gli atomi vengono rappresentati con caratteri standard MAIUSCOLI e gli unici presenti sono C, H, F, S, O, N, combinati in modi diversi. Gli atomi che formano un legame si trovano uno di seguito all’altro e sono inframezzati da simboli che corrispondono a legami covalenti: singoli (-), doppi (=) o tripli (#);
- valenza: ad ogni atomo, così come ad ogni legame, è associata una specifica valenza: C:4, F:1, S=2, N:3, H:1, O:2, "-" (legame singolo): 1, "="(legame doppio): 2, "#": 3 (legame triplo).
I singoli legami sono sempre validi per difetto: ciò significa che ogni atomo possiede legami con valenza minore o uguale alla propria;
- ramificazioni: sono sottostringe dell'encoding che iniziano con il simbolo "^" (che segue l'atomo da cui incipia la ramificazione), terminano con "&" e sono caratterizzate da almeno un legame. Se un atomo possiede ramificazioni, queste lo seguono immediatamente nella stringa. Inoltre, gli atomi all’interno della ramificazione non possiedono né legami tra loro né legami con altri atomi.
Stringhe FAME che non seguono queste regole sono considerate non valide.

## Struttura del codice:
Il progetto prevede l'utilizzo di quattro funzioni, di cui due destinate a svolgere le due verifiche principali:
1. `verifica_sintassi_fame()`: verifica la correttezza sintattica delle stringe FAME, seguendo le regole dell'encoding;
2. `verifica_valenza()`: funzione principale, la quale calcola la valenza dell'intera stringa FAME e viene eseguita solo se la sintassi dell'encoding risulta corretta.
Il calcolo della valenza prevede l'utilizzo di due altre funzioni:
    - `calcola_valenza_legami_ramificazione()`: cerca la ramificazione di prossimità all'atomo corrente nella stringa e, se la trova, ne individua i legami e calcola il contributo della valenza dell'atomo a cui essa è associata a sinistra;
    - `calcola_valenza_atomi_ramificazione()`: inidividua gli atomi nella ramificazione e ne calcola la valenza.

### Dettaglio funzioni:
### 1. `verifica_sintassi_fame()`:
**Parametri di input:**
- Stringa FAME

**Output:**
- bool: restiruisce `True` se la stringa è valida, altrimenti `False`

**Processo di funzionamento:**
1) scorre la stringa FAME fornita nell'argomento a partire dal primo carattere;
2) verifica che i caratteri siano validi, cioè presenti in `legami: str`, `atomi: str`e `delimitatori_ramificazione: str`;
3) se i caratteri sono validi, verifica che ogni atomo sia inframezzato da legami e che le ramificazioni siano racchiuse dai delimitatori: se ciò è vero restutuisce `True`, altrimenti restituisce `False` (la stringa è non valida). Per esempio, stringe FAME costituite da un unico atomo o in cui due atomi si susseguono vengono considerate non valide.

### 2. `verifica_valenza()`:
**Parametri di input:**
- stringa FAME valida

**Output:**
- bool: restituisce `True` se la valenza della molecola è valida, `False` se non lo è

**Processo di funzionamento:**

Sono posibili 3 casisistiche:
1)  Inizio della molecola: calcola la valenza dell'atomo considerando solo il carattere a destra del primo; deve però prima verificare se subito dopo l'atomo corrente vi è o meno una ramificazione. Chiama quindi la funzione `calcola_valenza_legami_ramificazione()` per cercare l'eventuale ramificazione e calcolare la valenza dei legami al suo interno: se la ramificazione è presente, viene calcolata prima la valenza dei legami e, successivamente, anche quella degli atomi al suo interno attraverso la funzione `calcola_valenza_atomi_ramificazione()`; se la ramificazione non è presente, `verifica_valenza()` calcola la valenza dell'atomo corrente considerando solo quella del legame subito successivo.
Dopodoché, la funzione si sposta oltre la ramificazione per verificare la presenza di ulteriori legami: se li trova aggiunge la loro valenza a quella trovata;
3) Fine della molecola: calcola la valenza dell'atomo considerando solo il carattere a sinistra dell'ultimo. L'iniziatore di una ramificazione non si può trovare sull'ultimo carattere della molecola: dunque la funzione `calcola_valenza_legami_ramificazione()` non viene chiamata, ma viene solo calcolata la valenza dell'atomo corrente considerando quella del legame subito precedente ad esso;
4) Interno della ramificazione: calcola la valenza di ogni singolo atomo considerando i caratteri presenti sia destra che a sinistra. Questo caso è analogo al primo: per verificare la presenza di un'eventuale ramificazione viene chiamata la funzione `calcola_valenza_legami_ramificazione()`: se la ramificazione è presente viene chiamata anche la funzione `calcola_valenza_atomi_ramificazione()`; se invece la ramificazione non è presente, la valenza dell'atomo corrente viene calcolata da `verifica_valenza()` considerando quella dei legami che lo seguono e precedono.

Dopo aver contemplato le varie casistiche, la funzione verifica che la valenza totale calcolata per ogni atomo corrisponda a quella massima stabilita nel protocollo FAME: se ciò risulta corretto restituisce 'True', altrimenti 'False'.

### `calcola_valenza_legami_ramificazione()`:
**Input:**
- `substr: str`: sottostringa successiva all'atomo corrente, che inizia con "^" e termina con "&"

**Output:**
- `tuple[int, int]`: il primo valore corrisponde alla lunghezza della ramificazione, il secondo alla valenza dei legami contenuti in essa. Se la ramificazione non è presente, viene restiuito 0,0.

**Processo di funzionamento:**
1) considera il primo carattere della stringa: se questo coincide con "^", significa che trova l'inizio della ramificazione (cioè della sottostringa), altrimenti la ramificazione non è presente;
2) se trova l'inizio della ramificazione, inizia a scorrere su di essa fino a trovare la terminazione ("&"), calcolandone la lunghezza: a questo punto, calcola la valenza dei singoli legami compresi nella ramificazione, che sono associati all'atomo a sinistra, e restituisce i valori della lunghezza e della valenza dei legami della ramificazione.

### `calcola_valenza_atomi_ramificazione()`:
**Input:**
- `ramificazione: str`: trovata da "calcola_valenza_legami_ramificazione"

**Output:**
- `verifica_valenza: bool`: restituisce True se tutti gli atomi interni alla ramificazione hanno valenza valida.

**Processo di funzionamento:**
1) 




  
   
