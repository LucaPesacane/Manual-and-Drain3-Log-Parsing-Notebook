# Log Analysis: Mutiny Dataset Parsing & Detection Framework

Questo repository contiene un framework di analisi dei log applicato a un subset del dataset **Mutiny** (log di Kubernetes). L'obiettivo principale è confrontare l'efficacia del parsing manuale rispetto a quello automatizzato tramite **Drain3**, analizzando come il pre-processing dei messaggi influenzi la capacità di rilevamento delle anomalie.

## Panoramica del Progetto

L'analisi esplora l'efficacia del log parsing in contesti di fault-injection (es. *less_resources*, *network faults*, ecc.) operando su due dimensioni:

### 1. Modalità di Analisi
* **Raw Mode**: I log vengono processati nel loro stato originale.
* **Preprocessed Mode**: I log subiscono una pulizia preventiva (rimozione di timestamp, ID esadecimali, indirizzi IP e variabili volatili) per stabilizzare i template.

### 2. Metodologie di Parsing
* **Analisi Manuale**: Basata su pattern euristici e regex definite dall'utente.
* **Drain3 (Automated)**: Un parser online basato su alberi a profondità fissa che raggruppa i log in cluster di template in modo dinamico.

---

## Caratteristiche Tecniche

* **Dataset**: Subset di 50 file estratti casualmente dal Mutiny Dataset (seed=42 per riproducibilità).
* **Preprocessing Pipeline**: Implementazione di filtri per la riduzione della cardinalità dei log.
* **Metrics Engine**: Calcolo automatico di TP, FP, FN, Precision, Recall e F1-Score per ogni categoria di test.
* **Validazione Statistica**: 
    * **Test di McNemar**: Per valutare la significatività della differenza tra Manual e Drain3.
    * **Kappa di Cohen**: Per misurare l'accordo sulla classificazione della baseline.

---

## Risultati e Performance

Il framework confronta le prestazioni attraverso diverse metriche. Di seguito i risultati tipici ottenuti dall'analisi:

| Metodo | Modalità | Recall | Precision | F1-Score | Cluster/Template |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Manual** | Preprocessed | ~0.615 | 1.000 | 0.762 | - |
| **Drain3** | Preprocessed | 1.000 | 1.000 | 1.000 | Ridotti del ~15-20% |
| **Manual** | Raw | 1.000 | 1.000 | 1.000 | - |
| **Drain3** | Raw | 1.000 | 1.000 | 1.000 | Alta cardinalità |

> **Nota**: Il preprocessing riduce drasticamente il numero di cluster (template) generati da Drain3, rendendo l'analisi più scalabile e meno soggetta a rumore.

---

## Requisiti e Installazione

Il progetto è sviluppato in Python 3.12+. Per eseguire il notebook, installa le dipendenze necessarie:

```bash
pip install drain3 pandas numpy scikit-learn matplotlib
