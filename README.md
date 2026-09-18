# Jevals eval data

The data behind [jevals.com](https://jevals.com): every published release board, the per-decision run logs each board was graded from, and the suite files that define the tasks. Licensed [CC BY 4.0](LICENSE); cite "Jevals (jevals.com), release <release>".

## Releases

- [2026-09-18](releases/2026-09-18/board.json): suite 0.1.0, 8 systems. Board: https://jevals.com/r/2026-09-18/

## Layout

- `releases/<release>/board.json`: the published board, one row per system and question type. The numbers on the site are these rows.
- `runs/<system>__<task>__<suite>.jsonl`: one log per system and task. The first line describes the run: system configuration, prices, harness version, where and when it ran. Every other line is one decision: `item_id`, `epoch` (repeat 0 to 4), `target` (index of the true label in the task's options), `order_seed` (which option order was shown), `output` (the typed answer with its stated probabilities), `usage`, `cost_usd`, `seconds`, `malformed`, `refusal` and `retries`.
- `suites/<suite>/`: `suite.json` and one file per task: the source dataset and revision, the options, the instructions, and each item's id, upstream row index and label. Item text is not included; every item points at its row in the public dataset.

## Recompute the numbers

The formulas (Decision Score, accuracy, ECE, confidence intervals and ranks, the confidence gate, flip rates, cost and latency) are at https://jevals.com/methodology/. Every number on a board can be recomputed from the logs here.

The same data is also served as JSON at https://jevals.com/api/v1/ (`releases`, `board`, `models`, `tasks`, `items`, `item`, `compare`), and item by item at https://jevals.com/explore/.

## Corrections

If a number is wrong, a label looks wrong, or a model deserves a row, open an issue with the release and the row or item id. Every correction lands in the [changelog](https://jevals.com/changelog/).
