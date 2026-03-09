# Simple LLM Application with LCEL

A text translation application built with **LangChain Expression Language (LCEL)** that translates English text into other languages. It uses the **Groq API** with the Llama 3.1 8B Instant model for fast inference and includes a FastAPI backend with a Streamlit frontend.

## Architecture

```
User Input (text, language)
       |
       v
+------------------+       +-------------------+       +------------------+
| ChatPromptTemplate| ----> | ChatGroq (LLM)   | ----> | StrOutputParser  |
| (system + user)  |       | llama-3.1-8b      |       | (plain text)     |
+------------------+       +-------------------+       +------------------+
       |
       v                    LCEL Chain: prompt | model | parser
  Translated Text
```

**Stack:**

| Component     | Technology                  |
|---------------|-----------------------------|
| LLM Framework | LangChain + LCEL            |
| Language Model| Groq (Llama 3.1 8B Instant) |
| Backend       | FastAPI + LangServe         |
| Frontend      | Streamlit                   |
| Server        | Uvicorn                     |

## Project Structure

```
Simple-LLM-with-LCEL/
├── serve.py              # FastAPI backend server with LCEL chain
├── client.py             # Streamlit frontend client
├── simplellmLCEL.ipynb   # Jupyter notebook tutorial
├── requirements.txt      # Python dependencies
├── .env                  # Environment variables (not committed)
└── .gitignore
```

## Prerequisites

- Python 3.10+
- A [Groq API key](https://console.groq.com) (free tier available)

## Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/sothulthorn/Simple-LLM-with-LCEL.git
   cd Simple-LLM-with-LCEL
   ```

2. **Create and activate a virtual environment**

   ```bash
   python -m venv .venv

   # Windows
   .venv\Scripts\activate

   # macOS/Linux
   source .venv/bin/activate
   ```

3. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

4. **Configure environment variables**

   Create a `.env` file in the project root:

   ```
   GROQ_API_KEY=your_groq_api_key_here
   ```

## Usage

### Running the Server

Start the FastAPI backend:

```bash
python serve.py
```

The server starts at `http://127.0.0.1:8000`. API docs are available at `http://127.0.0.1:8000/docs`.

### Running the Client

In a separate terminal, start the Streamlit frontend:

```bash
streamlit run client.py
```

The app opens at `http://localhost:8501`. Enter text to translate it to French.

### API Reference

**Endpoint:** `POST /chain/invoke`

**Request:**

```json
{
  "input": {
    "language": "French",
    "text": "Hello, how are you?"
  },
  "config": {},
  "kwargs": {}
}
```

**Response:**

```json
{
  "output": "Bonjour, comment allez-vous ?"
}
```

**cURL example:**

```bash
curl -X POST "http://127.0.0.1:8000/chain/invoke" \
  -H "Content-Type: application/json" \
  -d '{"input": {"language": "French", "text": "Hello, how are you?"}, "config": {}, "kwargs": {}}'
```

### Jupyter Notebook

The `simplellmLCEL.ipynb` notebook walks through the core concepts step by step:

1. Initializing a language model with ChatGroq
2. Using `SystemMessage` and `HumanMessage`
3. Parsing output with `StrOutputParser`
4. Chaining components with the LCEL pipe operator (`|`)
5. Building reusable prompt templates with `ChatPromptTemplate`
6. Composing the full chain: `prompt | model | parser`

## How It Works

The application uses three LangChain components chained together with LCEL:

1. **ChatPromptTemplate** - Formats the system instruction (`"Translate the following into {language}:"`) and user text into a structured prompt.
2. **ChatGroq** - Sends the prompt to the Groq API running Llama 3.1 8B Instant and returns the model's response.
3. **StrOutputParser** - Extracts the plain text content from the model's response object.

These are composed into a single chain using the pipe operator:

```python
chain = prompt_template | model | parser
```

The chain is then exposed as a REST API using LangServe's `add_routes`, which automatically handles serialization, deserialization, and provides an interactive playground at `/chain/playground`.
