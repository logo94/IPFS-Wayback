[![image](https://github.com/oduwsdl/ipwb/raw/master/docs/logo_stroked_400px.png)](https://pypi.python.org/pypi/ipwb)

# InterPlanetary Wayback (ipwb)

> Per il download e l'utilizzo del programma si rimanda al repository ufficiale https://github.com/oduwsdl/ipwb

> Per consultare la documentazione vedere sezione [wiki](https://github.com/logo94/IPFS-Wayback/wiki)

## Installazione

InterPlanetary Wayback (ipwb) utilizza Python 3.7+. In alternativa è disponibile l'immagine Docker (vedi [Sezione dedicata](https://github.com/logo94/IPFS-Wayback/wiki#utilizzo-tramite-docker)).

L'installazione tradizionale può essere effettuata per mezzo del comando pip:
```
$ pip install ipwb
```
La versione di sviluppo contenente gli aggiornamenti non ancora pubblicati ufficilamente può essere invece essere installata con i seguenti comandi:
```
$ git clone https://github.com/oduwsdl/ipwb
$ cd ipwb
$ pip install ./
```

## Prerequisiti
Per il corretto funzionamento di ipwb deve essere installato e in funzione IPFS daemon. Per l'istallazione di IPFS vedere la [pagina ufficiale](https://docs.ipfs.tech/install/).

Una volta installato IPFS sul proprio dispositivo per avviare il programma eseguire:
```
$ ipfs daemon
```
Qualora dovessero risultare conflitti con la porta API 5001, prima di avviare il demone è necessario modificare le impostazioni di IPFS affinchè sia dato l'accesso a una porta a propria scelta (nell'esempio seguente la porta 5002):
```
$ ipfs config Addresses.API /ip4/127.0.0.1/tcp/5002
```

## Indicizzazione
In un nuovo terminale rispetto a quello utilizzato per avviare ipfs daemon (o lo stesso qualora il demone sia stato avviato in background), per caricare i file WARC all'interno di IPFS e creare un indice CDXJ, eseguire il comando:
```
$ ipwb index (path del file warc oppure della versione compressa warc.gz)
```
Per esempio, utilizzando il campione di esempio del repository:
```
$ ipwb index samples/warcs/salam-home.warc
```
L'indice CDXJ generato dal comando verrà mostrato a terminale, per salvarlo all'interno di un file sarà sufficiente aggiungere il comando:
```
$ ipwb index samples/warcs/salam-home.warc >> NomeFile.cdxj
```

## Replay
Per il replay di un file WARC è necessario fornire a ipwb un indice CDXJ in modo da permettere al programma di ricomporre HTTP header e HTTP payload in un unico file. Per far ciò tra i parametri del comando deve essere riportato il path del file:
```
$ ipwb replay <path cdxj>
```
Qualora il file CDXJ sia disponibile online, è possibile passare al programma direttamente l'inidirizzo HTTP della risorsa:
```
$ ipwb replay http://myDomain/files/myIndex.cdxj
```
oppure, se conosciuto, il CID del file CDXJ:
```
$ ipwb replay QmYwAPJzv5CZsnANOTaREALhashYgPpHdWEz79ojWnPbdG
```
Una volta eseguito il comando la versione archiviata della risorsa sarà visibile tramite web browser all'indirizzo `http://localhost:2016` (o alla porta scelta)

Per eseguire il comando ad un indirizzo diverso da localhost (127.0.0.1) è necessario utilizzare un reverse proxy che supporti il protocollo HTTPS.

Per permettere a ipwb di riconoscere il proxy è necessario includere all'interno del comando i parametri `--proxy` o `-P` seguiti dall'URL del proxy, come riportato nel seguente esempio:
```
$ ipwb replay --proxy=https://ipwb.example.com <path cdxj>
```

## Utilizzo tramite Docker
Per l'utilizzo tramite Docker è disponibile un'immagine precompilata, eseguibile con il comando:
```
$ docker container run -it --rm -p 2016:2016 oduwsdl/ipwb
```
Il container avvia un'istanza IPFS daemon, esegue l'inidicizzazione di un file WARC d'esempio e ne esegue il replay utilizzando il file CDXJ creato. Dopo pochi secondi il replay sarà accesssibile tramite web browser all'indirizzo `hhtp://localhost:2016/`.

Per indicizzare e riprodurre file WARC custoditi localmente è necessario eseguire all'interno del container il muonting della cartella in cui sono contenuti tramite il parametro -v (o --volume) insieme ai relativi comandi. L'immagine Docker fa riferimento alla cartella di defualt `/data` in cui sono riportate le cartelle `warc`, `cdxj` e `ipfs`. Per passare i propri file WARC è sufficiente eseguire il muont della relativa cartella all'interno della directory `warc`.

Per la documentazione completa sull'utilizzo di Docker si rimanda al [repository ufficiale](https://github.com/oduwsdl/ipwb#using-docker).

## Elenco comandi
La lista di tutti i comandi supportati dal programma può essere ottenuta per mezzo del parametro `-h` oppure `--help`:
```
$ ipwb -h
utilizzo: ipwb [-h] [-d DAEMON_ADDRESS] [-v] [-u] {index,replay} ...

InterPlanetary Wayback (ipwb)

argomenti opzionali:
  -h, --help            stampa a terminale il messaggio di aiuto
  -d DAEMON_ADDRESS, --daemon DAEMON_ADDRESS
                        Multi-address del demone IPFS (default
                        /dns/localhost/tcp/5001/http)
  -v, --version         stampa a terminale la versione del software
  -u, --update-check    verifica disponibilità di eventuali aggiornamenti

comandi:
  Chiamata per l'esecuzione del programma: "ipwb <command>" (per esempio ipwb replay <cdxjFile>)

  {index,replay}
    index               Indicizza un file WARC per il replay tramite ipwb
    replay              Avvia la Wayback Machine tramite Web browser

```
```
$ ipwb index -h
utilizzo: ipwb [-h] [-e] [-c] [--compressFirst] [-o OUTFILE] [--debug]
            index <warc_path> [index <warc_path> ...]

Indicizza un file WARC file per il replay tramite ipwb

argomenti spaziali:
  index <warc_path>      Path al file WARC[.gz]

argomenti opzionali:
  -h, --help            stampa a terminale il messaggio di aiuto
  -e                    Crittografare il contenuto WARC prima di aggiungerlo a IPFS
  -c                    Comprimere il contenuto WARC prima di aggiungerlo a IPFS
  --compressFirst       Comprimere i dati prima della crittografia, quando applicabile
  -o OUTFILE, --outfile OUTFILE
                        Path al file CDXJ di output, defaults to STDOUT
  --debug               Parametro per facilitare il test e il DEBUG
```
```
$ ipwb replay -h
utilizzo: ipwb replay [-h] [-P [<host:port>]] [index]

Avvia la Wayack Machine per un utilizzo tramite web browser

argomenti spaziali:
  index                 path, URI, o multihash del file da riprodurre

optional arguments:
  -h, --help            stampa a terminale il messaggio di aiuto
  -P [<host:port>], --proxy [<host:port>]
                        Proxy URL

```


## Licenza

MIT
