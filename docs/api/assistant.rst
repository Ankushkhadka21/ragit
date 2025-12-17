RAGAssistant API
================

The ``RAGAssistant`` class provides a high-level interface for RAG operations.

.. warning::

   ``RAGAssistant`` is **NOT thread-safe**. Each thread should have its own instance.
   See :doc:`../integration` for thread-safe patterns.

Class Reference
---------------

.. autoclass:: ragit.RAGAssistant
   :members:
   :undoc-members:
   :show-inheritance:

Quick Reference
---------------

Constructor
^^^^^^^^^^^

.. code-block:: python

   from ragit import RAGAssistant

   assistant = RAGAssistant(
       source,                          # str, Path, or list of sources
       chunk_size=512,                  # Characters per chunk
       chunk_overlap=50,                # Overlap between chunks
       llm_model="llama3",              # LLM model name
       embedding_model="mxbai-embed-large",  # Embedding model name
       pattern="*.txt",                 # File pattern for directories
       recursive=False                  # Recursive directory search
   )

Methods
^^^^^^^

ask()
"""""

Ask a question and get an answer using RAG.

.. code-block:: python

   answer = assistant.ask("What is the installation process?")
   print(answer)

retrieve()
""""""""""

Retrieve relevant chunks without generating an answer.

.. code-block:: python

   results = assistant.retrieve("database configuration", top_k=5)

   for chunk, score in results:
       print(f"Score: {score:.3f}")
       print(f"Source: {chunk.doc_id}")
       print(f"Content: {chunk.content[:100]}...")

generate()
""""""""""

Direct LLM generation without retrieval.

.. code-block:: python

   response = assistant.generate("Explain what RAG means")
   print(response)

generate_code()
"""""""""""""""

Generate code with context from documents.

.. code-block:: python

   code = assistant.generate_code("Write a function to parse JSON")
   print(code)

Examples
--------

Basic Usage
^^^^^^^^^^^

.. code-block:: python

   from ragit import RAGAssistant

   # Create assistant
   assistant = RAGAssistant("docs/")

   # Ask questions
   answer = assistant.ask("How do I get started?")
   print(answer)

Custom Configuration
^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit import RAGAssistant

   assistant = RAGAssistant(
       "docs/",
       chunk_size=1024,                    # Larger chunks
       chunk_overlap=100,                  # More overlap
       llm_model="llama3:70b",             # Larger model
       embedding_model="nomic-embed-text"  # Different embeddings
   )

Multiple Document Sources
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit import RAGAssistant

   # Load from multiple sources
   assistant = RAGAssistant([
       "README.md",           # Single file
       "docs/",               # Directory
       "examples/tutorial/"   # Another directory
   ])

Recursive Loading
^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit import RAGAssistant

   # Load all markdown files recursively
   assistant = RAGAssistant(
       "project/",
       pattern="**/*.md",
       recursive=True
   )

Getting Sources with Answers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit import RAGAssistant

   assistant = RAGAssistant("docs/")

   question = "How do I configure logging?"

   # Get sources
   sources = assistant.retrieve(question, top_k=3)

   # Get answer
   answer = assistant.ask(question)

   print(f"Answer: {answer}\n")
   print("Based on:")
   for chunk, score in sources:
       print(f"  - {chunk.doc_id} (relevance: {score:.2f})")

Internal Attributes
-------------------

These attributes are available but considered internal:

.. code-block:: python

   # Access loaded chunks (immutable tuple)
   chunks = assistant._chunks
   print(f"Total chunks: {len(chunks)}")

   # Access embedding matrix (numpy array)
   matrix = assistant._embedding_matrix
   print(f"Matrix shape: {matrix.shape}")

   # Access provider
   provider = assistant.provider
   print(f"Provider: {provider.provider_name}")
