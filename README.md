# Simple RAG with LangChain

This project is a small Retrieval-Augmented Generation (RAG) application built
with LangChain. It loads the Markdown files in `data/books`, stores their
meaning as searchable vectors in Chroma, and uses the most relevant passages
to answer questions with an OpenAI chat model.

## What is RAG?

RAG combines document search with text generation:

1. **Load**: read the source documents.
2. **Split**: divide large documents into smaller overlapping chunks.
3. **Embed**: convert each chunk into a numerical vector that represents its meaning.
4. **Retrieve**: find the chunks most similar to a user's question.
5. **Generate**: send the question and retrieved context to a language model.

The model is instructed to answer only from the retrieved context. This makes
the answer relevant to the project documents instead of relying only on the
model's general training data. The returned source paths also show where the
context came from.

## How LangChain is used

LangChain connects the individual RAG components:

- `DirectoryLoader` loads Markdown documents from the books directory.
- `RecursiveCharacterTextSplitter` creates chunks of 300 characters with 100
    characters of overlap. Overlap helps preserve meaning across chunk boundaries.
- `OpenAIEmbeddings` converts documents and questions into vectors.
- `Chroma` stores the vectors and performs similarity search.
- `ChatPromptTemplate` builds the prompt containing the retrieved context.
- `ChatOpenAI` generates the final answer from that prompt.

## Project files

| File or directory | Purpose |
| --- | --- |
| `create_database.py` | Loads books, splits them into chunks, embeds the chunks, and saves the Chroma database. |
| `query_data.py` | Accepts a question, retrieves the three closest chunks, and generates an answer. |
| `compare_embeddings.py` | Creates an embedding for a word and compares the embedding distance between `apple` and `iphone`. |
| `data/books/` | Source Markdown documents used by the application. |
| `requirements.txt` | Python packages required by the project. |
| `chroma/` | Generated local vector database. It is recreated by `create_database.py` and is ignored by Git. |

## Setup

Create and activate a virtual environment from PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
pip install "unstructured[md]"
```

Create a `.env` file in the project root with your OpenAI API key:

```text
OPENAI_API_KEY=your_api_key_here
```

Do not commit `.env` or share the API key. These files are excluded by
`.gitignore`.

On Windows, Chroma may require the Microsoft C++ Build Tools. If installation
fails because of `onnxruntime`, install the build tools and then retry the
dependency installation.

## Create the vector database

Run this after adding or changing documents in `data/books`:

```powershell
python create_database.py
```

The script removes the existing `chroma/` directory, loads all `*.md` files,
creates the chunks, and stores their embeddings. It must be run successfully
before querying the application.

## Ask a question

```powershell
python query_data.py "How does Alice meet the Mad Hatter?"
```

The query script retrieves three relevant chunks. If the best similarity score
is below `0.7`, it declines to answer because the database did not find enough
relevant context. Otherwise, it prints the prompt, answer, and source paths.

Both database creation and querying use the OpenAI API, so the API key must be
valid and the account must have available credits.

## Explore embeddings

```powershell
python compare_embeddings.py
```

This utility demonstrates that related words can have a smaller embedding
distance than unrelated words. Embeddings capture semantic relationships, which
is why the RAG application can search for passages related to a question even
when they do not contain exactly the same words.

## Data flow

```text
Markdown files
            |
            v
DirectoryLoader -> text chunks -> OpenAI embeddings -> Chroma database
                                                                                                                    ^
                                                                                                                    |
User question -> question embedding -> similarity search
                                                                                                                    |
                                                     retrieved context + question
                                                                                                                    |
                                                                                                                    v
                                                                                             OpenAI chat model -> answer
```

## Limitations

- The application currently uses OpenAI for both embeddings and text generation.
- The Chroma database is local and must be rebuilt when source documents change.
- The answer quality depends on chunk size, similarity threshold, retrieved
    documents, and the available OpenAI model quota.
