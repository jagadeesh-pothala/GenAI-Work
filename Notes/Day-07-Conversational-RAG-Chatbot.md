
This code builds a **full conversational RAG chatbot** that:

1. Takes a user question
    
2. Converts it into embeddings
    
3. Searches relevant documents from **Chroma Vector DB (via REST API)**
    
4. Sends retrieved context to an **LLM (Groq – LLaMA 3.3)**
    
5. Generates an answer
    
6. Maintains **chat history** for follow-up questions
    

---

## PART 1: Common Setup (Environment & Config)

`load_dotenv()`

- Loads environment variables from `.env`
    
- Keeps secrets secure (API keys, DB details)
    

---

`CHROMA_HOST = os.getenv("CHROMA_HOST") CHROMA_API_KEY = os.getenv("CHROMA_API_KEY") CHROMA_TENANT = os.getenv("CHROMA_TENANT") CHROMA_DATABASE = os.getenv("CHROMA_DATABASE") COLLECTION_NAME = "Embeddings"`

- Configuration for **ChromaDB**
    
- Points to the same collection used during ingestion
    

---

`if not CHROMA_HOST.startswith("http"):     BASE_URL = f"https://{CHROMA_HOST}/api/v2"`

- Ensures correct REST API base URL
    
- Supports hosted Chroma (like `api.trychroma.com`)
    

---

`headers = {     "x-chroma-token": CHROMA_API_KEY,     "x-chroma-tenant": CHROMA_TENANT,     "x-chroma-database": CHROMA_DATABASE,     "Content-Type": "application/json" }`

- Authentication + multi-tenancy headers
    
- Required for Chroma REST API calls
    

---

## PART 2: Embeddings Initialization

`embedding_model = HuggingFaceEmbeddings(     model_name="sentence-transformers/all-mpnet-base-v2" )`

What this does:

- Converts text → numbers (embeddings)
    
- **Same model must be used** for:
    
    - Document ingestion
        
    - Query search
        

👉 This keeps everything in the **same vector space**

---

## PART 3: Fetching Collection ID (Important for REST API)

`collections_url = f"{BASE_URL}/tenants/{CHROMA_TENANT}/databases/{CHROMA_DATABASE}/collections"`

- REST endpoint to list all collections
    

---

`resp = requests.get(collections_url, headers=headers) collections = resp.json()`

- Fetches available collections from ChromaDB
    

---

`for col in collections:     if col['name'] == COLLECTION_NAME:         collection_id = col['id']`

- Finds the internal **collection ID**
    
- Required because REST API queries use **collection ID**, not name
    

---

## PART 4: Simple RAG Query (Single Question Flow)

`query = "How much did Microsoft pay to acquire GitHub?"`

- User’s question
    

---

`query_embedding = embedding_model.embed_query(query)`

- Converts the question into an embedding
    

---

`query_payload = {     "query_embeddings": [query_embedding],     "n_results": 5,     "include": ["documents", "metadatas", "distances"] }`

- Payload sent to ChromaDB
    
- Requests:
    
    - Top 5 similar chunks
        
    - Document text
        
    - Similarity scores
        

---

`query_resp = requests.post(query_url, headers=headers, json=query_payload) results = query_resp.json()`

- Performs **semantic search**
    
- Returns most relevant document chunks
    

---

`relevant_docs_content = results['documents'][0]`

- Extracts actual text chunks
    
- These chunks form the **RAG context**
    

---

## PART 5: Preparing Input for LLM

`combined_input = f""" Based on the following documents, please answer this question: {query} Documents: - doc1 - doc2 """`

Why this is important:

- LLM is forced to answer **only from retrieved documents**
    
- Prevents hallucination
    

---

## PART 6: LLM Initialization (Groq)

`model = ChatGroq(     model="llama-3.3-70b-versatile",     api_key=os.getenv("GROQ_API_KEY") )`

- Uses Groq’s ultra-fast inference
    
- LLaMA 3.3 (70B) for high-quality answers
    

---

`messages = [     SystemMessage(content="You are a helpful assistant."),     HumanMessage(content=combined_input), ]`

- System message sets behavior
    
- Human message contains question + context
    

---

`result = model.invoke(messages) print(result.content)`

- Generates final answer
    
- This is **RAG in action**
    

---

## PART 7: Conversational RAG (Second Script)

### Chat History

`chat_history = []`

- Stores previous questions and answers
    
- Enables follow-up questions
    

---

### Step 1: Contextualize Follow-Up Question

`if chat_history:     rewrite the question to be standalone`

Why?

- User might ask:
    
    > “How much did it cost?”
    
- LLM rewrites it to:
    
    > “How much did Microsoft pay to acquire GitHub?”
    

👉 This is **question rewriting**

---

### Step 2: Retrieval (Same as Before)

`query_embedding = embedding_model.embed_query(search_question)`

- Embedding for rewritten question
    

---

`results = requests.post(...).json() docs = results['documents'][0]`

- Retrieve relevant context again
    

---

### Step 3: Answer with Chat History

`messages = [     SystemMessage(...), ] + chat_history + [     HumanMessage(content=combined_input) ]`

This allows:

- Context-aware answers
    
- Natural conversation flow
    

---

### Step 4: Update Chat History

`chat_history.append(HumanMessage(...)) chat_history.append(AIMessage(...))`

- Saves conversation state
    

---

## PART 8: Chat Loop

`while True:     question = input("Your question:")`

- Simple terminal-based chatbot
    
- Runs until user types `quit`