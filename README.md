# Jevals: independent benchmark data for Jev and Jev-type models

The data behind [jevals.com](https://jevals.com/), an independent, ground-truth benchmark of TypeSafe AI's **Jev** (the first System One model, called as `typesafe-ai/jev` through Vercel AI Gateway) and the LLMs that answer the same typed questions: **noul** (yes or no), **choice** (pick one of N) and **score** (place the state on a rubric). Every answer carries probabilities and is scored against human labels for Decision Score, accuracy, calibration, cost and speed. New to Jev? [What is Jev? The System One model, explained](https://jevals.com/what-is-jev/).

**Findings, release 2026-09-18:** Jev ties the best of six LLMs on yes/no questions (PubMedQA) at 1/28 of the price, ties for second on Banking77 intents, and no model beats guessing on HelpSteer2. [Read the findings](https://jevals.com/notes/2026-09-18/).

## Latest results

Decision Score, release 2026-09-18 (100 = perfect, 0 = guessing the label base rates, below 0 = worse than that). Full boards with accuracy, calibration, cost per 1,000 decisions and latency at the links.

| Model | [noul](https://jevals.com/noul/) (PubMedQA) | [choice](https://jevals.com/choice/) (Banking77) | [score](https://jevals.com/score/) (HelpSteer2 helpfulness) |
| --- | ---: | ---: | ---: |
| [Gemini 3.8 Flash](https://jevals.com/models/gemini-3.8-flash/) | 73.0 | 74.1 | 4.6 |
| [Jev](https://jevals.com/models/jev/) | 69.0 | 67.8 | 9.2 |
| [GLM-5.3](https://jevals.com/models/glm-5.3/) | 60.6 | 66.8 | 7.8 |
| [DeepSeek V4.1 Flash](https://jevals.com/models/deepseek-v4.1-flash/) | 47.5 | 63.9 | −19.0 |
| [Qwen3.8 Flash](https://jevals.com/models/qwen3.8-flash/) | 62.4 | 62.4 | −1.4 |
| [Mistral Medium 3.5](https://jevals.com/models/mistral-medium-3.5/) | 58.0 | 59.5 | −13.7 |
| [Mercury 2.5](https://jevals.com/models/mercury-2.5/) | 55.7 | 54.0 | −5.5 |
| [Label prior](https://jevals.com/models/label-prior/) | 0.0 | 0.0 | 0.0 |

- **[noul](https://jevals.com/noul/):** Gemini 3.8 Flash (73.0) and Jev (69.0) tie for first in Decision Score on PubMedQA. Jev costs $0.029 per 1,000 decisions, p95 653 ms. Release 2026-09-18.
- **[choice](https://jevals.com/choice/):** Gemini 3.8 Flash (74.1) ranks first in Decision Score on Banking77. Jev (67.8, tied for rank 2 of 8): $0.043 per 1,000 decisions, p95 693 ms. Release 2026-09-18.
- **[score](https://jevals.com/score/):** No model clearly beats guessing on HelpSteer2 helpfulness. Jev (9.2, tied for rank 1 of 8): $0.036 per 1,000 decisions, p95 670 ms. Release 2026-09-18.

Head to head: [Jev vs Gemini 3.8 Flash](https://jevals.com/compare/jev-vs-gemini-3.8-flash/) · [Jev vs GLM-5.3](https://jevals.com/compare/jev-vs-glm-5.3/) · [Jev vs DeepSeek V4.1 Flash](https://jevals.com/compare/jev-vs-deepseek-v4.1-flash/) · [Jev vs Qwen3.8 Flash](https://jevals.com/compare/jev-vs-qwen3.8-flash/) · [Jev vs Mistral Medium 3.5](https://jevals.com/compare/jev-vs-mistral-medium-3.5/) · [Jev vs Mercury 2.5](https://jevals.com/compare/jev-vs-mercury-2.5/).

## Releases

- [2026-09-18](releases/2026-09-18/board.json): suite 0.1.0, 8 systems. Board: https://jevals.com/r/2026-09-18/

## Layout

- `releases/<release>/board.json`: the published board, one row per system and question type. The numbers on the site are these rows.
- `runs/<system>__<task>__<suite>.jsonl`: one log per system and task. The first line describes the run: system configuration, prices, harness version, where and when it ran. Every other line is one decision: `item_id`, `epoch` (repeat 0 to 4), `target` (index of the true label in the task's options), `order_seed` (which option order was shown), `output` (the typed answer with its stated probabilities), `usage`, `cost_usd`, `seconds`, `malformed`, `refusal` and `retries`.
- `suites/<suite>/`: `suite.json` and one file per task: the source dataset and revision, the options, the instructions, and each item's id, upstream row index and label. Item text is not included; every item points at its row in the public dataset.

### Option order

Choice questions show their options shuffled; noul and score questions keep the task file's order. Epochs 0 and 1 use `order_seed` 0 (identical requests, for the repeat flip rate), epochs 2 to 4 use seeds 1 to 3. The order for an item is `` shuffled(options, fnv1a(`${item_id}:${order_seed}`)) ``, where `options` is the task file's `options` array:

```js
function fnv1a(s) {
  let h = 0x811c9dc5;
  for (let i = 0; i < s.length; i++) { h ^= s.charCodeAt(i); h = Math.imul(h, 0x01000193); }
  return h >>> 0;
}
function mulberry32(seed) {
  let a = seed >>> 0;
  return () => {
    a = (a + 0x6d2b79f5) >>> 0;
    let t = a;
    t = Math.imul(t ^ (t >>> 15), t | 1);
    t ^= t + Math.imul(t ^ (t >>> 7), t | 61);
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}
function shuffled(xs, seed) {
  const a = xs.slice(), r = mulberry32(seed);
  for (let i = a.length - 1; i > 0; i--) { const j = Math.floor(r() * (i + 1)); [a[i], a[j]] = [a[j], a[i]]; }
  return a;
}
```

## Recompute the numbers

The formulas (Decision Score, accuracy, ECE, confidence intervals and ranks, the confidence gate, flip rates, cost and latency) are at https://jevals.com/methodology/. Every number on a board can be recomputed from the logs here.

The same data is also served as JSON at `https://jevals.com/api/v1/<endpoint>` (`releases`, `board`, `models`, `tasks`, `items`, `item`, `compare`; e.g. https://jevals.com/api/v1/board), and item by item at https://jevals.com/explore/.

## Licence and citation

[CC BY 4.0](LICENSE). Cite as: Jevals (jevals.com), release 2026-09-18, suite 0.1.0. CC-BY-4.0. The source datasets keep their own licences; see [NOTICE](NOTICE).

## Corrections

If a number is wrong, a label looks wrong, or a model deserves a row, open an issue with the release and the row or item id. Every correction lands in the [changelog](https://jevals.com/changelog/).
