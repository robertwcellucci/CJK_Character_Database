# CJK Character Database

**Which Chinese characters are truly universal?** This project answers that question by cross-referencing the official character education standards of Japan, Taiwan, and South Korea — and identifying the 2,573-character consensus that all three national systems independently ratify as foundational.

**Author:** Robert W. Cellucci  
**Stack:** Python 3.13 · pandas · SQLite3  
**Notebooks:** `CC_Database_Creation_Notebook.ipynb` · `CC_Database_Analysis_Notebook.ipynb`

---

## Key Finding

**2,573 characters** appear simultaneously on the official educational standards of Japan (Jōyō + Jinmeiyō kanji), Taiwan (Ministry of Education 4,808-character list), and South Korea (Educational Hanja list). These are codepoint-identical matches, verified against Unicode's Unihan dataset as the authoritative anchor.

This consensus set has a practical implication: a learner who masters these 2,573 characters has satisfied the foundational literacy requirements of three independent national curricula. No published cross-national map of this overlap previously existed.

### Pairwise Overlaps

| Comparison | Shared Characters |
|---|---|
| Japan ↔ Korea | 2,904 |
| Japan ↔ Taiwan | 2,599 |
| **Japan ↔ Taiwan ↔ Korea (core consensus)** | **2,573** |

The Japan–Taiwan overlap is smaller than Japan–Korea despite Taiwan's list being the largest (4,808 characters). This reflects Japan's postwar shinjitai script reforms: simplified character variants introduced codepoint-level divergences from the traditional forms that Taiwan continues to use.

### Japan-Exclusive Characters

**268 characters** from Japan's lists appear on no other regional standard. This set includes a significant proportion of Jinmeiyō kanji (approved for personal names but rarely encountered in general text) and a subset of true **kokuji** (国字) — characters invented in Japan with no Chinese etymological origin, such as 働 (*hataraku*, to work) and 峠 (*tōge*, mountain pass). Because kokuji were never part of the shared logographic pool, their absence from all other regional lists is definitionally expected.

---

## The Problem This Solves

Chinese, Japanese, and Korean each maintain independent official character standards for education and literacy. All three draw from the same logographic pool, yet there is no published map of where those curricula actually agree.

A Japanese learner working through the Jōyō list, a Mandarin learner working through HSK 3.0, a Traditional Chinese learner working through Taiwan's MoE standard, and a Korean learner working through the Educational Hanja list are all engaging with largely the same script — but no reference existed to quantify how much they overlap.

**This project builds that map.**

---

## Analytical Findings

The analysis notebook proceeds incrementally toward the central finding, using Japan's lists as the primary analytical anchor and building outward through pairwise comparisons to the three-way intersection.

### Section Map

| Section | Question | Lists Involved |
|---|---|---|
| **1** | Japan Exclusivity: which Japanese characters appear in no other regional list? | Japan |
| **2** | How Japan's traditional-form kanji relate to China's simplification reforms | Japan, `trad_simp_map` |
| **3** | Japan ↔ Korea pairwise overlap | Japan, Korea |
| **4** | Japan ↔ Taiwan pairwise overlap | Japan, Taiwan |
| **5** | Pan-CJK three-way intersection: core finding | Japan, Taiwan, Korea |

---

### Section 1: Japan Exclusivity

Japan maintains two official kanji lists: the **Jōyō kanji** (常用漢字), 2,136 characters for general literacy, and the **Jinmeiyō kanji** (人名用漢字), 863 characters approved for personal names. Together these define the full scope of officially sanctioned kanji in Japan, totaling 3,003 characters.

Of those, **268 characters** appear on no other regional educational list — characters that Japan's curriculum considers essential but that Taiwan's MoE, Korea's Educational Hanja list, and China's HSK all omit. The set includes a significant proportion of Jinmeiyō kanji (name-use characters with limited general circulation) and a smaller number of Jōyō kanji with no clear cognate in other CJK educational frameworks.

A subsection of these 268 are true **kokuji** (国字), characters invented in Japan with no Chinese etymological origin, such as 働 (*hataraku*, to work) and 峠 (*tōge*, mountain pass). Because kokuji were never part of the Chinese character pool, they are by definition absent from all other regional lists.

### Section 2: Japan's Kanji in Mainland China's Simplification Reforms

