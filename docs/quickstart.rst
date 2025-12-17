Quickstart Guide
================

This guide will get you up and running with ragit in just a few minutes. We'll cover the basics of creating a RAG assistant, asking questions, and optimizing your RAG pipeline.

Your First RAG Assistant
------------------------

The simplest way to use ragit is with the ``RAGAssistant`` class. It handles document loading, chunking, embedding, and retrieval automatically.

.. code-block:: python

   from ragit import RAGAssistant

   # Create an assistant with a directory of documents
   assistant = RAGAssistant("docs/")

   # Ask a question
   answer = assistant.ask("What is the main purpose of this project?")
   print(answer)

The assistant will:

1. Load all text files from the directory
2. Chunk them into smaller pieces
3. Create embeddings using Ollama
4. Find relevant chunks for your question
5. Generate an answer using the LLM

Loading Different Document Types
--------------------------------

ragit supports various document formats and loading patterns:

.. code-block:: python

   from ragit import RAGAssistant

   # Load from a single file
   assistant = RAGAssistant("README.md")

   # Load from a directory with a specific pattern
   assistant = RAGAssistant("docs/", pattern="*.rst")

   # Load recursively from nested directories
   assistant = RAGAssistant("docs/", pattern="**/*.md", recursive=True)

   # Load multiple sources
   assistant = RAGAssistant(["README.md", "docs/", "examples/"])

Customizing Chunk Settings
--------------------------

Control how documents are split into chunks:

.. code-block:: python

   from ragit import RAGAssistant

   # Larger chunks for more context
   assistant = RAGAssistant(
       "docs/",
       chunk_size=1024,      # Characters per chunk
       chunk_overlap=100     # Overlap between chunks
   )

   # Smaller chunks for more precise retrieval
   assistant = RAGAssistant(
       "docs/",
       chunk_size=256,
       chunk_overlap=50
   )

Retrieving Context
------------------

Sometimes you want to see what context the assistant finds:

.. code-block:: python

   from ragit import RAGAssistant

   assistant = RAGAssistant("docs/")

   # Get relevant chunks without generating an answer
   results = assistant.retrieve("How do I configure the database?", top_k=5)

   for chunk, score in results:
       print(f"Score: {score:.3f}")
       print(f"Content: {chunk.content[:200]}...")
       print(f"Source: {chunk.doc_id}")
       print("---")

Direct LLM Generation
---------------------

Use the assistant's LLM directly without RAG:

.. code-block:: python

   from ragit import RAGAssistant

   assistant = RAGAssistant("docs/")

   # Direct generation (no retrieval)
   response = assistant.generate("Explain what RAG means in AI")
   print(response)

   # Code generation with context
   code = assistant.generate_code(
       "Write a function to calculate fibonacci numbers"
   )
   print(code)

Using Custom Models
-------------------

Specify which Ollama models to use:

.. code-block:: python

   from ragit import RAGAssistant

   # Use specific models
   assistant = RAGAssistant(
       "docs/",
       llm_model="llama3:70b",              # Larger LLM for better answers
       embedding_model="nomic-embed-text"    # Different embedding model
   )

   answer = assistant.ask("What are the key features?")

Working with Document Loaders
-----------------------------

For more control over document loading, use the loader functions directly:

.. code-block:: python

   from ragit import load_text, load_directory, chunk_text, chunk_document

   # Load a single file
   doc = load_text("README.md")
   print(f"Loaded: {doc.id}, {len(doc.content)} characters")

   # Load all files from a directory
   docs = load_directory("docs/", pattern="*.rst")
   print(f"Loaded {len(docs)} documents")

   # Chunk text manually
   chunks = chunk_text(
       "Your long document text here...",
       chunk_size=512,
       chunk_overlap=50,
       doc_id="my_doc"
   )
   print(f"Created {len(chunks)} chunks")

   # Chunk a document
   doc = load_text("README.md")
   chunks = chunk_document(doc, chunk_size=256, chunk_overlap=25)

RST Document Chunking
---------------------

For RST documents, chunk by sections for better semantic grouping:

.. code-block:: python

   from ragit import load_text, chunk_rst_sections

   doc = load_text("docs/guide.rst")
   chunks = chunk_rst_sections(doc.content, doc_id=doc.id)

   for chunk in chunks:
       print(f"Section {chunk.chunk_index}: {chunk.content[:50]}...")

RAG Pipeline Optimization
-------------------------

Find the optimal hyperparameters for your RAG pipeline:

