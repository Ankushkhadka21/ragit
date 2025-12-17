Configuration
=============

ragit is configured through environment variables, which can be set directly or loaded from a ``.env`` file.

Environment Variables
---------------------

Ollama Connection
^^^^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 30 25 45

   * - Variable
     - Default
     - Description
   * - ``OLLAMA_BASE_URL``
     - ``http://localhost:11434``
     - Base URL for Ollama LLM API
   * - ``OLLAMA_EMBEDDING_URL``
     - Same as ``OLLAMA_BASE_URL``
     - URL for embedding API (useful for split deployments)
   * - ``OLLAMA_API_KEY``
     - None
     - API key for cloud Ollama providers
   * - ``OLLAMA_TIMEOUT``
     - ``120``
     - Request timeout in seconds

Model Defaults
^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 30 35 35

   * - Variable
     - Default
     - Description
   * - ``RAGIT_DEFAULT_LLM_MODEL``
     - ``qwen3-vl:235b-instruct``
     - Default LLM model for generation
   * - ``RAGIT_DEFAULT_EMBEDDING_MODEL``
     - ``mxbai-embed-large``
     - Default model for embeddings

Logging
^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 30 25 45

   * - Variable
     - Default
     - Description
   * - ``RAGIT_LOG_LEVEL``
     - ``INFO``
     - Logging level (DEBUG, INFO, WARNING, ERROR)

Using a .env File
-----------------

Create a ``.env`` file in your project root:

.. code-block:: bash

   # .env file
   OLLAMA_BASE_URL=http://localhost:11434
   OLLAMA_TIMEOUT=180

   RAGIT_DEFAULT_LLM_MODEL=llama3
   RAGIT_DEFAULT_EMBEDDING_MODEL=mxbai-embed-large

ragit automatically loads this file using ``python-dotenv``.

Configuration Examples
----------------------

Local Development
^^^^^^^^^^^^^^^^^

Minimal configuration for local Ollama:

.. code-block:: bash

   # .env
   OLLAMA_BASE_URL=http://localhost:11434
   RAGIT_DEFAULT_LLM_MODEL=llama3

Cloud Ollama Provider
^^^^^^^^^^^^^^^^^^^^^

Configuration for cloud-hosted Ollama:

.. code-block:: bash

   # .env
   OLLAMA_BASE_URL=https://api.your-ollama-cloud.com
   OLLAMA_API_KEY=your-api-key-here
   OLLAMA_TIMEOUT=300

   # Cloud might not support embeddings, use local for that
   OLLAMA_EMBEDDING_URL=http://localhost:11434

Split LLM and Embedding Servers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use different servers for LLM and embeddings:

.. code-block:: bash

   # .env
   # Powerful cloud server for LLM generation
   OLLAMA_BASE_URL=https://llm-server.example.com
   OLLAMA_API_KEY=llm-api-key

   # Local server for embeddings (faster, no API costs)
   OLLAMA_EMBEDDING_URL=http://localhost:11434

Production Deployment
^^^^^^^^^^^^^^^^^^^^^

Production configuration with longer timeouts:

.. code-block:: bash

   # .env
   OLLAMA_BASE_URL=https://ollama.internal.company.com
   OLLAMA_API_KEY=${OLLAMA_SECRET}  # From secrets manager
   OLLAMA_TIMEOUT=300

   RAGIT_DEFAULT_LLM_MODEL=llama3:70b
   RAGIT_DEFAULT_EMBEDDING_MODEL=mxbai-embed-large
   RAGIT_LOG_LEVEL=WARNING

Accessing Configuration in Code
-------------------------------

Access configuration values programmatically:

.. code-block:: python

   from ragit.config import config

   print(f"Ollama URL: {config.OLLAMA_BASE_URL}")
   print(f"Default LLM: {config.DEFAULT_LLM_MODEL}")
   print(f"Default Embedding: {config.DEFAULT_EMBEDDING_MODEL}")
   print(f"Timeout: {config.OLLAMA_TIMEOUT}")

Overriding Configuration
------------------------

Override defaults when creating instances:

.. code-block:: python

   from ragit import RAGAssistant

   # Override model defaults
   assistant = RAGAssistant(
       "docs/",
       llm_model="mistral",                  # Override LLM model
       embedding_model="nomic-embed-text"    # Override embedding model
   )

Provider-level overrides:

.. code-block:: python

   from ragit.providers import OllamaProvider

   # Use different base URL
   provider = OllamaProvider(
       base_url="http://other-server:11434",
       api_key="different-key",
       timeout=60
   )

Model Recommendations
---------------------

LLM Models
^^^^^^^^^^

For different use cases:

.. list-table::
   :header-rows: 1
   :widths: 25 25 50

   * - Model
     - Size
     - Use Case
   * - ``llama3``
     - 8B
     - General purpose, fast responses
   * - ``llama3:70b``
     - 70B
     - Higher quality, complex questions
   * - ``mistral``
     - 7B
     - Fast, good for simple queries
   * - ``codellama``
     - 7B-34B
     - Code-related questions

Embedding Models
^^^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 30 20 50

   * - Model
     - Dimensions
     - Characteristics
   * - ``mxbai-embed-large``
     - 1024
     - High quality, general purpose
   * - ``nomic-embed-text``
     - 768
     - Good for long documents
   * - ``all-minilm``
     - 384
     - Lightweight, fast

Troubleshooting Configuration
-----------------------------

Connection Issues
^^^^^^^^^^^^^^^^^

If ragit can't connect to Ollama:

.. code-block:: python

   from ragit.providers import OllamaProvider

   provider = OllamaProvider()
   if not provider.is_available():
       print("Cannot connect to Ollama")
       print(f"URL: {provider.base_url}")
       print("Check that Ollama is running: ollama serve")

Timeout Issues
^^^^^^^^^^^^^^

For large models or slow connections:

.. code-block:: bash

   # Increase timeout
   export OLLAMA_TIMEOUT=300

Model Not Found
^^^^^^^^^^^^^^^

If a model isn't available:

.. code-block:: bash

   # List available models
   ollama list

   # Pull missing model
   ollama pull llama3
   ollama pull mxbai-embed-large
