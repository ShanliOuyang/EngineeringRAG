# Agentic RAG Framework for Complex Engineering Calculations

This repository contains a research implementation of a cross-lingual, agentic Retrieval-Augmented Generation (RAG) framework for complex electrical engineering calculations. The project is organized as a **four-notebook Jupyter ablation study** and is designed to reduce hallucinations, formula mismatch, and reasoning drift when large language models solve multi-step engineering problems that require strict control of formulas, variables, units, and applicability conditions.

## Table of Contents

- [Project Overview](#project-overview)
- [Key Features](#key-features)
- [System Workflow](#system-workflow)
- [Core Algorithms and Methods](#core-algorithms-and-methods)
- [Experimental Results](#experimental-results)
- [Ablation Study Insights](#ablation-study-insights)
- [Repository Structure](#repository-structure)
- [Data Format](#data-format)
- [Implementation Details](#implementation-details)
- [How to Run](#how-to-run)
- [Dependencies](#dependencies)
- [Notes and Limitations](#notes-and-limitations)

## Project Overview

Traditional RAG pipelines often retrieve supporting knowledge only once for the entire question. That design is usually too coarse for engineering calculation tasks, where a single problem may contain several dependent computational steps and each step may require a different formula or applicability condition.

This project implements a structured **Plan-Retrieve-Solve-Critique** workflow:

1. Decompose a complex engineering problem into dependency-ordered sub-problems.
2. Retrieve formula knowledge for each sub-problem rather than for the question only once.
3. Re-rank retrieved candidates using an LLM to select the formula whose physical meaning best matches the current step.
4. Generate a constrained solution from retrieved engineering knowledge.
5. Critique the answer for logical consistency, formula correctness, and unit validity.
6. Iterate when the retrieval similarity or critique score is too low.

The framework also uses **cross-lingual reasoning**: Chinese questions are translated into English for decomposition and reasoning, while preserving the original Chinese input for alignment and comparison.

## Key Features

- Cross-lingual pipeline from Chinese engineering questions to English intermediate reasoning.
- Multi-step problem decomposition for compound numerical reasoning tasks.
- Knowledge retrieval over a structured engineering formula base.
- Dense retrieval with a replaceable embedding model. The current local example uses **Qwen3-Embedding**.
- LLM-based semantic re-ranking over Top-3 retrieved candidates.
- Multi-role agent loop with Planner, Retriever, Solver, Critic, and Editor.
- Automatic retry mechanism based on similarity thresholds and critique scores.
- Embedding precomputation and on-disk cache for faster repeated experiments.
- Four Jupyter notebook configurations for controlled ablation experiments.
- Batch evaluation over a benchmark set of engineering calculation questions.

## System Workflow

The full `EngineeringRAG-Standard.ipynb` pipeline works as follows:

1. **Translation**
   Chinese engineering questions are translated into English to improve downstream reasoning and retrieval consistency.

2. **Problem Decomposition**
   The Planner decomposes the translated question into ordered sub-problems.

3. **Task Normalization**
   Each sub-problem is converted into a standardized `task_description` style query so it matches the structure used in the engineering knowledge base.

4. **Dense Retrieval**
   The Retriever computes the query embedding and recalls the Top-3 most similar knowledge chunks from the formula base.

5. **Semantic Re-ranking**
   The retrieved Top-3 candidates are compared by an LLM, which selects the chunk whose variables, physical conditions, and formula intent best match the sub-problem.

6. **Prompt Construction**
   The selected chunks are assembled into a comprehensive prompt containing formulas, variable definitions, context, and task descriptions.

7. **Constrained Solving**
   The Solver generates a step-by-step answer under engineering-oriented constraints.

8. **Critique and Revision**
   The Critic scores the answer and decides whether revision is needed. If the score is below threshold or retrieval confidence is too low, the Editor adjusts the sub-problems and the loop runs again.

## Core Algorithms and Methods

### 1. Cross-Lingual Reasoning

- Input language: Chinese
- Intermediate reasoning language: English
- Purpose: improve decomposition quality and formula matching while keeping the system usable for Chinese-speaking users

### 2. Structured Knowledge Retrieval

The knowledge base is stored as JSONL chunks. Each chunk typically contains:

- a formula
- full contextual explanation
- variable definitions
- keywords
- potential question patterns
- a standardized task description
- chapter/section metadata
- difficulty metadata

This structure lets retrieval focus on procedural engineering intent rather than only keyword overlap.

### 3. Dense Embedding Retrieval

The retriever uses a dense embedding model to encode:

- knowledge chunk task descriptions
- normalized query task descriptions

In the current implementation, the local example model is **Qwen3-Embedding**, but users can replace it with another local or API-based embedding model if they keep the same retrieval interface.

Retrieval is performed by similarity scoring between the query embedding and precomputed chunk embeddings.

### 4. Semantic Re-ranking

In the full framework, the Top-3 retrieved chunks are passed to the LLM for semantic re-ranking. This stage checks:

- formula applicability
- variable compatibility
- physical meaning alignment
- whether the chunk matches the intended computational step

This is one of the most important contributions of the project because engineering formulas often share similar vocabulary while differing in scope and usage conditions.

### 5. Multi-Agent Self-Correction Loop

The full framework uses five roles:

- **Planner**: translates and decomposes the problem
- **Retriever**: generates task descriptions, recalls candidates, and selects the best chunk
- **Solver**: produces a solution using retrieved engineering knowledge
- **Critic**: checks correctness, consistency, completeness, and unit validity
- **Editor**: revises sub-problems for the next iteration when the critique fails

### 6. Threshold-Guided Iteration

The full notebook includes configurable control logic such as:

- maximum iteration rounds
- minimum similarity threshold
- minimum critique score threshold

If retrieval confidence is too low or the Critic detects major issues, the pipeline retries rather than accepting the current answer.

## Experimental Results

The framework was evaluated on a dataset of **300 electrical engineering problems**. The full configuration (**Standard**) significantly outperformed all baselines across Recall, Precision, and F1 Score.

### Main Results

| Configuration | Recall (%) | Precision (%) | F1 Score |
| :--- | :---: | :---: | :---: |
| Monolingual Chinese | 60.87 | 55.27 | 0.5567 |
| No Agent Loop | 59.00 | 47.49 | 0.5019 |
| No GPT Re-rank | 55.07 | 46.74 | 0.4735 |
| **Full Framework** | **72.96** | **60.49** | **0.6356** |

### Error Analysis

| Configuration | Miss Rate (%) | Redundancy Rate (%) |
| :--- | :---: | :---: |
| Monolingual Chinese | 39.13 | 44.73 |
| No Agent Loop | 41.00 | 52.51 |
| No GPT Re-rank | 44.93 | 53.26 |
| **Full Framework** | **27.04** | **39.51** |

## Ablation Study Insights

Based on the experimental results, the main findings are:

- **Semantic re-ranking is critical.** Removing the re-ranking stage caused the largest F1 drop (`0.6356 -> 0.4735`), showing that vector similarity alone is not reliable enough for engineering formulas.
- **The agent loop improves robustness.** Removing iterative self-correction reduced F1 from `0.6356` to `0.5019`, indicating that critique-driven revision recovers from incorrect intermediate steps.
- **Cross-lingual reasoning is beneficial.** Using English as an intermediate reasoning language improved Recall by `7.89%` over the Chinese-only baseline.

## Repository Structure

```text
EngineeringRAG-ToGit/
├── README.md
├── EngineeringRAG-Standard.ipynb
├── EngineeringRAG-noAgentLoop.ipynb
├── EngineeringRAG-noRerank.ipynb
├── EngineeringRAG-Chinese.ipynb
├── EE_Question/
│   ├── EE_Question(1-99).txt
│   ├── EE_Question(100-288).txt
│   └── EE_Question_Batch.txt
├── jsons_EE_Chi/
└── jsons_EE_Eng/
```

### Notebook Roles

These four notebooks are the core of the ablation study and correspond to four experimental settings rather than four unrelated demos.

- `EngineeringRAG-Standard.ipynb`
  Full framework with cross-lingual reasoning, Top-3 retrieval, LLM semantic re-ranking, and iterative agent correction.

- `EngineeringRAG-noAgentLoop.ipynb`
  Ablation variant that keeps decomposition and retrieval but removes iterative revision. The pipeline runs only once.

- `EngineeringRAG-noRerank.ipynb`
  Ablation variant that removes the semantic re-ranking stage and selects retrieved chunks only by embedding similarity.

- `EngineeringRAG-Chinese.ipynb`
  Chinese-first baseline used for cross-lingual comparison.

## Data Format

The engineering knowledge base is organized as JSONL files. A typical English chunk contains fields such as:

```json
{
  "chunk_id": "altitude_correction-001",
  "formula": "K_a = e^{q H / 8150}",
  "full_context": "...",
  "variable_definitions": [
    {
      "symbol": "K_a",
      "name": "Altitude correction factor",
      "meaning": "...",
      "unit": "None"
    }
  ],
  "keywords": ["altitude correction", "external insulation"],
  "potential_questions": ["..."],
  "task_description": "1. Determine ... 2. Compute ... 3. Output ...",
  "section": "Appendix B",
  "difficulty_level": "Medium"
}
```

The Chinese knowledge base uses the same structure with Chinese field names such as `公式`, `完整上下文`, `变量定义`, and `关键词`.

## Implementation Details

### Main Components in Code

- **Knowledge loading**
  Reads all JSONL chunks from the selected knowledge base directory.

- **Text normalization**
  Cleans LaTeX and Markdown fragments before embedding and prompt construction.

- **Embedding cache**
  Precomputes chunk embeddings and stores them on disk to avoid repeated encoding.

- **Task description generation**
  Converts each sub-problem into a standardized procedural retrieval query.

- **Top-K retrieval**
  Recalls the most similar chunks using embedding similarity.

- **Prompt assembly**
  Builds rich prompts that include original question, translated question, sub-problems, retrieved formulas, variable definitions, and contextual constraints.

- **Critique parsing**
  Parses LLM critique output into a structured score-and-feedback format for iterative control.

### Output Artifacts

During batch execution, the notebooks generate:

- per-question output prompt files
- answer logs
- run logs under `logs/`
- embedding cache folders such as `emb_cache_eng/`

Typical output directories referenced in the notebooks include:

- `Prompt_Standard/`
- `Prompt_noAgentLoop/`
- `Prompt_noRerank/`
- `Prompt_chi/`

## How to Run

This project is primarily notebook-based. A typical workflow is:

1. Prepare your API configuration in `config.py`.
2. Choose your own LLM API and embedding setup.
3. If you want to reproduce the current default implementation, prepare a local `Qwen3-Embedding` model and configure your API client accordingly.
4. Open the target notebook:
   - `EngineeringRAG-Standard.ipynb`
   - `EngineeringRAG-noAgentLoop.ipynb`
   - `EngineeringRAG-noRerank.ipynb`
   - `EngineeringRAG-Chinese.ipynb`
5. Set the paths for:
   - question file
   - JSONL knowledge base folder
   - output directory
   - log directory
6. If needed, replace the default LLM or embedding model calls with your own preferred API or local model.
7. Run the cells in order.
8. For batch evaluation, point the notebook to `EE_Question/EE_Question_Batch.txt`.

## Dependencies

The notebooks import and use the following main libraries:

- `openai`
- `torch`
- `transformers`
- `tqdm`
- `numpy`
- Python standard libraries such as `os`, `json`, `re`, `gc`, `time`, and `pathlib`

Model and service dependencies in the current default implementation:

- **LLM example**: `deepseek-ai/DeepSeek-V3`
- **Embedding example**: `Qwen3-Embedding`
- **API wrapper**: OpenAI-compatible client interface

Both model choices are replaceable. Users can freely switch to their own API provider, model endpoint, or embedding backend.

## Notes and Limitations

- The project is implemented mainly as research notebooks rather than a packaged Python module.
- Paths in the notebooks may need to be adjusted for a new environment.
- API credentials are not intended to be public and should be configured locally.
- The current notebooks show one concrete implementation choice for the LLM API and embedding model, but these are not mandatory fixed dependencies.
- The framework is specialized for structured engineering calculation tasks and may not transfer directly to open-ended QA without redesign.
- The repository includes the experimental assets and notebooks used for ablation analysis, so reproducibility is strongest at the notebook level.

<img width="1833" height="728" alt="system-overview" src="https://github.com/user-attachments/assets/d4cf6c64-c8ef-4e3c-b273-73b08e37350a" />
