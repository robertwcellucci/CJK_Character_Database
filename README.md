# CJK_Character_Database
A pandas + SQLite ETL pipeline that ingests Unicode's Unihan dataset and four regional CJK educational standards into a normalized relational schema, then runs cross-national set comparisons to surface a 2,573-character pan-CJK literacy consensus.

**Author:** Robert W Cellucci  
**Stack:** Python 3.13 · pandas · SQLite3  
**Notebooks:** `CC_Database_Creation_Notebook.ipynb` · `CC_Database_Analysis_Notebook.ipynb`

---

## Overview

This project builds a unified SQLite database of CJK (Chinese-Japanese-Korean) logographic characters, then queries it to find cross-national consensus on which characters are truly foundational to East Asian literacy.

The database integrates four independent educational and reference standards into a single relational schema anchored on Unicode codepoints: the Unicode Consortium's Unihan dataset, Taiwan's Ministry of Education list, China's HSK 3.0 proficiency exam, and Japan's Joyo and Jinmeiyou kanji lists. The analysis notebook queries this database to identify where the curricula of Japan, Taiwan, Korea, and China actually agree, producing a set of **2,573 characters** that have been independently ratified as essential by three national educational systems.

---

## The Central Question

Chinese, Japanese, and Korean each maintain their own official character standards for education and literacy. Yet all three draw from the same logographic pool. A Japanese learner working through the Jōyō list, a Mandarin learner working through HSK 3.0, a Traditional Chinese learner working through Taiwan's MoE standard, and a Korean learner working through the Educational Hanja list are all engaging with largely the same script, but with no published map of where those curricula actually overlap.

**This project builds that map.**

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
    ├── kanjidic2.xml
    └── kanji_freq_report.tsv
```

---

## Data Sources

| Source | Coverage | Format |
|---|---|---|
| **Unihan** (Unicode Consortium) | Pan-CJK readings, variants, dictionary references | TSV |
| **Taiwan MoE** (4,808-character list) | Traditional Chinese education standard | TSV (pre-processed) |
| **HSK 3.0** | Mainland Chinese proficiency exam characters | CSV |
| **Heisig RTK** (5th & 6th ed.) | Western learner kanji sequence | CSV |

---

## Technical Architecture

### Database Design

The database is structured around **Unicode codepoints as the universal primary key.** Every CJK character maps to exactly one codepoint, making codepoints a stable, encoding-agnostic anchor for all foreign-key relationships across tables.

#### Core Table: `character_table`

The spine of the entire schema. Every other table holds a foreign key back to `character_table.codepoint`. Characters accumulate here from each source via `INSERT OR IGNORE`, so no codepoint is duplicated as new sources are added.

| Column | Type | Description |
|---|---|---|
| `codepoint` | INTEGER (PK) | Unicode scalar value (e.g., `20013` for 中) |
| `literal` | TEXT | The actual character glyph, derived from `chr(codepoint)`. Redundant but human-readable. |

#### Supporting Tables

| Table | Description |
|---|---|
| `character_source` | Records which educational list(s) each codepoint belongs to (`China`, `Japan`, `Korea`, `Taiwan`) |
| `unihan_readings` | Mandarin, Cantonese, Korean, and Vietnamese readings from Unihan |
| `unihan_dict_indices` | Entry numbers in classical dictionaries (KangXi, Morohashi, Nelson, etc.) |
| `unihan_mappings` | Jōyō, Jinmeiyo, Korean MoE, and other list membership flags |
| `unihan_dict_data` | Stroke counts, radical numbers, frequency data |
| `unihan_variants` | Simplified ↔ Traditional and other variant codepoint mappings |
| `trad_simp_map` | Explicit Traditional-to-Simplified mapping derived from HSK data (3,165 pairs) |

### ETL Pipeline (Creation Notebook)

The creation notebook runs a standard extract-transform-load pipeline:

1. **Validate:** confirms all 11 required source files exist before any parsing begins
2. **Extract:** reads all Unihan TSV files (tab-separated, `#`-commented headers, no column names) into a single long-format DataFrame: `codepoint | field | value`
3. **Transform:** pivots the long DataFrame into a wide per-codepoint format; parses XML (KANJIDIC2) and CSV sources; derives traditional-form proxies for HSK characters
4. **Load:** writes to SQLite using `INSERT OR IGNORE` everywhere, making all runs idempotent; `CREATE TABLE IF NOT EXISTS` guards prevent duplicate table creation on re-runs
5. **Enforce referential integrity:** `PRAGMA foreign_keys = ON` is issued on every connection (SQLite disables FK enforcement by default and does not persist the setting across connections)

