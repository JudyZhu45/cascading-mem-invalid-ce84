# Resources Catalog

## Summary

This document catalogs all resources gathered for the research project on **Cascading Memory Invalidation in User Memory Graphs**. The core hypothesis concerns propagating invalidation signals through typed dependency edges (LOCATED_IN for explicit context shifts, CONFLICTS_WITH for implicit preference drift).

---

## Papers

Total papers downloaded: 15

| Title | Authors | Year | File | Relevance |
|-------|---------|------|------|-----------|
| LoCoMo: Evaluating Very Long-Term Conv. Memory | Maharana et al. | 2024 | `papers/2402.17753_locomo_longterm_conv_memory.pdf` | **Primary dataset; temporal event graph** |
| HorizonBench: Long-Horizon Personalization | Li et al. | 2026 | `papers/2604.17283_horizonbench_evolving_prefs.pdf` | **Directly implements typed dep. edges** |
| PersonaMem-v2: Implicit User Personas | Jiang et al. | 2025 | `papers/2512.06688_personamem_v2.pdf` | **Agentic memory + dynamic prefs** |
| MemGPT: Towards LLMs as Operating Systems | Packer et al. | 2024 | `papers/2310.08560_memgpt_llm_os.pdf` | **Hierarchical memory baseline** |
| 2310.09104 (non-AI paper, incorrect ID) | Kalmes, Przestacki | 2023 | `papers/2310.09104_unknown.pdf` | Not relevant (mathematics) |
| Memory OS of AI Agent (MemoryOS) | Kang et al. | 2025 | `papers/memoryos_agent.pdf` | Strong LoCoMo baseline (+49% F1) |
| MemoryBank: Enhancing LLMs w/ Long-Term Memory | Zhong et al. | 2023 | `papers/memorybank_longterm_memory.pdf` | Temporal decay / Ebbinghaus model |
| Generative Agents: Interactive Simulacra | Park et al. | 2023 | `papers/generative_agents_park.pdf` | Foundational agent memory architecture |
| Survey on Memory Mechanism of LLM Agents | Zhang et al. | 2024 | `papers/survey_llm_agent_memory.pdf` | Memory taxonomy |
| ENGRAM: Lightweight Memory Orchestration | Patel, Patel | 2025 | `papers/engram_memory_conversational.pdf` | State-of-the-art LoCoMo; typed memory |
| Mnemosyne: Human-Inspired LT Memory | Jonelagadda et al. | 2025 | `papers/mnemosyne_longterm_memory_edge.pdf` | Graph memory + temporal decay |
| TOBUGraph: KG-Based Retrieval | Kashmira et al. | 2024 | `papers/tobugraph_kg_retrieval.pdf` | KG retrieval beyond RAG |
| Memoria: Personalized Conversational AI | Sarin et al. | 2025 | `papers/memoria_personalized_conv_ai.pdf` | KG-based user modeling |
| MetaMem: Evolving Meta-Memory | Xin et al. | 2026 | `papers/metamem_meta_memory.pdf` | Self-reflective memory optimization |
| SR-FoT (downloaded as PAMU, wrong ID) | Wan et al. | 2025 | `papers/pamu_preference_aware_memory.pdf` | Not relevant to this research |

See `papers/README.md` for detailed descriptions.

---

## Datasets

Total datasets located: 3 (2 directly accessible, 1 downloadable)

| Name | Source | Size | Task | Location | Status |
|------|--------|------|------|----------|--------|
| LoCoMo | GitHub: snap-research/locomo | 10 conversations, ~5.6MB | Long-term dialogue QA | `code/locomo/data/locomo10.json` | **Available locally** |
| HorizonBench (sample) | HuggingFace: stellalisy/HorizonBench | 10 items (sample) | Preference evolution MCQ | `datasets/horizonbench/samples.json` | Sample downloaded |
| PersonaMem-v2 | HuggingFace: bowen-upenn/PersonaMem-v2 | 1K users, 20K+ prefs | Implicit personalization | Not yet downloaded | Requires HF token |

See `datasets/README.md` for detailed download instructions.

**LoCoMo key statistics** (from `datasets/locomo/dataset_summary.json`):
- 10 conversations, avg 548 turns, avg 198 QA pairs
- All 5 QA categories present in most conversations
- Temporal event graph data included in conversation structure

---

## Code Repositories

Total repositories cloned: 4

