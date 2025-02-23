---
title:       "AI Engineering Goes Visual: Building an LLM RAG with PyFlyde & LangChain"
subtitle:    "Use Flyde to design and wire up LangChain, Vector Store, and Ollama or OpenAI API to perform Retrieval Augmented Generation"
description: "This part builds upon the Scraper app that we created in the part 1 and uses our visual programming skills to dive deeper in the LLM engineering world. This time we turn our article database into a RAG app, making use of LangChain, Vector Store and local LLaMA3 or OpenAI API."
date:        2025-02-23
tags:        ["fbp", "data", "llm", "python"]
categories:  ["Flow-based Programming" ]
draft:       false
---

## Introduction and recap of Part 1

Welcome back to our journey into Visual AI Engineering with PyFlyde! In this second part of our tutorial, we'll dive deeper into the fascinating world of AI and data engineering, combining modern tools and libraries with the intuitive power of visual programming.

In [Part 1 of this tutorial](/post/visual-ai-engineering-with-pyflyde-pt1-scraper), we embarked on an exciting journey to build a flexible data extraction flow. We successfully created a web scraper that fetches articles from the Software Leads Weekly newsletter, cleans them up, and saves them as local Markdown files. Here’s a quick visual recap of what we accomplished:

{{< figure src="/img/post/pyflyde/scrape_v3.flyde.avif" alt="Scrape.flyde version 3" width=50% >}}

Throughout the first part, we learned how to:

- Create visual flows with Flyde in VSCode
- Code Flyde nodes in Python using PyFlyde
- Utilize BS4, Playwright, and Markdownify to scrape a website
- Run, update, and optimize our flow

In this part, we will take our project to the next level by transforming the scraped content into a powerful Retrieval Augmentation Generation (RAG) system. We will leverage LangChain, a vector database, LLaMA3, and Flyde to seamlessly integrate these components. By the end of this tutorial, you will have a comprehensive understanding of how to build advanced AI engineering workflows using visual programming tools.

So, let’s get started and unlock the full potential of our data with PyFlyde!

## Prerequisites for this tutorial part

Before we proceed, ensure you have the following:

