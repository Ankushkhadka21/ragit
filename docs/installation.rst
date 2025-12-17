Installation
============

This guide covers how to install ragit for different use cases.

Prerequisites
-------------

- Python 3.12 or higher (up to 3.14)
- pip package manager
- Ollama server (local or cloud) for LLM and embedding operations

Installing from PyPI
--------------------

The easiest way to install ragit is using pip:

.. code-block:: bash

   pip install ragit

Installing from Source
----------------------

If you want to install from the source code:

.. code-block:: bash

   git clone https://github.com/rodmena-limited/ragit.git
   cd ragit
   pip install .

Development Installation
------------------------

For development purposes, install in editable mode with development dependencies:

.. code-block:: bash

   git clone https://github.com/rodmena-limited/ragit.git
   cd ragit
   pip install -e ".[dev]"

This installs additional tools for testing, linting, and type checking:

- pytest, pytest-cov (testing)
- ruff (linting and formatting)
- mypy (type checking)

Setting Up Ollama
-----------------

ragit uses Ollama for LLM and embedding operations. You can run Ollama locally or use a cloud provider.

Local Ollama
^^^^^^^^^^^^

1. Install Ollama from https://ollama.ai/download

2. Pull the required models:

.. code-block:: bash

   # Pull an LLM model
   ollama pull llama3

   # Pull an embedding model
   ollama pull mxbai-embed-large

3. Start the Ollama server:

.. code-block:: bash

   ollama serve

Cloud Ollama
^^^^^^^^^^^^

For cloud Ollama providers, set the appropriate environment variables:

.. code-block:: bash

   export OLLAMA_BASE_URL="https://your-cloud-ollama-url"
   export OLLAMA_API_KEY="your-api-key"

Verification
------------

To verify that ragit is installed correctly:

.. code-block:: python

   import ragit
   print(ragit.__version__)

   # Check if Ollama is available
   from ragit.providers import OllamaProvider
   provider = OllamaProvider()
   print(f"Ollama available: {provider.is_available()}")

If both commands run without error, ragit is properly installed and configured.

Next Steps
----------

- Follow the :doc:`quickstart` guide to build your first RAG application
- Read about :doc:`configuration` options for customizing ragit