Japan and Mainland China both reformed their official character sets in the twentieth century, independently simplifying stroke structures that had remained stable for centuries. Japan's Jōyō revisions (1946, 1981, 2010) produced *shinjitai* (新字体) variants; China's reforms (1956, 1964) produced *simplified characters* (简体字). The two systems did not coordinate, resulting in a complex mapping where some forms converge, others diverge, and some characters were simplified in only one system.

This section joins Japan's Jōyō list against the `trad_simp_map` table (3,165 Traditional → Simplified pairs derived from HSK data) to surface **463 Jōyō kanji** that have a recorded Mainland Chinese simplified counterpart. Results display the traditional form as used in Japan alongside the simplified form as used in the PRC. The query is scoped to Jōyō only; Jinmeiyō characters are unlikely to appear in HSK-derived simplification data.

### Section 3: Japan ↔ Korea Pairwise Overlap

Korea's Educational Hanja list (한문 교육용 기초 한자) designates 1,800 characters for middle and high school instruction. Like Japan's kanji system, Korean Hanja are traditional-form characters drawn from the same script pool that predates both Japan's shinjitai reforms and China's simplification program. This shared traditional-form heritage makes the Japan–Korea comparison methodologically cleaner than any comparison involving China's simplified forms.

Japan's combined kanji lists and Korea's Educational Hanja list share **2,904 characters** — the largest pairwise overlap between any two individual national lists in this dataset.

### Section 4: Japan ↔ Taiwan Pairwise Overlap

Taiwan's Ministry of Education list (國語常用字表) designates 4,808 characters for standard Mandarin literacy instruction. Like Korea's Hanja list, it operates entirely in traditional forms. Taiwan's list is the largest of the four regional lists in this database, making it a generous comparator.

Japan's combined kanji lists and Taiwan's MoE standard share **2,599 characters** — a slightly smaller overlap than Japan–Korea despite Taiwan's larger list size. The reduction reflects the influence of Japan's shinjitai variants, which introduced codepoint-level divergences from the traditional forms Taiwan continues to use.

### Section 5: Core Finding — The Pan-CJK Three-Way Intersection

**2,573 characters** appear simultaneously on Japan's Jōyō list, Taiwan's Ministry of Education standard, and Korea's Educational Hanja list.

This set has been independently ratified by three distinct national educational authorities, across three languages, over decades of curriculum revision. No single institution designed this overlap — it emerged from convergent pedagogical judgment about what logographic literacy requires. It represents the strongest available evidence for a cross-culturally validated foundation of CJK character literacy.

The `source_count = 3` column confirms every returned row genuinely satisfies all three membership conditions. This is a strict intersection, not a union or majority vote.

**Why is China (HSK) excluded from the core intersection?**  
HSK 3.0 is represented in this database in its traditional character forms (`hsk_trad`), not the simplified forms actually used in PRC classrooms. Including it in the intersection would compare traditional-form proxies against the genuine traditional-script standards of Japan, Taiwan, and Korea — a methodologically unsound equivalence. The 2,573-character set therefore represents the consensus of the three living traditional-character educational systems, untainted by the simplification asymmetry.

---

## Technical Implementation

### Database Design

The database is structured around **Unicode codepoints as the universal primary key.** Every CJK character maps to exactly one codepoint, making codepoints a stable, encoding-agnostic anchor for all foreign-key relationships across tables.

#### Core Table: `character_table`

The spine of the schema. Every other table holds a foreign key back to `character_table.codepoint`. Characters accumulate here from each source via `INSERT OR IGNORE`, so no codepoint is duplicated as new sources are added.

| Column | Type | Description |
|---|---|---|
| `codepoint` | INTEGER (PK) | Unicode scalar value (e.g., `20013` for 中) |
| `literal` | TEXT | The character glyph, derived from `chr(codepoint)`. Redundant but human-readable. |

#### Supporting Tables

| Table | Description |
|---|---|
| `character_source` | Records which educational list(s) each codepoint belongs to (`China`, `Japan`, `Korea`, `Taiwan`) |
| `dic_index_table` | Entry numbers in classical dictionaries (KangXi, Morohashi, Nelson, etc.) |
| `trad_simp_map` | Explicit Traditional-to-Simplified mapping derived from HSK data (3,165 pairs) |

### ETL Pipeline

