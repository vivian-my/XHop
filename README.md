<div align="center">

# XHop: Multilingual Multi-Hop Reasoning Dataset

**Cross-lingual multi-hop question answering, one hop at a time.**

[![Hugging Face](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Dataset-FFD21E?style=for-the-badge)](https://huggingface.co/datasets/CHANGE-ME/XHop)
[![Website](https://img.shields.io/badge/Website-XHop-4A90D9?style=for-the-badge&logo=googlechrome&logoColor=white)](https://CHANGE-ME.github.io/XHop)
[![Paper](https://img.shields.io/badge/Paper-arXiv-B31B1B?style=for-the-badge&logo=arxiv&logoColor=white)](https://arxiv.org/abs/CHANGE-ME)

[![Languages](https://img.shields.io/badge/languages-5-3d5a80)](#splits)
[![Records](https://img.shields.io/badge/records-6%2C560-3d5a80)](#splits)
[![Hops](https://img.shields.io/badge/hops-2%20%7C%203%20%7C%204-3d5a80)](#splits)
[![License](https://img.shields.io/badge/license-per%20source-6E7781)](LICENSES/)

</div>

---

**XHop** extends two English multi-hop question-answering datasets,
[HotpotQA](https://hotpotqa.github.io/) and [MuSiQue](https://github.com/stonybrooknlp/musique),
to five languages: English, Chinese, French, Arabic, and Russian. By varying the language
of each supporting document, **XHop** enables fine-grained evaluation of multilingual
multi-hop reasoning, revealing how model performance changes when the evidence required to
answer a question is distributed across languages. As illustrated below, the model must
integrate information from two documents written in different languages to derive the
final answer.

<div align="center">
<img alt="A 2-hop XHop example. An English question, one context block holding a French bridging passage that names Briana Corrigan and a Chinese answer-bearing passage identifying her as a Northern Irish singer, and the English answer." src="assets/example.svg" width="860">
</div>





## XHop Splits

| Number of hops | Amount | Sources |  Languages included |
|:---:|---:|:---:|---|
| **2** | 803 | MuSiQue + HotpotQA | English · French · Russian · Arabic · Chinese |
| **3**  | 327 | MuSiQue |  English · French · Russian · Arabic · Chinese |
| **4**  | 182 | MuSiQue |  English · French · Russian · Arabic · Chinese |








## Quick start

```python
import json

def load(split, source, lang):
    with open(f"data/{split}/{source}/{lang}.jsonl", encoding="utf-8") as f:
        return [json.loads(line) for line in f]

en, ru, zh = (load("two_hop", "hotpotqa", L) for L in ("en", "ru", "zh"))

# Files are aligned, so line i is the same question in every language.
i = 0
# English Two-Hop Query
question = en[i]
bridge_pos, answer_pos = q["hop_seq"]        # reasoning order -> positions in `answers`

# Query in English, bridging hop in Russian, answer-bearing hop in Chinese.
passages = [None, None]
passages[bridge_pos] = ru[i]["answers"][bridge_pos]
passages[answer_pos] = zh[i]["answers"][answer_pos]
```


## Record schema

```jsonc
{
  "id": "hotpotqa_5a8b57f25542995d1e6f1371",
  "source": "hotpotqa",            // "musique" | "hotpotqa"
  "type": "bridge",                // upstream reasoning-shape label
  "n_hops": 2,
  "question": "...",
  "answer": "Chief of Protocol",

  "answers":     [[title, [sentence, ...]], ...],   // the n gold passages
  "non_answers": [[title, [sentence, ...]], ...],   // distractors
  "supporting_facts": [[title, sentence_index], ...],

  "hop_seq": [1, 0],               // reasoning order -> indices into `answers`;
                                   // hop 1 = bridging, hop n = answer-bearing
  "hop_seq_verified": true,        // false => placeholder order, see below

  "sub_q1": {"question": "...", "answer": "Shirley Temple"},   // null on MuSiQue
  "sub_q2": {"question": "...",
             "answer": "Chief of Protocol",   // == `answer`, normalized
             "answer_raw": null}
  
}
```
### Two-Hop Question decomposition

The 176 HotpotQA records carry a gold decomposition into two single-hop questions, stored
as fields on the record so questions and passages cannot drift apart.

To load — e.g. the French decomposition:

```python
import json

hotpot = [json.loads(line) for line
          in open("data/two_hop/hotpotqa/fr.jsonl", encoding="utf-8")]

r = hotpot[0]
r["question"]              # "Quel poste gouvernemental occupait la femme qui
                           #  interprétait Corliss Archer dans le film Kiss and Tell ?"

r["sub_q1"]["question"]    # hop 1 — find the bridge entity
                           # "quelle femme a incarné Corliss Archer dans le film
                           #  Kiss and Tell ?"
r["sub_q1"]["answer"]      # "Shirley Temple"  <- the bridge entity

r["sub_q2"]["question"]    # hop 2 — asks about that entity
                           # "Quel poste gouvernemental occupait Shirley Temple ?"
r["sub_q2"]["answer"]      # "Chef du Protocole"  == r["answer"], always
```

## Citation

```bibtex
@misc{xhop,
  title  = {XHop: Cross-lingual Multi-Hop Question Answering},
  author = {CHANGE-ME},
  year   = {2026},
  url    = {https://github.com/vivian-my/XHop}
}
```
