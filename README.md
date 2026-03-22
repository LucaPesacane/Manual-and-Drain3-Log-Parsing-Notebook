# Log Parsing e Fault Injection in Kubernetes: Analisi del Dataset Mutiny

Questo repository contiene il framework di analisi sviluppato per l'elaborato di tesi triennale in Ingegneria Informatica. Il progetto si occupa del **log parsing** applicato a sistemi orchestrati con Kubernetes, con l'obiettivo di valutare la capacità di identificare anomalie derivanti da attività di **fault injection**.

## Descrizione del Progetto
L'analisi mette a confronto due diverse metodologie di estrazione di template (parsing) e valuta l'impatto di una fase di pre-processing sui risultati finali.

### Metodologie a confronto:
1.  **Analisi Manuale**: Basata su pattern euristici e regole predefinite.
2.  **Drain3**: Parser automatizzato basato su un algoritmo di clustering online (alberi a profondità fissa).

### Modalità di esecuzione:
* **Raw Mode**: I log vengono processati così come estratti dal sistema.
* **Preprocessed Mode**: I log subiscono una pulizia preventiva (rimozione di timestamp, indirizzi IP, variabili volatili e ID esadecimali) per ridurre il rumore e la cardinalità dei template.

## Organizzazione del Notebook

Il notebook è strutturato in due macro-sezioni principali. Sebbene la logica di parsing sia la stessa, esse rispondono a esigenze diverse del flusso di lavoro:

### 1. Analisi su Subset (Validazione e Statistica)
Questa prima parte processa un campione di **50 file** selezionati casualmente (usando `seed=42` per garantire la stabilità dei risultati).
* **Scopo**: Validare rapidamente l'efficacia delle espressioni regolari (regex) e dei parametri di Drain3.
* **Output**: Generazione immediata dei test statistici (**McNemar** e **Cohen's Kappa**) e delle metriche di performance utilizzate per la discussione dei risultati sperimentali (es. Capitolo 4 della tesi).

### 2. Analisi Full Dataset (Produzione Risultati)
Questa sezione applica il framework all'intero dataset Mutiny (circa **2893 file**). 
* **Scopo**: Ottenere una panoramica completa delle anomalie su larga scala.
* **Gestione Checkpoint**: Data la mole di dati e i tempi di esecuzione prolungati (~1 ora), questa sezione implementa un sistema di **checkpointing**. I risultati parziali vengono salvati nella directory `/content/checkpoints`, permettendo di riprendere l'analisi in caso di interruzione senza dover ricominciare da capo.

> **Nota Tecnica**: Le funzioni di preprocessing e le classi di parsing sono condivise. Se si apportano modifiche alla logica di pulizia dei log o ai parametri del parser, è bene testarle nella Sezione 1 prima di avviare l'esecuzione massiva nella Sezione 2.

## Personalizzazione e Parametri

Per adattare l'analisi a nuovi requisiti o per testare diverse configurazioni del parser, è possibile intervenire sui seguenti parametri all'interno del notebook:

### 1. Parametri del Parser Drain3
L'algoritmo **Drain3** può essere tarato modificando l'istanza di `TemplateMiner`. I parametri principali su cui agire sono:
* `sim_th` (Similarity Threshold): Definisce quanto due messaggi di log debbano essere simili per finire nello stesso cluster. Un valore più alto crea più template (più specifici), un valore più basso ne crea meno (più generici).
* `depth`: La profondità massima dell'albero di parsing.
* `max_children`: Numero massimo di nodi figli per ogni nodo dell'albero.

### 2. Preprocessing (Regex Masking)
Nella sezione dedicata al preprocessing, è presente una lista di tuple denominata `masking_instructions` (o similare). È possibile aggiungere o rimuovere espressioni regolari per:
* Mascherare nuove variabili (es. ID specifici di sessione, nomi di file, nuovi formati di timestamp).
* Ridurre il rumore: aggiungere maschere permette di raggruppare log che differiscono solo per parametri volatili, aumentando l'efficacia del clustering.

### 3. Configurazione del Dataset
Nella prima cella di setup, si possono modificare le variabili di controllo del flusso:
* `n_files`: Numero di file da includere nel subset di test (default: 50).
* `seed`: Il seme per la generazione casuale. Cambiando questo valore si testerà il modello su un set di file differente, utile per verificare la robustezza statistica dei risultati.
* `base_path`: Stringa del percorso principale del dataset (necessaria se si sposta il dataset su un'altra cartella di Drive o in locale).

### 4. Gestione Checkpoint
Per la sezione "Full Dataset", è possibile configurare la frequenza di salvataggio dei risultati parziali o la cartella di destinazione dei file `.pkl` o `.csv` generati durante l'elaborazione massiva, utile per prevenire perdite di dati in caso di disconnessione di Google Colab.

## How-to: Guida all'Utilizzo

### 1. Requisiti
Il codice è sviluppato in Python 3.12 ed è ottimizzato per l'esecuzione su **Google Colab**.
Le librerie necessarie sono:
```bash
pip install drain3 pandas numpy scikit-learn matplotlib
```

### 2. Configurazione del Dataset
Il notebook è configurato per interfacciarsi con Google Drive.

Caricare la cartella Mutiny_dataset 2 nel proprio Drive.

Il percorso atteso dal codice è: /content/drive/MyDrive/Mutiny_dataset 2/.

Il dataset deve essere organizzato in sottocartelle corrispondenti alle classi di test (es. baseline, less_resources, network, more_resources).

### 3. Esecuzione
Aprire il file logAnalysis.ipynb in Google Colab o Jupyter.

Campionamento: Il codice seleziona automaticamente un subset di 50 file casuali per velocizzare l'analisi, utilizzando un seed=42 per garantire che l'esperimento sia perfettamente riproducibile.

Eseguire le celle in sequenza: il notebook caricherà i file, eseguirà il parsing (manuale e Drain3) sia in modalità Raw che Preprocessed, e infine genererà i report.

## Analisi Statistica e Risultati
Il framework non si limita al calcolo delle metriche classiche, ma fornisce strumenti di validazione scientifica:

Metriche: Precision, Recall e F1-Score per ogni scenario di guasto.

Riduzione della Cardinalità: Calcolo della percentuale di riduzione dei cluster identificati da Drain3 grazie al preprocessing.

Test di McNemar: Validazione statistica per confrontare se le differenze di accuratezza tra i modelli sono significative.

Kappa di Cohen: Misura dell'accordo tra il parser e la baseline reale.

## Struttura del Repository

logAnalysis.ipynb: Notebook principale contenente l'intera pipeline di analisi.

README.md: Questo file di documentazione.
