# CJK Character Database

**Which Chinese characters are truly universal?** This project answers that question by cross-referencing the official character education standards of Japan, Taiwan, and South Korea — and identifying the 1,579-character consensus that all three national systems independently ratify as foundational.

**Author:** Robert W. Cellucci
**Stack:** Python 3.13 · pandas · SQLite3
**Notebooks:** `CC_Database_Creation_Notebook.ipynb` · `CC_Database_Analysis_Notebook.ipynb`

---

## Key Finding

**1,579 characters** appear simultaneously on the official educational standards of Japan (Jōyō + Jinmeiyō kanji, 2,999 characters), Taiwan (Ministry of Education, 4,808), and South Korea (Educational Hanja, 1,800). These are codepoint-identical matches, verified against Unicode's Unihan dataset as the authoritative anchor.

The interesting part is not the number but its distribution. **87.7% of Korea's entire Educational Hanja list falls inside the three-way core** — only 19 of its 1,800 characters appear on neither of the other two standards. The consensus set is, to a first approximation, Korea's list minus 221 characters. Japan and Taiwan both teach nearly everything Korea does, plus substantially more that they do not share with each other.

A learner who masters these 1,579 characters has satisfied the large majority of the foundational literacy requirements of three independent national curricula.

### Pairwise Overlaps

| Comparison | Shared | Share of the smaller list |
|---|---|---|
| Japan ↔ Taiwan | 2,395 | 79.9% of Japan's 2,999 |
| Korea ↔ Taiwan | 1,745 | 96.9% of Korea's 1,800 |
| Japan ↔ Korea | 1,615 | 89.7% of Korea's 1,800 |
| **Japan ↔ Taiwan ↔ Korea (core)** | **1,579** | **87.7% of Korea's 1,800** |

### List Composition

| Standard | List size | Exclusive | Shared by two | Shared by all three |
|---|---|---|---|---|
| Japan (Jōyō + Jinmeiyō) | 2,999 | 568 (18.9%) | 852 (28.4%) | 1,579 (52.7%) |
| Korea (Educational Hanja) | 1,800 | 19 (1.1%) | 202 (11.2%) | 1,579 (87.7%) |
| Taiwan (MoE) | 4,808 | 2,247 (46.7%) | 982 (20.4%) | 1,579 (32.8%) |

Korea's list is almost entirely inside the core. Taiwan's is the most divergent — 46.7% of it appears on neither of the others, the largest exclusive share of the three. So the consensus is close to the whole of the smallest list, carved out of two substantially larger and more divergent inventories, rather than one oversized list absorbing two smaller ones.

### Japan-Exclusive Characters

**451 characters** from Japan's lists appear on none of the other three national standards (Taiwan MoE, Korean Educational Hanja, HSK 3.0). This set includes a large proportion of Jinmeiyō kanji — approved for personal names but rarely encountered in general text — and a smaller number of Jōyō kanji with no clear cognate elsewhere.

True **kokuji** (国字), characters invented in Japan with no Chinese etymological origin such as 働 (*hataraku*, to work) and 峠 (*tōge*, mountain pass), are a structural subset of this group. This database cannot isolate them directly: doing so requires etymological data that Unihan's IRG source fields only approximate. The 451-character set is therefore best read as *characters absent from the other national curricula*, of which true kokuji are a small minority.

### Korea-Exclusive Characters

Only **19** characters appear on Korea's Educational Hanja list and neither of the others: 敎, 淸, 靑, 硏, 擧, 墻, 尙, 屛, 慙, 旣, 槪, 毁, 畓, 竝, 鄕, 鑛, 飮, 鬪, and one extension-block character. Most are older glyph forms that Korean retains while Japan and Taiwan use variants sitting at *different Unicode codepoints* — not different characters so much as different encodings of the same character. This is the mirror image of the Han unification limitation noted below.

---

## Methodology: What Counts as a National Standard

The database holds nine sources across four regions, and they are not equivalent. Two are not national standards at all:

