# P8: 'Around the World in 80 days'

## Informazioni tecniche

Il file Progetto.ipynb contiene il codice ufficiale del progetto.

Il file Proggetto.Albani.ipynb contiene il codice relativo ad una criticità del percorso, dovuta alle regole del gioco, che discuterà Albani. (nelle slides)

Il file PresentazioneProgetto.pptx contiene i lucidi utili alla presentazione orale del codice.

Il file world5.png contiene l'immagine di tutti i possibili collegamenti da ogni città del mondo verso le tre più vicine.

Il file UK9.png contiene l'ingrandimento del grafico precedente nei dintorni di Londra destinazione.

Il file worldcities.xlsx è il dataset da cui il programma attinge i dati. 


## Pacchetti utili per l'esecuzione del codice 

import pandas as pd

import geopandas as gpd

import numpy as np

import matplotlib.pyplot as plt 

import folium 

import math

from sklearn.metrics.pairwise import haversine_distances

from sklearn.neighbors import BallTree

from typing import List

from collections import defaultdict

import networkx as nx


## Problema 

Il problema si ispira al romanzo avventuroso dell'autore francese Jules Verne, pubblicato per la prima volta nel 1873. Il londinese Phileas Fogg ed il suo cameriere francese Passepartout tentano di circumnavigare il mondo in 80 giorni, per vincere una scommessa di 20.000 sterline stipulata con gli altri soci del Reform Club.

Si consideri l'insieme dei dati che descrivono alcune delle principali città del mondo e si supponga che sia sempre possibile viaggiare da ogni città fino alle tre più vicine. Il tempo di viaggio richiede 2 ore per la prima città più vicina, 4 per la seconda e 8 ore per la terza. 
Inoltre, occorrono 2 ore aggiuntive se la città di destinazione ha più di 200.000 abitanti e altre 2 ore aggiuntive se la città di partenza e arrivo non sono nella stessa nazione.
Partendo da Londra e viaggiando esclusivamente verso Est, è possibile compiere il giro del mondo in 80 giorni? 


## Strutture dati e manipolazione dati

Dapprima si rende necessario il caricamento del dataset nel DataFrame [df], con i soli attributi utili.
Segue una manipolazione dei valori relativi alla longitudine, dovuta alla necessità di viaggiare verso Oriente: la longitudine relativa ad ogni città viene riscalata del valore pari alla longitudine di Londra, in modo da avere longitudine 0° per la partenza e longitudine 360° per Londra destinazione; poi i valori negativi della longitudine subiscono la seguente trasformazione matematica max_lng +||xlng|-max_lng|, dove max_lng indica la massima longitudine positiva presente nella colonna relativa del DataFrame; xlng è il valore della longitudine della singola città. 
Questa operazione si rivelerà utile per l'ordinamento delle città nel DataFrame in ordine di longitudine crescente, considerando il movimento obbligatorio verso Est. 
Ad esempio, la città in posizione i-esima, potrà teoricamente comunicare solo con le città dalla i+1-esima in poi. 
Come ultima riga del DataFrame vengono copiati i valori riga di Londra-partenza (che nel DataFrame ordinato occuperà la prima posizione), con la sola differenza che la longitudine avrà valore 360°, ad indicare Londra-destinazione. 

Per confrontare i dati di latitudine e longitudine sulla stessa scala di misura occorre una standardizzazione, dividendo tutti i valori della colonna relativa per il rispettivo valore massimo. 

Rammentando la precedente normalizzazione della longitudine, i valori di quest'ultima saranno compresi tra 0 e 2, quelli della latitudine tra -1 e 1; tuttavia ai fini del calcolo della distanza di Haversine tra le città il diverso intervallo non sortirà alcun effetto. 
E' stata adottata la metrica di Haversine per calcolare la distanza tra due città perchè è la linea di minor lunghezza che congiunge due punti sulla superficie terrestre e giace sulla superficie stessa.


## Algoritmo di riempimento del dizionario e costruzione del grafo

Il codice presenta tre possibili algoritmi di riempimento per il grafo e il dizionario, introdotto di seguito, ai fini dell'avvio dell'esecuzione dell'algoritmo di Dijkstra. 
D'ora in poi, ogni città verrà indicata tramite l'indice posizionale nel DataFrame ordinato per longitudine crescente. 
Il dizionario [C] ha come chiavi gli interi da 0 a [number_city-1]; ogni chiave è relativa ad una lista di massimo tre tuple, riempite ciascuna con l'indice della prima città raggiungibile e il tempo di viaggio, l'indice della possibile seconda città di destinazione e il tempo e l'indice della terza città utile e le ore di spostamento. 


