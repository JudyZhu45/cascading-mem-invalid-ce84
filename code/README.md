# Cloned Repositories

## Repo 1: LoCoMo
- **URL**: https://github.com/snap-research/locomo
- **Purpose**: Primary evaluation dataset and benchmark code; temporal event graph generation pipeline; baseline RAG evaluation scripts
- **Location**: `code/locomo/`
- **Key Files**:
  - `data/locomo10.json` — The evaluation dataset (10 conversations)
  - `task_eval/` — Evaluation scripts for QA, event summarization
  - `generative_agents/` — Agent architecture for data generation
  - `scripts/generate_conversations.sh` — Pipeline to generate new conversations
- **Notes**: Dataset uses temporal event graph structure (causal connections between life events). This is the closest existing structure to our proposed memory graph with typed dependency edges.
- **Installation**:
  ```bash
  pip install -r code/locomo/requirements.txt
  # Requires OpenAI API key for evaluation scripts
  ```

---

## Repo 2: HorizonBench
- **URL**: https://github.com/stellalisy/HorizonBench
- **Purpose**: Benchmark for long-horizon personalization with evolving preferences; implements typed dependency edges and cascading preference updates in data generation; evaluation framework
- **Location**: `code/horizonbench/`
- **Key Files**:
  - `evaluate.py` — Main evaluation script (litellm-based, supports any frontier model)
  - `src/` — Data generation source code
  - `data/` — Local data (if available)
  - `requirements.txt` — Dependencies
- **Key Entry Points**:
  ```bash
  # Quick sanity check (10 items)
  python evaluate.py --model gpt-4o --config sample
  # Full benchmark
  python evaluate.py --model claude-sonnet-4-20250514
  ```
- **Notes**: This repo implements the closest existing version of our research hypothesis. The data generator in `src/` uses a mental state graph with typed dependency edges and cascading preference propagation (p_evo=0.15, 2-5 preferences per life event). Study this code carefully for our implementation.
- **Installation**:
  ```bash
  cd code/horizonbench && pip install -r requirements.txt
  export ANTHROPIC_API_KEY="..."
  ```

---

## Repo 3: PersonaMem
- **URL**: https://github.com/bowen-upenn/PersonaMem
- **Purpose**: PersonaMem dataset generation and evaluation code; agentic memory framework for personalization
- **Location**: `code/personamem/`
- **Key Files**:
  - `inference.py` — LLM inference for personalization
  - `prepare_data.py` — Dataset preparation
  - `prompts.py` — Prompts for persona extraction and memory update
- **Notes**: Useful for studying the agentic memory update mechanism (RFT-trained compact memory). The compact 2K-token memory that outperforms full 32K context is architecturally relevant.

---

## Repo 4: Letta (formerly MemGPT)
- **URL**: https://github.com/letta-ai/letta
- **Purpose**: Production-ready MemGPT successor with hierarchical memory (in-context working memory + external recall/archival storage); function-calling-based memory management
- **Location**: `code/letta/`
- **Key Files**:
  - See `letta/` module for agent and memory implementations
  - `examples/` — Usage examples
- **Notes**: The working context in Letta/MemGPT is the flat-text equivalent of our typed memory graph. We can adapt this architecture by replacing the flat working context with a structured graph representation that supports typed dependency edges and propagation.
- **Installation**:
  ```bash
  pip install letta
  # or: pip install -e code/letta/
  ```

---

## Relationship to Research Hypothesis

```
LoCoMo temporal event graph ─────────────────► Our memory graph (LOCATED_IN edges)
HorizonBench mental state graph ─────────────► Cascading invalidation (CONFLICTS_WITH edges)
PersonaMem agentic memory ───────────────────► Compact memory representation
Letta/MemGPT working context ────────────────► Base architecture to extend
```

### For Experiment Implementation:

1. **Scenario 1 (LOCATED_IN)**: Use LoCoMo conversations; build graph nodes from temporal events; add LOCATED_IN edges from location-dependent memories to root location node; test whether changing root invalidates downstream nodes.

2. **Scenario 2 (CONFLICTS_WITH)**: Use HorizonBench generator code as reference; implement LLM inference at write time to identify semantic conflicts; compare to HorizonBench's ground-truth dependency edges as upper bound.