| `source` | `source_region` | codepoints | national standard? |
|---|---|---|---|
| `Joyo` | Japan | 2,136 | yes — general-use kanji |
| `Jinmeiyo` | Japan | 863 | yes — name-use kanji |
| `RememberingTheKanji5th` | Japan | 3,030 | **no** — Heisig, a Western textbook |
| `RememberingTheKanji6th` | Japan | 3,000 | **no** — Heisig, a Western textbook |
| `KoreanEducationHanja` | Korea | 1,800 | yes — middle/high school Hanja |
| `KoreanName` | Korea | 6,367 | **no** — name inventory, not a curriculum |
| `TaiwanEducationHanzi` | Taiwan | 4,808 | yes — MoE literacy standard |
| `hsk_trad` / `hsk_simp` | China | 3,164 / 2,999 | excluded (see below) |

Every comparison query therefore filters on `source`, **not** `source_region`. Filtering on region silently unions Heisig into the Japanese operand and a 6,367-character name inventory into the Korean one, inflating every overlap figure.

An intersection can never exceed its smallest operand, so **1,800 is the hard ceiling on every figure in this project**. The analysis notebook asserts that ceiling directly, and the tier decomposition asserts that each standard's tiers reconcile to its published list size.

**Errata.** Earlier versions of this README and notebook reported the three-way core at 2,573 and Japan↔Korea at 2,904 — both above the 1,800 ceiling, and therefore arithmetically impossible. The cause was `source_region` filtering. All figures above are recomputed with correct scoping.

**Why China (HSK) is excluded from the core.** HSK 3.0 is represented in this database in its traditional character forms (`hsk_trad`), not the simplified forms actually used in PRC classrooms. Including it would compare traditional-form proxies against the genuine traditional-script standards of Japan, Taiwan, and Korea — a methodologically unsound equivalence. China's data is used descriptively in the simplification mapping section, not as a fourth operand.

---

## The Problem This Solves

Chinese, Japanese, and Korean each maintain independent official character standards for education and literacy. All three draw from the same logographic pool, yet there is no widely available map of where those curricula actually agree.

A Japanese learner working through the Jōyō list, a Mandarin learner working through HSK 3.0, a Traditional Chinese learner working through Taiwan's MoE standard, and a Korean learner working through the Educational Hanja list are all engaging with largely the same script — with no reference quantifying how much they overlap.

**This project builds that map.**

---

## Analytical Findings

The analysis notebook proceeds incrementally toward the central finding, using Japan's lists as the primary analytical anchor and building outward through pairwise comparisons to the three-way intersection.

| Section | Question | Standards Involved |
|---|---|---|
| **0** | Methodology: which lists count as a national standard? | all |
| **1** | Japan exclusivity: which Japanese characters appear on no other standard? | Japan |
| **2** | How Japan's traditional-form kanji relate to China's simplification reforms | Japan, `trad_simp_map` |
| **3** | Japan ↔ Korea pairwise overlap | Japan, Korea |
| **4** | Japan ↔ Taiwan pairwise overlap | Japan, Taiwan |
| **5** | Pan-CJK three-way intersection: core finding | all three |
| **6** | Window-function decomposition of list composition | all three |

### Section 2: Japan's Kanji in Mainland China's Simplification Reforms

Japan and Mainland China both reformed their official character sets in the twentieth century, independently simplifying stroke structures that had remained stable for centuries. Japan's Jōyō revisions (1946, 1981, 2010) produced *shinjitai* (新字体) variants; China's reforms (1956, 1964) produced *simplified characters* (简体字). The two systems did not coordinate, resulting in a complex mapping where some forms converge, others diverge, and some characters were simplified in only one system.

This section joins Japan's Jōyō list against the `trad_simp_map` table (**1,210** Traditional → Simplified pairs derived from HSK data) to surface **463 Jōyō kanji** with a recorded Mainland Chinese simplified counterpart. The query is scoped to Jōyō only; Jinmeiyō characters are unlikely to appear in HSK-derived simplification data.

### Sections 3–4: Pairwise Overlaps

Japan ↔ Taiwan is the largest pair at 2,395 characters, which is the direction list size alone predicts — Taiwan's standard is 2.7× the size of Korea's. Japan's shinjitai divergence is still visible: 2,395 of Japan's 2,999 is 79.9%, short of the near-total coverage a 4,808-character superset might otherwise imply.