- Python 3.10 or newer
- Basic Python programming experience
- Visual Studio Code with the [Flyde extension](https://marketplace.visualstudio.com/items?itemName=flyde.flyde-vscode)
- Basic knowledge of LLMs and data processing (helpful but not required)
- A GPU or Apple Silicon chip with enough RAM to run an LLM locally, or an API key for OpenAI or another LLM of your choice

Here are some useful links to the tools and libraries we will use:

- [Flyde](https://www.flyde.dev) - a visual programming tool and VSCode extension.
- [PyFlyde](https://github.com/trustmaster/pyflyde) - a Python library and runtime for Flyde.
- [Ollama](https://ollama.com/) - the easiest way to run LLMs on your local machine.
- [LangChain](https://python.langchain.com/docs/introduction/) - an abstraction over LLM-related libraries.

### Installing dependencies

You can checkout the code of the project from the [SWLWi Github repo](https://github.com/trustmaster/swlwi/).

Here is the updated `dependencies` section of our `pyproject.toml`:

```toml
dependencies = [
    "beautifulsoup4",               # HTML parsing
    "langchain",                    # LLM abstraction layer
    "langchain-community >= 0.3.2", # Community packages for LangChain, including SQLiteVec
    "langchain-huggingface",        # HuggingFace model loader and API
    "langchain-core",               # Core
    "langchain-ollama",             # Ollama wrapper
    "langchain-text-splitters",     # Needed for chunking
    "markdownify",                  # HTML to Markdown conversion
    "playwright",                   # JavaScript-enabled scraping
    "pyflyde >= 0.0.11",            # Flyde runtime for Python
    "requests",                     # Basic HTTP requests
    "sentence-transformers",        # Needed to create embeddings
    "streamlit",                    # Simple web UI
    "sqlite-vec",                   # Vector Store on top of SQLite
]
```

To enter the Python virtual environment and install all the dependencies at once, run this command in thr project directory:

```bash
source .venv/bin/activate # On Windows: .venv\Scripts\activate
pip install .             # Install the dependencies
```

Next, let's get our LLM ready. If you haven't installed [Ollama](https://ollama.com/) yet, now is the good time. With Ollama on board, we can install our LLaMA3.2-3B model in the terminal:

```bash
ollama run llama3.2
```

Ollama will download the model, install it, and run automatically. You can enjoy a conversation with it right away in your CLI, or say `/bye` to exit and get back to coding.

#### Note on alternative LLMs

The `llama-3.2` is a relatively small model with 3 billion parameters, requiring just ~1GB of VRAM. It works fast on an Apple Silicon Mac or a relatively inexpensive discrete GPU.

If you want to try a slightly slower but larger model, you can run:

```bash
ollama run phi4
```

Phi-4 is a 14 billion parameter optimized model from Microsoft. It needs 2GB of VRAM and works slightly slower than llama-3.2-3b.

No GPU? No problem. You can still use an LLM in the cloud. For example, if you want to use OpenAI chat, you can use LangChain's OpenAI wrapper. We will cover it in more detail later in this article.

## So you need a RAG...

Our goal for today is to build a simple Retrieval-Augmented Generation (RAG) model using LLaMA3 (or a similar Large Language Model), Langchain, and SQLiteVec. We will leverage the LLaMA3/OpenAI model to generate text based on the input we provide and use SQLiteVec to store the vectors of the text data we have scraped.

### What is Retrieval-Augmented Generation (RAG)?

Retrieval-Augmented Generation (RAG) is a technique that enhances the capabilities of a standard Large Language Model (LLM) by incorporating external knowledge from a specific document database. This approach does not involve changing or retraining the LLM itself; instead, the LLM is used as-is. The key idea is to perform a search on the document database first and then insert the relevant documents into the LLM's context before generating a response.

### How does RAG Work?

RAG consists of four steps:

1. **Document database**: first, you need a database of documents. These documents are pre-processed and stored in a way that makes them easy to search. In our case, we will use SQLiteVec to store the vectors of the text data we have scraped.
2. **Search and retrieval**: when a query is made to the LLM, a search is performed on the document database to find the most relevant documents. This search can be based on various techniques, such as keyword matching or vector similarity. We are going to use vector similarity search, turning the text into vector embeddings.
3. **Context insertion**: the retrieved documents are then inserted into the context of the LLM. This means that the LLM will consider these documents when generating its response.
4. **Response generation**: finally, the LLM generates a response based on the input query and the additional context provided by the retrieved documents.

### When to use RAG

Here are some benefits of using RAG technique:

- _Enhanced knowledge_: RAG allows the LLM to access and utilize information that is not part of its pre-trained knowledge base. This is particularly useful for domain-specific applications where the LLM needs to be aware of specialized or up-to-date information.
- _Cost-effectivess_: retraining a large language model is both time-consuming and expensive. With RAG, you can update the document database without needing to retrain the model. This means you can keep the LLM's responses current with new information simply by updating the database.
- _Scalability_: RAG can handle large volumes of documents. Instead of embedding all possible knowledge into the LLM, which is impractical, RAG allows the model to dynamically retrieve and use relevant information from an external source.

It has certain limitations though:

- _Quality of documents_: the effectiveness of RAG is highly dependent on the quality and relevance of the documents in the database.
- _Search implementation_: the search mechanism needs to be efficient and accurate to retrieve the most relevant documents.
- _Context space_: the retrieved documents occupy space in the LLM's context, which is already limited. This means there is a trade-off between the amount of additional context and the complexity of the input query.

Now that we have better understanding of what RAG is, it's time to proceed to implementing it.

## Indexing documents and saving them in a Vector Store

Currently, we have our Knowledge Base saved on disk as Markdown files in issue subfolders. Unfortunately, an LLM cannot magically work with our local Markdown files. We need to turn our Markdown articles into a searchable database first. This process is called indexing, and we'll implement it as the `Index.flyde` flow:

{{< figure src="/img/post/pyflyde/index.flyde.avif" alt="Index.flyde" caption="The document Indexer flow in Flyde" width=50% >}}

For RAG to work, we first need to create a searchable document index in a vector store. A vector store is a database that treats data chunks as vectors of numbers, allowing it to easily find similarities between different documents and query inputs. It does not work well with large bodies of plain text out of the box, so we will have to do two things before we store our articles in a vector store:

1. Split the articles into smaller chunks. This makes it easier to find relevant pieces and ensures they do not occupy too many tokens in the LLM's context.
2. Encode the plain text chunks into embeddings—a representation the vector search can understand.

There are plenty of dedicated vector stores available, but in our app, we will use one of the quickest options that doesn't require running a separate database server — [sqlite-vec](https://github.com/asg017/sqlite-vec). Yes, you guessed it, it's a vector search on top of SQLite.

Now that we have an idea of the data flow, we can move on to coding some Python nodes.

### Implementing the indexer nodes

For the RAG part, we will create another Python module `swlwi/rag.py`.

We are going to use [LangChain](https://python.langchain.com/docs/tutorials/) as an abstraction over various chunking, embedding encoder/decoder methods and models, as well as a vector store.

#### ListArticles component

The `ListArticles` component is pretty straightforward. It lists all the Markdown files across all issues that we scraped with the scraper flow. To do that, it goes over all `issue-` folders in the index and gets paths of all `*.md` files inside them.

```python
class ListArticles(Component):
    """Lists all articles in the index."""

    inputs = {"path": Input(description="Path to the index", type=str, mode=InputMode.STICKY, value="./index")}

    outputs = {"article_path": Output(description="Stream of paths to articles", type=str)}

    def process(self, path: str) -> None:
        for issue in os.listdir(path):
            # Skip folders that don't start with 'issue-'
            if not issue.startswith("issue-"):
                continue
            issue_path = os.path.join(path, issue)
            for article in os.listdir(issue_path):
                # Skip files that don't end with '.md'
                if not article.endswith(".md"):
                    continue
                article_path = os.path.join(issue_path, article)
                self.send("article_path", article_path)
        # Send EOF when finished listing
        self.stop()
```

#### DocumentLoader component

The `DocumentLoader` reads the contents of a Markdown file from the specified path and converts it into a LangChain Document.

{{< figure src="/img/post/pyflyde/rag_document_loader.avif" alt="DocumentLoader component view in Flyde" width=25% >}}

The Markdown content is saved in the `page_content` of the document, which will be used to search and return the text.

Other attributes such as the title, source URL, reading time, and summary are saved as `metadata`. We won't necessarily use them in the first version, but we may need some of them to implement extra functionality in the future.

```python
from langchain_core.documents import Document # add this to imports


class DocumentLoader(Component):
    """Loads Markdown from file as a Langchain Document."""

    inputs = {"path": Input(description="Path to the markdown file", type=str)}

    outputs = {"document": Output(description="Langchain Document", type=Document)}

    def process(self, path: str) -> dict[str, Document]:
        with open(path, "r") as f:
            text = f.read()

        # Parse header
        header = text.split("\n---\n")[0]
        lines = header.split("\n")
        title = lines[0].strip("# \t\r\n")
        lines = lines[1:]
        source_url = ""
        reading_time = ""
        summary_lines = []

        for line in lines:
            if line.startswith("Source:"):
                source_url = line.split("(")[-1].strip(")")
            elif line.startswith("Reading time:"):
                reading_time = line.split(":")[-1].strip()
            else:
                summary_lines.append(line)

        summary = "\n".join(summary_lines).strip()

        doc = Document(
            page_content=text,
            metadata={
                "path": path,
                "title": title,
                "source_url": source_url,
                "reading_time": reading_time,
                "summary": summary,
            },
        )

        return {"document": doc}
```

#### DocumentSplitter component

The next node, `DocumentSplitter`, uses LangChain's `MarkdownDocumentSplitter` to split each `Document` into smaller `Document` chunks of a specific size.

{{< figure src="/img/post/pyflyde/rag_document_splitter.avif" alt="DocumentSplitter component view in Flyde" width=25% >}}

The `MarkdownTextSplitter` from LangChain knows how to split our source `document` into chunks of up to `chunk_size` characters. We set `chunk_overlap` so that these chunks overlap slightly, increasing the probability of having enough context for the request.

```python
from langchain_text_splitters import MarkdownTextSplitter


class DocumentSplitter(Component):
    """Splits markdown documents into chunks."""

    inputs = {
        "document": Input(description="Document to split", type=Document),
        "chunk_size": Input(description="Size of each chunk",
            type=int, mode=InputMode.STICKY, value=2000),
    }

    outputs = {"documents": Output(description="Chunks of text")}

    def process(self, document: Document, chunk_size: int) -> dict[str, list[Document]]:
        splitter = MarkdownTextSplitter(chunk_size=chunk_size, chunk_overlap=50)
        texts = [document.page_content]
        metadatas = [document.metadata]
        documents = splitter.create_documents(texts, metadatas)
        return {"documents": documents}
```

For each input document, the `DocumentSplitter` sends a list of documents. Therefore, our next node, `VectorStore`, should be designed to handle this type of input.

#### VectorStore component

We are going to create our `VectorStore` component as a wrapper around the `SQLiteVec` class from LangChain. This component will only add new documents to the store. Unlike conventional Repository classes in object-oriented programming, in flow-based programming, a node typically has just one main function. For example, document search will be a different component.

However, we need to ensure the `SQLiteVec` store itself stays in memory all the time and we don't recreate it on every new input. That's why we will implement an `_init()` method which initializes the store only once.

{{< figure src="/img/post/pyflyde/rag_vector_store.avif" alt="VectorStore component view in Flyde" width=25% >}}

While `process()` is called on every new input list of documents, we call `_init()` inside of it to ensure the embeddings encoder and the SQLiteVec instance are created once.

```python
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_community.vectorstores import SQLiteVec


class VectorStore(Component):
    """Stores documents as vectors in a vector store."""

    inputs = {
        "documents": Input(description="Documents to store"),
        "path": Input(description="Path to the vector store",
            type=str, mode=InputMode.STICKY, value="./index/vectors"),
    }

    def _init(self, path: str):
        if not hasattr(self, "_embeddings"):
            # Load the embeddings encoder/decoder model from HuggingFace
            self._embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
        if not hasattr(self, "_vector_store"):
            # Create path if not exists
            os.makedirs(path, exist_ok=True)
            # Create the Vector Store (opens an existing database or creates a new one)
            # and pass the embeddings encoder/decoder model to it
            self._vector_store = SQLiteVec(
                table="swlwi_embeddings", connection=None,
                db_file=f"{path}/db.sqlite3", embedding=self._embeddings
            )

    def process(self, documents: list, path: str):
        # Load the store and embeddings if needed
        self._init(path)

        logger.info(f"Adding {len(documents)} documents from {documents[0].metadata['path']}
            to the vector store")
        self._vector_store.add_documents(documents)
```

As you see in the code, our vector store needs a `HuggingFaceEmbeddings` object to perform vector searches on the documents. What does this Embeddings object do? When we store new documents, it converts text sentences into searchable vectors. When we query the store, it translates the query into an embedding vector as well. Then the store can perform similarity searches among vectors.

Since we use Ollama, it might be tempting to assume that we need to use Ollama Embeddings in the vector store, as LangChain provides it. However, we are only going to use these embeddings to perform the search and retrieval of documents; we are not going to send these embeddings to LLaMA, as LLaMA receives our prompt as plain text. And since Ollama's embeddings encoder is quite slow, we are using a much simpler `all-MiniLM-L6-v2` model, which is much faster and sufficient for our needs.

### Running the flow

With all the components implemented, we make them available in Flyde with:

```bash
pyflyde gen swlwi/rag.py
```

Next, create a new `Index.flyde` visual flow with Flyde in VSCode. Wire the nodes together as shown in the diagram at the beginning of this section.

Then, run it to index our article database:

```bash
pyflyde Index.flyde
```

This creates an SQLite database file in our index folder. We will use it in the next section to perform actual Retrieval Augmented Generation.

### Potential improvements to the indexer

As you may want to update your RAG database from time to time, our `Index.flyde` can be improved to support incremental indexing. We have implemented such an optimization in `Scrape.flyde` so that we don't scrape the issues that were scraped earlier. However, our vector database is populated from scratch every time we run `Index.flyde`.

To avoid recreating the database from scratch or adding the same documents repeatedly, we can load the metadata from SQLite about the issues and articles that were indexed earlier and exclude those from the list returned by `ListArticles`. Alternatively, we can create a unique index in the `swlwi_embeddings` table and ignore items with the same path or hash upon adding them to the store.

## Retrieving documents into LLM's context

Once we have populated the database with vectors of chunked documents, the actual retrieval of documents relevant to a search `query` and embedding of the search results into LLaMA's prompt is quite simple:

{{< figure src="/img/post/pyflyde/rag.flyde.avif" alt="Rag.flyde" caption="Flow for Retrieval Augmentation of Ollama prompt" width=60% >}}

### Retriever component

We provide the same path to the vector DB to the `Retriever` as an inline value. The `query` input comes from outside the flow, and the `response` output is used to send the results. There is no UI to get the query and display the results in this flow yet, because we are actually going to integrate it into an external Streamlit application in the next section.

{{< figure src="/img/post/pyflyde/rag_retriever.avif" alt="Retriever component view in Flyde" width=25% >}}

The Retriever implements the reader part of the vector store and reuses the same vector store logic for initialization. In addition, it uses LangChain's `as_retriever()` interface to perform the actual retrieval of matching documents.

```python
class Retriever(Component):
    """Retrieves context from a vector store."""

    inputs = {
        "query": Input(description="Query text", type=str),
        "path": Input(description="Path to the vector store",
            type=str, mode=InputMode.STICKY, value="./index/vectors"),
    }

    outputs = {"context": Output(description="Context retrieved from the vector store", type=str)}

    def _init(self, path: str):
        if not hasattr(self, "_embeddings"):
            self._embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
        if not hasattr(self, "_vector_store"):
            self._vector_store = SQLiteVec(
                table="swlwi_embeddings", connection=None,
                db_file=f"{path}/db.sqlite3", embedding=self._embeddings
            )
        if not hasattr(self, "_retriever"):
            # New stuff here: initializing as_retriever() interface to the vector store
            self._retriever = self._vector_store.as_retriever()

    def process(self, query: str, path: str) -> dict[str, str]:
        # Initialize the store, embeddings, and retriever if needed
        self._init(path)

        # Retrieve the documents matching the query
        docs = self._retriever.invoke(query)

        logger.info(f"Retrieved {len(docs)} documents from the vector store for query '{query}'")
        print(f"Retrieved {len(docs)} documents from the vector store for query '{query}'")

        if not docs:
            return {"context": ""}

        # Simply joining the documents as string
        context = "\n\n".join([doc.page_content for doc in docs])
        return {"context": context}
```

The `Retriever` retrieves documents (or chunks of articles, to be more precise) that match the `query`. It then combines them into a `context` for the prompt using simple concatenation.

### OllamaChat node

Finally, let's look at our LLaMA wrapper component:

{{< figure src="/img/post/pyflyde/rag_ollama_chat.avif" alt="OllamaChat component view in Flyde" width=25% >}}

This node receives the unmodified `query` and the `context` string from the `Retriever`. It then creates a prompt and sends it to a LLaMA chat to get a response.

```python
import ollama


class OllamaChat(Component):
    """Chat with the Ollama model."""

    inputs = {
        "query": Input(description="Query text", type=str),
        "context": Input(description="Context text", type=str),
    }

    outputs = {"response": Output(description="Response from the Ollama model", type=str)}

    def process(self, query: str, context: str) -> dict[str, str]:
        logger.info(f"Loaded context:\n\n {context}\n\n")

        # System prompt tells the agent about their role and sets ground rules
        system_prompt = ("Given a question and context by user,"
            "use the context and your prior knowledge to answer the user's question.")

        # Our user prompt
        prompt = f"Question: {query}\n\nContext: {context}"

        # Invoke the model
        response = ollama.chat(
            model="llama3.2", # Replace this with other model string if needed, e.g. "phi4"
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": prompt}
            ],
        )
        return {"response": response["message"]["content"]}
```

Our `Rag.flyde` is different from `Index.flyde` and `Scrape.flyde` in that it is not designed as a standalone flow that you can just run with `pyflyde`. It is meant to be embedded in another flow or Python program.

This was intentional because we want to build the UI in a more conventional way and just embed `Rag.flyde` in that application. The next section shows how we can do it.

### Using other Chat APIs, e.g., OpenAI

Replacing a local LLM with one running in the cloud is easy. For example, let's add `OpenAIChat` instead of `OllamaChat` to our RAG.

First, install the dependencies and add your OpenAI API key to the environment variables:

```bash
pip install langchain-openai
export OPENAI_API_KEY="your-api-key"
```

Next, implement the `OpenAIChat` component with the same inputs and outputs schema as `OllamaChat` we implemented before:

```python
from langchain_openai import ChatOpenAI # Add this to imports


class OpenAIChat(Component):
    """Chat with the OpenAI model."""

    inputs = {
        "query": Input(description="Query text", type=str),
        "context": Input(description="Context text", type=str),
    }

    outputs = {"response": Output(description="Response from the OpenAI model", type=str)}

    def _init(self):
        if not hasattr(self, "_llm"):
            self._llm = ChatOpenAI(model="gpt-4o")

    def process(self, query: str, context: str) -> dict[str, str]:
        logger.info(f"Loaded context:\n\n {context}\n\n")

        # Load the model if needed
        self._init()

        # Construct the prompts
        system_prompt = "Given a question and context by user, use the context and your prior knowledge to answer the user's question."
        prompt = f"Question: {query}\n\nContext: {context}"

        # Invoke the model
        response = self._llm.invoke([("system", system_prompt), ("human", prompt)])

        return {"response": response.content}
```

Looks very similar to how we work with Ollama, but the prompt and response structure is a bit different. We also used the `_init()` method to avoid reinitializing the model on every input.

> **Note:** If you get `openai.RateLimitError: Error code: 429` when using this component, it most likely means your OpenAI credit balance is zero.

### Putting it all together

Now that we have our components implemented in `rag.py`, it's time to update the TypeScript stubs and create a new visual flow in Flyde:

```bash
pyflyde gen swlwi/rag.py
```

Then invoke the `Flyde: New Visual Flow` command in VSCode and create the `Rag.flyde` flow as shown in the diagram above.

This time, we cannot just run `pyflyde Rag.flyde` to talk to the chatbot. Our flow is designed to be embedded in a UI app. We are going to implement it next.

## Adding a Streamlit UI

In case you are not familiar with Streamlit, it is a simple web-based UI library. It differs from similar toolkits in that it does not have traditional views, templates, or reactive handlers. Instead, it provides a set of UI components that you can use in your Python script. The essential feature of Streamlit is that it re-runs your script each time an interaction happens or the view needs re-rendering.

This might sound complicated and unusual, but in practice, it can be simpler than conventional UI toolkits. Let's have a look at our Streamlit `app.py`:

```python
"""Streamlit app for the UI to query the RAG."""

from dataclasses import dataclass
import os
import signal
import streamlit as st
from queue import Queue
from flyde.flow import Flow
from flyde.io import EOF


# A structure to hold our appication's state
@dataclass
class FlowWrapper:
    flow: Flow
    query: Queue
    response: Queue


# Cache resource is a way to keep state between re-renders in Streamlit.
# wrap_flow() is effectively called just once in our app.
@st.cache_resource
def wrap_flow() -> FlowWrapper:
    # Load a PyFlyde flow from a .flyde file
    flow = Flow.from_file("Rag.flyde")

    # Get the query queue from the flow object
    query_q = flow.node.inputs["query"].queue
    # Attach a new queue to the flow's output
    response_q: Queue = Queue()
    flow.node.outputs["response"].connect(response_q)

    # Run the flow (in a separate thread)
    flow.run()

    return FlowWrapper(flow, query_q, response_q)


# UI header
st.title("Software Leads Weekly Index KB")
st.caption("A collection of articles, papers, and other resources for software leads.")
st.markdown("---")


f = wrap_flow()

query = st.text_input("Ask a question or type /bye to finish the conversation", "")
if query:
    if query.strip() == "/bye":
        query = EOF
    f.query.put(query)
    response = f.response.get()
    if response == EOF:
        st.write("Goodbye!")
        st.stop()
        os.kill(os.getpid(), signal.SIGKILL)
    else:
        st.write(response)
```

First, we define a data class (structure) that will hold the state of our application. It consists of a PyFlyde Flow and two queues to send the query input and receive the response output from that flow.

Then we use Streamlit's `st.cache_resource` decorator. This makes Streamlit remember the return value of the decorated function during subsequent calls. So, our `wrap_flow()` function is only called once, and subsequent calls return the same dataclass instance.

After that, we initialize the header part of the UI and call `wrap_flow` to retrieve the state. Then we create a text field to receive the user's query.

Note that we use `if query` rather than a typical `while` loop. This is because the loop happens outside of our `app.py`; Streamlit calls our `app.py` every time the UI needs to be refreshed. That's why we just check if the query is non-empty and react to its value.

Our script supports the `/bye` command to terminate the dialog. In this case, it sends an `EOF` input to the PyFlyde flow, which shuts it down gracefully.

Any other text input is considered a query and triggers our `Rag.flyde` flow from the previous section to retrieve, augment, and generate a response.

> **Known caveat:** Streamlit suppresses the logs in our PyFlyde code that we do with the standard Python logger interface. You may need to use good old `print()` to debug and see actual messages when running with Streamlit.

To run the application, invoke:

```bash
streamlit run app.py
```

Here is how the UI of our application looks in action in a browser window:

{{< figure src="/img/post/pyflyde/streamlit_app_example.avif" alt="Streamlit App UI" width=60% >}}

When finished chatting, type `/bye` to shut down our PyFlyde flow gracefully, and hit `Ctrl + C` to close the Streamlit app in the terminal.

## Conclusion

In this tutorial, we embarked on an exciting journey to create an interactive knowledge base with an LLM-powered chat interface. From scraping thematic blog posts to building a search index, integrating Retrieval Augmented Generation, and wrapping it all up in a user-friendly UI, we've covered a wide range of practical applications.

### Key takeaways

Here's a recap of the valuable lessons we've learned:

- **Data flow perspective**:
  - Identifying critical data inputs, outputs, and transformation steps.
  - Designing key nodes that effectively process and transform data.

- **Reusable components with PyFlyde**:
  - Defining clear interfaces with inputs and outputs.
  - Implementing the logic in Python.
  - Handling various input types and sending output effectively.

- **Visual data flows in VSCode with Flyde**:
  - Leveraging Flyde's visual language and component library.
  - Creating and refining visual flows seamlessly within VSCode.

- **AI Engineering libraries and tools**:
  - Utilizing LangChain for data processing and integration.
  - Working with LLMs locally using Ollama and remotely with OpenAI.
  - Implementing Vector Store for semantic search.
  - Enhancing LLM prompts with Retrieval Augmented Generation.

- **Iterative development of Flow-based applications**:
  - Starting with simple flows and achieving end-to-end functionality.
  - Gradually adding complexity and covering more use cases.
  - Embedding data flows into conventional Python applications.

#### Reflecting on Flow-Based Programming

Flow-based programming offers a unique approach to Data and AI Engineering. It aligns perfectly with the flow-based nature of data pipelines, providing a powerful tool for designing, structuring, and visualizing applications. This approach simplifies onboarding, enhances collaboration, and makes the development process more enjoyable.

However, it's important to acknowledge that tools like Flyde and PyFlyde are still in their early stages. While they are excellent for learning, prototyping, and small to medium-sized projects, they may not yet be suitable for large-scale or corporate environments due to potential bugs and stability issues.

#### Embracing AI Coding Assistants

One of the standout benefits of this paradigm is its compatibility with AI coding assistants. By defining clear data flows and interfaces, AI assistants like Copilot can efficiently generate implementation code, making the development process smoother and more efficient. In fact, this is how this project built: I took more of an architect's seat defining the flows and the interfaces of the black boxes, and Copilot wrote most of the implementation code.

#### Next Steps

Now that you have Flyde and PyFlyde in your toolkit, your journey begins. Whether you're building innovative prototypes, exploring new AI applications, or enhancing existing workflows, these tools help you to bring your ideas to life.

If you're eager to dive deeper and explore more tools, here are some useful links:

- [Flyde](https://www.flyde.dev)
- [PyFlyde](https://github.com/trustmaster/pyflyde)
- [Flow-Based Programming Wiki](https://github.com/flowbased/flow-based.org/wiki)
- [KNIME](https://www.knime.com/) - enterprise-level visual data engineering workflows

Thank you for joining me on this journey. I hope this tutorial has inspired you to embrace the power of visual AI engineering and flow-based programming. Now, go forth and create something amazing!
