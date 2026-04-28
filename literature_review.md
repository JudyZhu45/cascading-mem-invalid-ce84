# Literature Review: Cascading Memory Invalidation in User Memory Graphs

## Research Area Overview

This review covers the intersection of **long-term memory systems for LLM agents**, **preference evolution and personalization**, and **knowledge graph-based memory architectures**. The research hypothesis concerns propagating invalidation signals through typed dependency edges in a memory graph — a problem that decomposes into (1) structural graph invalidation (e.g., LOCATED_IN edges when a user moves city) and (2) semantic conflict detection (e.g., inferring that "preferring quiet" conflicts with "going to bars").

The field is active and rapidly growing. Key findings from the literature: frontier LLMs consistently fail at *belief-update* even when they succeed at memory retrieval; temporal reasoning is the hardest sub-task in long-term dialogue benchmarks; and compact agentic memory architectures outperform full-history retrieval at a fraction of the token cost.

---

## Key Papers

### Paper 1: Evaluating Very Long-Term Conversational Memory of LLM Agents (LoCoMo)
- **Authors**: Maharana et al. (Snap Research, UNC, USC)
- **Year**: 2024
- **Source**: ACL 2024 (arXiv 2402.17753)
- **Key Contribution**: Introduces LoCoMo — a dataset of 10 very long-term conversations (300+ turns, 9K tokens avg., up to 35 sessions) grounded in **temporal event graphs** representing causally linked life events. Proposes three evaluation tasks: QA (5 categories), event summarization, and multi-modal dialogue generation.
- **Methodology**: Machine-human pipeline: LLM agents with Park et al. generative architecture generate dialogues conditioned on persona + temporal event graph G; human annotators fix long-range inconsistencies (~15% of turns edited).
- **Datasets Used**: LoCoMo (their own), MSC dataset for seed personas
- **Key Results**:
  - Human performance: 87.9% F1 overall; best LLM: 41.4% (RAG w/ observations)
  - RAG with top-5 observations outperforms raw dialogue RAG (+5%)
  - Long-context LLMs hallucinate more on adversarial questions (2.1% vs 22.1%)
  - **Temporal reasoning is the hardest category** (LLMs lag humans by 73%)
  - Event summarization: long-context models *underperform* base models (lose 3–8.7%)
- **Evaluation Metrics**: F1 (QA), FactScore + ROUGE (event summarization), MMRelevance (multimodal)
- **Code Available**: Yes — https://github.com/snap-research/locomo
- **Relevance to Our Research**: LoCoMo is our primary evaluation dataset. The temporal event graph structure in LoCoMo directly mirrors what we propose: causal dependency edges between life events. However, LoCoMo does not model **cascading invalidation** — events do not automatically down-weight previously stored memories. This gap is our target.

---

### Paper 2: HorizonBench: Long-Horizon Personalization with Evolving Preferences
- **Authors**: Li et al. (UW, Meta, OpenAI, Stanford)
- **Year**: 2026 (preprint arXiv 2604.17283)
- **Source**: Under review
- **Key Contribution**: Introduces long-horizon personalization as a formal problem; builds HorizonBench with 4,245 items from 360 simulated users with 6-month histories (~163K tokens). Uses a **mental state graph** with typed dependency edges — life events propagate through edges and update downstream preferences. Pre-evolution values serve as hard-negative distractors for diagnosing *belief-update failure* vs. *retrieval failure*.
- **Methodology**: State-first data generation: conversations produced from structured mental state graph G. Preference evolution: with p_evo=0.15, a life event alters 2-5 preferences simultaneously via cascading typed dependency edges. Expression-tracking system: separates *when preferences evolve* from *when they are stated*. 5-LLM consensus filter ensures items require full history.
- **Datasets Used**: HorizonBench (their own synthetic dataset)
- **Key Results**:
  - Best frontier model (Claude-opus-4.5): 52.8%; most models at or below 20% chance
  - All 25 models select pre-evolution distractor at >25% of wrong answers (belief-update failure)
  - Gap persists even in short-horizon variant (0-14 days): Evo∆ = -5.7pp
  - Implicit expression makes belief-update failure worse (distractor rate 49.7% → 58.8%)
  - **State-tracking, not retrieval, is the bottleneck**