Japan ↔ Korea reaches 1,615, or 89.7% of Korea's list. Korea ↔ Taiwan is higher still at 1,745 (96.9% of Korea's list) — Korea's Hanja inventory is nearly contained within Taiwan's.

### Section 6: Window-Function Decomposition

A single query computes the exclusive / shared-by-two / shared-by-three tiers for all three standards at once, using `COUNT(*) OVER (PARTITION BY codepoint)` to count standards per character without collapsing rows, and a nested `SUM(COUNT(*)) OVER (PARTITION BY standard)` to express each tier as a percentage of its own standard's list. The output produces the composition table above.

---

## Technical Implementation

### Database Design

The database is structured around **Unicode codepoints as the universal primary key.** Every CJK character maps to exactly one codepoint, making codepoints a stable, encoding-agnostic anchor for all foreign-key relationships across tables.

#### Core Table: `character_table`

The spine of the schema (9,840 rows). Every other table holds a foreign key back to `character_table.codepoint`. Characters accumulate here from each source via `INSERT OR IGNORE`, so no codepoint is duplicated as new sources are added.

| Column | Type | Description |
|---|---|---|
| `codepoint` | INTEGER (PK) | Unicode scalar value (e.g., `20013` for 中) |
| `literal` | TEXT | The character glyph, derived from `chr(codepoint)`. Redundant but human-readable. |

#### Supporting Tables

| Table | Rows | Description |
|---|---|---|
| `character_source` | 28,167 | Which educational list(s) each codepoint belongs to, with both a `source` and a `source_region` column |
| `dic_index_table` | 9,840 | Entry numbers in classical dictionaries (KangXi, Morohashi, Nelson, etc.) |
| `trad_simp_map` | 1,210 | Explicit Traditional-to-Simplified mapping derived from HSK data |

### ETL Pipeline

The creation notebook runs a standard extract-transform-load pipeline:

1. **Validate** — confirms all required source files are present before any parsing begins; raises `FileNotFoundError` on missing inputs
2. **Extract** — reads all Unihan TSV files (tab-separated, `#`-commented headers) into a single long-format DataFrame: `codepoint | field | value`
3. **Transform** — pivots the long DataFrame into a wide per-codepoint format; parses CSV sources; handles the many-to-one simplified↔traditional character mapping (e.g., 發 and 髮 both simplify to 发) via an explode step
4. **Load** — writes to SQLite using `INSERT OR IGNORE` everywhere, making all runs idempotent; `CREATE TABLE IF NOT EXISTS` guards prevent duplicate table creation on re-runs
5. **Enforce referential integrity** — `PRAGMA foreign_keys = ON` is issued on every connection (SQLite disables FK enforcement by default and does not persist the setting)

The analysis notebook opens with a validation section that checks table presence, row counts, and `PRAGMA foreign_key_check` before any analytical query runs. The current build reports zero foreign key violations.

### Dependencies

```python
import pandas as pd    # DataFrame-based ETL staging
import sqlite3         # CPython built-in; no pip install needed
import matplotlib      # analysis notebook charts only
```

Python 3.13 (conda base environment).

---

## Data Sources

| Source file | Provides | Format |
|---|---|---|
| `Unihan_OtherMappings.tsv` | Japan's Jōyō and Jinmeiyō lists, Korea's Educational Hanja and name inventory (via the `kJoyoKanji`, `kJinmeiyoKanji`, `kKoreanEducationHanja`, `kKoreanName` fields) | TSV |
| `Unihan_Readings.tsv`, `Unihan_DictionaryIndices.tsv`, `Unihan_DictionaryLikeData.tsv`, `Unihan_IRGSources.tsv`, `Unihan_Variants.tsv` | Pan-CJK readings, variants, dictionary references | TSV |
| `moe_4808_unicode.tsv` | Taiwan MoE 4,808-character standard | TSV (pre-processed) |
| `hsk_3.0_characters.csv` | Mainland Chinese proficiency exam characters (9 levels, 2021 revision) | CSV |
| `heisig-kanjis.csv` | Heisig RTK 5th/6th ed. learner sequence, stroke counts, JLPT level | CSV |

