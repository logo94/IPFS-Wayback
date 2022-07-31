[![image](https://github.com/oduwsdl/ipwb/raw/master/docs/logo_stroked_400px.png)](https://pypi.python.org/pypi/ipwb)

# InterPlanetary Wayback (ipwb)

> Per il download e l'utilizzo del programma si rimanda al repository ufficiale https://github.com/oduwsdl/ipwb 

L’archiviazione delle risorse che circolano su internet avviene per mezzo di strumenti specifici denominati web crawlers. 
I web crawlers, o bot, da un dato indirizzo HTTP eseguono delle istantanee dinamiche di ogni link attraverso cui passano, generando dei file WARC (Web ARChive File Format), un formato contenitore appositamente progettato per la conservazione a lungo termine di questa tipologia di risorse.

Un file WARC, mediamente della dimensione di 1GB, si presenta come una concatenazione di record WARC: ogni record WARC è composto da tre elementi, ovvero un WARC header, un HTTP header e un HTTP payload.

All’interno del WARC header sono raccolte le informazioni relative al tipo di contenuto, data e motivo di raccolta, indirizzo IP del dispositivo da cui è stato lanciato l’harvesting, codifica del contenuto, codice identificativo della transazione ed eventuali conversioni o modifiche apportate al file. 
I blocco di contenuto è suddiviso in un HTTP header, contenente lo stato della risposta HTTP, e HTTP payload  e all'interno del quale sono riportati i contenuti delle pagine web:

![image](https://github.com/logo94/IPFS-Wayback/blob/master/docs/HTMLtoWARC.png)

Per la conservazione a lungo termine dei file WARC la prassi ne prevede il deposito all’interno di repositories certificati e il replay dei contenuti può avvenire sia per mezzo della Open Wayback Machine di Internet Archive che tramite Wayback Machines installate localmente. 

Se da un lato la scelta di affidare l’archiviazione dei propri archivi digitali a depositi certificati o servizi in cloud può essere una scelta forzata, soprattutto per chi non sia in possesso delle risorse adeguate per il mantenimento di un sistema di conservazione e replay locale, dall’altro apre la strada ad una serie di minacce. 

Il primo rischio è rappresentato dalla mole crescente di dati da archiviare e lo sforzo economico necessario per garantire la scalabilità dell’azione conservativa; l’esponenziale incremento della quantità di dati prodotti e pubblicati ogni giorno sul web richiede infatti capacità hardware e tecniche sempre più onerose, il rischio è quindi quello di non disporre delle risorse economiche sufficienti a rinnovare i contratti con i fornitori dei servizi. 

Un ulteriore fattore di rischio è rappresentato dal numero limitato di copie di backup messe a disposizione dai fornitori di servizi di storage, normalmente non superiore a una o due copie depositate in server remoti, aprendo la strada al ‘single point of failure’ in cui errore umano, malfunzionamenti o problemi relativi al sistema dei nomi di dominio (DNS) possono portare alla perdita definitiva dei dati o dell’accesso agli stessi.

# IPFS Wayback

InterPlanetary Wayback (ipwb) è un software basato su Python pywb (Core Web Archive Toolkit) che permette la gestione e la riproduzione dei file WARC all’interno di IPFS (InterPlanetary File System): protocollo per lo scambio di dati all'interno di una rete di computer che mira a connettere tutti i dispositivi computazionali tramite un unico sistema di file. IPFS introduce un sistema alternativo per l’identificazione e la gestione di dati, tale modalità prende il nome di [content-based addressing](https://github.com/logo94/IPFS-Wayback/wiki/Content-based-addressing): le risorse invece che essere individuate attraverso il proprio URL (Uniform Resource Locator) e quindi la propria posizione all’interno della rete, vengono identificate attraverso un codice identificativo univoco, desunto direttamente dal contenuto di una risorsa digitale, che prende il nome di [Content Identifier](https://github.com/logo94/IPFS-Wayback/wiki/Content-based-addressing#cid---content-identifier) (o CID). Il CID di blocchi equivalenti rimarrà invariato identificando con lo stesso link tutti i contenuti uguali; mentre la modifica del contenuto porterà sempre alla generazione di un CID diverso, garantendo così l’autenticità e l’integrità delle risorse.

A differenza del sistema client-server utilizzato oggi su internet, all'interno di IPFS la comunicazione avviene per mezzo di un’architettura peer-to-peer in cui ogni dispositivo dotato di connessione a internet può operare come nodo della rete. ll funzionamento del sistema peer-to-peer è gestito automaticamente da [Libp2p](https://github.com/logo94/IPFS-Wayback/wiki/Libp2p), uno stack di rete modulabile in grado di gestire le diverse fasi della comunicazione tra i nodi, dalla negoziazione dei protocolli di trasporto allo scambio dei pacchetti.

L’identificazione di ogni nodo avviene per mezzo dell’impronta digitale della sua chiave pubblica (vedi [Peer Identity](https://github.com/logo94/IPFS-Wayback/wiki/Libp2p#peerid---peer-identity)), un Peer ID invia quindi alla rete una richiesta per ottenere un contenuto specifico (CID): l’instradamento delle richieste avviene per mezzo di tabelle distribuite (vedi [Distributed Hash Table](https://github.com/logo94/IPFS-Wayback/wiki/Libp2p#routing-instradamento)) tra i nodi che al loro interno contengono coppie di CID e dei Peer ID in loro possesso. Tramite l’interrogazione di queste tabelle è quindi possibile risalire ai nodi che ospitano i contenuti richiesti. 

Prima dell’invio della richiesta avviene una negoziazione tra protocolli per stabilire quelli supportati dai nodi coinvolti nello scambio. Una volta stabilito il percorso e il protocollo per il corretto dialogo tra nodi viene inviata in rete la richiesta per uno specifico contenuto. I peers in possesso dei blocchi richiesti codificano il contenuto con la chiave pubblica del peer che ha eseguito la richiesta e inviano una risposta, in questo modo solo il nodo in possesso della chiave privata corretta potrà decodificare correttamente il contenuto dei blocchi ricevuti. Lo scambio tra nodi può avvenire sia in modo diretto tra mittente e destinatario, che indiretto utilizzando altri nodi come tramite. 

Ogni volta che un nodo ottiene una risorsa, una copia della stessa viene salvata sotto forma di file cache a livello locale; tramite l’utilizzo di Pinset i nodi possono gestire automaticamente o manualmente il mantenimento o la cancellazione dei file da loro posseduti, selezionando quali file salvare e quali invece scartare. Nel momento in cui un nodo seleziona la risorsa per il mantenimento locale, PeerID del nodo e CID vengono pubblicati all’interno della DHT.

Anche qualora il nodo che ha caricato la risorsa all’interno di IPFS si disconnettesse, fintanto che i nodi della rete saranno in possesso del numero sufficiente di dati necessari a ricomporre integralmente un file, l’accesso ai contenuti non verrà compromesso.
Un sistema di conservazione basato su IPFS è quindi in grado di garantire un adeguato fattore di replicazione delle risorse mentre l’accesso alle risorse è garantito dalla comunicazione diretta tra nodi, senza la necessità di passare per il sistema dei nomi di dominio, quindi scongiurando i rischi di isolamento o censura passibili di cambiamenti di ordine economico o politico.

Per il suo funzionamento InterPlanetary Wayback dispone di due script: [indexer.py](https://github.com/logo94/IPFS-Wayback/wiki#indicizzazione) e [replay.py](https://github.com/logo94/IPFS-Wayback/wiki#replay).

### Indexer.py

Lo script indexer.py permette di estrarre direttamente dal WARC Store (archivio locale di file .warc) un record WARC alla volta, divide HTTP header e HTTP payload dei record WARC e carica le due parti all’interno della rete IPFS.
 
Al momento dell’upload nella rete i file vengono suddivisi in blocchi di dati, a ciascuno dei quali viene attribuito un codice identificativo univoco, desunto direttamente dal proprio contenuto, che prende il nome di Content Identifier (o CID).

![image](https://github.com/logo94/IPFS-Wayback/blob/master/docs/IPFS-upload.png)

Una volta caricati i file WARC all’interno di IPFS e ottenuti i CIDs di HTTP Header e HTTP Payload di ogni risorsa, viene generato un file CDXJ contenente i metadati necessari per recuperare e riprodurre il contenuto. 

Un record CDXJ appare quindi come:

![image](https://github.com/logo94/IPFS-Wayback/blob/master/docs/CDXJ-example.png)

### Replay.py

Replay.py è la componente di InterPlanetary Wayback che si occupa del recupero e della riproduzione dei file WARC attraverso web browser; gli utenti possono accedere alle risorse archiviate per mezzo del loro URL che può essere inserito direttamente all’interno della maschera di ricerca della Wayback Machine (vedi immagine): 

![image](https://github.com/logo94/IPFS-Wayback/blob/master/docs/browser-snapshot.png)

In seguito ad una richiesta il software interroga l’indice CDXJ all’interno del quale sono contenuti i CIDs di HTTP header e HTTP payload della risorsa desiderata, vengono interrogati i nodi della rete IPFS per ottenere i contenuti desiderati, una volta ottenuti  replay.py ricompone il file WARC e ne riproduce il contenuto.

L’intero processo di funzionamento di ipwb può essere rappresentato con il seguente schema:

![image](https://github.com/logo94/IPFS-Wayback/blob/master/docs/diagram_72.png)

in cui i numeri in rosso rappresentano il processo di inserimento e indicizzazione:
1. Indexer.py estrae HTTP header e payload dele risorse web contenute in un file WARC;
2. [3-4-5] HTTP header e HTTP payload vengono caricati all’interno della rete IPFS, il processo porta alla generazione dei CIDs corrispettivi;

6. Viene generato un indice CDXJ che riporta le coppie di CID relativi a HTTP header e HTTP payload di ogni risorsa insieme ad altre informazioni per il recupero e il replay; 

mentre i numeri in blu mostrano il processo di recupero e replay dei contenuti:
1. L’utente richiede un contenuto attraverso il suo URL;
2. Replay.py interroga l’indice CDXJ per sapere quali CID sono necessari per ricomporre il contenuto richiesto;
3. Vengono individuati i CIDs di HTTP header e HTTP payload della risorsa richiesta;
4. [5-6-7] il software inoltra una richiesta alla rete per ottenere i contenuti desiderati; i nodi in possesso di quei contenuti inviano i dati al nodo che ha eseguito la richiesta;

8. HTTP header e HTTP payload vengono riassemblati e riprodotti tramite Wayback Machine.

L’integrazione di IPFS Wayback con IPFS cluster permette di garantire un’adeguata ridondanza dei dati, non più limitata a una o due copie di backup, mentre la possibilità di creare una rete privata, all’interno della quale solo i nodi in possesso in possesso dell’apposita chiave possono inviare e ricevere dati, offre la possibilità di creare un sistema di conservazione e accesso completamente distribuito, scalabile e resistente alla censura.


### Citazione progetto

> Mat Kelly, Sawood Alam, Michael L. Nelson, and Michele C. Weigle. __InterPlanetary Wayback: Peer-To-Peer Permanence of Web Archives__. In _Proceedings of the 20th International Conference on Theory and Practice of Digital Libraries_, pages 411–416, Hamburg, Germany, June 2016.

```bib
@INPROCEEDINGS{ipwb-tpdl2016,
  AUTHOR    = {Mat Kelly and
               Sawood Alam and
               Michael L. Nelson and
               Michele C. Weigle},
  TITLE     = {{InterPlanetary Wayback}: Peer-To-Peer Permanence of Web Archives},
  BOOKTITLE = {Proceedings of the 20th International Conference on Theory and Practice of Digital Libraries},
  PAGES     = {411--416},
  MONTH     = {June},
  YEAR      = {2016},
  ADDRESS   = {Hamburg, Germany},
  DOI       = {10.1007/978-3-319-43997-6_35}
}
```

## Licenza

MIT
