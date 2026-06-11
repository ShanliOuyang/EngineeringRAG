# Agentic RAG Framework for Complex Engineering Calculations

This repository contains the implementation of a Cross-Lingual Agentic Retrieval-Augmented Generation (RAG) framework designed for solving complex calculation problems in electrical engineering. The project addresses specific challenges Large Language Models (LLMs) face with domain-specific formulas, such as hallucinations and logical drift in multi-step reasoning.

## Project Overview

Traditional RAG pipelines often fail in engineering contexts because they retrieve context only once for the entire query. This project introduces a "Plan-Execute-Review" architecture that mimics human problem-solving:

1.  **Decomposition:** Breaking complex problems into linear sub-tasks.
2.  **Targeted Retrieval:** Performing granular, task-oriented search for each step.
3.  **Self-Verification:** Employing a Critic agent to ensure logical consistency and unit correctness.

By leveraging cross-lingual processing (Chinese input, English reasoning), the system utilizes the stronger logical capabilities of LLMs while remaining accessible to Chinese-speaking users.

## Experimental Results

The framework was evaluated on a dataset of **300 electrical engineering problems**. The full configuration (**Standard**) significantly outperformed all baselines across Recall, Precision, and F1 Score.

### Main Results

| Configuration | Recall (%) | Precision (%) | F1 Score |
| :--- | :---: | :---: | :---: |
| Monolingual Chinese  | 60.87 | 55.27 | 0.5567 |
| No Agent Loop | 59.00 | 47.49 | 0.5019 |
| No GPT Re-rank | 55.07 | 46.74 | 0.4735 |
| **Full Framework** | **72.96** | **60.49** | **0.6356** |

### Error Analysis

| Configuration | Miss Rate (%) | Redundancy Rate (%) |
| :--- | :---: | :---: |
| Monolingual Chinese  | 39.13 | 44.73 |
| No Agent Loop  | 41.00 | 52.51 |
| No GPT Re-rank | 44.93 | 53.26 |
| **Full Framework** | **27.04** | **39.51** |

## Ablation Study Insights

Based on the experimental results, we observed the following contributions of each component:

*   **Semantic Re-ranking (Most Critical):** Removing the GPT-based re-ranker caused the largest drop in performance (F1: 0.6356 → 0.4735). Vector similarity alone is insufficient for engineering formulas that share similar keywords but differ in applicability conditions.
*   **Agent Loop (Error Recovery):** The self-correction mechanism provided a significant boost (F1: 0.5019 → 0.6356), allowing the system to recover from incorrect formula applications.
*   **Cross-Lingual Processing:** Using English as an intermediate reasoning language improved performance by 7.89% in Recall compared to the Chinese-only version.

## Repository Structure

The repository is organized into datasets and experimental notebooks used for the ablation studies.

### Datasets
*   `EE_Question/`: Source data for the engineering problems.
*   `jsons_EE_Chi/`: Structured JSON data for Chinese-language engineering problems.
*   `jsons_EE_Eng/`: Structured JSON data for English-language engineering problems (used for intermediate reasoning).

### Configuration

*   `config.py`: API configuration file (not included in the repository). Please refer to the note below.
*   **Embedding Model**: The project uses Qwen3-Embedding for vector embeddings. The model files are not included in this repository due to size constraints. Users can download the Qwen3-Embedding model locally and configure the path, or replace it with an API-based embedding service. For reference configuration, see the `Qwen3-Embedding/` directory structure in the codebase.

> **Note:** The `config.py` file is not uploaded to this repository as it contains private API keys. If you wish to use this project, please create your own `config.py` file and configure it with your own API credentials.

### Experimental Notebooks

All four notebooks share the **same API configuration** to ensure a fair comparison across different experimental settings. They correspond exactly to the experimental configurations in the paper:

*   `EngineeringRAG-Standard.ipynb`
    *   **Description:** The complete framework. Includes Chinese-to-English translation, problem decomposition, LLM-based semantic re-ranking, and the multi-agent self-correction loop.

*   `EngineeringRAG-noAgentLoop.ipynb`
    *   **Description:** Ablation study removing the agent loop. This runs the pipeline only once (Plan → Retrieve → Solve → Output) without self-verification.

*   `EngineeringRAG-noRerank.ipynb`
    *   **Description:** Ablation study removing the LLM-based re-ranker. It relies solely on vector embedding similarity (Top-1 selection).

*   `EngineeringRAG-Chinese.ipynb`
    *   **Description:** Comparison baseline running entirely in Chinese.

## Methodology Summary

1.  **Planner:** Decomposes the query into dependent sub-problems using Chain-of-Thought (CoT).
2.  **Retriever:** Uses a dual-stage search (Embedding + Re-ranking) to fetch precise formulas from a knowledge base of 555 structured entries.
3.  **Solver:** Generates calculation steps constrained strictly by the retrieved context.
4.  **Critic:** Reviews the solution for unit consistency and logic, triggering re-retrieval if the score is low.

<img width="1833" height="728" alt="image" src="https://github.com/user-attachments/assets/d4cf6c64-c8ef-4e3c-b273-73b08e37350a" />
