# P8: 'Around the World in 80 days'

## Informazioni tecniche

Il progetto è stato diviso in due programmi, entrambi i codici eseguono le stesse operazioni:

1)Progetto Cropelli_Non_Ottimizzato.ipynb
2)Progetto Cropelli_Ottimizzato.ipynb

Progetto Cropelli_Ottimizzato.ipynb tuttavia presenta numerose ottimizzazioni del codice che permettono di risolvere il
problema con un minore tempo macchina.


## Problema 

Il problema è un alterazione dell'esecizio P8: 'Around the World in 80 days'.

Scopo del programma è quello di trovare il percorso più breve che collega una città di partenza ad una di arrivo utilizzando l'aereo come mezzo unico di trasporto.

Sono stati identificati dei file di dati utili per lo scopo:
airports.csv contenente una tabella rappresentante una serie di aereoporti associati alle città e paesi di appartenenza.
routes.csv contenente una tabella rappresentante delle tratte aeree di collegamento tra aereoporti.

Uniti a worldcities.csv che contiene i dati di tutte le città da cui vogliamo poter arrivare o partire per il viaggio abbiamo dati sufficienti a creare un grafo rappresentante i collegamenti tra tutte le città.


Il codice è suddiviso in 3 passi principali:

-Passo 1 associazione per ogni città degli aereoporti corrispondenti o limitrofi - vengono associati per ogni città di worldcities.csv tutti gli aereoporti di airports.csv relativi. (ogni città può avere più aereoporti)

L'associazione è fatta per coppia di valori città-paese. (la sola associazione per nome di città non è sufficiente, vi sono città con lo stesso nome ma presenti in due stati differenti)
Vi sono dei casi tuttavia, per incompletezza del database airports.csv, per incongruenza tra le rappresentazioni dei paesi tra i due csv oppure per effettiva assenza di aereoporti collegati alle città che questa associazione non è possibile 
in maniera diretta.
Per citare un esempio si è notato che le città in sud korea non hanno corrispondenza univoca in quanto in airports.csv la rappresentazione dello stato è la stringa "korea_south"
mentre nel file worldcities.csv la corrispettiva rappresentazione è la stringa "south_korea".
 
In questi casi si è deciso di associare alle città l'aereoporto più vicino per distanza, se si riscontra che l'areoporto più vicino è effettivamente associato alla città il problema era la rappresentazione dello stato
altrimenti significa che la città non è veramente servita da alcun aereoporto e se ne terrà conto aumentando il tempo di tragitto di 2 ore.


-Passo 2 associazione dei tragitti per ogni città e creazione del grafo

Una volta associati ad ogni città uno o più aereoporti si utilizza i dati di routes.csv per effettuare i collegamenti tra gli aereoporti e qundi le città e calcolare il tempo di percorrenza di ogni tratta basandosi 
su una velocità stimata di crociera costante. ( La lunghezza della tratta è calcolata analizzando latitudine e longitudine degli aereoporti di partenza e di arrivo. ) 
Per finire è stato creato un dizionario dove le chiavi sono tutte le città di worldcities.csv e i valori sono delle tuple rappresentanti le città raggiungibili dalla città chiave e il relativo tempo di percorrenza della tratta.

-Passo 3 creazione grafo e utilizzo dell'agoritmo my_Dijkstra per calcolare i tragitti

Per finire questo dizionario è stato dato in input all'algoritmo my_Dijkstra, è sufficiente inserire l'indice della città di partenza e l'indice della città di arrivo e questo algoritmo calcolerà il tragitto più breve che collega queste città, sempre assumendo che esso esista.



## Algoritmo my_Dijkstra

L'algoritmo di Dijkstra è un algoritmo utilizzato per cercare i cammini minimi in un grafo con pesi non negativi sugli archi. Tale algoritmo trova applicazione nell'ottimizzazione nella 
realizzazione di reti idriche, telecomunicazioni, stradali e  circuitali. 


Ottimizzazioni:

Progetto Cropelli_Non_Ottimizzato.ipynb utilizza fortemente e direttamente i dataframe Pandas per eseguire le associazioni tra i dataframe, eseguire query e calcoli.
Il tempo computazionale risulta molto elevato in quanto più volte vengono richieste query che implicano lo scorrimento e la ricerca di dati tra dataframe relativamente estesi.

2)Progetto Cropelli_Ottimizzato.ipynb si pone come filosofia quella di utilizzare il più possibile dizionari di appoggio, ricavati dai vari dataframe, in modo da eseguire sempre le ricerche tramite chiave.
In questo modo il tempo computazionale cala sensibilmente sfruttando la più ottimizzata indicizzazione delle chiavi univoche dei dizionari.

