Contributing
============

Thank you for your interest in contributing to ragit! This guide will help you get started.

Development Setup
-----------------

1. Clone the repository:

.. code-block:: bash

   git clone https://github.com/rodmena-limited/ragit.git
   cd ragit

2. Create a virtual environment:

.. code-block:: bash

   python -m venv .venv
   source .venv/bin/activate  # Linux/macOS
   # or
   .venv\Scripts\activate     # Windows

3. Install in development mode:

.. code-block:: bash

   pip install -e ".[dev]"

4. Verify the installation:

.. code-block:: bash

   pytest
   ruff check .
   mypy --strict ragit/

Code Quality Standards
----------------------

All code must pass these checks before merging:

Linting
^^^^^^^

.. code-block:: bash

   # Check for issues
   ruff check .

   # Auto-fix issues
   ruff check --fix .

Formatting
^^^^^^^^^^

.. code-block:: bash

   # Check formatting
   ruff format --check .

   # Apply formatting
   ruff format .

Type Checking
^^^^^^^^^^^^^

.. code-block:: bash

   # Strict type checking
   mypy --strict ragit/

Testing
^^^^^^^

.. code-block:: bash

   # Run all tests
   pytest

   # Run with coverage (minimum 90%)
   pytest --cov=ragit --cov-report=term-missing

   # Run specific test file
   pytest tests/unit/test_assistant.py

   # Run integration tests (requires Ollama)
   pytest -m integration

Code Style Guidelines
---------------------

1. **Type hints**: All functions must have complete type hints (mypy --strict compliant)

2. **Docstrings**: Use NumPy-style docstrings for public API:

.. code-block:: python

   def my_function(param1: str, param2: int = 10) -> list[str]:
       """
       Short description of the function.

       Parameters
       ----------
       param1 : str
           Description of param1.
       param2 : int, optional
           Description of param2 (default: 10).

       Returns
       -------
       list[str]
           Description of return value.

       Examples
       --------
       >>> result = my_function("hello", 5)
       >>> print(result)
       """
       pass

3. **Line length**: Maximum 120 characters

4. **Imports**: Sorted by ruff (isort compatible)

5. **No emojis/unicode**: Keep output ASCII-compatible

6. **Immutability**: Prefer tuples over lists for data that shouldn't change

File Headers
------------

All Python files must include:

.. code-block:: python

   #
   # Copyright RODMENA LIMITED 2025
   # SPDX-License-Identifier: Apache-2.0
   #

Testing Guidelines
------------------

Test Structure
^^^^^^^^^^^^^^

.. code-block:: text

   tests/
   ├── conftest.py              # Shared fixtures
   ├── unit/                    # Unit tests (mocked dependencies)
   │   ├── test_assistant.py
   │   ├── test_config.py
   │   └── ...
   └── integration/             # Integration tests (real Ollama)
       └── test_highway_rag.py

Writing Tests
^^^^^^^^^^^^^

.. code-block:: python

   import pytest
   from ragit import RAGAssistant

   class TestRAGAssistant:
       """Tests for RAGAssistant class."""

       def test_ask_returns_string(self, mock_provider):
           """ask() should return a string answer."""
           assistant = RAGAssistant("docs/")
           result = assistant.ask("What is this?")
           assert isinstance(result, str)

       def test_retrieve_returns_list(self, mock_provider):
           """retrieve() should return list of (chunk, score) tuples."""
           assistant = RAGAssistant("docs/")
           results = assistant.retrieve("query", top_k=3)
           assert isinstance(results, list)
           assert len(results) <= 3

Using Fixtures
^^^^^^^^^^^^^^

.. code-block:: python

   # In conftest.py
   import pytest
   from unittest.mock import MagicMock

   @pytest.fixture
   def mock_provider():
       """Mock OllamaProvider for unit tests."""
       provider = MagicMock()
       provider.is_available.return_value = True
       provider.generate.return_value = MagicMock(text="Test answer")
       provider.embed.return_value = MagicMock(
           embedding=tuple([0.1] * 1024),
           dimensions=1024
       )
       return provider

Adding New Features
-------------------

1. **Create an issue** first to discuss the feature

2. **Write tests** before implementing

3. **Implement the feature** following code style guidelines

4. **Update documentation** if needed

5. **Submit a pull request**

Adding New Providers
--------------------

To add a new provider (e.g., OpenAI, Anthropic):

1. Create ``ragit/providers/newprovider.py``

2. Inherit from ``BaseLLMProvider`` and/or ``BaseEmbeddingProvider``

3. Implement required methods:

.. code-block:: python

   from ragit.providers.base import (
       BaseLLMProvider,
       BaseEmbeddingProvider,
       LLMResponse,
       EmbeddingResponse
   )

   class NewProvider(BaseLLMProvider, BaseEmbeddingProvider):
       @property
       def provider_name(self) -> str:
           return "newprovider"

       @property
       def dimensions(self) -> int:
           return self._dimensions

       def generate(self, prompt: str, model: str, ...) -> LLMResponse:
           # Implementation
           pass

       def embed(self, text: str, model: str) -> EmbeddingResponse:
           # Return tuple for embedding, not list
           return EmbeddingResponse(embedding=tuple(embedding), ...)

       def embed_batch(self, texts: list[str], model: str) -> list[EmbeddingResponse]:
           # Single API call for all texts
           pass

       def is_available(self) -> bool:
           # Health check
           pass

4. Add configuration to ``ragit/config.py``

5. Export in ``ragit/providers/__init__.py``

6. Write tests in ``tests/unit/test_providers_newprovider.py``

Pull Request Process
--------------------

1. Fork the repository

2. Create a feature branch:

.. code-block:: bash

   git checkout -b feature/my-new-feature

3. Make your changes

4. Run all checks:

.. code-block:: bash

   ruff check .
   ruff format .
   mypy --strict ragit/
   pytest --cov=ragit

5. Commit with a clear message:

.. code-block:: bash

   git commit -m "Add feature: description of what it does"

6. Push and create a pull request:

.. code-block:: bash

   git push origin feature/my-new-feature

Issue Tracking
--------------

This project uses ``issuedb-cli`` for issue tracking. See the project's CLAUDE.md for commands.

.. code-block:: bash

   # Create an issue
   issuedb-cli create -t "Bug: description" --priority high

   # Start working on an issue
   issuedb-cli start ID

   # Close when done
   issuedb-cli stop --close

Getting Help
------------

- Open an issue on GitHub for bugs or feature requests
- Check existing issues before creating new ones
- Provide clear reproduction steps for bugs

License
-------

By contributing, you agree that your contributions will be licensed under the Apache-2.0 license.
