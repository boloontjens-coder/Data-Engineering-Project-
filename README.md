# Predicting Commercial Success in Contemporary Cinema

A Data Engineering course project that combines three public sources into a
single integrated analytical workflow: API ingestion, web scraping, regular
expression cleaning, relational storage, distributed computation, graph
analytics, and natural language processing.

**Course.** Data Engineering, Prof. Dr. Sofie Goethals.
**Programme.** Master of Business Engineering, University of Antwerp.
**Academic year.** 2025 to 2026.
**Authors.** Bo Loontjens, Timon Van Reeth, Thibaud Verschueren.

The written report and the slide deck are submitted separately on
Blackboard. This repository contains the code and figures only.

---

## Research question

Which observable attributes of a film are associated with its commercial
success, how does the country of production mediate that success, and is the
sentiment of news coverage surrounding a film related to its audience
reception?

The question is deliberately simple. Its main role is to scaffold an
end-to-end demonstration of the course's core techniques against a single,
small, real dataset.

---

## Data sources

| Source | Role | Access |
|---|---|---|
| The Movie Database (TMDb) | Catalogue of ~200 popular films with budget, revenue, runtime, genres, and production countries | REST API, requires a free API key |
| REST Countries | Demographic context (region, sub-region, population) for every ISO-3166 country code | REST API, no authentication |
| Google News RSS | Recent news headlines for the top ten films, used as raw material for sentiment analysis | RSS feed, parsed with BeautifulSoup |

---

## Pipeline

1. **Ingest** — TMDb (`requests`), REST Countries (`requests`), Google News RSS (`feedparser` + `BeautifulSoup`).
2. **Clean** — Regular expressions (`re`) and `pandas` for date parsing, title normalisation, whitespace, and ASCII transliteration.
3. **Store** — SQLite three-table schema (`countries`, `movies`, `reviews`) with INNER JOIN, GROUP BY / HAVING, and nested SELECT queries.
4. **Compute** — Apache Spark DataFrame (`groupBy`, `Window` + `rank`).
5. **Graph** — NetworkX weighted genre co-occurrence graph with degree, betweenness, and closeness centrality, plus greedy modularity community detection.
6. **NLP** — HuggingFace `transformers` pipeline running `distilbert-base-uncased-finetuned-sst-2-english` on the scraped headlines.
7. **Visualise** — `matplotlib` figures saved to `figures/`.

---

## Repository layout

```
.
├── DE_Project_Movie_Success.ipynb   Main notebook with all cell outputs
├── requirements.txt                 Python dependencies
├── README.md                        This file
├── .gitignore                       Patterns excluded from version control
└── figures/
    ├── fig1_budget_revenue.png      Budget against revenue, log scale
    ├── fig2_genre_rating_revenue.png  Revenue and rating by genre
    ├── fig3_genre_network.png       Genre co-occurrence network
    └── fig4_seasonality.png         Mean revenue by release month
```

The notebook is self-contained: every figure and every printed table is
already saved as cell output, so the notebook can be read top to bottom on
GitHub without re-running it.

---

## Setup

### Google Colab (easiest, no local install)

1. Open <https://colab.research.google.com>.
2. `File → Upload notebook` and pick
   `DE_Project_Movie_Success.ipynb` from this repository.
3. `Runtime → Run all`.
4. Paste a free TMDb v3 API key (the 32-character one from
   <https://www.themoviedb.org/settings/api>) when the `getpass` cell
   prompts for it.

End-to-end runtime is roughly five to ten minutes, dominated by the
HuggingFace model download (~250 MB) the first time it runs.

### Local Jupyter

```bash
git clone https://github.com/boloontjens-coder/Data-Engineering-Project-.git
cd Data-Engineering-Project-

python -m venv .venv
# Windows
.\.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt

# Optional: skip the prompt by exporting the key
export TMDB_API_KEY="your_v3_key_here"

jupyter notebook DE_Project_Movie_Success.ipynb
```

PySpark requires a Java runtime. If you do not have one,
[Adoptium Temurin 17](https://adoptium.net) is a good free option.

---

## Outputs the notebook produces

- `movies.db` — SQLite database with the cleaned, joined dataset (created
  locally when you run the notebook, not committed).
- `figures/fig1_budget_revenue.png` — budget against revenue, log scale.
- `figures/fig2_genre_rating_revenue.png` — revenue and rating by genre.
- `figures/fig3_genre_network.png` — genre co-occurrence network.
- `figures/fig4_seasonality.png` — mean revenue by release month.
- Inline tables and plots embedded in the notebook itself.

---

## Key findings

- The budget-to-revenue relationship is positive but heteroscedastic.
  Larger budgets raise the ceiling, not the floor.
- Action, adventure, and science-fiction dominate revenue; drama and
  documentary lead on audience rating. Commercial and critical success are
  only weakly aligned in this sample.
- Drama is the highest-centrality node in the genre co-occurrence network
  on all three measures.
- Mean revenue peaks in the summer and late-autumn release windows, which
  is consistent with studio release strategy.
- The news-headline sentiment signal is weakly positive but tentative
  given the small per-film sample.

A full discussion is in the written report submitted on Blackboard.

---

## Reflection on AI assistant use

Claude Code was used during development for code scaffolding (TMDb API
boilerplate, PySpark window syntax, NetworkX patterns, SQL query shape).
ChatGPT was used as a secondary tool for spelling and grammar review of the
written report. Correctness was verified by:

1. Running every suggested code block in the notebook and inspecting its
   output against a handful of manually verified cases.
2. Cross-checking figures produced by assistant-generated code against
   independent queries on the same dataframe.
3. Re-deriving every empirical claim in the report directly from the
   notebook output rather than from the assistant's prose.

---

## Limitations

- The sample is small (~200 films) and non-random. No inferential claim
  is warranted.
- TMDb's popularity endpoint is biased toward recent and English-language
  films, which under-represents non-Anglophone cinema.
- The sentiment model was fine-tuned on movie reviews rather than news
  headlines. Domain mismatch is a known source of error.

---

## License

This is a course project. No commercial use intended.