The creation notebook runs a standard extract-transform-load pipeline:

1. **Validate** — confirms all 11 required source files are present before any parsing begins; raises `FileNotFoundError` on missing inputs
2. **Extract** — reads all Unihan TSV files (tab-separated, `#`-commented headers) into a single long-format DataFrame: `codepoint | field | value`
3. **Transform** — pivots the long DataFrame into a wide per-codepoint format; parses CSV sources; handles the many-to-one simplified↔traditional character mapping (e.g., 發 and 髮 both simplify to 发) via an explode step
4. **Load** — writes to SQLite using `INSERT OR IGNORE` everywhere, making all runs idempotent; `CREATE TABLE IF NOT EXISTS` guards prevent duplicate table creation on re-runs
5. **Enforce referential integrity** — `PRAGMA foreign_keys = ON` is issued on every connection (SQLite disables FK enforcement by default and does not persist the setting)

**Note on Unihan pivot:** The six Unihan TSV files share a three-column schema and are concatenated before pivoting. This produces a wide table where each row is one codepoint and each column is a Unihan property field.

### Dependencies

```python
import pandas as pd    # DataFrame-based ETL staging
import sqlite3         # CPython built-in; no pip install needed
```

No third-party dependencies beyond pandas. Python 3.13 (conda base environment).

---

## Data Sources

| Source | Coverage | Format |
|---|---|---|
| **Unihan** (Unicode Consortium) | Pan-CJK readings, variants, dictionary references | TSV |
| **Taiwan MoE** (4,808-character list) | Traditional Chinese education standard | TSV (pre-processed) |
| **HSK 3.0** | Mainland Chinese proficiency exam characters (9 levels, 2021 revision) | CSV |
| **Heisig RTK** (5th & 6th ed.) | Western learner kanji sequence with stroke counts and readings | CSV |

The Heisig RTK data is included in the database schema for extensibility — it provides learner-sequence metadata (Heisig index numbers, stroke counts, JLPT level) joinable to the core character table — but is not the subject of a standalone analytical section in the current version.

---

## Repository Structure

```
├── CC_Database_Creation_Notebook.ipynb   # ETL pipeline: builds the database from source files
├── CC_Database_Analysis_Notebook.ipynb   # Read-only queries and analytical findings
├── Chinese_Character_Database.db         # The compiled SQLite database (output of creation notebook)
└── sources/
    ├── Unihan_Readings.tsv
    ├── Unihan_DictionaryIndices.tsv
    ├── Unihan_OtherMappings.tsv
    ├── Unihan_DictionaryLikeData.tsv
    ├── Unihan_IRGSources.tsv
    ├── Unihan_Variants.tsv
    ├── moe_4808_unicode.tsv
    ├── hsk_3.0_characters.csv
    ├── heisig-kanjis.csv
```

---

## Getting Started

### Prerequisites

- Python 3.x with `pandas` installed (e.g., via Anaconda)
- All source files present in a `sources/` subdirectory

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
- **Staging via temporary tables:** complex transformations (e.g., the HSK traditional-form derivation) are built in pandas and written to `_temp_char` before being inserted into the target table and dropped, keeping the ETL auditable.
- **`INSERT OR IGNORE` everywhere:** allows sources to be re-processed or added incrementally without needing to clear the database first.
- **Analysis notebook is strictly read-only:** creation and analysis responsibilities are separated into two notebooks to prevent accidental data modification during analytical work.
- **Japan as the analytical anchor:** the analysis notebook uses Japan's Jōyō and Jinmeiyō lists as the primary operand for all comparisons, building outward to Korea, Taiwan, and the three-way intersection rather than treating all four systems as co-equal inputs.

---

## Known Limitations

**Han Unification overcounts true overlap.**  
Unicode merged visually similar CJK characters from Chinese, Japanese, and Korean into single codepoints (Han Unification). Characters that are functionally distinct across regions but share a codepoint due to glyph unification are counted as one, inflating apparent overlap between scripts. Regional glyph variation is not captured by Unihan alone.

**HSK is represented in traditional forms only.**  
The `hsk_trad` source contains traditional written forms of HSK 3.0 vocabulary, not the simplified forms used in PRC classrooms. Cross-script comparisons for China reflect traditional character inventories, which are no longer the characters Mainland learners actually study.
