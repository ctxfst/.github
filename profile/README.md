# ctxfst · Coach every problem line as you write it

> Most knowledge tools try to extract structure from half-written notes *after* the fact.
> `ctxfst` coaches you **while writing** so each problem line (`問題線`) leaves its complete shape in the text itself—and surfaces where you hit the same problem years ago.

---

## Repository

| Repository | Description |
|---|---|
| **[`skill-line-coach`](https://github.com/ctxfst/skill-line-coach)** | Agent skill for coaching **Four-Slot problem lines (`張力句四槽教練`)** in Markdown diaries/notes and linking recurring problem shapes with **`shape_layer.py`** |

---

## 1. Write-Time Four-Slot Line Coach (`張力句四槽教練`)

When writing a diary entry or work log, people naturally record *who* did *what* (Slots 1 & 2) or praise a new tool ("very convenient"), while leaving out the **concrete obstacle** that triggered the action (Slot 3) and **whether that obstacle actually receded** afterward (Slot 4). Months later, the note cannot be found—or if found, cannot tell you why you tried that approach or where it stalled.

`skill-line-coach` coaches you at write time across four role slots:

| Slot | Role | What counts |
|---|---|---|
| **1. 誰在做 (Actor)** | Who is acting | Explicit actor, or omitted subject when context uniquely identifies one actor |
| **2. 動作＋對象 (Action + Object)** | What was done to what | Concrete action and target along this problem line |
| **3. 被什麼擋 (Concrete Blocker)** | What stood in the way | A specific, observable scene or obstacle (vague labels like *"bad state"* or *"no time"* count as missing) |
| **4. 戰報 (Outcome / State Change)** | Did the obstacle recede? | Direct update on that same obstacle—victory, failure, partial improvement, or stalemate (praising a tool's convenience does not count) |

### Coaching Modes & Line Separation

- **Line Separation (`分線`)**: A single diary entry often mixes several distinct problem lines. The coach separates them so each line is self-contained (passing the **Standalone Test** without ambiguous pronouns) and can optionally be bounded by `%% line-start id="..." %%` / `%% line-end %%`.
- **Four Interactive Modes**:
  - **`教練` (Coach, default)**: Summarizes the structure in one sentence and asks **one** non-leading probe question to help you supply the missing blocker or outcome.
  - **`評比` (Evaluate)**: Marks each slot as present (`有`) or missing (`缺`) per problem line using quotes from your text—never a numeric score.
  - **`改寫` (Rewrite)**: Reorganizes your draft without inventing facts or motives.
  - **`比較` (Compare)**: Compares before/after drafts to verify what structural piece was added.

See [`references/four-slot-coach.md`](https://github.com/ctxfst/skill-line-coach/blob/main/references/four-slot-coach.md) for the full coaching specification.

---

## 2. Recurring Problem Shape Cards (`shape_layer.py`)

Once problem lines are written clearly, how do you connect the same recurring problem across 15 years when the surface topic, tools, and objects have completely changed?

In controlled evaluations across 72 personal notes spanning 2011–2026:
- **Post-hoc structure extraction** from uncoached notes failed: entity overlap scored `0.51` pair AUC (chance), and retroactive slot extraction scored `0.629` because nearly half of old notes never recorded the blocker or outcome in the first place.
- **Write-time structure + Shape Cards** succeeded: writing the role relationships directly into the note raised cross-era embedding similarity from `0.623` to `0.858`, and ranking notes with a hand-written **Shape Card** probe (`confirmed instances − 0.5 × rejected notes`) plus LLM judging reached **`0.866` mean card AUC**.

| Method | Mean card AUC |
|---|---:|
| Nearest confirmed note (embedding) | 0.682 |
| LLM judge against the card + examples | 0.780 |
| Card probe with your rejections | 0.834 |
| **Card probe with your rejections + LLM judge** | **0.866** |

Each **Shape Card** ([`references/shape-cards.md`](https://github.com/ctxfst/skill-line-coach/blob/main/references/shape-cards.md)) captures:
- **`## 問題形狀` (Problem shape)** & **`## 核心張力` (Core tension)**
- **`## 崩潰／放棄原因（疫苗欄）` (Breakdown / Vaccine)**: what you tried in previous years, where it stalled, and what to check so you do not get stuck in the same place again
- **`## 最像但不同的線` (Near misses)** & **`## 觸發條件` (Triggers)**
- **`## 化身清單` (Confirmed instances)** & **`## 退回` (Rejected notes)**

---

## Quickstart

Install the skill in your agent environment or run the shape layer CLI directly from **[`ctxfst/skill-line-coach`](https://github.com/ctxfst/skill-line-coach)**:

```bash
git clone https://github.com/ctxfst/skill-line-coach.git
cd skill-line-coach

# Build embedding index, link notes to shape cards, and apply review feedback
python3 scripts/shape_layer.py --env .env build ~/notes --out index --exclude 'private/*'
python3 scripts/shape_layer.py --env .env link index --cards ~/notes/shapes
python3 scripts/shape_layer.py review index --cards ~/notes/shapes
```

- **Specification**: [`references/ctxfst-spec.md`](https://github.com/ctxfst/skill-line-coach/blob/main/references/ctxfst-spec.md)
- **Legacy v2 (entity graph & world state)**: preserved on the [`legacy-entity-graph`](https://github.com/ctxfst/skill-line-coach/tree/legacy-entity-graph) branch of `skill-line-coach`.
