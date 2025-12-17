Core Concepts
=============

This page explains the fundamental concepts behind ragit and RAG systems in general.

What is RAG?
------------

RAG (Retrieval-Augmented Generation) is a technique that enhances LLM responses by retrieving relevant context from a knowledge base before generating an answer.

The RAG Pipeline
^^^^^^^^^^^^^^^^

.. code-block:: text

   User Question
        |
        v
   +------------------+
   |  1. Embed Query  |  Convert question to vector
   +------------------+
        |
        v
   +------------------+
   |  2. Retrieve     |  Find similar chunks from index
   +------------------+
        |
        v
   +------------------+
   |  3. Augment      |  Add context to prompt
   +------------------+
        |
        v
   +------------------+
   |  4. Generate     |  LLM produces answer
   +------------------+
        |
        v
   Final Answer

Why RAG Matters
^^^^^^^^^^^^^^^

- **Current Information**: LLMs have knowledge cutoffs; RAG provides up-to-date context
- **Domain Knowledge**: Add specialized knowledge without fine-tuning
- **Reduced Hallucination**: Grounding responses in retrieved facts
- **Transparency**: Know exactly what sources informed the answer

Document Chunking
-----------------

Documents are split into smaller pieces called "chunks" for efficient retrieval.

Chunk Size
^^^^^^^^^^

The number of characters in each chunk significantly affects retrieval quality:

- **Small chunks (128-256)**: More precise retrieval but may lose context
- **Medium chunks (512-1024)**: Good balance of precision and context
- **Large chunks (2048+)**: More context but less precise matching

.. code-block:: python

   from ragit import chunk_text

   # Small chunks for precise retrieval
   small_chunks = chunk_text(text, chunk_size=256, chunk_overlap=25)

   # Large chunks for more context
   large_chunks = chunk_text(text, chunk_size=1024, chunk_overlap=100)

Chunk Overlap
^^^^^^^^^^^^^

Overlap ensures information at chunk boundaries isn't lost:

.. code-block:: text

   Without overlap:
   [Chunk 1: "The quick brown"][Chunk 2: "fox jumps over"]

   With overlap:
   [Chunk 1: "The quick brown fox"][Chunk 2: "brown fox jumps over"]

Typical overlap values: 10-20% of chunk size.

Embeddings
----------

Embeddings are numerical vector representations of text that capture semantic meaning.

How Embeddings Work
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit.providers import OllamaProvider

   provider = OllamaProvider()

   # Similar sentences have similar embeddings
   emb1 = provider.embed("The cat sat on the mat", model="mxbai-embed-large")
   emb2 = provider.embed("A feline rested on the rug", model="mxbai-embed-large")
   emb3 = provider.embed("Python is a programming language", model="mxbai-embed-large")

   # emb1 and emb2 will be similar (both about cats)
   # emb3 will be different (about programming)

Embedding Models
^^^^^^^^^^^^^^^^

ragit supports any embedding model available through Ollama:

- **mxbai-embed-large**: General-purpose, 1024 dimensions
- **nomic-embed-text**: Good for long documents, 768 dimensions
- **all-minilm**: Lightweight, 384 dimensions

Vector Similarity Search
------------------------

When you ask a question, ragit finds chunks with similar embeddings.

Cosine Similarity
^^^^^^^^^^^^^^^^^

ragit uses cosine similarity to compare embeddings:

.. code-block:: text

   similarity = dot(query, document) / (|query| * |document|)

   Range: -1 to 1
   - 1.0: Identical meaning
   - 0.0: Unrelated
   - -1.0: Opposite meaning

Pre-normalized Embeddings
^^^^^^^^^^^^^^^^^^^^^^^^^

ragit pre-normalizes embeddings at index time for fast retrieval:

.. code-block:: python

   # Instead of computing full cosine similarity each time:
   # similarity = dot(a, b) / (norm(a) * norm(b))

   # We normalize once at index time:
   # normalized_a = a / norm(a)
   # normalized_b = b / norm(b)

   # Then similarity is just a dot product:
   # similarity = dot(normalized_a, normalized_b)

This makes retrieval O(1) per vector instead of O(n).

Top-K Retrieval
---------------

The ``top_k`` parameter controls how many chunks are retrieved:

.. code-block:: python

   from ragit import RAGAssistant

   assistant = RAGAssistant("docs/")

   # Retrieve more chunks for complex questions
   results = assistant.retrieve("Explain the architecture", top_k=10)

   # Retrieve fewer for simple lookups
   results = assistant.retrieve("What is the version?", top_k=2)

Trade-offs
^^^^^^^^^^

- **Higher top_k**: More context, but may include irrelevant information
- **Lower top_k**: More focused, but may miss relevant information

Typical values: 3-5 for focused answers, 5-10 for comprehensive responses.

Prompt Augmentation
-------------------

Retrieved chunks are added to the LLM prompt:

.. code-block:: text

   System: You are a helpful assistant. Answer based on the context.

   Context:
   [Chunk 1 content]
   [Chunk 2 content]
   [Chunk 3 content]

   Question: {user_question}

   Answer:

The LLM then generates a response grounded in the provided context.

RAG Evaluation Metrics
----------------------

ragit evaluates RAG quality using three metrics:

Answer Correctness
^^^^^^^^^^^^^^^^^^

How well the generated answer matches the expected answer.

.. code-block:: python

   # Evaluates semantic similarity between:
   # - Generated answer
   # - Ground truth answer

Context Relevance
^^^^^^^^^^^^^^^^^

How relevant the retrieved chunks are to the question.

.. code-block:: python

   # Evaluates whether retrieved chunks contain
   # information needed to answer the question

Faithfulness
^^^^^^^^^^^^

Whether the answer is supported by the retrieved context.

.. code-block:: python

   # Checks that the answer doesn't contain
   # information not present in the context

Combined Score
^^^^^^^^^^^^^^

The final score combines all three metrics:

.. code-block:: python

   final_score = mean(answer_correctness, context_relevance, faithfulness)

Hyperparameter Optimization
---------------------------

RAG quality is sensitive to hyperparameters. ragit optimizes:

Indexing Parameters
^^^^^^^^^^^^^^^^^^^

- **chunk_size**: Characters per chunk (256, 512, 1024)
- **chunk_overlap**: Overlap between chunks (25, 50, 100)

Inference Parameters
^^^^^^^^^^^^^^^^^^^^

- **num_chunks**: Chunks to retrieve (3, 5, 7)
- **llm_model**: Model for generation
- **embedding_model**: Model for embeddings

The optimization process tests combinations and ranks by evaluation score.

Thread Safety
-------------

Important: ``RAGAssistant`` and ``SimpleVectorStore`` are **NOT thread-safe**.

.. code-block:: python

   # WRONG: Sharing assistant between threads
   assistant = RAGAssistant("docs/")

   def handle_request(question):
       return assistant.ask(question)  # Race condition!

   # CORRECT: Each thread gets its own instance
   def handle_request(question):
       assistant = RAGAssistant("docs/")  # Or use thread-local storage
       return assistant.ask(question)

For web applications, see the :doc:`integration` guide for proper patterns.
