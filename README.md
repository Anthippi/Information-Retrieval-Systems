# Σύστημα Ανάκτησης Πληροφοριών. <a name="greek"></a> [![Static Badge](https://img.shields.io/badge/English-orange)](README.en.md)
Το σύστημα στοχεύει στην αναζήτηση επιστημονικών περιλήψεων από Ευρωπαϊκά Ερευνητικά Έργα (EU Research Projects), χρησιμοποιώντας τόσο κλασικές (Lexical) όσο και σύγχρονες (Semantic) μεθόδους.

### Φάση 1: Κλασική Ανάκτηση ([`phase1_elastic.ipynb`](https://github.com/Anthippi/Information-Retrieval-Systems/blob/main/notebooks/phase1_elastic.ipynb))
* **Σκοπός:** Υλοποίηση του Baseline συστήματος χρησιμοποιώντας **Elasticsearch** και τον αλγόριθμο **BM25**.
* **Λειτουργίες:**
    * Προεπεξεργασία κειμένου (Tokenization, Stemming, Lemmatization, Stopword Removal).
    * Δημιουργία ευρετηρίου (Indexing) στο Elasticsearch.
    * Εκτέλεση Boolean ερωτημάτων για βελτιστοποίηση Ακρίβειας/Ανάκλησης.
    * Αξιολόγηση αποτελεσμάτων (MAP, Precision@k, Recall@k).
```mermaid
graph TD
    %% Input
    Docs["Documents"]
    Queries["Queries"]

    %% Process
    subgraph Preprocessing ["Text Processing"]
        Clean["Cleaning & Stemming"]
    end

    subgraph Search_Engine ["Search Engine"]
        Index[("Elasticsearch Index")]
        Algorithm["BM25 Algorithm"]
    end

    %% Output
    Results["Ranked Results"]

    %% Flow
    Docs --> Clean --> Index
    Queries --> Clean --> Algorithm
    Index -.-> Algorithm
    Algorithm --> Results

    %% Styles
    style Preprocessing fill:#e6fcf5,stroke:#0c9,color:#000
    style Search_Engine fill:#fff4e6,stroke:#f66,color:#000
    style Results fill:#e6e6ff,stroke:#333,color:#000
```
### Φάση 2: Σημασιολογική Ανάκτηση ([`phase2_minilm.ipynb`](https://github.com/Anthippi/Information-Retrieval-Systems/blob/main/notebooks/phase2_minilm.ipynb))
* **Σκοπός:** Υλοποίηση συστήματος Dense Retrieval χρησιμοποιώντας **Transformers** και **FAISS**.
* **Λειτουργίες:**
    * Χρήση του προ-εκπαιδευμένου μοντέλου `all-MiniLM-L6-v2` για τη δημιουργία Embeddings (διανυσμάτων).
    * Αναζήτηση ομοιότητας συνημιτόνου (Cosine Similarity) σε διανυσματικό χώρο.
    * Αξιολόγηση της ικανότητας του μοντέλου να αντιλαμβάνεται το νόημα χωρίς λέξεις-κλειδιά.
```mermaid
graph TD
    %% Input Data
    Docs["Documents"]
    Queries["Queries"]

    %% Step 1: Preprocessing
    subgraph Prep ["Data Preparation"]
        Clean["Text Cleaning (Python)"]
    end

    %% Step 2: Vectorization
    subgraph Model ["AI Vectorization (BERT)"]
        Encoder["Transformer Model (MiniLM)"]
        Vectors["Dense Vectors (Embeddings)"]
    end

    %% Step 3: Search
    subgraph Search ["Similarity Search (FAISS)"]
        Index["FAISS Index"]
        Sim["Cosine Similarity Calculation"]
    end

    %% Output
    Results["Top-k Semantic Results"]

    %% Flow Connections
    Docs --> Clean
    Queries --> Clean
    
    Clean --> Encoder
    Encoder --> Vectors
    
    Vectors --> Index
    Index --> Sim
    Sim --> Results

    %% Styles (Pastel Colors + Black Text)
    style Prep fill:#e6fcf5,stroke:#0c9,color:#000
    style Model fill:#ffffcc,stroke:#cccc00,color:#000
    style Search fill:#e6e6ff,stroke:#333,color:#000
    style Vectors fill:#fff,stroke:#333,color:#000
```
### Φάση 3: Υβριδική Ανάκτηση ([`phase3_hybrid.ipynb`](https://github.com/Anthippi/Information-Retrieval-Systems/blob/main/notebooks/phase3_hybrid.ipynb))
* **Σκοπός:** Συνδυασμός των δύο προηγούμενων μεθόδων (Hybrid Approach - Retrieve & Re-rank).
* **Λειτουργίες:**
    * **Retrieve:** Ανάκτηση των top-200 υποψηφίων εγγράφων από το Elasticsearch (για υψηλό Recall).
    * **Re-rank:** Επανακατάταξη των υποψηφίων με χρήση BERT Embeddings (για υψηλό Precision).
    * **Fusion:** Τελική συγχώνευση των κατατάξεων με τον αλγόριθμο **Reciprocal Rank Fusion (RRF)**.