Japan's and Korea's standards are not separate downloads — they are derived from Unihan membership fields, which is why the source file list contains no file named for either country.

Heisig RTK is included in the schema for extensibility. It is **excluded from every comparison in this project** and is not a national standard; see the Methodology section.

---

## Repository Structure

```
├── CC_Database_Creation_Notebook.ipynb   # ETL pipeline: builds the database from source files
├── CC_Database_Analysis_Notebook.ipynb   # Read-only queries and analytical findings
├── Chinese_Character_Database.db         # The compiled SQLite database (output of creation notebook)
├── Unihan_Readings.tsv
├── Unihan_DictionaryIndices.tsv
├── Unihan_OtherMappings.tsv
├── Unihan_DictionaryLikeData.tsv
├── Unihan_IRGSources.tsv
├── Unihan_Variants.tsv
├── moe_4808_unicode.tsv
├── hsk_3.0_characters.csv
├── heisig-kanjis.csv
└── LICENSE
```

All source files live at the repository root, not in a subdirectory.

---

## Getting Started

### Prerequisites

- Python 3.x with `pandas` and `matplotlib` installed (e.g., via Anaconda)
- All source files present in the working directory

### Running the Project

Run the notebooks **in order:**

**Step 1 — Build the database**
```
CC_Database_Creation_Notebook.ipynb
```
Run all cells top-to-bottom. Source file validation runs first and will raise `FileNotFoundError` if any required file is missing. Output: `Chinese_Character_Database.db`.

**Step 2 — Run the analysis**
```
CC_Database_Analysis_Notebook.ipynb
```
Read-only with respect to the database. No data is written back to disk. Requires the `.db` file produced in Step 1.

Re-runs of the creation notebook are safe. The `INSERT OR IGNORE` and `CREATE TABLE IF NOT EXISTS` guards prevent duplication.

---

## Design Decisions

- **Codepoint as integer PK** rather than the raw `U+XXXX` string: avoids string parsing on every join, keeps the schema encoding-agnostic, and allows `chr(codepoint)` to regenerate the literal at any time.
- **`literal` column kept despite redundancy:** human-readable queries and spot-checking are significantly easier when the glyph is visible directly in query output.
- **Both `source` and `source_region` retained:** region is useful for describing the database's contents; `source` is what analysis must filter on. Keeping both, and documenting which to use, is safer than dropping either.
- **Staging via temporary tables:** complex transformations (e.g., the HSK traditional-form derivation) are built in pandas and written to `_temp_char` before being inserted into the target table and dropped, keeping the ETL auditable.
- **`INSERT OR IGNORE` everywhere:** allows sources to be re-processed or added incrementally without needing to clear the database first.
- **Analysis notebook is strictly read-only:** creation and analysis responsibilities are separated into two notebooks to prevent accidental data modification during analytical work.
- **Japan as the analytical anchor:** the analysis notebook uses Japan's Jōyō and Jinmeiyō lists as the primary operand for all comparisons, building outward to Korea, Taiwan, and the three-way intersection rather than treating all four systems as co-equal inputs.

---

## Known Limitations

**Han Unification overcounts true overlap.**
Unicode merged visually similar CJK characters from Chinese, Japanese, and Korean into single codepoints. Characters that are functionally distinct across regions but share a codepoint due to glyph unification are counted as one, inflating apparent overlap. Regional glyph variation is not captured by Unihan alone.

**Codepoint divergence undercounts it in the other direction.**
Korea's 19 exclusive characters are older glyph forms of characters Japan and Taiwan also teach, encoded at different codepoints. A codepoint-anchored comparison treats those as non-matches. Both effects are present, they do not cancel, and neither is quantified here.

**HSK is represented in traditional forms only.**
The `hsk_trad` source contains traditional written forms of HSK 3.0 vocabulary, not the simplified forms used in PRC classrooms. Cross-script comparisons for China reflect traditional character inventories, which are no longer the characters Mainland learners actually study.

**Non-standard sources are present but excluded.**
Heisig's *Remembering the Kanji* and the Korean name inventory are retained in the database for extensibility. Any query filtering on `source_region` rather than `source` will silently include them.