- **Evaluation Metrics**: Multiple-choice accuracy, pre-evolution distractor selection rate, evolved-vs-static accuracy gap
- **Code/Data Available**: Yes — https://github.com/stellalisy/HorizonBench; https://huggingface.co/datasets/stellalisy/HorizonBench
- **Relevance to Our Research**: **Highest relevance.** HorizonBench provides the closest existing formalization of the research hypothesis. Their typed dependency edges and cascading preference propagation directly implement scenario (1) from our hypothesis. The paper proves that:
  - Structural graph edges (their "preference dependency edges") can propagate changes
  - The hard problem is **belief-update**: systems retrieve stale values and fail to integrate life events
  - This maps exactly onto our CONFLICTS_WITH edge problem — the question is HOW to establish semantic conflict edges

---

### Paper 3: PersonaMem-v2: Towards Personalized Intelligence via Learning Implicit User Personas and Agentic Memory
- **Authors**: Jiang et al. (UPenn, MIT, UW, UCSB, Meta)
- **Year**: 2025 (arXiv 2512.06688)
- **Source**: Preprint
- **Key Contribution**: State-of-the-art personalization dataset with 1,000 user personas, 20K+ user preferences, 128K-token context windows. Focuses on **implicit** preference revelation (most real-world preferences are never stated directly). Introduces an agentic memory framework trained via RL fine-tuning (GRPO) to maintain 2K-token compact memory.
- **Methodology**: Users reveal preferences implicitly through everyday tasks (email polishing, translation, etc.). Agentic memory: trained via RFT to distill 32K-token conversation histories into 2K-token user memory; the memory becomes the sole personalization context. Dynamic preferences included (e.g., vegetarian → high-protein after fitness change).
- **Key Results**:
  - Frontier LLMs: 37–48% accuracy on implicit personalization
  - Agentic memory model: 55% accuracy using 16× fewer tokens (2K vs 32K)
  - RFT-trained Qwen3-4B outperforms GPT-5 (53%) on implicit personalization
- **Evaluation Metrics**: Multiple-choice and open-ended accuracy
- **Code/Data Available**: https://huggingface.co/datasets/bowen-upenn/PersonaMem-v2
- **Relevance to Our Research**: Important for scenario (2). The agentic memory framework (compact, updatable) is a strong baseline architecture. Their dynamic preference scenario (vegetarian → high-protein) is analogous to our implicit preference drift problem. The key limitation: they do not model *why* preferences change or establish conflict detection — changes are treated as ground truth annotations rather than inferred from semantic conflict signals.

---

