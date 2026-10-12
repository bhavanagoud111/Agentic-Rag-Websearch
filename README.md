# Agentic RAG with Web Search using CrewAI

An advanced Retrieval-Augmented Generation (RAG) system that enhances local document querying with real-time web search capabilities. This application leverages a multi-agent team built with CrewAI to provide comprehensive answers by searching both a user-uploaded PDF and the web.

## Features

- **PDF Knowledge Base**: Upload a PDF to create a dynamic, searchable knowledge base.
- **Hybrid Search**: Combines semantic search on your local document with real-time web search using Exa.
- **Multi-Agent System**: Utilizes a CrewAI team of specialized agents for database search, web search, and answer generation.
- **Vector Storage**: Powered by Qdrant for efficient vector storage and similarity search.
- **Conversational Interface**: An intuitive chat interface built with Streamlit.
- **AI Observability**: Optional AgentOps integration for tracing and monitoring agent performance.

## Architecture

The system uses a sequential CrewAI process:

1. **PDF Processing**: A user-uploaded PDF is processed by `pdfplumber`, converted into embeddings using OpenAI, and stored in a Qdrant vector database.
2. **DB Search Agent**: This agent first queries the Qdrant database to find context relevant to the user's query from the uploaded document.
3. **Web Search Agent**: Next, an agent uses the EXA Search tool to gather up-to-date, relevant information from the web.
4. **Answer Agent**: Finally, a master agent synthesizes the information from both the PDF context and the web search results to generate a comprehensive, well-formatted answer.

```
┌────────────────┐   ┌──────────────────┐    ┌─────────────────┐
│   PDF Upload   │──▶│  OpenAI Embeddings │───▶│  Qdrant VectorDB│
└────────────────┘   └──────────────────┘    └─────────────────┘
        │                                              │
        │                                              ▼
┌────────────────┐   ┌──────────────────┐    ┌─────────────────┐
│   User Query   │──▶│      CrewAI      │◀───│ DB Search Agent │
└────────────────┘   │ (Sequential Flow)│    └─────────────────┘
        │            └──────────────────┘              │
        │                      │                       ▼
        │                      │             ┌─────────────────┐
        │                      └────────────▶│ Web Search Agent│
        │                                    │    (Exa Tool)   │
        │                                    └─────────────────┘
        │                                              │
        ▼                                              ▼
┌────────────────┐   ┌──────────────────┐    ┌─────────────────┐
│ RAG Response   │◀──│   Answer Agent   │◀───│  Combined Context │
└────────────────┘   └──────────────────┘    └─────────────────┘
```

## Quick Start

### Prerequisites

- Python 3.11+
- OpenAI API Key
- Qdrant API Key & URL
- Exa API Key
- AgentOps API Key (Optional, for observability)

### Installation

1. **Clone the repository**:
    ```bash
    git clone https://github.com/bhavanagoud111/Agentic-Rag-Websearch.git
    cd Agentic-Rag-Websearch
    ```

2. **Install dependencies**:
    This project uses `uv` for package management. No lockfile is committed, so `uv sync` resolves the dependencies from `pyproject.toml` and creates a local `uv.lock`.
    ```bash
    pip install uv
    uv sync
    ```

3. **Set up environment variables**:
    Copy `.env.example` to `.env` and fill in your API keys:
    ```env
    OPENAI_API_KEY="your_openai_api_key"
    QDRANT_API_KEY="your_qdrant_api_key"
    QDRANT_URL="your_qdrant_cluster_url"
    EXA_API_KEY="your_exa_api_key"
    AGENTOPS_API_KEY="your_agentops_api_key"   # optional
    QDRANT_COLLECTION_NAME="rag_with_web_search"   # optional, this is the default
    ```
    `OPENAI_API_KEY`, `QDRANT_URL`, `QDRANT_API_KEY` and `EXA_API_KEY` are required; the app shows an error and stops if any of them is missing. AgentOps tracing is only enabled when `AGENTOPS_API_KEY` is set. `QDRANT_COLLECTION_NAME` sets the Qdrant collection the PDF is loaded into.

4. **Run the application**:
    ```bash
    uv run streamlit run main.py
    ```

## Usage Guide

1. **Check API Keys**: The keys are read from your `.env` file when the app starts; the sidebar shows the configured Qdrant and Exa values.
2. **Upload a PDF**: Use the file uploader in the sidebar to select a PDF. The application will automatically process it and load it into your Qdrant collection.
3. **Ask a Question**: Once the PDF is loaded, use the chat input to ask a question.
4. **Get an Answer**: The agent crew will start its process. The final, synthesized answer, combining knowledge from the PDF and the web, will be displayed in the chat.

## Configuration

The core logic is defined in `crews.py` and `qdrant_tool.py`.

### Agents & Tasks (`crews.py`)

- **`db_search_agent`**: Searches the Qdrant vector database.
- **`search_agent`**: Searches the web using `EXASearchTool`.
- **`answer_agent`**: Compiles the final response.
- The `Crew` is configured to run these agents in a sequential process.

### Qdrant & Embeddings (`qdrant_tool.py`)

- **PDF Extraction**: Uses `pdfplumber` to extract text.
- **Embeddings**: Generates embeddings using OpenAI's `text-embedding-3-large` model.
- **Vector Store**: Creates a collection in Qdrant and upserts the document vectors. The collection size is configured for `3072` dimensions.

## Key Components & Technologies

- **[CrewAI](https://github.com/crewAI/crewAI)**: Multi-agent framework for orchestrating the RAG workflow.
- **[Streamlit](https://streamlit.io/)**: Web interface for the chat application.
- **[Qdrant](https://qdrant.tech/)**: Vector database for storing and searching PDF embeddings.
- **[Exa](https://exa.ai/)**: AI-powered search engine for real-time web queries.
- **[OpenAI](https://openai.com/)**: For generating embeddings and powering the agents.
- **[AgentOps](https://agentops.ai/)**: For monitoring and tracing the agent execution flow.
- **[PDFPlumber](https://github.com/jsvine/pdfplumber)**: For robust text extraction from PDF files.


