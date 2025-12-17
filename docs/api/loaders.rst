Loaders API
===========

The loaders module provides utilities for loading documents and splitting them into chunks.

Document Loading
----------------

load_text()
^^^^^^^^^^^

Load a single text file as a Document.

.. autofunction:: ragit.load_text

.. code-block:: python

   from ragit import load_text

   # Load a single file
   doc = load_text("README.md")

   print(f"ID: {doc.id}")           # "README" (filename without extension)
   print(f"Length: {len(doc.content)}")
   print(f"Metadata: {doc.metadata}")
   # {"source": "README.md", "filename": "README.md"}

load_directory()
^^^^^^^^^^^^^^^^

Load all matching files from a directory.

.. autofunction:: ragit.load_directory

.. code-block:: python

   from ragit import load_directory

   # Load all .txt files
   docs = load_directory("docs/", pattern="*.txt")
   print(f"Loaded {len(docs)} documents")

   # Load all .rst files
   docs = load_directory("docs/", pattern="*.rst")

   # Load recursively
   docs = load_directory("project/", pattern="**/*.md", recursive=True)

   # Process loaded documents
   for doc in docs:
       print(f"  {doc.id}: {len(doc.content)} chars")

Text Chunking
-------------

chunk_text()
^^^^^^^^^^^^

Split text into overlapping chunks.

.. autofunction:: ragit.chunk_text

.. code-block:: python

   from ragit import chunk_text

   text = """
   This is a long document that needs to be split into smaller
   chunks for efficient retrieval. Each chunk should contain
   enough context to be meaningful on its own, while also
   overlapping with adjacent chunks to preserve continuity.
   """

   # Basic chunking
   chunks = chunk_text(text, chunk_size=100, chunk_overlap=20)
   print(f"Created {len(chunks)} chunks")

   for chunk in chunks:
       print(f"Chunk {chunk.chunk_index}: {len(chunk.content)} chars")
       print(f"  Content: {chunk.content[:50]}...")

   # Custom document ID
   chunks = chunk_text(
       text,
       chunk_size=256,
       chunk_overlap=50,
       doc_id="my_document"
   )

chunk_document()
^^^^^^^^^^^^^^^^

Split a Document into overlapping chunks.

.. autofunction:: ragit.chunk_document

.. code-block:: python

   from ragit import load_text, chunk_document

   # Load and chunk a document
   doc = load_text("guide.txt")
   chunks = chunk_document(doc, chunk_size=512, chunk_overlap=50)

   print(f"Document '{doc.id}' split into {len(chunks)} chunks")

   for chunk in chunks:
       print(f"  Chunk {chunk.chunk_index}: {chunk.content[:50]}...")

chunk_by_separator()
^^^^^^^^^^^^^^^^^^^^

Split text by a separator (e.g., paragraphs, sections).

.. autofunction:: ragit.chunk_by_separator

.. code-block:: python

   from ragit import chunk_by_separator

   text = """
   First paragraph with some content.

   Second paragraph with different content.

   Third paragraph with more information.
   """

   # Split by double newline (paragraphs)
   chunks = chunk_by_separator(text, separator="\n\n", doc_id="article")

   for chunk in chunks:
       print(f"Paragraph {chunk.chunk_index}: {chunk.content}")

   # Split by custom separator
   markdown = """
   # Section 1
   Content for section 1.

   ---

   # Section 2
   Content for section 2.
   """

   sections = chunk_by_separator(markdown, separator="\n---\n", doc_id="readme")

chunk_rst_sections()
^^^^^^^^^^^^^^^^^^^^

Split RST documents by section headers.

.. autofunction:: ragit.chunk_rst_sections

.. code-block:: python

   from ragit import chunk_rst_sections

   rst_content = """
   Introduction
   ============

   This is the introduction section.

   Installation
   ------------

   How to install the software.

   Usage
   -----

   How to use the software.
   """

   chunks = chunk_rst_sections(rst_content, doc_id="manual")

   for chunk in chunks:
       print(f"Section {chunk.chunk_index}:")
       print(f"  {chunk.content[:100]}...")

Data Classes
------------

Chunk
^^^^^

.. autoclass:: ragit.core.experiment.experiment.Chunk
   :members:
   :undoc-members:

.. code-block:: python

   from ragit.core.experiment.experiment import Chunk

   chunk = Chunk(
       content="The actual text content of the chunk",
       doc_id="document_name",
       chunk_index=0,
       embedding=None  # Set after embedding
   )

   print(chunk.content)
   print(chunk.doc_id)
   print(chunk.chunk_index)

Document
^^^^^^^^

.. autoclass:: ragit.Document
   :members:
   :undoc-members:

.. code-block:: python

   from ragit import Document

   doc = Document(
       id="readme",
       content="Full document content here...",
       metadata={
           "source": "README.md",
           "version": "1.0",
           "author": "John Doe"
       }
   )

Complete Examples
-----------------

Loading and Chunking a Project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit import load_directory, chunk_document

   # Load all Python files
   docs = load_directory("src/", pattern="**/*.py", recursive=True)
   print(f"Loaded {len(docs)} Python files")

   # Chunk all documents
   all_chunks = []
   for doc in docs:
       chunks = chunk_document(doc, chunk_size=512, chunk_overlap=50)
       all_chunks.extend(chunks)
       print(f"  {doc.id}: {len(chunks)} chunks")

   print(f"\nTotal chunks: {len(all_chunks)}")

Processing Different File Types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit import load_text, chunk_document, chunk_rst_sections

   def load_and_chunk(path: str) -> list:
       """Load and chunk a file based on its type."""
       doc = load_text(path)

       if path.endswith(".rst"):
           # Use section-based chunking for RST
           return chunk_rst_sections(doc.content, doc_id=doc.id)
       else:
           # Use overlap chunking for other formats
           return chunk_document(doc, chunk_size=512, chunk_overlap=50)

   # Usage
   rst_chunks = load_and_chunk("docs/guide.rst")
   md_chunks = load_and_chunk("README.md")
   txt_chunks = load_and_chunk("notes.txt")

Building a Document Index
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit import load_directory, chunk_document
   from ragit.providers import OllamaProvider

   # Load documents
   docs = load_directory("knowledge_base/", pattern="*.txt")

   # Chunk all documents
   all_chunks = []
   for doc in docs:
       chunks = chunk_document(doc, chunk_size=512, chunk_overlap=50)
       all_chunks.extend(chunks)

   # Create embeddings
   provider = OllamaProvider()
   texts = [chunk.content for chunk in all_chunks]
   embeddings = provider.embed_batch(texts, model="mxbai-embed-large")

   # Build index (chunks with embeddings)
   indexed_chunks = [
       {
           "content": chunk.content,
           "doc_id": chunk.doc_id,
           "chunk_index": chunk.chunk_index,
           "embedding": emb.embedding
       }
       for chunk, emb in zip(all_chunks, embeddings, strict=True)
   ]

   print(f"Indexed {len(indexed_chunks)} chunks")
