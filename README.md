# HDS Black-Box Translator
````md
# HDS Black-Box Translator (Chess / Shogi)

> **Reproducibility-first explanation layer.**  
> This does **NOT** replace an engine. It translates **official engine outputs** into **HDS (Objective → Axes → Alternatives → Risk(EVR) → Choice)**.

- ✅ Goal: **Deterministic, reproducible, log-grounded causal comments** (“why this move?”)  
- ❌ Non-goal: convenience-first packaging, reverse engineering, internal recovery (BB-L3)

---

## Table of Contents

- [JP](#jp)
  - [TL;DR](#jp-tldr)
  - [Capability & Pledge](#jp-capability--pledge)
  - [Scope (BB-L1/L2 only)](#jp-scope-bb-l1l2-only)
  - [Two-Phase Demo](#jp-two-phase-demo)
  - [Reproducibility Contract](#jp-reproducibility-contract)
  - [Input Contract](#jp-input-contract)
  - [Output Contract](#jp-output-contract)
  - [Recommended Flow / Process](#jp-recommended-flow--process)
  - [How to Run (Notebooks)](#jp-how-to-run-notebooks)
  - [Optional Shogi Dataset Reference](#jp-optional-shogi-dataset-reference)
  - [License / Disclaimer](#jp-license--disclaimer)
  - [Contributing / Policy](#jp-contributing--policy)
- [EN](#en)
  - [TL;DR](#en-tldr)
  - [Capability & Pledge](#en-capability--pledge)
  - [Scope (BB-L1/L2 only)](#en-scope-bb-l1l2-only)
  - [Two-Phase Demo](#en-two-phase-demo)
  - [Reproducibility Contract](#en-reproducibility-contract)
  - [Input Contract](#en-input-contract)
  - [Output Contract](#en-output-contract)
  - [Recommended Flow / Process](#en-recommended-flow--process)
  - [How to Run (Notebooks)](#en-how-to-run-notebooks)
  - [Optional Shogi Dataset Reference](#en-optional-shogi-dataset-reference)
  - [License / Disclaimer](#en-license--disclaimer)
  - [Contributing / Policy](#en-contributing--policy)

---

# JP

## JP; TL;DR

このリポジトリは **「便利に使わせる」ためのものではない**。  
目的はただ一つ：**再現性（決定性）を担保したログ出力として、「なぜその手なのか」を納得できる因果で説明する**こと。

- Chess：**決定論・再現性**の証明（“説明器の器”が正しく動く）
- Shogi：学習データ/エンジンログ接続で **有用性**の証明（ここまでできる）

---

## JP; Capability & Pledge

私は、モデル/エンジンの内部挙動（評価・探索・表現）を外部I/Oから推定する実装を **行い得ます**。  
しかし本リポジトリは **「翻訳（HDSによる構造化）」** に限定し、**内部推定・復元・模倣に繋がる実装や手順は、可能であっても意図的に含めません／公開しません**。

### 禁止と誓約（HDS / 神域の禁止を本リポジトリへ写像）
- **禁止**：BB-L3（内部因果）に相当する、内部機構の復元・推定・模倣・識別・逆解析を目的とする利用  
- **禁止**：人間の優劣スコアリング、序列化、差別・排除に直結する利用  
- **誓約**：本リポジトリは BB-L1/L2 の「ログに根拠を置いた翻訳」に限定し、**決定論的ログとして検証可能な因果コメント**を出力する

---

## JP; Scope (BB-L1/L2 only)

本リポジトリが指す「可視化」は次に限定：

- **BB-L1（外形の構造化）**：出力（候補手/評価/ログ）を HDS で整理し、人間が追える形にする  
- **BB-L2（ログ根拠つき翻訳）**：PV/深さ推移/比較手など “ログ根拠” を添えて説明を強化する  
- **BB-L3（内部因果）**：NN内部表現や探索機構の因果まで説明する（**対象外 / 禁止**）

---

## JP; Two-Phase Demo

### Phase 1（Chess）：決定性の担保と全体像
- **エンジン不要でも動く**（軽量な解釈可能特徴量）
- 同一入力 → 同一出力（コメント・差分）を保証し、**説明器が決定論で動く**ことを示す

### Phase 2（Shogi）：AIログ接続による翻訳（有用性）
- 将棋エンジンのログ（候補手＋評価＋任意でPV等）を入力
- HDS 5層ログ、要約（コメント）、簡易可視化を出し、**“AIの手を翻訳して教える”**を実演する

---

## JP; Reproducibility Contract

ここが本リポジトリの中核。**再現性は「努力」ではなく「仕様」**。

### 1) 決定論（Determinism）要件
同一入力に対して、次が **完全一致**すること：
- 候補手テーブル
- HDS 5層ログ（Objective → Axes → Alternatives → Risk(EVR) → Choice）
- コメント（要約）
- 因果トレース（根拠ログ）

※一致しない場合は、入力/環境/データ/乱数/順序規則のどれかが仕様違反。

### 2) 環境固定（Environment Freeze）手順（必須）
「環境差でズレる」を潰すため、**公表/検証時は必ず記録**する：

- Python バージョン
- OS / CPU / GPU 有無
- 主要依存ライブラリの完全版（pip freeze）

実行例：
~~~bash
python --version
python -c "import platform; print(platform.platform())"
python -m pip freeze
~~~

> 本リポジトリは利便性を目的としないが、**再現性の証拠（環境ログ）は必須**。

### 3) 乱数（RNG）規約
- 乱数を使う場合：**seed固定は必須**
- 乱数を使わない設計なら：その事実を明記する
- 乱数が関与する箇所があるなら：どの乱数源（Python / NumPy / 他）かを明記し、全て固定

### 4) 順序・タイブレーク（Order / Tie-break）規約
- 入力候補手の順序、同点時の扱い、丸め規則、文字列正規化（例：空白、改行、ロケール）  
  **どれも再現性を破壊するので、実装と一致する形で明文化**すること。
- 推奨：**入力側で候補手順序を決定論的に固定**し、出力側はその順序を尊重する。

### 5) データ（Shogi）規約
Shogi 側で学習データ/外部データを使う場合：
- データセット名
- バージョン / 取得元URL
- 取得日
- 可能ならファイルハッシュ（SHA256等）
- 前処理（あれば）を手順として完全記述

---

## JP; Input Contract

### Chess Input
- `fen`：FEN文字列
- `move_uci`：UCI手（例：`e2e4`）
- `perspective`：`"white"` / `"black"`

### Shogi Input（ログ取り込み最小スキーマ）
~~~json
{
  "position_id": "any_id",
  "sfen": "....",
  "side_to_move": "black",
  "candidate_moves": [
    { "move": "7g7f", "eval": 300, "policy": 0.45, "label": "best", "pv": "...", "desc": "任意" },
    { "move": "2g2f", "eval": 250, "policy": 0.30, "label": "second", "pv": "...", "desc": "任意" }
  ]
}
~~~

- `eval`：センチポーン等のスカラー（符号の向きは運用で固定）
- `pv` / `policy` / `desc`：任意（あるほど BB-L2 の忠実性が上がる）

---

## JP; Output Contract

ご主人様の評価軸を、そのまま仕様として固定：

- **有用なコメントが出る**
- **なぜその手なのか**が納得できる **因果**として説明される
- それが **ログ（構造化トレース）**として出る
- そして **決定論的に再現できる**

### 推奨ログ構造（因果の最小分解）
コメント本文の裏に、最低限これが残る形が望ましい：
- Premise（観測事実：差分/候補/評価/PV…）
- Reason（因果：何がどう変化したからその手が良い/悪い）
- Tradeoff（捨てたもの：リスク/副作用）
- Decision（採択理由：最終の決め手）

> 重要：雰囲気コメントは禁止。**根拠ログと1対1で紐づく因果**のみを出す。

---

## JP; Recommended Flow / Process

“便利導線”ではなく、**再現性を担保する検証プロセス**。

1) Phase 1（Chess）を実行  
   - 決定論（同一入力→同一コメント/差分）を確認  
   - 環境ログ（python version / pip freeze）を記録  
2) Phase 2（Shogi）を実行  
   - 同一入力→同一HDSログ/同一コメントを確認  
   - 使ったデータ（あれば）を Data Contract に従い記録  
3) 期待出力（Expected Logs）と一致することを確認  
   - 一致しないなら仕様違反：原因は必ず追跡・明文化する

---

## JP; How to Run (Notebooks)

このリポジトリは Notebook ベース。

- `notebook41b2dffc34.ipynb`
- `notebook7f1cfc6470.ipynb`

実行原則：
- 上から順に実行（セル順序は仕様）
- 外部API不要（オフラインで完結）
- 乱数要素があるなら seed 固定

使用例（概念）：
~~~python
# Chess
comment, deltas = explainer.explain_move(fen_string, best_move_uci, perspective="white")
print(comment)
print(deltas)

# Shogi
df, hds_log, summary = explain_position(sample_position_dict)
print(hds_log)
print(summary)
~~~

---

## JP; Optional Shogi Dataset Reference

> ※以下は配布元の説明を要約したもの。利用条件（ライセンス/規約）は各配布元に従ってください。

本リポジトリ自体はデータセット必須ではないが、将棋側の「ログ接続デモ」を作る際に eval 付き局面データがあると便利。

- 知識蒸留済みデータセット（約1.1億局面）
- `shogi_suisho5_depth9` をベースに、Kanadeで指し手と評価値を書き換え
- シャッフル済みで学習に使用可能
- `Eval_Coef=285` でDLモデルのvalueと評価値を変換
- 3ファイル構成：Qsearch適用psv / Qsearch不適用psv / Qsearch不適用hcpe
- バグがある可能性、品質保証なし

Links:
- https://huggingface.co/datasets/nodchip/shogi_suisho5_depth9
- https://fuuppi.booth.pm/items/7108913

---

## JP; License / Disclaimer

- License: MIT（`LICENSE` を参照）
- 免責：研究/デモ用途。実運用は自己責任。

---

## JP; Contributing / Policy

- BB-L3（内部因果）へ寄せるPR/issueは **方針として対象外**  
- 逆解析・模倣・識別など、内部復元に直結する変更は受け付けない  
- “再現性を壊す変更”は、仕様条件・手順・ログを更新できない限りマージしない

---

# EN

## EN; TL;DR

This repository is **not** a convenience-first toolkit.  
Its only goal is: **deterministic, reproducible, log-grounded causal comments** (“why this move?”) as a verifiable trace.

- Chess: **proof of determinism** (the explanation pipeline behaves deterministically)
- Shogi: **proof of usefulness** (how far we can go with real logs / data)

---

## EN; Capability & Pledge

I can implement external I/O–based inference of internal engine/model behavior (evaluation, search, representations).  
However, this repository is intentionally limited to **translation (HDS-based structuring)** and therefore **will not include or publish implementations/procedures that enable internal recovery, reconstruction, imitation, identification, or reverse engineering**, even if technically feasible.

### Prohibitions & Pledge (mapped from HDS/Shiniki commitments)
- **Prohibited**: BB-L3-equivalent use (internal recovery / reconstruction / imitation / identification / reverse engineering)
- **Prohibited**: direct human ranking/profiling/discrimination/exclusion use
- **Pledge**: This repo stays within BB-L1/L2 as **log-grounded translation**, producing **deterministic causal comments** as auditable logs

---

## EN; Scope (BB-L1/L2 only)

- **BB-L1 (structured outputs)**: organize outputs (candidates/evals/logs) into an HDS trace
- **BB-L2 (log-grounded translation)**: strengthen explanations with PV/depth trends/comparisons, etc.
- **BB-L3 (internal causality)**: causal explanations of NN/search internals (**out of scope / prohibited**)

---

## EN; Two-Phase Demo

### Phase 1 (Chess): determinism + overall pipeline
- Can run without a strong engine (small interpretable features)
- Proves **deterministic explanation behavior** (same input → same output)

### Phase 2 (Shogi): translation via real engine logs (usefulness)
- Input: shogi engine logs (candidate moves + eval + optional PV, etc.)
- Output: HDS 5-layer logs, comments, and simple visualizations

---

## EN; Reproducibility Contract

Reproducibility is not “effort”; it is a **spec**.

### 1) Determinism Requirements
For the same input, these must match **exactly**:
- candidate table
- HDS 5-layer trace
- comment (summary)
- causal trace (evidence log)

### 2) Environment Freeze (MUST)
Record at publication/verification time:
- Python version
- OS / CPU / GPU availability
- full dependency versions (`pip freeze`)

~~~bash
python --version
python -c "import platform; print(platform.platform())"
python -m pip freeze
~~~

### 3) RNG Rules
- If randomness exists: seeds must be fixed
- If no randomness: explicitly state it
- If multiple RNG sources exist: fix all of them

### 4) Order / Tie-break Rules
Order, tie-break, rounding, and normalization can silently break reproducibility.  
They must be **explicitly documented to match the implementation**.  
Recommended: keep input ordering deterministic and preserve it through the pipeline.

### 5) Data Rules (Shogi)
If any dataset is used:
- dataset name
- version / source URL
- retrieval date
- hash if possible
- full preprocessing procedure (if any)

---

## EN; Input Contract

### Chess
- `fen`: FEN string
- `move_uci`: UCI move (e.g., `e2e4`)
- `perspective`: `"white"` / `"black"`

### Shogi (minimal schema for log ingestion)
~~~json
{
  "position_id": "any_id",
  "sfen": "....",
  "side_to_move": "black",
  "candidate_moves": [
    { "move": "7g7f", "eval": 300, "policy": 0.45, "label": "best", "pv": "...", "desc": "optional" },
    { "move": "2g2f", "eval": 250, "policy": 0.30, "label": "second", "pv": "...", "desc": "optional" }
  ]
}
~~~

- `eval`: scalar score (centipawn-like); keep sign conventions consistent
- `pv` / `policy` / `desc`: optional (more fields → stronger BB-L2 faithfulness)

---

## EN; Output Contract

The core success criterion:
- **Useful comment**
- **Causal explanation** (“why this move?”) grounded in logs
- As a **structured trace**
- **Deterministic and reproducible**

Recommended minimal causal trace structure:
- Premise (observations: deltas/candidates/evals/PV…)
- Reason (causality: what changed and why it justifies the move)
- Tradeoff (risks / what was sacrificed)
- Decision (final decisive factor)

---

## EN; Recommended Flow / Process

This is not a “convenience guide”; it is a reproducibility protocol.

1) Run Phase 1 (Chess)  
   - verify determinism  
   - record environment logs  
2) Run Phase 2 (Shogi)  
   - verify deterministic HDS trace & comment  
   - record dataset contract if used  
3) Compare with expected logs  
   - any mismatch means a spec violation: locate and document the cause

---

## EN; How to Run (Notebooks)

Notebooks:
- `notebook41b2dffc34.ipynb`
- `notebook7f1cfc6470.ipynb`

Rules:
- run top-to-bottom (cell order is part of the spec)
- offline-ready (no external APIs)
- fix seeds if randomness exists

~~~python
# Chess
comment, deltas = explainer.explain_move(fen_string, best_move_uci, perspective="white")
print(comment)
print(deltas)

# Shogi
df, hds_log, summary = explain_position(sample_position_dict)
print(hds_log)
print(summary)
~~~

---

## EN; Optional Shogi Dataset Reference

> The following is a summarized description from distributors. Follow the original license/terms.

Links:
- https://huggingface.co/datasets/nodchip/shogi_suisho5_depth9
- https://fuuppi.booth.pm/items/7108913

---

## EN; License / Disclaimer

- License: MIT (see `LICENSE`)
- Disclaimer: research/demo use only; use at your own risk

---

## EN; Contributing / Policy

- PRs/issues toward BB-L3 (internal recovery) are out of scope
- no reverse engineering / imitation / identification oriented changes
- changes that can break determinism must update the spec, procedure, and expected logs

