Platform Integration
====================

This guide shows how to integrate ragit with popular frameworks and tools.

Flask Integration
-----------------

Basic Flask Application
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from flask import Flask, request, jsonify
   from ragit import RAGAssistant

   app = Flask(__name__)

   # Create assistant at startup (NOT thread-safe, see below)
   assistant = RAGAssistant("docs/")

   @app.route("/ask", methods=["POST"])
   def ask():
       data = request.json
       question = data.get("question", "")

       if not question:
           return jsonify({"error": "No question provided"}), 400

       answer = assistant.ask(question)
       return jsonify({"answer": answer})

   @app.route("/health")
   def health():
       return jsonify({"status": "ok"})

   if __name__ == "__main__":
       app.run(debug=True)

Thread-Safe Flask Application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For production with multiple workers, use a factory pattern:

.. code-block:: python

   from flask import Flask, request, jsonify, g
   from ragit import RAGAssistant

   app = Flask(__name__)

   def get_assistant():
       """Get or create assistant for current request context."""
       if "assistant" not in g:
           g.assistant = RAGAssistant(
               "docs/",
               chunk_size=512,
               chunk_overlap=50
           )
       return g.assistant

   @app.teardown_appcontext
   def teardown_assistant(exception):
       """Clean up assistant after request."""
       g.pop("assistant", None)

   @app.route("/ask", methods=["POST"])
   def ask():
       data = request.json
       question = data.get("question", "")

       if not question:
           return jsonify({"error": "No question provided"}), 400

       assistant = get_assistant()
       answer = assistant.ask(question)

       return jsonify({
           "question": question,
           "answer": answer
       })

   @app.route("/retrieve", methods=["POST"])
   def retrieve():
       data = request.json
       question = data.get("question", "")
       top_k = data.get("top_k", 3)

       assistant = get_assistant()
       results = assistant.retrieve(question, top_k=top_k)

       return jsonify({
           "question": question,
           "results": [
               {
                   "content": chunk.content,
                   "doc_id": chunk.doc_id,
                   "score": score
               }
               for chunk, score in results
           ]
       })

   if __name__ == "__main__":
       app.run(debug=True, threaded=True)

Using with curl:

.. code-block:: bash

   # Ask a question
   curl -X POST http://localhost:5000/ask \
        -H "Content-Type: application/json" \
        -d '{"question": "How do I install ragit?"}'

   # Retrieve context
   curl -X POST http://localhost:5000/retrieve \
        -H "Content-Type: application/json" \
        -d '{"question": "What is RAG?", "top_k": 5}'

FastAPI Integration
-------------------

Basic FastAPI Application
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from fastapi import FastAPI, HTTPException
   from pydantic import BaseModel
   from ragit import RAGAssistant

   app = FastAPI(title="RAG API", version="1.0.0")

   # Global assistant (for simple deployments)
   assistant = RAGAssistant("docs/")

   class QuestionRequest(BaseModel):
       question: str
       top_k: int = 3

   class AnswerResponse(BaseModel):
       question: str
       answer: str

   class RetrieveResponse(BaseModel):
       question: str
       results: list[dict]

   @app.post("/ask", response_model=AnswerResponse)
   async def ask(request: QuestionRequest):
       if not request.question.strip():
           raise HTTPException(status_code=400, detail="Question cannot be empty")

       answer = assistant.ask(request.question)
       return AnswerResponse(question=request.question, answer=answer)

   @app.post("/retrieve", response_model=RetrieveResponse)
   async def retrieve(request: QuestionRequest):
       results = assistant.retrieve(request.question, top_k=request.top_k)

       return RetrieveResponse(
           question=request.question,
           results=[
               {
                   "content": chunk.content,
                   "doc_id": chunk.doc_id,
                   "score": float(score)
               }
               for chunk, score in results
           ]
       )

   @app.get("/health")
   async def health():
       return {"status": "ok"}

