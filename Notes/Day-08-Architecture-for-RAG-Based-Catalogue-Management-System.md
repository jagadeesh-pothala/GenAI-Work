
## 1. What is the CMS used for?

The Catalogue Management System (CMS) stores **all product information** in one place.

It supports **two types of AI**:

1. **Estimating AI**
    
    - Uses **prices, SKUs, dimensions**
        
    - Needs **correct and structured data**
        
2. **Designer / Chatbot AI**
    
    - Uses **PDFs, images, links**
        
    - Needs **search by meaning (RAG)**
        

So the CMS must handle:

- Text
    
- Images
    
- PDFs
    
- Prices
    
- Links
    

---

## 2. Why We Use RAG in CMS

Normal search looks for **exact words**.  
RAG search looks for **meaning**.

Example:

- “running shoes”
    
- “sports footwear”
    

Both mean the same thing → RAG understands this.

---

## 3. Why We Use Two Databases

### Structured Database (SQL / MongoDB)

Used for:

- Prices
    
- SKUs
    
- Dimensions
    
- Calculations
    

Because:

- Prices must be **100% accurate**
    
- No guessing allowed
    

---

### Vector Database

Used for:

- PDFs
    
- Text descriptions
    
- Images
    

Because:

- AI needs to **understand meaning**
    
- Helps chatbots answer questions correctly
    

---

## 4. How PDFs Are Handled

### What we do:

1. Upload PDF catalogues
    
2. Read text using OCR
    
3. Split text into small parts
    
4. Store:
    
    - Important fields → Structured DB
        
    - Full text → Vector DB
        

### Tools Used (Simple)

|Tool|Why|
|---|---|
|OCR (Textract / Tesseract)|Reads text from PDFs|
|LangChain|Splits and processes text|
|Human review|OCR can make mistakes|

### Alternatives

- Google Document AI
    
- Azure Form Recognizer
    

---

## 5. How Images Are Handled

### What we do:

1. Store images in cloud storage
    
2. Link each image to a SKU
    
3. Convert images into embeddings
    
4. Store embeddings in Vector DB
    

### Why?

- So users can search images using text
    
- Example: “modern kitchen cabinet”
    

### Tools Used

| Tool      | Why                   |
| --------- | --------------------- |
| Vector DB | Search similar images |

---

## 6. How Online Links Are Handled

### What we do:

- Save product links (manuals, specs, install guides)
    
- Link them to SKUs
    
- Show them in AI answers
    

### Why?

- Customers need original sources
    
- Designers need reference documents
    

---

## 7. How Prices Are Managed

### What we do:

1. Upload CSV / Excel price files
    
2. Validate the data
    
3. Store as **single source of truth**
    

### Why?

- AI must never guess prices
    
- Only stored values are used
    

### Tools Used

|Tool|Why|
|---|---|
|PostgreSQL / MySQL|Reliable data|
|Pandas|Read Excel / CSV|
|API|Secure uploads|

    

---

## 8. Vector Database

Vector DB stores **numbers that represent meaning**.

Used for:

- Searching PDFs
    
- Searching images
    
- Answering chatbot questions
    

### Tools Used

| Tool     | Why                |
| -------- | ------------------ |
| ChromaDB | Easy to use        |
