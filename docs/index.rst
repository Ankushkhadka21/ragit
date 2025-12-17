ragit Documentation
===================

Welcome to the official documentation for **ragit**, a RAG (Retrieval-Augmented Generation) optimization library that helps you build and optimize RAG pipelines with ease.

.. toctree::
   :maxdepth: 2
   :caption: Getting Started

   installation
   quickstart

.. toctree::
   :maxdepth: 2
   :caption: User Guide

   concepts
   configuration
   optimization
   integration

.. toctree::
   :maxdepth: 2
   :caption: API Reference

   api/assistant
   api/experiment
   api/providers
   api/loaders

.. toctree::
   :maxdepth: 1
   :caption: Community

   contributing
   changelog


About ragit
-----------

ragit provides automatic RAG hyperparameter optimization, a high-level RAG assistant for document Q&A, and pluggable provider architecture. It's designed to help you find the optimal configuration for your RAG pipelines without manual tuning.

Key Features
^^^^^^^^^^^^

- **RAG Hyperparameter Optimization**: Automatically find the best chunk size, overlap, and retrieval parameters
- **High-Level RAG Assistant**: Simple API for document Q&A with ``RAGAssistant``
- **Pluggable Providers**: Support for Ollama (local and cloud) with extensible architecture
- **Document Loading**: Built-in utilities for loading and chunking documents
- **Performance Optimized**: Pre-normalized embeddings, batch API calls, vectorized operations

Quick Example
^^^^^^^^^^^^^

.. code-block:: python

   from ragit import RAGAssistant

   # Create assistant with your documents
   assistant = RAGAssistant("docs/")

   # Ask questions
   answer = assistant.ask("How do I create a new user?")
   print(answer)


Why ragit?
^^^^^^^^^^

RAG systems are powerful but sensitive to hyperparameters. The wrong chunk size or retrieval count can dramatically reduce answer quality. ragit solves this by:

1. **Automated Optimization**: Grid search over hyperparameter combinations
2. **Evaluation Metrics**: Built-in scoring for answer correctness, context relevance, and faithfulness
3. **Simple Integration**: Works with Flask, FastAPI, CLI tools, and more


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
