RAG Optimization
================

This guide covers how to use ragit's optimization engine to find the best hyperparameters for your RAG pipeline.

Why Optimize?
-------------

RAG quality is highly sensitive to hyperparameters:

- **Chunk size**: Too small loses context, too large dilutes relevance
- **Chunk overlap**: Affects information continuity at boundaries
- **Top-k retrieval**: More chunks = more context but also more noise
- **Model selection**: Different models have different strengths

Manual tuning is time-consuming. ragit automates this process.

Setting Up an Experiment
------------------------

Prepare Documents
^^^^^^^^^^^^^^^^^

First, prepare your documents:

.. code-block:: python

   from ragit import Document, load_directory

   # Option 1: Create documents manually
   documents = [
       Document(
           id="intro",
           content="ragit is a RAG optimization library...",
           metadata={"source": "intro.txt"}
       ),
       Document(
           id="install",
           content="Install ragit using pip install ragit...",
           metadata={"source": "install.txt"}
       ),
   ]

   # Option 2: Load from files
   from ragit import load_directory
   docs = load_directory("docs/", pattern="*.txt")
   documents = [
       Document(id=doc.id, content=doc.content, metadata=doc.metadata)
       for doc in docs
   ]

Create Benchmark Questions
^^^^^^^^^^^^^^^^^^^^^^^^^^

Create questions with expected answers:

.. code-block:: python

   from ragit import BenchmarkQuestion

   benchmark = [
       BenchmarkQuestion(
           question="How do I install ragit?",
           ground_truth="Install using pip install ragit",
           context="Installation documentation"
       ),
       BenchmarkQuestion(
           question="What is the default chunk size?",
           ground_truth="512 characters",
           context="Configuration section"
       ),
       BenchmarkQuestion(
           question="Is RAGAssistant thread-safe?",
           ground_truth="No, RAGAssistant is not thread-safe",
           context="Thread safety documentation"
       ),
       BenchmarkQuestion(
           question="What embedding models are supported?",
           ground_truth="mxbai-embed-large, nomic-embed-text, all-minilm",
           context="Model documentation"
       ),
       BenchmarkQuestion(
           question="How do I configure Ollama URL?",
           ground_truth="Set the OLLAMA_BASE_URL environment variable",
           context="Configuration section"
       ),
   ]

Guidelines for good benchmark questions:

- Use 5-20 questions for meaningful results
- Cover different aspects of your documents
- Make ground truth answers clear and specific
- Include both simple lookups and complex reasoning questions

Running the Experiment
----------------------

Basic Experiment
^^^^^^^^^^^^^^^^

Run with default search space:

.. code-block:: python

   from ragit import RagitExperiment, Document, BenchmarkQuestion

   experiment = RagitExperiment(documents, benchmark)
   results = experiment.run()

   # Results are sorted by score (best first)
   print(f"Tested {len(results)} configurations")

   best = results[0]
   print(f"\nBest configuration: {best.pattern_name}")
   print(f"Final score: {best.final_score:.3f}")
   print(f"Execution time: {best.execution_time:.1f}s")

Custom Search Space
^^^^^^^^^^^^^^^^^^^

Define your own hyperparameter ranges:

.. code-block:: python

   from ragit import RagitExperiment

   experiment = RagitExperiment(documents, benchmark)

   # Define custom search space
   configs = experiment.define_search_space(
       chunk_sizes=[256, 512, 1024, 2048],
       chunk_overlaps=[0, 25, 50, 100],
       num_chunks=[3, 5, 7, 10],
       llm_models=["llama3", "mistral"],
       embedding_models=["mxbai-embed-large"],
       max_configs=50  # Limit total configurations
   )

   print(f"Search space: {len(configs)} configurations")

   # Run with custom configs
   results = experiment.run(configs=configs)

Limiting Configurations
^^^^^^^^^^^^^^^^^^^^^^^

For faster iteration, limit the search:

.. code-block:: python

   # Quick test with fewer configs
   configs = experiment.define_search_space(
       chunk_sizes=[512],
       chunk_overlaps=[50],
       num_chunks=[3, 5],
       max_configs=10
   )

   results = experiment.run(configs=configs)

Understanding Results
---------------------

Examining Results
^^^^^^^^^^^^^^^^^

.. code-block:: python

   from ragit import RagitExperiment

   experiment = RagitExperiment(documents, benchmark)
   results = experiment.run()

   # Top 5 configurations
   print("Top 5 Configurations:")
   print("-" * 60)

   for i, result in enumerate(results[:5], 1):
       print(f"\n{i}. {result.pattern_name}")
       print(f"   Score: {result.final_score:.3f}")
       print(f"   Time: {result.execution_time:.1f}s")
       print(f"   Indexing: {result.indexing_params}")
       print(f"   Inference: {result.inference_params}")

       # Detailed scores
       for metric, values in result.scores.items():
           print(f"   {metric}: {values['mean']:.3f}")

Result Attributes
^^^^^^^^^^^^^^^^^

Each ``EvaluationResult`` contains:

.. code-block:: python

   result = results[0]

   # Configuration name
   result.pattern_name        # "Pattern_1"

   # Indexing hyperparameters
   result.indexing_params     # {"chunk_size": 512, "chunk_overlap": 50}

   # Inference hyperparameters
   result.inference_params    # {"num_chunks": 5, "llm_model": "llama3"}

   # Evaluation scores
   result.scores              # {"answer_correctness": {"mean": 0.85}, ...}

   # Combined score
   result.final_score         # 0.82

   # Time taken
   result.execution_time      # 45.3 seconds