| Name | URL | Purpose | Location | Status |
|------|-----|---------|----------|--------|
| locomo | github.com/snap-research/locomo | Dataset + evaluation scripts | `code/locomo/` | Cloned, data available |
| HorizonBench | github.com/stellalisy/HorizonBench | Benchmark + mental state graph generator | `code/horizonbench/` | Cloned |
| PersonaMem | github.com/bowen-upenn/PersonaMem | Agentic memory framework | `code/personamem/` | Cloned |
| Letta (MemGPT) | github.com/letta-ai/letta | Production hierarchical memory | `code/letta/` | Cloned |

See `code/README.md` for detailed descriptions.

---

## Resource Gathering Notes

### Search Strategy
1. Used `paper-finder` with query "cascading memory invalidation user memory graphs LLM" — returned 21 papers
2. Directly fetched metadata for all 5 user-specified arXiv IDs
3. Downloaded PDFs for all specified papers plus top relevance-scored papers from search
4. Searched for repositories via GitHub links in papers

### Selection Criteria
- **Datasets**: Must be directly relevant to long-term memory evaluation (LoCoMo primary; HorizonBench for preference evolution)
- **Papers**: Focused on (a) memory system architectures, (b) temporal/preference evaluation, (c) knowledge graph-based memory
- **Repositories**: Selected for direct relevance to experiment implementation

### Challenges Encountered
1. `snap-research/locomo` not on HuggingFace Hub — resolved by using GitHub repo directly (data in `code/locomo/data/`)
2. arXiv ID `2310.09104` is a mathematics paper, not an AI paper — included download for completeness but flagged as not relevant
3. arXiv ID `2501.11599` (downloaded as PAMU) resolved to SR-FoT (syllogistic reasoning), not the intended PAMU paper — flagged
4. PersonaMem-v2 (HuggingFace) requires authentication for full dataset download

### Gaps and Workarounds
- **PAMU (Preference-Aware Memory Update)** paper by Sun et al. (2025): Correct paper is on Semantic Scholar but arXiv ID 2501.11599 maps to a different paper. The PAMU mechanism (SW + EMA for preference representation) is documented in the literature review from paper-finder abstract.
- Full HorizonBench dataset (4,245 items) not downloaded due to size (~50GB+ for 163K token conversations). Sample split (10 items) is available and sufficient for method validation.

---

## Recommendations for Experiment Design

### 1. Primary Dataset: LoCoMo
Use `code/locomo/data/locomo10.json` for all experiments. Structure: load conversations, extract temporal event graphs, build memory graph with typed edges, evaluate QA tasks with focus on category 3 (temporal reasoning) and category 2 (multi-hop).

### 2. Baseline Methods (in order of priority)
1. **RAG with observations** — from LoCoMo paper (41.4% F1): retrieve top-5 speaker observation assertions
2. **MemoryOS** — strongest reported baseline (+49% F1 on GPT-4o-mini); code at https://github.com/BAI-LAB/MemoryOS
3. **ENGRAM** — state-of-the-art; typed episodic/semantic/procedural memory
4. **Full-context LLM** — GPT-4o or Claude-sonnet with full conversation history

### 3. Evaluation Metrics
- **Primary**: F1 score stratified by QA category (especially temporal, cat 3)
- **Secondary**: Pre-evolution distractor selection rate (adapted from HorizonBench for our memory graph invalidation)
- **Diagnostic**: Separate "invalidated memory retrieval" vs "valid memory retrieval" accuracy

### 4. Proposed System Architecture
```python
# Pseudo-code for cascading memory invalidation system
class MemoryGraph:
    nodes: Dict[str, MemoryNode]   # memory_id -> node
    edges: List[Edge]              # typed dependency edges
    
    def add_memory(self, content, timestamp):
        node = MemoryNode(content, timestamp)
        # LLM inference at write time: identify conflicts and dependencies
        conflicts = self.infer_conflicts(content)      # CONFLICTS_WITH edges
        dependencies = self.infer_dependencies(content)  # LOCATED_IN, REQUIRES_CONTEXT edges
        # ...
    
    def propagate_invalidation(self, root_node_id, new_value):
        # BFS/DFS through dependency edges
        # Down-weight/invalidate all dependent nodes
        # ...
```

### 5. Experiment Phases
1. **Phase 1**: Replicate LoCoMo baselines (RAG, MemoryOS) to establish comparison points
2. **Phase 2**: Implement flat memory graph (nodes only, no edges) as intermediate baseline
3. **Phase 3**: Add LOCATED_IN edges (structural, straightforward)
4. **Phase 4**: Add CONFLICTS_WITH edges via LLM inference at write time
5. **Phase 5**: Evaluate on LoCoMo temporal QA; compare to HorizonBench belief-update failure metrics