Production FastAPI with Dependency Injection
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from contextlib import asynccontextmanager
   from fastapi import FastAPI, Depends, HTTPException
   from pydantic import BaseModel
   from ragit import RAGAssistant
   from ragit.providers import OllamaProvider
   import threading

   # Thread-local storage for assistants
   _local = threading.local()

   def get_assistant() -> RAGAssistant:
       """Get thread-local assistant instance."""
       if not hasattr(_local, "assistant"):
           _local.assistant = RAGAssistant(
               "docs/",
               chunk_size=512,
               chunk_overlap=50
           )
       return _local.assistant

   @asynccontextmanager
   async def lifespan(app: FastAPI):
       # Startup: verify Ollama is available
       provider = OllamaProvider()
       if not provider.is_available():
           raise RuntimeError("Ollama server not available")
       yield
       # Shutdown: cleanup if needed

   app = FastAPI(
       title="RAG API",
       version="1.0.0",
       lifespan=lifespan
   )

   class QuestionRequest(BaseModel):
       question: str
       top_k: int = 3

   class AnswerResponse(BaseModel):
       question: str
       answer: str
       sources: list[dict] | None = None

   @app.post("/ask", response_model=AnswerResponse)
   async def ask(
       request: QuestionRequest,
       assistant: RAGAssistant = Depends(get_assistant)
   ):
       if not request.question.strip():
           raise HTTPException(status_code=400, detail="Question cannot be empty")

       # Get answer with sources
       context = assistant.retrieve(request.question, top_k=request.top_k)
       answer = assistant.ask(request.question)

       return AnswerResponse(
           question=request.question,
           answer=answer,
           sources=[
               {"doc_id": chunk.doc_id, "score": float(score)}
               for chunk, score in context
           ]
       )

   @app.get("/models")
   async def list_models():
       """List available models."""
       provider = OllamaProvider()
       return {
           "llm_models": ["llama3", "mistral", "codellama"],
           "embedding_models": ["mxbai-embed-large", "nomic-embed-text"]
       }

Running FastAPI:

.. code-block:: bash

   # Install uvicorn
   pip install uvicorn

   # Run the server
   uvicorn app:app --reload --host 0.0.0.0 --port 8000

   # With multiple workers (production)
   uvicorn app:app --workers 4 --host 0.0.0.0 --port 8000

Command Line Interface
----------------------

Basic CLI Tool
^^^^^^^^^^^^^^

.. code-block:: python

   #!/usr/bin/env python3
   """Simple RAG CLI tool."""

   import argparse
   import sys
   from ragit import RAGAssistant
   from ragit.providers import OllamaProvider

   def main():
       parser = argparse.ArgumentParser(
           description="RAG-powered Q&A from documents"
       )
       parser.add_argument(
           "docs_path",
           help="Path to documents directory"
       )
       parser.add_argument(
           "-q", "--question",
           help="Question to ask (interactive if not provided)"
       )
       parser.add_argument(
           "--chunk-size",
           type=int,
           default=512,
           help="Chunk size (default: 512)"
       )
       parser.add_argument(
           "--top-k",
           type=int,
           default=3,
           help="Number of chunks to retrieve (default: 3)"
       )
       parser.add_argument(
           "--show-sources",
           action="store_true",
           help="Show source chunks with answer"
       )

       args = parser.parse_args()

       # Check Ollama availability
       provider = OllamaProvider()
       if not provider.is_available():
           print("Error: Ollama server not available", file=sys.stderr)
           print("Start with: ollama serve", file=sys.stderr)
           sys.exit(1)

       # Create assistant
       print(f"Loading documents from {args.docs_path}...")
       assistant = RAGAssistant(
           args.docs_path,
           chunk_size=args.chunk_size
       )
       print("Ready!\n")

       def ask_question(question: str):
           if args.show_sources:
               # Get sources first
               results = assistant.retrieve(question, top_k=args.top_k)
               print("\nSources:")
               for i, (chunk, score) in enumerate(results, 1):
                   print(f"  {i}. [{chunk.doc_id}] (score: {score:.3f})")
                   print(f"     {chunk.content[:100]}...")
               print()

           answer = assistant.ask(question)
           print(f"Answer: {answer}\n")

       if args.question:
           # Single question mode
           ask_question(args.question)
       else:
           # Interactive mode
           print("Enter questions (Ctrl+C to exit):\n")
           try:
               while True:
                   question = input("Q: ").strip()
                   if question:
                       ask_question(question)
           except KeyboardInterrupt:
               print("\nGoodbye!")

   if __name__ == "__main__":
       main()

Save as ``rag_cli.py`` and use:

.. code-block:: bash

   # Single question
   python rag_cli.py docs/ -q "How do I install?"

   # Interactive mode with sources
   python rag_cli.py docs/ --show-sources

   # Custom settings
   python rag_cli.py docs/ --chunk-size 1024 --top-k 5

