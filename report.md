# Report: Progress Summary + Next Steps

**Project theme:** _Leadership Style Impact on Trustworthiness in Mobile Crowdsourcing Systems_  
**Datasets:**

- **SeeClickFix**: `SeeClickFix_Public_Service_Requests.csv` (civic issue reports + timestamps)
- **WebCrowd25K**: `webcrowd25k/webcrowd25k/crowd_judgements.csv`, `webcrowd25k/webcrowd25k/gold_judgements.txt` (+ optional `webcrowd25k/webcrowd25k/behaviorDataRelease.json`)

This report documents what was completed in:

- `data_cleaning_preprocessing.ipynb`
- `workers_vs_orgnizations.ipynb`

…and defines the next step notebook:

- `expermenting_text_analysis.ipynb` (NLP/text analysis plan)

---

## What was done in `data_cleaning_preprocessing.ipynb`

### Purpose

This notebook establishes a clean foundation by:

- loading both datasets robustly (handling embedded newlines/quotes),
- running **basic EDA** (missingness, distributions, duplicates),
- performing **minimal cleaning/preprocessing** to make later scoring/modeling reliable.

### Loading strategy (robust parsing)

#### SeeClickFix

- Loaded the SeeClickFix CSV with robust parsing to handle special characters / embedded newlines.
- Confirmed schema and previewed rows.

**Columns observed (raw):**  
`['X','Y','OBJECTID','GlobalID','id','summary','description','status','address','category','source','agency','comments_count','image_url','image_square_url','url','created_at','acknowledged_at','closed_at','reopened_at']`

#### WebCrowd25K

- Loaded `crowd_judgements.csv` using robust parsing (important because `rationale` can contain newlines).
- Loaded `gold_judgements.txt` as space-delimited text and dropped the “unused” column per dataset documentation.

**Crowd columns:**  
`['wid','feedback','url','mapping','label','start','tid','design','rationale','duration','did','ID']`

**Gold columns:**  
`['topic_id','document_id','gold_judgement']`

### EDA performed (high-signal findings)

#### SeeClickFix EDA

- Confirmed dataset size: **7,121 rows × 20 columns**
- Checked:
  - **dtypes**
  - **missingness counts + missingness %**
  - **duplicate rows** and duplicate IDs
  - **agency/source distributions**
  - **numeric summary** (including coordinate ranges)

Key missingness highlights:

- `reopened_at` has very high missingness
- `acknowledged_at` has substantial missingness
- image-related columns have high missingness

#### WebCrowd crowd judgments EDA

- Confirmed dataset size: **25,119 rows × 12 columns**
- Checked:
  - missingness (low for core columns)
  - duplicates
  - **label distribution** (including `-1` = no-response)
  - **duration distribution** + outliers
  - **coverage**: judgments per (topic, document) pair
  - **worker activity**: judgments per worker + average duration patterns

Important computed patterns:

- **No-response** rate is about **2%** (`label == -1`)
- Coverage is close to expected (~5 judgments per item); most pairs have exactly 5 judgments.

#### WebCrowd gold judgments EDA

- Confirmed dataset size: **14,432 rows × 3 columns**
- Examined:
  - gold label distribution across {-2,0,1,2,3,4}
  - confirmed **50 topics**

### Minimal cleaning/preprocessing steps

#### Clean SeeClickFix

- Dropped high-missing columns: `reopened_at`, `image_url`, `image_square_url`
- Standardized column names to **snake_case**
- Parsed timestamps:
  - `created_at`, `acknowledged_at`, `closed_at` → datetime
- Created derived operational features:
  - `was_acknowledged`
  - `time_to_acknowledge_hours`

#### Agency responsiveness analysis (operational leadership proxy)

Computed per-agency metrics (with a filter like ≥10 issues), including:

- total issues
- acknowledged count
- acknowledgment rate
- mean/median time-to-acknowledge

This provides an initial operational view of “leadership style” (Option A) as **responsiveness patterns**.

#### Clean WebCrowd crowd judgments

- Removed no-response rows (`label == -1`)
  - Dropped **502** rows → remaining **24,617**
- Dropped `url` column (not required for modeling/scoring)
- Enforced types:
  - `tid`, `label` → nullable integer
  - `duration` → numeric

#### Join crowd ↔ gold (objective evaluation backbone)

- Joined on `(tid, did)` to attach `gold_judgement` to each crowd row.
- Gold coverage after join is nearly complete (~99.9%).

#### Worker trustworthiness metrics (foundation)

Prepared worker trustworthiness computation using:

- accuracy vs gold (with **gold→crowd label mappings**),
- majority-vote agreement (peer consistency),
- behavioral/effort proxies from `duration` and `rationale`,
- stability over time (bin-based accuracy trend).

---

## What was done in `workers_vs_orgnizations.ipynb`

### Purpose

This notebook explicitly **does not redo EDA**. It builds comparable **trustworthiness scores** for:

- participants/workers (data quality / reliability proxies),
- agencies/organizations (responsiveness + governance consistency proxies),
- WebCrowd worker trust (gold-based),
- WebCrowd “platform governance” proxy at topic level,
  and exports structured CSV outputs.

### Shared scoring framework

A common composite score is used:

\[
T(e)=\sum\_{k=1}^{K} w_k \tilde{m}\_k(e)
\quad \text{with} \quad
w_k \ge 0,\ \sum w_k = 1
\]

Utilities implemented:

- robust 0–1 scaling via quantile clipping (reduces outlier distortion)
- metric direction handling: “higher is better” vs “lower is better” (inversion)
- evidence thresholding: only rank entities with enough data (e.g., min reports/issues/judgments)

### SeeClickFix: “Participants” trustworthiness (source × category groups)

Because SeeClickFix does **not** provide stable reporter user IDs, “participants” are modeled as:

**`participant_group = source × category`**

Per-report text quality proxies computed and aggregated by group:

- missingness of `summary`/`description`
- median lengths of `summary`/`description`
- combined text length median
- `comments_count` (engagement proxy)
- duplicate-risk proxy: top summary share (`top_summary_share`)

Composite score:

- normalized metrics combined with equal weights
- only groups with **n_reports ≥ 30** ranked

**Output written:**

- `outputs/participants_sourceXcategory_scores.csv`

### SeeClickFix: Agency (organization) trustworthiness

Agency is treated as the organization unit. Metrics computed:

- **Responsiveness**
  - `ack_rate`
  - `ack_time_median` (hours)
- **Follow-through**
  - `close_rate`
  - `close_time_median`
- **Stability / governance consistency**
  - month-level variability (std) of acknowledgment rate
  - month-level variability (std) of ack-time median
- **Engagement proxy**
  - `comments_mean` (note: comment _count_, not staff response text)

Composite score:

- normalized metrics combined with equal weights
- only agencies with **n_issues ≥ 10** ranked

**Output written:**

- `outputs/agencies_scores.csv`

### WebCrowd25K: Worker trustworthiness (gold-based, defensible)

Pipeline:

- load crowd judgments + gold labels
- remove no-response (`label == -1`)
- join gold
- compute per-worker metrics:
  - accuracy vs gold (after mapping gold scale to crowd scale)
  - within-1 tolerance, MAE
  - agreement with majority vote (excluding ties)
  - effort proxies: duration, “very fast” rate, empty rationale rate
  - stability proxies (accuracy across sequential bins)

Composite score:

- normalized metrics combined with equal weights
- only workers with **total_judgments ≥ 20** ranked

**Output written:**

- `outputs/webcrowd_worker_scores.csv`

### WebCrowd25K: “Platform governance” proxy (topic-level)

Interprets topic `tid` as a governance unit (how well the process yields correct aggregated outcomes):

- majority-vote accuracy vs gold
- tie rate (ambiguity/disagreement proxy)
- non-response rate (from raw `label == -1`)

Composite governance score:

- higher majority accuracy/within-1 is better
- lower tie rate / lower non-response is better

**Output written:**

- `outputs/webcrowd_governance_topic_scores.csv`

### Sensitivity analysis (weights robustness)

To test whether rankings are stable under reasonable weight changes:

- sampled weights using a Dirichlet distribution around baseline weights
- computed Spearman rank correlations vs baseline
- reported distribution (p10/median/p90/min)

### Interpretation notes (explicit limitations)

- **SeeClickFix participant groups** reflect group-level report-quality proxies (not individuals).
- **Agency score** reflects operational responsiveness/follow-through/stability (proxy for operational leadership style).
- **WebCrowd worker score** is the most objective because it is validated against gold labels.
- **Transparency** cannot be measured directly from SeeClickFix here (no staff response text); comment count is only a weak proxy.

---

## Next step: `expermenting_text_analysis.ipynb` (detailed plan)

### Goal

Implement **text-driven analysis** to complement the operational/scoring pipelines:

- **Worker/participant side (WebCrowd):** use `rationale` (+ optional `feedback`) to estimate additional trust signals and improve/validate worker scoring.
- **Leadership style side (SeeClickFix, Option A):** use citizen text (`summary`, `description`) to quantify **trust signals expressed by citizens**, and test how those signals relate to operational leadership styles (responsiveness patterns).