**Note on Unihan Pivot:** The six Unihan TSV files share a three-column schema and are concatenated before pivoting. This produces a wide table where each row is one codepoint and each column is a Unihan property field.

### Dependencies

```python
import pandas as pd    # DataFrame-based ETL staging
import sqlite3         # CPython built-in; no pip install needed
```

No third-party dependencies beyond pandas. Python 3.13 (conda base environment).

---

## Getting Started

### Prerequisites

- Python 3.x with `pandas` installed (e.g., via Anaconda)
- All 11 source files present in a `sources/` subdirectory (see Repository Structure above)

### Running the Project

Run the notebooks **in order:**

**Step 1: Build the database**
```
CC_Database_Creation_Notebook.ipynb
```
Run all cells top-to-bottom. The notebook validates source files first and will raise `FileNotFoundError` if any are missing. Output: `Chinese_Character_Database.db`.

**Step 2: Run the analysis**
```
CC_Database_Analysis_Notebook.ipynb
```
Read-only with respect to the database. No data is modified. Requires the `.db` file produced in Step 1.

Re-runs of the creation notebook are safe. The `INSERT OR IGNORE` and `CREATE TABLE IF NOT EXISTS` guards prevent duplication.

---

## Analytical Findings

The analysis notebook builds toward one central finding: the cross-validated, multi-institutional consensus on foundational CJK character literacy. It starts from Japan's lists in isolation, moves through pairwise comparisons, and arrives at the full three-way intersection. The sections below follow that progression.

### Section Map

| Section | Question | Source Lists Involved |
|---|---|---|
| **1** | Japan Exclusivity: which Japanese characters appear in no other regional list? | `Japan` |
| **2** | How characters from Japan's lists were simplified in China's script reforms | `Japan`, `trad_simp_map` |
| **3** | Japan ↔ Korea pairwise overlap | `Japan`, `Korea` |
| **4** | Japan ↔ Taiwan pairwise overlap | `Japan`, `Taiwan` |
| **5** | Pan-CJK three-way intersection: core finding | `Japan`, `Taiwan`, `Korea` |

---

### Section 1: Japan Exclusivity

Japan maintains two official kanji lists: the **Jōyō kanji** (常用漢字), 2,136 characters for general literacy, and the **Jinmeiyō kanji** (人名用漢字), 863 characters approved for personal names. Together these define the full scope of officially sanctioned kanji in Japan, totaling 3,003 characters.

Of those, **268 characters** appear on no other regional educational list. These are characters that Japan's curriculum considers essential but that Taiwan's MoE, Korea's Educational Hanja list, and China's HSK all omit. The set includes a significant proportion of Jinmeiyō kanji (name-use characters with limited general circulation) and a smaller number of Jōyō kanji with no clear cognate in other CJK educational frameworks.

A subsection of these 268 are true **kokuji** (国字), characters invented in Japan with no Chinese etymological origin, such as 働 (*hataraku*, to work) and 峠 (*tōge*, mountain pass). Because kokuji were never part of the Chinese character pool, they are by definition absent from all other regional lists.

### Section 2: Japan's Kanji in Mainland China's Simplification Reforms

Japan and Mainland China both reformed their official character sets in the twentieth century, independently simplifying stroke structures that had remained stable for centuries. Japan's Jōyō revisions (1946, 1981, 2010) produced *shinjitai* (新字体) variants; China's reforms (1956, 1964) produced *simplified characters* (简体字). The two systems did not coordinate, resulting in a complex mapping where some forms converge, others diverge, and some characters were simplified in only one system.

This section joins Japan's Jōyō list against the `trad_simp_map` table (3,165 Traditional → Simplified pairs derived from HSK data) to surface **463 Jōyō kanji** that have a recorded Mainland Chinese simplified counterpart. Results display the traditional form as used in Japan alongside the simplified form as used in the PRC. The query is scoped to Jōyō only; Jinmeiyō characters are unlikely to appear in HSK-derived simplification data.

