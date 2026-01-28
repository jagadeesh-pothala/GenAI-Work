## 1. Embeddings

**definition:**  
Embeddings convert data into **numbers that represent meaning**.  
They are like **GPS coordinates for words**.

---

### Why embeddings are needed

- Computers understand **only numbers**
    
- Words must be converted into numbers
    
- Similar meanings get **similar numbers**
    

---

### Common example (chatbot)

User types:

> “Where is my order?”

The chatbot:

- Converts the sentence into embeddings
    
- Matches it with:
    
    - “Track my shipment”
        
    - “Order not delivered”
        

Even though the words are different, the **meaning is the same**.

---

### Simple explanation

- GPS coordinates show **location**
    
- Embeddings show **meaning**

TASK

Requirements to build a Video/Image Chatbot with streaming using embeddings

Build a chatbot that supports **image sliding** using embeddings. Each image is converted into a single embedding vector and stored in a vector database with its image URL. When a user enters text, the text is converted into an embedding using the same model. The system compares the query embedding with stored image embeddings using similarity search.