.. code-block:: python

   from ragit import RagitExperiment, Document, BenchmarkQuestion

   # Prepare your documents
   documents = [
       Document(
           id="doc1",
           content="Your document content here...",
           metadata={"source": "manual.txt"}
       ),
       Document(
           id="doc2",
           content="Another document...",
           metadata={"source": "guide.txt"}
       ),
   ]

   # Create benchmark questions with expected answers
   benchmark = [
       BenchmarkQuestion(
           question="What is the installation process?",
           ground_truth="Install using pip install ragit",
           context="Installation section of the documentation"
       ),
       BenchmarkQuestion(
           question="How do I configure the database?",
           ground_truth="Set the DATABASE_URL environment variable",
           context="Configuration section"
       ),
   ]

   # Run the experiment
   experiment = RagitExperiment(documents, benchmark)
   results = experiment.run()

   # Get the best configuration
   best = results[0]
   print(f"Best configuration: {best.pattern_name}")
   print(f"Score: {best.final_score:.3f}")
   print(f"Chunk size: {best.indexing_params['chunk_size']}")
   print(f"Overlap: {best.indexing_params['chunk_overlap']}")

Custom Search Space
-------------------

Define your own hyperparameter search space:

.. code-block:: python

   from ragit import RagitExperiment, Document, BenchmarkQuestion

   experiment = RagitExperiment(documents, benchmark)

   # Define custom search space
   configs = experiment.define_search_space(
       chunk_sizes=[256, 512, 1024],
       chunk_overlaps=[25, 50, 100],
       num_chunks=[3, 5, 7],
       llm_models=["llama3", "mistral"],
       max_configs=20  # Limit number of configurations to test
   )

   print(f"Testing {len(configs)} configurations")

   # Run with custom configs
   results = experiment.run(configs=configs)

Using the Provider Directly
---------------------------

For low-level control, use the Ollama provider directly:

.. code-block:: python

   from ragit.providers import OllamaProvider

   provider = OllamaProvider()

   # Check availability
   if provider.is_available():
       print("Ollama is running")

   # Generate text
   response = provider.generate(
       prompt="Explain quantum computing in simple terms",
       model="llama3",
       temperature=0.7,
       max_tokens=500
   )
   print(response.text)

   # Create embeddings
   embedding = provider.embed(
       text="This is a test sentence",
       model="mxbai-embed-large"
   )
   print(f"Embedding dimensions: {embedding.dimensions}")

   # Batch embeddings (more efficient)
   texts = ["First text", "Second text", "Third text"]
   embeddings = provider.embed_batch(texts, model="mxbai-embed-large")
   print(f"Created {len(embeddings)} embeddings")

Complete Example: Building a Q&A System
---------------------------------------

Here's a complete example that ties everything together:

.. code-block:: python

   from ragit import RAGAssistant, load_directory

   def create_qa_system(docs_path: str) -> RAGAssistant:
       """Create a Q&A system from a directory of documents."""
       return RAGAssistant(
           docs_path,
           chunk_size=512,
           chunk_overlap=50,
           llm_model="llama3",
           embedding_model="mxbai-embed-large"
       )

   def answer_question(assistant: RAGAssistant, question: str) -> dict:
       """Answer a question and return detailed results."""
       # Get context
       context = assistant.retrieve(question, top_k=3)

       # Generate answer
       answer = assistant.ask(question)

       return {
           "question": question,
           "answer": answer,
           "sources": [
               {
                   "content": chunk.content[:200],
                   "doc_id": chunk.doc_id,
                   "score": score
               }
               for chunk, score in context
           ]
       }

   # Usage
   assistant = create_qa_system("docs/")

   result = answer_question(assistant, "How do I get started?")
   print(f"Answer: {result['answer']}")
   print(f"\nSources used:")
   for source in result['sources']:
       print(f"  - {source['doc_id']} (score: {source['score']:.3f})")

Error Handling
--------------

Handle common errors gracefully:

.. code-block:: python

   from ragit import RAGAssistant
   from ragit.providers import OllamaProvider

   # Check if Ollama is available before creating assistant
   provider = OllamaProvider()
   if not provider.is_available():
       print("Error: Ollama server is not running")
       print("Start it with: ollama serve")
       exit(1)

   try:
       assistant = RAGAssistant("docs/")
       answer = assistant.ask("What is this about?")
       print(answer)
   except FileNotFoundError:
       print("Error: Document directory not found")
   except Exception as e:
       print(f"Error: {e}")

What's Next?
------------

Now that you've completed the quickstart, you can:

- Read about :doc:`concepts` to understand the core ideas behind ragit
- Learn about :doc:`configuration` for environment variables and settings
- Explore :doc:`optimization` for RAG hyperparameter tuning
- Check the :doc:`integration` guide for Flask, FastAPI, and CLI examples
- Dive into the :doc:`api/assistant` for detailed API documentation