> Thesis framing for Option A: measure **operational leadership style** (responsiveness/consistency) and its impact on **trustworthiness outcomes and citizen-expressed trust signals**, not leadership personality traits.

### Data preparation (inputs/joins)

#### SeeClickFix

- Text inputs: `summary`, `description`
- Operational targets/fields:
  - `acknowledged_at`, `closed_at`, `status`, `agency`, `category`, `source`, `created_at`
- Derived variables to compute/reuse:
  - `time_to_ack_hours`, `time_to_close_hours`
  - `was_acknowledged`, `was_closed`
- Unit of analysis:
  - issue-level text → aggregated by agency/category/time windows

#### WebCrowd

- Text inputs: `rationale`, `feedback` (optional)
- Labels/targets:
  - `label` (crowd), `gold_judgement` (expert)
- Join:
  - crowd ↔ gold on `(tid, did)`

### NLP/text feature extraction plan

#### A) SeeClickFix text → citizen “trust signal” measures (no staff text available)

Compute issue-level features, then aggregate by agency:

1. **Sentiment / emotion intensity**

- quantify negativity/frustration
- aggregate by agency/category/month

2. **Responsiveness-complaint language**
   Create an interpretable indicator/lexicon for terms such as:

- “ignored”, “no response”, “still not fixed”, “again”, “never”, “always”, “nobody”, “unacceptable”
  Output:
- per-issue indicator/score → per-agency rate

3. **Urgency / harm language**
   Detect urgency/safety terms such as:

- “dangerous”, “hazard”, “accident”, “urgent”, “immediately”
  Use as a control/proxy for severity differences.

4. **Topic modeling / clustering (optional but useful)**
   Use embeddings (e.g., sentence transformers) to cluster issues into themes beyond the provided `category`, then:

- compare agencies within similar clusters (fairer governance comparisons)

#### B) WebCrowd rationale text → worker trust signals (validated)

1. **Rationale completeness**

- length, emptiness, repetition, template-like patterns

2. **Rationale–label consistency (“supportiveness”)**
   Estimate whether rationale supports the chosen label:

- baseline: keyword heuristics
- stronger: embedding-based classifier trained to predict gold agreement

3. **Predictive modeling (optional)**
   Predict whether a worker judgment matches gold using:

- rationale features (text)
- behavior proxies already available (`duration`, etc.)
  Evaluate on held-out judgments; aggregate to worker-level predicted reliability.

### Leadership style (Option A) operationalization (what to test)

Define agency operational leadership style as measurable dimensions:

- **speed/urgency orientation:** median/p90 ack time, median/p90 close time
- **engagement discipline:** acknowledgment rate
- **follow-through:** closure rate
- **consistency:** month-to-month variability of responsiveness

Then test relationships:

- do “slower/less consistent” agencies show higher citizen negativity and more responsiveness-complaint language?
- do relationships hold after controlling for issue type (category/topic clusters)?

### Evaluation & thesis-ready evidence

Because SeeClickFix has no ground-truth “trust” labels:

- treat findings as **associations** between operational style and citizen-expressed trust signals
- include robustness checks:
  - within-category comparisons
  - time-window sensitivity (by year/quarter/month)
  - alternative text models (lexicon vs embedding approach)

Because WebCrowd has gold labels:

- provide strong validation:
  - correlation between rationale-derived trust signals and gold-accuracy
  - predictive performance for correctness

### Planned outputs from the next notebook

Proposed outputs (CSV + figures):

- `outputs/scfx_issue_text_signals.csv` (issue-level sentiment/lexicon/topic fields)
- `outputs/scfx_agency_text_signal_summary.csv` (agency-level aggregated trust signals)
- `outputs/webcrowd_rationale_features.csv` (judgment-level rationale features)
- `outputs/webcrowd_worker_text_enhanced_scores.csv` (worker-level text-augmented trust score)
- figures:
  - agency style vs citizen negativity plots
  - clusters/topics vs response times
  - worker accuracy vs rationale quality plots

---

## Scope alignment with the thesis

With the current datasets:

- **Strongly measurable**
  - WebCrowd worker trustworthiness (gold-validated)
  - SeeClickFix agency responsiveness and consistency (operational leadership style)
  - citizen-expressed trust signals in text (perception)
- **Not directly measurable**
  - transparency (requires staff response text/policy documents)
  - true intent/honesty (only proxies)

This scope matches Option A and keeps claims defensible.
