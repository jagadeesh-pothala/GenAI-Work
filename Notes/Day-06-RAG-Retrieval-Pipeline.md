## Purpose of This Code 

This script:

1. Connects to an existing **Chroma vector database**
    
2. Loads the **same embedding model** used during ingestion
    
3. Converts the user question into an embedding
    
4. Retrieves the **most relevant document chunks**
    
5. Prints the context for answering the question
    

👉 This is the **retrieval step of RAG**.

---

## 1. Import Statements

`from langchain_chroma import Chroma`

- LangChain wrapper to interact with **ChromaDB**
    
- Used for searching vectors easily
    

---

`from langchain_huggingface import HuggingFaceEmbeddings`

- Loads the **embedding model**
    
- Must be the **same model** used during ingestion
    

---

`from dotenv import load_dotenv import chromadb import os`

- `load_dotenv()` → loads secrets from `.env`
    
- `chromadb` → native Chroma client
    
- `os` → access environment variables
    

---

`load_dotenv()`

- Loads:
    
    - `CHROMA_HOST`
        
    - `CHROMA_API_KEY`
        
    - `CHROMA_TENANT`
        
    - `CHROMA_DATABASE`
        

---

## 2. Load the Embedding Model

`embedding_model = HuggingFaceEmbeddings(     model_name="sentence-transformers/all-mpnet-base-v2" )`

What this does:

- Converts user questions into **embeddings**
    
- Ensures embeddings are in the **same vector space**
    
- Very important rule in RAG:
    
    > Same embedding model for **store** and **search**
    

---

## 3. Connect to ChromaDB (Remote)

`client = chromadb.HttpClient(     host=os.getenv("CHROMA_HOST"),     ssl=True,     headers={"x-chroma-token": os.getenv("CHROMA_API_KEY")},     tenant=os.getenv("CHROMA_TENANT"),     database=os.getenv("CHROMA_DATABASE") )`

This:

- Connects to **remote ChromaDB**
    
- Uses API key for authentication
    
- Selects the correct tenant and database
    

---

`print(f"Connecting to ChromaDB and embedding model...")`

- Log message for clarity during execution
    

---

## 4. Load the Existing Vector Store

`db = Chroma(     client=client,     collection_name="Embeddings",     embedding_function=embedding_model,     collection_metadata={"hnsw:space": "cosine"}   )`

What happens here:

- Connects to the **already-created collection**
    
- Uses cosine similarity for search
    
- Does NOT create embeddings again
    
- Only used for **retrieval**
    

---

## 5. User Query

`query = "What was the original name of Microsoft before it became Microsoft?"`

- This is the **user question**
    
- It will be converted into an embedding internally
    

---

## 6. Create a Retriever (MMR Search)

`retriever = db.as_retriever(     search_type="mmr",     search_kwargs={"k": 5, "fetch_k": 20} )`

### What is happening here?

- `as_retriever()` converts the vector store into a **retrieval engine**
    
- `search_type="mmr"` means **Maximum Marginal Relevance**
    

### Why MMR?

- Avoids duplicate or very similar chunks
    
- Returns **diverse but relevant** context
    

### Parameters:

- `fetch_k=20` → fetch top 20 candidates
    
- `k=5` → return best 5 results
    

---

## 7. Alternative Search (Commented)

`# search_type="similarity_score_threshold" # score_threshold=0.3`

This method:

- Returns only chunks with similarity ≥ 0.3
    
- Useful when you want **high-confidence answers**
    
- Avoids weak matches
    

---

## 8. Retrieve Relevant Documents

`relevant_docs = retriever.invoke(query)`

What this does internally:

1. Convert query → embedding
    
2. Compare with stored embeddings
    
3. Find closest matches
    
4. Return matching document chunks
    

---

## 9. Print the Results

`print(f"User Query: {query}")`

- Displays the question
    

---

`for i, doc in enumerate(relevant_docs, 1):     print(doc.page_content)`

- Prints retrieved **context**
    
- This context will later be:
    
    - Passed to an LLM
        
    - Used to generate the final answer