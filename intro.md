# pdf.ai
Summarize the pdf and can ask questions on the uploaded pdf
## how?
1. Take all the text from pdf and send that along with prompt to GPT
    - Cant work since there is limit on tokens per prompt
    - Even if we could put int as many wors as we can, not a good idea since a ton of text will likely confuse the model to find right meaning
        - also a financial issue as too monay tokens
2. When pdf uploaded, extract the text, divy to chunks
    - run algo on each chunk to gen summary and store inDB
    - after that when use asks questions, figure out what they are asking about and look at text summaries in the DB, then find most relavent text
    - sent that summary and the prompt to GPT now
    
    a. Divide text into chunk size 1024 chars for eg
    b. convert each chunk into an embedding: list of embeddings.
    c. store these into a vector DB like SQL, but best ot use vector store.
    d. When prompt submitted, convert it to embedding vector
    e. find most relavent chunk to the propmt: find closest embedding in the vector store to this using a distance measure
    f. take the prompt, closest chunk and then create another prompt contianing both of these
        - Dear GPT, Did you know that "chunk"? Also, "prompt"?
    g. send this prompt to GPT
    h. GPT just regurgitates the part of pdf we sent it, the crucial thing was embedding, chunking, looking up embeddings

## embedding
In: str, Out: arr of numbers. Always 1536 elements. each num is from -1 to 1. This distills the essesnce of what text is talkin about. each number in array is a score that talks about what the text is discussing, eg idx 0 could be sentiment, idx 1 can be objects, idx 2 can be verb

Embedding can be made for a variable chunk size but output is same size array. Now we can do math on the array.

## why use langchain?
### LangChain 
Gives tools to automate all the steps aboe in pdf.ai.
1. open pdf, parse and extract text from it using its class: unstructuredPDFLoader
```python
from langchain.document_loaders import UnstructuredPDFLoader
loader = UnstructuredPDFLoader("a.pdf")
data = loader.load()
print(data) # Document(page_content="qwerty...")
```
We can load other types too it has classes for: PyPDF, JSON, HTML, S3FileLoader (from AWS), Discord chat, Excel, Github issues, google drive
2. Store results in a vectorstore. It supports classes for: PGVector(extension on postgres), Pinecone, Chroma, Deep Lake, Weaviate, Redis. Specialized to handle embeddings
```python
from langchain.vectorstores.pgvector import PGVector

db = PGVector.from_documents(
    embeddings=embeddings,
    documents=docs,
    collection_name="a",
    connection_string=CONNECTION_STRING,
)
query = "<PROMPT>"
docs = db.similarity_search(query)

from langchain.vectorstores.redis import Redis

rds = Redis.from_documents(
    embeddings=embeddings,
    documents=docs,
    redis_url="redis://localhost:6379",
    index_name="a",
)
query = "<PROMPT>"
docs = rds.similarity_search(query)

from langchain.vectorstores.pinecone import Pinecone

db = Pinecone.from_documents(
    embeddings=embeddings,
    dpcuments=docs,
    index_name="a"
)
query = "<PROMPT>"
docs = db.similarity_search(query)

```
- Langchain provides interchangeable tools to automate each step of text generation piepline.
- load data, parse it, store it, query it, pass it to models like gpt
- integrates with many diff services provided by many companies
- easy to swap out moels, can use cluade, deepseek, etc instead of gpt anytime

 