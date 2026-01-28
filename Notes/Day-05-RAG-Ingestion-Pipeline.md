
## Purpose of This Code 

1. Loads company documents (`.txt` files)
    
2. Splits them into small chunks
    
3. Converts text into embeddings
    
4. Stores embeddings in **Chroma Vector Database**
    
5. Prepares data for **RAG (Retrieval-Augmented Generation)**
    

---

## 1. Import Statements

`import os`

- Used to work with files and directories (check if folder exists, read paths)
    

---

`from langchain_community.document_loaders import TextLoader, DirectoryLoader`

- `TextLoader`: reads a single `.txt` file
    
- `DirectoryLoader`: reads **multiple files from a folder**
    

---

`from langchain_text_splitters import CharacterTextSplitter`

- Splits large text into **smaller chunks**
    
- Required because LLMs cannot process very large text at once
    

---

`from langchain_huggingface import HuggingFaceEmbeddings`

- Converts text into **embeddings (numbers)**
    
- Uses Hugging Face sentence-transformer model
    

---

`from langchain_chroma import Chroma`

- LangChain wrapper for **Chroma vector database**
    

---

`from dotenv import load_dotenv import chromadb`

- `load_dotenv()` loads secrets from `.env` file
    
- `chromadb` is the **native Chroma client** (used for remote DB)
    

---

`load_dotenv()`

- Loads environment variables like:
    
    - `CHROMA_HOST`
        
    - `CHROMA_API_KEY`
        
    - `CHROMA_DATABASE`
        

---

## 2. Loading Documents Function

`def load_documents(docs_path="docs"):`

- Function to load all text files from the `docs/` folder
    

---

`if not os.path.exists(docs_path):`

- Checks if `docs` folder exists
    
- Prevents runtime errors
    

---

`loader = DirectoryLoader(     path=docs_path,     glob="*.txt",     loader_cls=TextLoader,     loader_kwargs={"encoding": "utf-8"} )`

What this does:

- Reads **all `.txt` files**
    
- Uses UTF-8 encoding
    
- Converts each file into a `Document` object
    

---

`documents = loader.load()`

- Actually loads the files into memory
    

---

`if len(documents) == 0:`

- Ensures at least one file exists
    
- Prevents empty vector database
    

---

`print(doc.page_content[:100])`

- Prints preview of text
    
- Used only for **debugging / learning**
    

---

`return documents`

- Returns list of loaded documents
    

---

## 3. Splitting Documents into Chunks

`def split_documents(documents, chunk_size=1000, chunk_overlap=0):`

- Splits documents into chunks of **1000 characters**
    
- No overlap between chunks
    

---

`text_splitter = CharacterTextSplitter(     chunk_size=chunk_size,      chunk_overlap=chunk_overlap )`

Why splitting is needed:

- LLMs have **token limits**
    
- Small chunks improve **search accuracy**
    

---

`chunks = text_splitter.split_documents(documents)`

- Converts each document into multiple smaller chunks
    

---

`chunk.page_content`

- Actual text inside the chunk
    

---

`chunk.metadata`

- Keeps track of source file name
    

---

`return chunks`

- Returns chunked documents
    

---

## 4. Creating Vector Store (Embeddings + ChromaDB)

`def create_vector_store(chunks):`

- This function creates embeddings and stores them
    

---

`embedding_model = HuggingFaceEmbeddings(     model_name="sentence-transformers/all-mpnet-base-v2" )`

What this does:

- Converts text → vectors
    
- Same model must be used later for **query search**
    

---

`client = chromadb.HttpClient(     host=os.getenv("CHROMA_HOST"),     ssl=True,     headers={"x-chroma-token": os.getenv("CHROMA_API_KEY")},     tenant=os.getenv("CHROMA_TENANT"),     database=os.getenv("CHROMA_DATABASE") )`

This:

- Connects to **remote ChromaDB**
    
- Uses API key authentication
    
- Uses cosine similarity
    

---

`collection = client.get_or_create_collection(     name="Embeddings",     metadata={"hnsw:space": "cosine"} )`

- Creates a collection if it doesn’t exist
    
- Uses **cosine similarity** (most common for embeddings)
    

---

## 5. Adding Documents in Batches

`vectorstore = Chroma(     client=client,     collection_name="Embeddings",     embedding_function=embedding_model )`

- Wraps ChromaDB using LangChain
    
- Automatically:
    
    - Converts text → embeddings
        
    - Stores them
        

---

`vectorstore.add_documents(batch)`

- Stores:
    
    - Text chunks
        
    - Embeddings
        
    - Metadata
        

Batching is used to:

- Avoid memory issues
    
- Improve performance
    

---

## 6. Main Function (Pipeline Execution)

`def main():`

- Entry point of the program
    

---

`documents = load_documents(docs_path)`

- Step 1: Load text files
    

---

`chunks = split_documents(documents)`

- Step 2: Split documents
    

---

`vectorstore = create_vector_store(chunks)`

- Step 3: Create embeddings + store in ChromaDB
    

---

`print("Ingestion complete!")`

- Confirms data is ready for RAG
    

---

`if __name__ == "__main__":     main()`

- Ensures script runs only when executed directly