```mermaid
graph TD
    %% --- INPUT ---
    RawDocs[("Raw Documents")]
    RawQueries[("Raw Queries")]

    %% --- STEP 0: SHARED CLEANING ---
    subgraph Preprocessing ["Data Cleaning (Python)"]
        CleanFunc[("clean_text Function")]
        CleanDocs["Cleaned Documents"]
        CleanQueries["Cleaned Queries"]
        
        CleanFunc --> CleanDocs
        CleanFunc --> CleanQueries
    end

    %% --- STEP 1: ELASTICSEARCH ---
    subgraph Stage1 ["Retrieval (Elasticsearch)"]
        Elastic["Elasticsearch Index (BM25)"]
        Candidates["Top-200 Candidate IDs"]
    end

    %% --- STEP 2: BERT RE-RANKING ---
    subgraph Stage2 ["Re-ranking (BERT Model)"]
        Filter["Filter: Keep only Top-200 Texts"]
        BERT["BERT Model (MiniLM)"]
        SemScores["Semantic Similarity Scores"]
    end

    %% --- STEP 3: FUSION ---
    subgraph Stage3 ["Fusion (RRF)"]
        RRF["Reciprocal Rank<br/>Fusion Algorithm"]
    end

    %% --- OUTPUT ---
    Final["Final Top-50 Results"]

    %% --- FLOW CONNECTIONS ---
    
    %% Cleaning Flow
    RawDocs --> CleanFunc
    RawQueries --> CleanFunc

    %% Stage 1 Flow
    CleanDocs --> Elastic
    CleanQueries --> Elastic
    Elastic --> Candidates

    %% Stage 2 Flow
    Candidates --> Filter
    CleanDocs --> Filter
    
    Filter -- "Text of Top-200" --> BERT
    CleanQueries -- "Text of Query" --> BERT
    
    BERT --> SemScores

    %% Stage 3 Flow
    Candidates -- "Rank A (Lexical)" --> RRF
    SemScores -- "Rank B (Semantic)" --> RRF
    
    RRF --> Final

    %% --- STYLING ---
    style Preprocessing fill:#e6fcf5,stroke:#0c9,color:#000
    style Stage1 fill:#fff4e6,stroke:#f66,color:#000
    style Stage2 fill:#ffffcc,stroke:#cccc00,color:#000
    style Stage3 fill:#99ccff,stroke:#003366,color:#000
    style Final fill:#fff,stroke:#333,color:#000
```
---

## Προαπαιτούμενα & Εγκατάσταση

Για να εκτελέσετε τα notebooks, χρειάζεστε τα εξής:

* **Python 3.8+**
* **Elasticsearch** (Τοπική εγκατάσταση): Πρέπει να τρέχει στη θύρα `9200`.
* **trec_eval**: Το εκτελέσιμο αρχείο για την αξιολόγηση (συνήθως τοποθετείται σε φάκελο `../../trec_eval/`).

### Εγκατάσταση Βιβλιοθηκών Python
Εκτελέστε την παρακάτω εντολή για να εγκαταστήσετε τα απαραίτητα πακέτα:

```bash
pip install pandas numpy elasticsearch sentence-transformers faiss-cpu nltk scikit-learn matplotlib
