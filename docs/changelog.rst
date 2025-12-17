Changelog
=========

All notable changes to ragit are documented here.

Version 0.7.1 (Current)
-----------------------

**Performance Optimizations**

- Pre-normalized embedding matrices for O(1) cosine similarity
- Batch embedding API calls instead of individual calls
- Efficient top-k selection using ``numpy.argpartition``
- Immutable ``EmbeddingResponse`` with tuple embeddings

**Code Quality**

- Full mypy --strict compliance
- 94% test coverage with 150 tests
- Thread-safety documentation for non-thread-safe classes

Version 0.7.0
-------------

**New Features**

- High-level ``RAGAssistant`` class for document Q&A
- Document loading utilities (``load_text``, ``load_directory``)
- Multiple chunking strategies (overlap, separator, RST sections)
- Code generation with ``generate_code()``

**API Changes**

- ``EmbeddingResponse.embedding`` is now a tuple (immutable)
- Added ``embed_batch()`` for efficient batch embeddings

Version 0.4.0
-------------

**New Features**

- ``RagitExperiment`` for hyperparameter optimization
- Grid search over chunk sizes, overlaps, and retrieval parameters
- Evaluation metrics: answer correctness, context relevance, faithfulness

**Improvements**

- Better error handling in provider classes
- Progress bars for long-running operations

Version 0.3.0
-------------

**New Features**

- Ollama provider for LLM and embedding operations
- Environment variable configuration
- Support for cloud Ollama providers

Version 0.2.0
-------------

**Initial Release**

- Core experiment framework
- Document and chunk data structures
- Basic vector store implementation
- Evaluation result classes

Versioning
----------

ragit follows `Semantic Versioning <https://semver.org/>`_:

- **MAJOR**: Incompatible API changes
- **MINOR**: New functionality, backwards compatible
- **PATCH**: Bug fixes, backwards compatible

Deprecation Policy
------------------

- Deprecated features are marked in documentation
- Deprecated features remain for at least one minor version
- Removal is announced in changelog

Upgrade Guide
-------------

0.6.x to 0.7.x
^^^^^^^^^^^^^^

**Breaking Changes**

1. ``EmbeddingResponse.embedding`` is now a tuple:

.. code-block:: python

   # Before (0.6.x)
   embedding = response.embedding  # list[float]

   # After (0.7.x)
   embedding = response.embedding  # tuple[float, ...]

   # If you need a list
   embedding_list = list(response.embedding)

2. ``embed_batch()`` is now required for custom providers:

.. code-block:: python

   class MyProvider(BaseEmbeddingProvider):
       def embed_batch(self, texts: list[str], model: str) -> list[EmbeddingResponse]:
           # Required implementation
           pass

**Migration Steps**

1. Update any code that modifies embeddings (they're now immutable)
2. Implement ``embed_batch()`` in custom providers
3. Run tests to verify compatibility