Evaluation Metrics
^^^^^^^^^^^^^^^^^^

Each configuration is scored on three metrics:

1. **Answer Correctness**: Semantic similarity between generated and expected answers
2. **Context Relevance**: How relevant the retrieved chunks are to the question
3. **Faithfulness**: Whether the answer is supported by the retrieved context

.. code-block:: python

   result = results[0]

   print("Detailed Scores:")
   print(f"  Answer Correctness: {result.scores['answer_correctness']['mean']:.3f}")
   print(f"  Context Relevance: {result.scores['context_relevance']['mean']:.3f}")
   print(f"  Faithfulness: {result.scores['faithfulness']['mean']:.3f}")

Applying Optimal Settings
-------------------------

Use the best configuration in your application:

.. code-block:: python

   from ragit import RAGAssistant, RagitExperiment

   # Run experiment
   experiment = RagitExperiment(documents, benchmark)
   results = experiment.run()
   best = results[0]

   # Extract optimal parameters
   chunk_size = best.indexing_params["chunk_size"]
   chunk_overlap = best.indexing_params["chunk_overlap"]
   llm_model = best.inference_params.get("llm_model", "llama3")

   # Create optimized assistant
   assistant = RAGAssistant(
       "docs/",
       chunk_size=chunk_size,
       chunk_overlap=chunk_overlap,
       llm_model=llm_model
   )

   # Use the optimized assistant
   answer = assistant.ask("Your question here")

Saving and Loading Results
--------------------------

Export results for analysis:

.. code-block:: python

   import json
   from ragit import RagitExperiment

   experiment = RagitExperiment(documents, benchmark)
   results = experiment.run()

   # Export to JSON
   results_data = [result.to_dict() for result in results]
   with open("experiment_results.json", "w") as f:
       json.dump(results_data, f, indent=2)

   # Export best config
   best = results[0]
   config = {
       "chunk_size": best.indexing_params["chunk_size"],
       "chunk_overlap": best.indexing_params["chunk_overlap"],
       "num_chunks": best.inference_params.get("num_chunks", 5),
       "llm_model": best.inference_params.get("llm_model"),
       "final_score": best.final_score
   }
   with open("best_config.json", "w") as f:
       json.dump(config, f, indent=2)

Advanced Optimization
---------------------

Custom Provider
^^^^^^^^^^^^^^^

Use a custom provider for the experiment:

.. code-block:: python

   from ragit import RagitExperiment
   from ragit.providers import OllamaProvider

   # Custom provider with different settings
   provider = OllamaProvider(
       base_url="http://gpu-server:11434",
       timeout=300
   )

   experiment = RagitExperiment(
       documents,
       benchmark,
       provider=provider
   )
   results = experiment.run()

Progress Tracking
^^^^^^^^^^^^^^^^^

The experiment shows progress using tqdm:

.. code-block:: text

   Evaluating configurations: 100%|██████████| 24/24 [05:32<00:00, 13.83s/it]

For programmatic progress tracking:

.. code-block:: python

   from ragit import RagitExperiment

   experiment = RagitExperiment(documents, benchmark)

   # Access results incrementally
   configs = experiment.define_search_space(max_configs=10)

   for i, config in enumerate(configs):
       result = experiment.evaluate_config(config)
       print(f"Config {i+1}/10: {result.final_score:.3f}")

Optimization Tips
-----------------

Start Broad, Then Narrow
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   # Phase 1: Broad search
   configs = experiment.define_search_space(
       chunk_sizes=[256, 512, 1024],
       chunk_overlaps=[25, 50, 100],
       num_chunks=[3, 5, 7],
       max_configs=30
   )
   results = experiment.run(configs=configs)

   # Find best chunk_size from Phase 1
   best_chunk_size = results[0].indexing_params["chunk_size"]

   # Phase 2: Fine-tune around best
   fine_configs = experiment.define_search_space(
       chunk_sizes=[best_chunk_size - 128, best_chunk_size, best_chunk_size + 128],
       chunk_overlaps=[25, 50, 75, 100],
       num_chunks=[4, 5, 6],
       max_configs=20
   )
   final_results = experiment.run(configs=fine_configs)

Quality vs Speed Trade-offs
^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **Smaller chunks**: Faster embedding, more precise retrieval
- **Fewer num_chunks**: Faster generation, less context
- **Smaller LLM**: Faster responses, potentially lower quality

.. code-block:: python

   # Optimize for speed
   fast_configs = experiment.define_search_space(
       chunk_sizes=[256, 512],
       num_chunks=[2, 3],
       llm_models=["mistral"]  # Fast model
   )

   # Optimize for quality
   quality_configs = experiment.define_search_space(
       chunk_sizes=[512, 1024],
       num_chunks=[5, 7, 10],
       llm_models=["llama3:70b"]  # Large model
   )

Representative Benchmark
^^^^^^^^^^^^^^^^^^^^^^^^

Your benchmark should reflect real usage:

- Include questions users actually ask
- Cover different difficulty levels
- Test edge cases and corner cases
- Update benchmark as usage patterns change