I -- Il primo meccanismo di riempimento è di semplice comprensione e prevede, per la città i-esima nel DataFrame, il calcolo della distanza di Haversine tra la stessa e le succesive città i+1-esima,...,end. 
Segue l'ordinamento crescente del vettore delle distanze e la selezione dei soli primi tre indici: gli indici individuano le tre città prossime alla città considerata. Il 'ciclo for' termina con il riempimento effettivo del dizionario, abbinando ciascun indice al tempo di viaggio opportuno. 

II -- Il secondo meccanismo sfrutta l'algoritmo di clustering KNeighbors, che restituisce per ogni città, in ordine di distanza, le località più vicine. A causa del movimento vincolato verso Est, dopo aver ottenuto come output una lista di città, è richiesta la valutazione della longitudine: la longitudine della città di partenza deve sempre essere inferiore alla città di destinazione. 
Una criticità da non trasurare è la seguente: se il metodo I richiede un tempo di calcolo asintoticamente pari a 𝜭(n² log₂(n)), dove n è il numero di città nel DataFrame; il II è molto più lento qualora si ponesse in KNeighbors K=number_city-1, cioè tutte le città, esclusa la città in esame, venissero ordinate per distanza di Haversine dalla stessa. E' plausibile ridurre K in modo ragionevole, tuttavia la semplificazione rimane ragionevole per il DataFrame corrente e per uno più ricco di quello corrente. 


III-- Il terzo meccanismo sfrutta l'intuzione di dividere il Mondo 2-dimensionale tramite una griglia di 180x180 rettangoli e aggiudicare ogni città ad una sola cella di tale griglia. 
La griglia assume la struttura dati di un dizionario a doppia entrata, dove la prima chiave indica il numero della cella sulla latitudine e la seconda chiave è in riferimento al numero del rettangolo sulla longitudine. 
Il valore associato al doppio indice è la lista degli indici delle città lì posizionate. 
L'idea è quella di ridurre il calcolo delle distanze di Haversine tra la città i-esima e le circostanti, racchiuse in una cornice verso Est. Se la cornice non contenesse tre città utili, cioè con longitudine maggiore della città i-esima, viene ampliata progressivamente. Ad ogni iterazione vengono, inoltre, valutate anche tutte le città che risiedono in una cornice di riserva attorno a quella selezionata dalla città i-esima. 
Mediamente, la funzione del calcolo delle distante di Haversine lavora con la città in valutazione e un array con meno di 200 città; risultato soddisfacente rispetto alla media di 13000 città del metodo I. 


## Algoritmo my_Dijkstra

L'algoritmo di Dijkstra è un algoritmo utilizzato per cercare i cammini minimi in un grafo con pesi non negativi sugli archi. Tale algoritmo trova applicazione nell'ottimizzazione nella realizzazione di reti idriche, telecomunicazioni, stradali e  circuitali. 

L'algoritmo <<my_Dijkstra>> prevede l'utilizzo di una lista che raccoglie la soluzione tramite gli indici delle città raggiunte, da Londra a Londra. Chiama la funzione <<my_path>> per risalire a tali indici tramite il vettore prodotto dall'algoritmo <<my_Dijkstra>>, che in corrispondenza di ogni nodo, indicizzato da 0 a number_city-1, ne scrive il predecessore, cioè il nodo tramite il quale è stato raggiunto durante il percorso. 

Segue il calcolo dei tempi, che indipendentemente dal metodo di riempimento che si adotta, è sempre inferiore alla metà del tempo a disposizione, cioè inferiore a 40 giorni. 

E' interessante come, dal confronto tra l'algoritmo <<my_Dijkstra>> e l'algoritmo di Dijkstra implementato da Python, il tempo di percorrenza è identico, segno della correttezza della nostra implementazione. 

## Grafici

Il codice termina con la presentazione del Mondo 2-dimensionale con l'aggiunta, dapprima di tutte le città nel DataFrame, poi delle sole raggiunte dal viaggio con senso di percorrenza; infine delle sole città valicate con relativo nome nel flag. 

