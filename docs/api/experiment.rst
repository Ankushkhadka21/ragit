Experiment API
==============

The experiment module provides RAG hyperparameter optimization.

RagitExperiment
---------------

.. autoclass:: ragit.RagitExperiment
   :members:
   :undoc-members:
   :show-inheritance:

Data Classes
------------

Document
^^^^^^^^

.. autoclass:: ragit.Document
   :members:
   :undoc-members:

.. code-block:: python

   from ragit import Document

   doc = Document(
       id="readme",
       content="Your document content here...",
       metadata={"source": "README.md", "version": "1.0"}
   )

BenchmarkQuestion
^^^^^^^^^^^^^^^^^

.. autoclass:: ragit.BenchmarkQuestion
   :members:
   :undoc-members:

.. code-block:: python

   from ragit import BenchmarkQuestion

   question = BenchmarkQuestion(
       question="How do I install the library?",
       ground_truth="Use pip install ragit",
       context="Installation section of documentation"
   )

EvaluationResult
^^^^^^^^^^^^^^^^

.. autoclass:: ragit.core.experiment.results.EvaluationResult
   :members:
   :undoc-members:

.. code-block:: python

   # Accessing result attributes
   result = results[0]

   print(result.pattern_name)        # "Pattern_1"
   print(result.final_score)         # 0.85
   print(result.execution_time)      # 45.3
   print(result.indexing_params)     # {"chunk_size": 512, ...}
   print(result.inference_params)    # {"num_chunks": 5, ...}
   print(result.scores)              # {"answer_correctness": {...}, ...}

ExperimentResults
^^^^^^^^^^^^^^^^^

.. autoclass:: ragit.core.experiment.results.ExperimentResults
   :members:
   :undoc-members:

.. code-block:: python

   from ragit import RagitExperiment

   experiment = RagitExperiment(documents, benchmark)
   results = experiment.run()

   # Iterate over results
   for result in results:
       print(result)

   # Get best results
   top_5 = results.get_best(k=5)

   # Get sorted results
   sorted_results = results.sorted(reverse=True)

   # Check if configuration was tested
   cached_score = results.is_cached(
       indexing_params={"chunk_size": 512},
       inference_params={"num_chunks": 5}
   )

Quick Reference
---------------

Creating an Experiment
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit import RagitExperiment, Document, BenchmarkQuestion

   # Prepare documents
   documents = [
       Document(id="doc1", content="...", metadata={}),
       Document(id="doc2", content="...", metadata={}),
   ]

   # Create benchmark
   benchmark = [
       BenchmarkQuestion(
           question="Question 1?",
           ground_truth="Expected answer 1",
           context="Context 1"
       ),
       BenchmarkQuestion(
           question="Question 2?",
           ground_truth="Expected answer 2",
           context="Context 2"
       ),
   ]

   # Create experiment
   experiment = RagitExperiment(documents, benchmark)

Running with Default Settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   # Run with default search space
   results = experiment.run()

   # Get best configuration
   best = results[0]
   print(f"Best score: {best.final_score:.3f}")

Custom Search Space
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   # Define custom search space
   configs = experiment.define_search_space(
       chunk_sizes=[256, 512, 1024],
       chunk_overlaps=[25, 50, 100],
       num_chunks=[3, 5, 7],
       llm_models=["llama3", "mistral"],
       embedding_models=["mxbai-embed-large"],
       max_configs=50
   )

   # Run with custom configs
   results = experiment.run(configs=configs)

Evaluating Single Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit.core.experiment.experiment import RAGConfig

   # Create a specific configuration
   config = RAGConfig(
       chunk_size=512,
       chunk_overlap=50,
       num_chunks=5,
       llm_model="llama3",
       embedding_model="mxbai-embed-large"
   )

   # Evaluate single config
   result = experiment.evaluate_config(config)
   print(f"Score: {result.final_score:.3f}")

Analyzing Results
^^^^^^^^^^^^^^^^^

.. code-block:: python

   results = experiment.run()

   # Summary statistics
   scores = results.scores
   print(f"Min score: {min(scores):.3f}")
   print(f"Max score: {max(scores):.3f}")
   print(f"Mean score: {sum(scores)/len(scores):.3f}")

   # Best by different criteria
   for result in results[:5]:
       print(f"\n{result.pattern_name}:")
       print(f"  Final: {result.final_score:.3f}")
       print(f"  Correctness: {result.scores['answer_correctness']['mean']:.3f}")
       print(f"  Relevance: {result.scores['context_relevance']['mean']:.3f}")
       print(f"  Faithfulness: {result.scores['faithfulness']['mean']:.3f}")

Exporting Results
^^^^^^^^^^^^^^^^^

.. code-block:: python

   import json

   results = experiment.run()

   # Export all results
   export_data = []
   for result in results:
       export_data.append(result.to_dict())

   with open("experiment_results.json", "w") as f:
       json.dump(export_data, f, indent=2)

   # Export best config
   best = results[0]
   best_config = {
       "chunk_size": best.indexing_params["chunk_size"],
       "chunk_overlap": best.indexing_params["chunk_overlap"],
       "num_chunks": best.inference_params.get("num_chunks"),
       "score": best.final_score
   }

   with open("best_config.json", "w") as f:
       json.dump(best_config, f, indent=2)

SimpleVectorStore
-----------------

Internal vector store used by the experiment engine.

.. warning::

   ``SimpleVectorStore`` is **NOT thread-safe**.

.. autoclass:: ragit.core.experiment.experiment.SimpleVectorStore
   :members:
   :undoc-members:

.. code-block:: python

   from ragit.core.experiment.experiment import SimpleVectorStore, Chunk

   # Create vector store
   store = SimpleVectorStore()

   # Add chunks with embeddings
   chunk = Chunk(
       content="Some text content",
       doc_id="doc1",
       chunk_index=0,
       embedding=(0.1, 0.2, 0.3, ...)  # Pre-computed embedding
   )
   store.add(chunk)

   # Search
   query_embedding = (0.15, 0.25, 0.35, ...)
   results = store.search(query_embedding, top_k=5)

   for chunk, score in results:
       print(f"{chunk.doc_id}: {score:.3f}")

   # Clear store
   store.clear()