Advanced CLI with Click
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #!/usr/bin/env python3
   """Advanced RAG CLI with Click."""

   import click
   import json
   from ragit import RAGAssistant, RagitExperiment, Document, BenchmarkQuestion
   from ragit.providers import OllamaProvider

   @click.group()
   def cli():
       """RAG-powered document Q&A tool."""
       pass

   @cli.command()
   @click.argument("docs_path")
   @click.option("-q", "--question", help="Question to ask")
   @click.option("--chunk-size", default=512, help="Chunk size")
   @click.option("--top-k", default=3, help="Number of chunks")
   @click.option("--json-output", is_flag=True, help="Output as JSON")
   def ask(docs_path, question, chunk_size, top_k, json_output):
       """Ask questions about documents."""
       assistant = RAGAssistant(docs_path, chunk_size=chunk_size)

       if question:
           answer = assistant.ask(question)
           if json_output:
               click.echo(json.dumps({"question": question, "answer": answer}))
           else:
               click.echo(f"Answer: {answer}")
       else:
           # Interactive mode
           while True:
               try:
                   q = click.prompt("Question", default="", show_default=False)
                   if not q:
                       continue
                   answer = assistant.ask(q)
                   click.echo(f"Answer: {answer}\n")
               except click.Abort:
                   break

   @cli.command()
   @click.argument("docs_path")
   @click.argument("query")
   @click.option("--top-k", default=5, help="Number of results")
   def search(docs_path, query, top_k):
       """Search for relevant document chunks."""
       assistant = RAGAssistant(docs_path)
       results = assistant.retrieve(query, top_k=top_k)

       for i, (chunk, score) in enumerate(results, 1):
           click.echo(f"\n{i}. Score: {score:.3f} | Source: {chunk.doc_id}")
           click.echo(f"   {chunk.content[:200]}...")

   @cli.command()
   def check():
       """Check if Ollama is available."""
       provider = OllamaProvider()
       if provider.is_available():
           click.echo("Ollama is available")
           click.echo(f"URL: {provider.base_url}")
       else:
           click.echo("Ollama is NOT available", err=True)
           raise SystemExit(1)

   if __name__ == "__main__":
       cli()

Usage:

.. code-block:: bash

   # Check Ollama
   python rag_cli.py check

   # Ask question
   python rag_cli.py ask docs/ -q "What is ragit?"

   # Search documents
   python rag_cli.py search docs/ "installation instructions"

   # Interactive mode
   python rag_cli.py ask docs/

Jupyter Notebook Integration
----------------------------

Using ragit in Jupyter notebooks:

.. code-block:: python

   # Cell 1: Setup
   from ragit import RAGAssistant, load_directory
   from ragit.providers import OllamaProvider

   # Check Ollama
   provider = OllamaProvider()
   print(f"Ollama available: {provider.is_available()}")

   # Cell 2: Create assistant
   assistant = RAGAssistant(
       "docs/",
       chunk_size=512,
       chunk_overlap=50
   )
   print(f"Loaded {len(assistant._chunks)} chunks")

   # Cell 3: Interactive Q&A
   def ask(question: str):
       """Helper function for notebook Q&A."""
       print(f"Q: {question}\n")

       # Show sources
       results = assistant.retrieve(question, top_k=3)
       print("Sources:")
       for chunk, score in results:
           print(f"  - {chunk.doc_id} (score: {score:.3f})")

       # Generate answer
       answer = assistant.ask(question)
       print(f"\nA: {answer}")

   # Usage
   ask("How do I configure the database?")

   # Cell 4: Visualization (optional)
   import matplotlib.pyplot as plt

   # Visualize chunk scores for a query
   results = assistant.retrieve("What is RAG?", top_k=10)
   scores = [score for _, score in results]
   labels = [f"Chunk {i}" for i in range(len(scores))]

   plt.figure(figsize=(10, 5))
   plt.bar(labels, scores)
   plt.xlabel("Chunks")
   plt.ylabel("Similarity Score")
   plt.title("Chunk Relevance Scores")
   plt.xticks(rotation=45)
   plt.tight_layout()
   plt.show()

Docker Deployment
-----------------

Dockerfile for ragit application:

.. code-block:: dockerfile

   FROM python:3.12-slim

   WORKDIR /app

   # Install dependencies
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt

   # Copy application
   COPY app.py .
   COPY docs/ ./docs/

   # Environment variables
   ENV OLLAMA_BASE_URL=http://ollama:11434
   ENV RAGIT_DEFAULT_LLM_MODEL=llama3

   EXPOSE 8000

   CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]

Docker Compose with Ollama:

.. code-block:: yaml

   version: '3.8'

   services:
     ollama:
       image: ollama/ollama
       ports:
         - "11434:11434"
       volumes:
         - ollama_data:/root/.ollama

     rag-api:
       build: .
       ports:
         - "8000:8000"
       environment:
         - OLLAMA_BASE_URL=http://ollama:11434
       depends_on:
         - ollama

   volumes:
     ollama_data:

Running:

.. code-block:: bash

   # Start services
   docker-compose up -d

   # Pull models in Ollama container
   docker-compose exec ollama ollama pull llama3
   docker-compose exec ollama ollama pull mxbai-embed-large

   # Test the API
   curl http://localhost:8000/ask \
        -H "Content-Type: application/json" \
        -d '{"question": "What is ragit?"}'
