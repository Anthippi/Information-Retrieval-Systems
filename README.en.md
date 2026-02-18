# Information Retrieval System
The system aims to search scientific abstracts from EU Research Projects, utilizing both classical (Lexical) and modern (Semantic) methods.

### Phase 1: Classical Retrieval ([`phase1_elastic.ipynb`](https://github.com/Anthippi/Information-Retrieval-Systems/blob/main/notebooks/phase1_elastic.ipynb))
* **Goal:** Implementation of the Baseline system using Elasticsearch and the BM25 algorithm.
* **Features:**
  * Text Preprocessing (Tokenization, Stemming, Lemmatization, Stopword Removal).
  * Indexing in Elasticsearch.
  * Execution of Boolean queries to optimize Precision/Recall.
  * Result Evaluation (MAP, Precision@k, Recall@k).
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
### Phase 2: Semantic Retrieval ([`phase2_minilm.ipynb`](https://github.com/Anthippi/Information-Retrieval-Systems/blob/main/notebooks/phase2_minilm.ipynb))
* **Goal:** Implementation of a Dense Retrieval system using Transformers and FAISS.
* **Features:**
  * Utilization of the pre-trained `all-MiniLM-L6-v2` model for generating Embeddings (vectors).
  * Cosine Similarity search in vector space.
  * Evaluation of the model's ability to perceive meaning without exact keywords.
```mermaid
graph TD
    %% Input Data
    Docs["Documents"]
    Queries["Queries"]

    %% Step 1: Preprocessing
    subgraph Prep ["1. Data Preparation"]
        Clean["Text Cleaning (Python)"]
    end

    %% Step 2: Vectorization
    subgraph Model ["2. AI Vectorization (BERT)"]
        Encoder["Transformer Model (MiniLM)"]
        Vectors["Dense Vectors (Embeddings)"]
    end

    %% Step 3: Search
    subgraph Search ["3. Similarity Search (FAISS)"]
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
### Phase 3: Hybrid Retrieval ([`phase3_hybrid.ipynb`](https://github.com/Anthippi/Information-Retrieval-Systems/blob/main/notebooks/phase3_hybrid.ipynb))
* **Goal:** Combination of the two previous methods (Hybrid Approach - Retrieve & Re-rank).
* **Features:**
  * Retrieve: Retrieval of top-200 candidate documents from Elasticsearch (for High Recall).
  * Re-rank: Re-ranking of candidates using BERT Embeddings (for High Precision).
  * Fusion: Final merging of rankings using the Reciprocal Rank Fusion (RRF) algorithm.
```mermaid
graph TD
    %% --- INPUT ---
    RawDocs[("Raw Documents")]
    RawQueries[("Raw Queries")]

    %% --- STEP 0: SHARED CLEANING ---
    subgraph Preprocessing ["0. Data Cleaning (Python)"]
        CleanFunc[("clean_text Function")]
        CleanDocs["Cleaned Documents"]
        CleanQueries["Cleaned Queries"]
        
        CleanFunc --> CleanDocs
        CleanFunc --> CleanQueries
    end

    %% --- STEP 1: ELASTICSEARCH ---
    subgraph Stage1 ["1. Retrieval (Elasticsearch)"]
        Elastic["Elasticsearch Index (BM25)"]
        Candidates["Top-200 Candidate IDs"]
    end

    %% --- STEP 2: BERT RE-RANKING ---
    subgraph Stage2 ["2. Re-ranking (BERT Model)"]
        Filter["Filter: Keep only Top-200 Texts"]
        BERT["BERT Model (MiniLM)"]
        SemScores["Semantic Similarity Scores"]
    end

    %% --- STEP 3: FUSION ---
    subgraph Stage3 ["3. Fusion (RRF)"]
        RRF["Reciprocal Rank Fusion Algorithm"]
    end

    %% --- OUTPUT ---
    Final["Final Top-50 Results"]

    %% --- FLOW CONNECTIONS ---
    
    %% Cleaning Flow
    RawDocs --> CleanFunc
    RawQueries --> CleanFunc

    %% Stage 1 Flow (Uses Clean Data)
    CleanDocs --> Elastic
    CleanQueries --> Elastic
    Elastic --> Candidates

    %% Stage 2 Flow (Uses Clean Data + Candidates)
    Candidates --> Filter
    CleanDocs --> Filter
    
    Filter -- "Text of Top-200" --> BERT
    CleanQueries -- "Text of Query" --> BERT
    
    BERT --> SemScores

    %% Stage 3 Flow (Fusion)
    Candidates -- "Rank A (Lexical)" --> RRF
    SemScores -- "Rank B (Semantic)" --> RRF
    
    RRF --> Final

    %% --- STYLING (Pastel + Black Text) ---
    style Preprocessing fill:#e6fcf5,stroke:#0c9,color:#000
    style Stage1 fill:#fff4e6,stroke:#f66,color:#000
    style Stage2 fill:#ffffcc,stroke:#cccc00,color:#000
    style Stage3 fill:#99ccff,stroke:#003366,color:#000
    style Final fill:#fff,stroke:#333,color:#000
```
---
## Prerequisites & Installation
To run the notebooks, you need the following:
* **Python 3.8+**
* **Elasticsearch** (Local installation): Must be running on port `9200`.
* **trec_eval**: The executable file for evaluation (usually placed in a `../../trec_eval/` folder).

### Python Libraries Installation
Run the following command to install the necessary packages:

```Bash
pip install pandas numpy elasticsearch sentence-transformers faiss-cpu nltk scikit 
```