### Paper 4: MemGPT: Towards LLMs as Operating Systems
- **Authors**: Packer et al. (UC Berkeley)
- **Year**: 2024 (arXiv 2310.08560)
- **Source**: MLSys/ICLR
- **Key Contribution**: Proposes virtual context management for LLMs, inspired by OS virtual memory. Memory hierarchy: main context (working context + FIFO queue) and external context (recall storage + archival storage). LLM manages its own memory via function calls.
- **Methodology**: Working context = read/write fixed-size block for key facts/preferences. Recall storage = full history database (BM25 + semantic search). Queue eviction policy triggers LLM to summarize/compress when context nears limit.
- **Key Results**: Outperforms window-based approaches on multi-session chat (better persona consistency and long-term memory); handles documents far exceeding context window
- **Code Available**: Yes — now Letta (https://github.com/letta-ai/letta)
- **Relevance to Our Research**: MemGPT's working context (persistent facts about user) is the precursor to typed memory graphs. The key gap: MemGPT's working context is flat text — no typed dependency edges, no propagation mechanism. Our research proposes to add LOCATED_IN and CONFLICTS_WITH edges that trigger cascading updates when root context nodes change.

---

### Paper 5: Memory OS of AI Agent (MemoryOS)
- **Authors**: Kang et al.
- **Year**: 2025
- **Source**: Preprint (arXiv 2501.01999)
- **Key Contribution**: OS-inspired hierarchical memory (short/mid/long-term) for AI agents. FIFO-based short-to-mid transfer; segmented page organization for mid-to-long. Evaluated on LoCoMo.
- **Key Results**: +49.11% F1 and +46.18% BLEU-1 over baselines on LoCoMo (GPT-4o-mini)
- **Code Available**: https://github.com/BAI-LAB/MemoryOS
- **Relevance**: Strong LoCoMo baseline for our experiments. Shows that hierarchical memory with proper update policies significantly outperforms naive RAG on LoCoMo.

---

### Paper 6: ENGRAM: Effective, Lightweight Memory Orchestration for Conversational Agents
- **Authors**: Patel and Patel
- **Year**: 2025
- **Source**: Preprint
- **Key Contribution**: Lightweight memory system with three canonical types (episodic, semantic, procedural) and single router/retriever. Dense retrieval with typed memories outperforms complex architectures.
- **Key Results**: State-of-the-art on LoCoMo; +15 points on LongMemEval over full-context baseline using ~1% of tokens
- **Relevance**: Direct LoCoMo baseline; demonstrates that typed memory (episodic/semantic/procedural) is powerful without complex graphs. Relevant to our hypothesis about typed edges.

---

### Paper 7: MemoryBank: Enhancing Large Language Models with Long-Term Memory
- **Authors**: Zhong et al.
- **Year**: 2023 (arXiv 2309.11276)
- **Source**: AAAI 2024
- **Key Contribution**: Memory mechanism with Ebbinghaus Forgetting Curve — memories decay based on time elapsed and significance. Supports both closed-source (ChatGPT) and open-source (ChatGLM) models.
- **Key Results**: Demonstrated via long-term AI companion (SiliconFriend); memory decay + update improves user personality understanding
- **Relevance**: First paper to model **temporal decay** in LLM memory. Our CONFLICTS_WITH edge propagation is analogous to — but more structured than — their significance-based decay.

---

### Paper 8: Preference-Aware Memory Update Mechanism (PAMU)
- **Authors**: Sun et al.
- **Year**: 2025
- **Source**: Preprint
- **Key Contribution**: Sliding window averages (SW) + exponential moving averages (EMA) fused to create preference-aware representation capturing short-term fluctuations and long-term tendencies.
- **Datasets Used**: LoCoMo (5 task scenarios)
- **Relevance**: Directly relevant to scenario (2). PAMU provides a continuous-weight decay mechanism for preferences but no graph-based invalidation. Their approach is complementary: our structural edges handle explicit context shifts; PAMU-style EMA handles gradual drift.

---

### Paper 9: Generative Agents: Interactive Simulacra of Human Behavior
- **Authors**: Park et al. (Stanford)
- **Year**: 2023
- **Source**: UIST 2023
- **Key Contribution**: Generative agent architecture with memory stream, retrieval (recency + importance + relevance), and reflection. Foundational architecture for LoCoMo, HorizonBench, and numerous memory systems.
- **Relevance**: Baseline architecture. Memory is flat stream of observations; no typed dependency edges or invalidation propagation. Our research extends this with structured graph edges.

---

### Paper 10: A Survey on the Memory Mechanism of LLM-based Agents
- **Authors**: Zhang et al.
- **Year**: 2024
- **Source**: Preprint (arXiv 2403.13318)
- **Key Contribution**: Comprehensive survey of memory types (sensory/short-term/long-term), storage formats (in-context, external, parametric), operations (reading/writing/management)
- **Relevance**: Provides taxonomy for contextualizing our work. Our typed dependency graph falls under "structured external memory" with explicit "memory management/invalidation" operations.

---

## Common Methodologies

| Method | Papers |
|--------|--------|
| Temporal event graphs for grounding | LoCoMo, HorizonBench |
| RAG with typed memory units (observations/summaries/sessions) | LoCoMo, MemGPT, ENGRAM |
| Hierarchical memory (short/mid/long-term) | MemoryOS, MemGPT, CraniMem |
| Ebbinghaus-style temporal decay | MemoryBank, Mnemosyne |
| Agentic memory (compact, RFT-trained) | PersonaMem-v2, MetaMem |
| Knowledge graph-based memory | Memoria, TOBUGraph, PersonalAI |
| LLM inference at write time | MemGPT (working context), ENGRAM |

---

## Standard Baselines

| Baseline | Description | Typical Performance on LoCoMo |
|----------|-------------|-------------------------------|
| Base LLM (4K context) | No memory, truncated history | ~22-32% F1 |
| Long-context LLM (16K) | Full history in context | ~37.8% F1 |
| RAG (dialog) | BM25/semantic retrieval over dialog turns | ~35.8% F1 |
| RAG (observations) | Retrieval over extracted assertions | ~41.4% F1 |
| RAG (summaries) | Retrieval over session summaries | ~32.5% F1 |
| MemoryOS | Hierarchical OS-inspired memory | ~+49% F1 over GPT-4o-mini baseline |
| ENGRAM | Typed episodic/semantic/procedural | State-of-the-art LoCoMo |

---

## Evaluation Metrics

| Metric | Task | When to Use |
|--------|------|-------------|
| F1 (exact match) | QA | Standard for LoCoMo QA |
| FactScore | Event summarization | Factual accuracy in summaries |
| ROUGE-1/2/L | Summarization | Surface overlap |
| Multiple-choice accuracy | Preference tracking | HorizonBench, PersonaMem-v2 |
| Pre-evolution distractor rate | Belief-update failure | HorizonBench diagnostic |
| Evolved-vs-static accuracy gap | Belief-update capability | HorizonBench controlled experiment |
| BLEU-1 | Dialogue generation | LoCoMo multimodal task |

---

## Datasets in the Literature

| Dataset | Papers | Task | Size | Notes |
|---------|--------|------|------|-------|
| LoCoMo | This study, MemoryOS, ENGRAM, PAMU | Long-term dialogue QA/summarization | 10 conversations, ~1986 QA pairs | Gold standard for memory evaluation |
| HorizonBench | This study | Preference evolution tracking | 4,245 items, 360 users | Pre-evolution distractors, typed dependency edges |
| PersonaMem-v2 | This study | Implicit personalization | 1,000 users, 20K+ prefs, 128K ctx | RFT training data available |
| LongMemEval | ENGRAM | Long-term memory QA | - | ENGRAM reports +15pp over full-context |
| MSC (Multi-Session Chat) | LoCoMo | Multi-session dialogue | 53.3 turns avg | Used as persona seed in LoCoMo |

---

## Gaps and Opportunities

1. **No existing system models cascading invalidation through typed dependency edges.** HorizonBench gets closest (propagates preference changes through dependency edges at data generation time), but no *memory system* implements this at inference time.

2. **CONFLICTS_WITH edge establishment is unsolved.** All existing approaches either (a) use ground-truth labels, or (b) treat preference changes as independently observed. No system automatically infers semantic conflicts (e.g., "quiet" vs. "bars") without explicit annotation.

3. **Temporal reasoning remains the hardest task** across all benchmarks (73% below human on LoCoMo temporal questions; HorizonBench short-horizon belief-update failure is *worse* than long-horizon). Existing systems retrieve stale values rather than integrating life events.

4. **The retrieval vs. belief-update distinction** is newly established by HorizonBench (2026). No prior memory system is designed specifically to address belief-update failure as distinct from retrieval failure.

5. **LLM inference at write time for semantic bridging** is underexplored. MemGPT uses LLM to manage its own memory but does not use it to infer conflict edges. PersonaMem-v2 uses RFT to improve personalization but not for conflict graph construction.

---

## Recommendations for Our Experiment

**Recommended datasets**:
- Primary: **LoCoMo** (10 conversations with temporal event graphs, existing baselines, QA with temporal reasoning category)
- Secondary: **HorizonBench** (sample split available; evolved vs. static preference split enables direct measurement of belief-update failure)

**Recommended baselines**:
1. Base GPT-4o-mini/Claude-sonnet-4 (4K context, truncated)
2. RAG with observations (best LoCoMo baseline from original paper)
3. MemoryOS (strongest reported LoCoMo baseline: +49% F1)
4. ENGRAM (state-of-the-art, lightweight, typed memory)

**Recommended metrics**:
- F1 (QA), stratified by category (especially temporal reasoning)
- Pre-evolution distractor selection rate (if adapting HorizonBench evaluation)
- Evolved-vs-static accuracy gap

**Methodological considerations**:
- **Scenario (1) - Explicit context shift (LOCATED_IN)**: Implement graph with structural dependency edges; test on LoCoMo conversations where persona location/role shifts (e.g., job change events). Straightforward to implement as post-processing on existing temporal event graph.
- **Scenario (2) - Implicit preference drift (CONFLICTS_WITH)**: Two candidate approaches:
  - **(a) LLM inference at write time**: When storing new memory node, prompt LLM to identify potential conflicts with existing graph nodes; store CONFLICTS_WITH edges. Cost: 1 LLM call per memory write.
  - **(b) Co-occurrence signals**: Track behavioral patterns across sessions; down-weight memories with low co-occurrence with recent activity patterns. No additional LLM calls but requires sufficient history.
- The key engineering challenge is the **semantic bridging problem**: LLM must bridge "preferring quiet" → "conflicts with bars" without explicit user statement. Start with approach (a) as it is more controllable and directly testable.
