### Step-by-Step Instructions with Code Snippets

1. **Import Necessary Modules:**
   ```python
   import os
   from dotenv import load_dotenv
   from langchain_openai import ChatOpenAI
   from langchain.prompts.prompt import PromptTemplate
   from langchain.chains.llm import LLMChain
   from langchain_pinecone import PineconeVectorStore
   from langchain_openai import OpenAIEmbeddings
   from constants import *
   ```

2. **Load Environment Variables:**
   ```python
   load_dotenv()

   openai_key = os.getenv('OPENAI_API_KEY2')
   pinecone_key = os.getenv('PINECONE_API_KEY')
   ```

3. **Initialize Embeddings:**
   ```python
   # Note: Use the same embedding model that was used when uploading the documents
   embeddings = OpenAIEmbeddings(model=EMBEDDING_MODEL)
   ```

4. **Setup Vector Store and Retriever:**
   ```python
   # Querying the vector database for relevant documents
   document_vectorstore = PineconeVectorStore(index_name=PINECONE_INDEX, embedding=embeddings)
   retriever = document_vectorstore.as_retriever()
   ```

5. **Initialize the Language Model (LLM):**
   ```python
   llm = ChatOpenAI(temperature=0.7, model_name="gpt-4o-mini")
   ```

6. **Define Prompt Template:**
   ```python
   template = PromptTemplate(template="{query} Context: {context}", input_variables=["query", "context"])
   ```

7. **Create Context and Retrieve Relevant Documents:**
   ```python
   prompt = "Can you tell me about the best overall methods for day trading?"

   # Create context by retrieving relevant documents based on the prompt
   context = retriever.get_relevant_documents(prompt)
   ```

8. **Display Relevant Documents:**
   ```python
   # Show the thought process by looping over all relevant documents
   for doc in context:
       print(f"Source: {doc.metadata}\nContext: {doc.page_content}\n\n")
       print("******************************")
   ```

9. **Build Prompt with Context:**
   ```python
   # Build a prompt with the query and the retrieved context
   prompt_with_context = template.invoke({"query": prompt, "context": context})
   ```

10. **Get Response from the LLM:**
    ```python
    # Ask the LLM for a response using the prompt with context
    results = llm.invoke(prompt_with_context)
    ```

11. **Print the LLM's Response:**
    ```python
    # Print the response content
    print(results.content)
    `
