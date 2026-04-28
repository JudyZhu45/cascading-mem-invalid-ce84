# Datasets

This directory contains datasets for the research project. Large data files are NOT committed to git due to size. Follow the download instructions below.

---

## Dataset 1: LoCoMo

### Overview
- **Source**: GitHub repository (snap-research/locomo) — data embedded in repo
- **Size**: 10 conversations, ~1,986 total QA pairs, ~5.6MB JSON
- **Format**: JSON (single file `locomo10.json`)
- **Task**: Long-term conversational memory evaluation (QA, event summarization, multi-modal generation)
- **Splits**: All 10 conversations in one evaluation set
- **License**: See https://github.com/snap-research/locomo

### Download Instructions

**Option 1 — Clone the repository (already available in code/locomo/):**
```python
import json
with open('code/locomo/data/locomo10.json') as f:
    data = json.load(f)
# data is a list of 10 conversation dicts
```

**Option 2 — Direct download:**
```bash
wget https://raw.githubusercontent.com/snap-research/locomo/main/data/locomo10.json \
     -O datasets/locomo/locomo10.json
```

### Loading the Dataset
```python
import json
with open('datasets/locomo/locomo10.json') as f:
    data = json.load(f)

# Each item has:
# - sample_id: conversation ID
# - conversation: dict of sessions (session_1, session_2, ...) with turns
# - qa: list of QA pairs with category, question, answer, evidence
# - event_summary: ground-truth event annotations per session
# - observation: pre-generated speaker observations per session
# - session_summary: pre-generated session summaries
```

### Dataset Statistics
| sample_id | sessions | total_turns | qa_pairs |
|-----------|----------|-------------|----------|
| conv-26   | 19       | 419         | 199      |
| conv-30   | 19       | 369         | 105      |
| conv-41   | 32       | 663         | 193      |
| conv-42   | 29       | 629         | 260      |
| conv-43   | 29       | 680         | 242      |
| conv-44   | 28       | 675         | 158      |
| conv-47   | 31       | 689         | 190      |
| conv-48   | 30       | 681         | 239      |
| conv-49   | 25       | 509         | 196      |
| conv-50   | 30       | 568         | 204      |

**QA Categories**: 1=Single-hop, 2=Multi-hop, 3=Temporal, 4=Open-domain knowledge, 5=Adversarial

### Notes
- Category 3 (temporal reasoning) is the hardest for LLMs (73% below human performance)
- Observations and session summaries are pre-generated and included for RAG baselines
- LoCoMo includes temporal event graphs linking life events — critical for our invalidation experiments

---

## Dataset 2: HorizonBench

### Overview
- **Source**: HuggingFace — stellalisy/HorizonBench
- **Size**: 4,245 items from 360 simulated users, 6-month histories (~163K tokens each)
- **Format**: HuggingFace Dataset (Parquet)
- **Task**: Long-horizon personalization — 5-option multiple choice with pre-evolution distractors
- **Splits**: sample (10 items), full dataset (4,245 items)
- **License**: See https://huggingface.co/datasets/stellalisy/HorizonBench

### Download Instructions

**Using HuggingFace (sample config already loaded):**
```python
from datasets import load_dataset

# Sample (10 items, fast)
ds = load_dataset('stellalisy/HorizonBench', 'sample')

# Full benchmark (4,245 items)
ds = load_dataset('stellalisy/HorizonBench')
ds.save_to_disk('datasets/horizonbench/full')
```

### Loading the Dataset
```python
from datasets import load_from_disk
ds = load_from_disk('datasets/horizonbench/full')

# Each item has:
# - id: item identifier
# - generator: which LLM generated the conversation (sonnet-4.5, o3, gemini-3-flash)
# - user_id: simulated user identifier
# - conversation: full 6-month conversation history
# - correct_letter: ground truth answer (A-E)
# - options: dict of 5 response options
# - has_evolved: whether this item tests an evolved preference (True/False)
# - preference_domain: type of preference being tested
# - distractor_letter: which option is the pre-evolution distractor
# - preference_evolution: description of how preference changed
```

### Notes
- 59% of items involve evolved preferences (our primary target)
- Pre-evolution distractor rate >25% indicates belief-update failure
- Three generators ensure results aren't generator-specific artifacts
- Sample config (10 items) is sufficient for initial method validation

---

## Dataset 3: PersonaMem-v2

### Overview
- **Source**: HuggingFace — bowen-upenn/PersonaMem-v2
- **Size**: 1,000 personas, 20K+ preferences, 128K-token contexts; 5K test QA + 20K train QA
- **Format**: HuggingFace Dataset
- **Task**: Implicit personalization — multiple-choice and open-ended QA from long conversation history

### Download Instructions
```python
from datasets import load_dataset
ds = load_dataset('bowen-upenn/PersonaMem-v2')
ds.save_to_disk('datasets/personamem_v2/')
```

### Notes
- Primarily useful for training the memory update mechanism
- Contains dynamic preference examples (preference changes across sessions)
- Requires HuggingFace token for full access