### Section 3: Japan ↔ Korea Pairwise Overlap

Korea's Educational Hanja list (한문 교육용 기초 한자) designates 1,800 characters for middle and high school instruction. Like Japan's kanji system, Korean Hanja are traditional-form characters, drawn from the same script pool that predates both Japan's shinjitai reforms and China's simplification program. This shared traditional-form heritage makes Japan-Korea comparisons methodologically cleaner than any comparison involving China's simplified forms.

Japan's combined kanji lists and Korea's Educational Hanja list share **2,904 characters**, the largest pairwise overlap between any two individual national lists in this dataset.

### Section 4: Japan ↔ Taiwan Pairwise Overlap

Taiwan's Ministry of Education list (國語常用字表) designates 4,808 characters for standard Mandarin literacy instruction. Like Korea's Hanja list, it operates entirely in traditional forms. Taiwan's list is the largest of the four regional lists in this database, making it a generous comparator.

Japan's combined kanji lists and Taiwan's MoE standard share **2,599 characters**, a slightly smaller overlap than Japan-Korea despite Taiwan's larger list size. The reduction reflects the influence of Japan's shinjitai variants, which introduced codepoint-level divergences from the traditional forms Taiwan continues to use.

### Section 5: Core Finding: The Pan-CJK Three-Way Intersection

**2,573 characters** appear simultaneously on Japan's Jōyō list, Taiwan's Ministry of Education standard, and Korea's Educational Hanja list.

This set has been independently ratified by three distinct national educational authorities, across three languages, over decades of curriculum revision. No single institution designed this overlap. It emerged from convergent pedagogical judgment about what logographic literacy requires and represents the strongest available evidence for a cross-culturally validated foundation of CJK character literacy.

The `source_count = 3` column confirms every returned row genuinely satisfies all three membership conditions. This is a strict intersection, not a union or majority vote.

**Why is China (HSK) excluded from the core intersection?**  
HSK 3.0 is represented in this database in its traditional character forms (`hsk_trad`), not the simplified forms actually used in PRC classrooms. Including it in the intersection would compare traditional-form proxies against the genuine traditional-script standards of Japan, Taiwan, and Korea, which is a methodologically unsound equivalence. The 2,573-character set therefore represents the consensus of the three living traditional-character educational systems, untainted by the simplification asymmetry.

---

## Known Limitations

**Han Unification overcounts true overlap.**  
Unicode merged visually similar CJK characters from Chinese, Japanese, and Korean into single codepoints (Han Unification). Characters that are functionally distinct across regions but share a codepoint due to glyph unification are counted as one, inflating apparent overlap between scripts. Regional glyph variation is not captured by Unihan alone.

**HSK is represented in traditional forms only.**  
The `hsk_trad` source contains traditional written forms of HSK 3.0 vocabulary, not the simplified forms used in PRC classrooms. Cross-script comparisons for China reflect traditional character inventories, which are no longer the characters Mainland learners actually study.

**Kokuji coverage in Unihan is not exhaustive.**  
The kokuji identification in Section 1 uses Unihan's IRG source field as a proxy. Characters confirmed by this method are genuine kokuji; the true count may be slightly higher than what the query returns.

---

## Design Decisions

- **Codepoint as integer PK** rather than the raw `U+XXXX` string: avoids string parsing on every join, keeps the schema encoding-agnostic, and allows `chr(codepoint)` to regenerate the literal at any time.
- **`literal` column kept despite redundancy:** human-readable queries and spot-checking are significantly easier when the glyph is visible directly in query output.
- **Staging via temporary tables:** complex transformations (e.g., the HSK traditional-form derivation) are built in pandas and written to `_temp_char` before being inserted into the target table and dropped, keeping the ETL auditable.
- **`INSERT OR IGNORE` everywhere:** allows sources to be re-processed or added incrementally without needing to clear the database first.
- **Analysis notebook is strictly read-only:** the creation and analysis responsibilities are separated into two notebooks to prevent accidental data modification during analytical work.
- **Japan as the analytical anchor:** the analysis notebook uses Japan's Jōyō and Jinmeiyō lists as the primary operand for all comparisons, building outward to Korea, Taiwan, and the three-way intersection rather than treating all four systems as co-equal inputs.
