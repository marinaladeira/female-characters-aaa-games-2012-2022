# Female Playable Characters in Critically Acclaimed Video Games (2012-2022)

Data, notebooks and figures for the paper *Central but Not Controllable: Female Playable Characters in Critically Acclaimed Video Games, 2012--2022*.

The study combines game metadata from the [RAWG API](https://rawg.io/apidocs) with the character-level coding of the Kaggle dataset [*Gender Representation in Video Games*](https://www.kaggle.com/datasets/br33sa/gender-representation-in-video-games) (Palomar, 2022) to test four hypotheses on 60 critically acclaimed games (Metacritic >= 80), AAA and independent, released between 2012 and 2022:

- **H1:** the proportion of critically acclaimed games with a playable female protagonist is below 50%;
- **H2:** female characters are more likely to be sexualized than male characters;
- **H3:** the mean sexualization of playable female characters decreased over 2012-2022;
- **H4:** the sexualization of female characters differs across game genres.

## Repository structure

```
.
├── data/
│   ├── LICENSE                     # CC BY-NC 4.0 (data)
│   ├── raw/                        # snapshots used in the study (input of notebook 01)
│   │   ├── rawg_games_raw.csv      # RAWG API collection (1,280 titles, 28 Sep 2026)
│   │   ├── rawg_collection_info.json # date, time and parameters of the collection
│   │   ├── kaggle_games.csv        # Kaggle dataset - games table
│   │   ├── kaggle_characters.csv   # Kaggle dataset - characters table
│   │   └── kaggle_sexualization.csv# Kaggle dataset - sexualization index
│   ├── processed/                  # output of notebook 01, input of notebook 02
│   │   ├── games.csv               # 60 games, one row per game
│   │   └── characters.csv          # 578 characters, one row per character
│   └── qualitative/                # qualitative validation (Section III-D of the paper)
│       └── verification_flags.csv  # ambiguous codings, impact and sensitivity checks
├── figures/                        # figures produced by notebook 02
├── notebooks/
│   ├── 01_data_processing.ipynb    # collection, integration, cleaning and export
│   └── 02_data_analysis.ipynb      # descriptive analysis and hypothesis tests
├── requirements.txt
├── CITATION.cff
├── LICENSE                         # MIT (code)
└── .env.example
```

## How to reproduce

Requires Python 3.12.

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cd notebooks
jupyter nbconvert --to notebook --execute --inplace 01_data_processing.ipynb 02_data_analysis.ipynb
```

Or open the notebooks with `jupyter lab` and run them in order.

**Downloads are disabled by default.** `01_data_processing.ipynb` has `download_datasets = False`, so it reads the snapshots in `data/raw/` and reproduces the study sample exactly; no API key or Kaggle account is needed. Setting `download_datasets = True` collects fresh data from the RAWG API and Kaggle (copy `.env.example` to `.env` and fill in `RAWG_API_KEY`; Kaggle credentials are handled by `kagglehub`). Because RAWG is continuously updated, a new collection may return a different sample.

## Pipeline

1. **Collection** - RAWG API, all titles released 2010-2022 with a Metacritic score between 80 and 100 (`metacritic=80,100`, `ordering=-metacritic`), paging until the last page: 1,280 games.
2. **Integration** - titles normalized in both sources (lower case, apostrophes, accents, years in parentheses and punctuation removed, edition suffixes stripped, Roman numerals converted to Arabic) and joined by exact inner merge; five titles that differ between the sources are matched through manually verified equivalences. 60 of the 64 Kaggle games are matched (65 rows).
3. **Cleaning** - deduplication of editions and remasters (60 unique games), keeping the RAWG edition released in the same year as recorded in the Kaggle dataset, or the earliest one otherwise. Release-year typos in the Kaggle dataset (Far Cry 3, Hades) are documented in the notebook.
4. **Export** - `games.csv` and `characters.csv` (characters joined with the four dimensions of the sexualization index).

## Processed data

`games.csv` keeps the columns of the Kaggle games table plus:

| Column | Description |
|---|---|
| `year` | Release year (from RAWG, for the edition matched to the Kaggle release year) |
| `Metacritic_RAWG` / `Metacritic_Kaggle` | Metacritic score from each source |
| `Rating_RAWG_Audience`, `RatingsCount_RAWG` | RAWG user rating (0-5) and number of ratings |
| `team_pct_num` | % of women in the development team (numeric) |
| `char_pct_num` | % of non-male relevant characters (numeric) |

`characters.csv` keeps the columns of the Kaggle characters table (`Game` is the foreign key to `Game_Id` in `games.csv`) plus the four binary dimensions `Sexualized_clothing`, `Trophy`, `Damsel in Distress`, `Sexualized Cutscenes` and their sum `Total` (sexualization index, 0-4). Narrative relevance (`Relevance`) follows the original coding: `PA` protagonist, `DA` deuteragonist, `MC` main character, `SC` secondary character, `SK` sidekick, `MA` main antagonist.

## Main results

| Hypothesis | Primary test | Result | Decision |
|---|---|---|---|
| H1 | One-sided exact binomial | 14 of 60 games (23.33%), p = 2.11e-5 | Supported |
| H2 | One-sided Mann-Whitney U | U = 37623.5, p = 2.20e-11 | Supported |
| H3 | Pearson correlation (playable, N = 29) | r = -0.18, p = 0.342 | Not supported (insufficient evidence) |
| H4 | Permutation test on Kruskal-Wallis H (all female, N = 183) | H = 16.98, p = 0.010 | Exploratory, not robustly supported (game-level permutation p = 0.127) |

Robustness checks reported in the paper (all in `02_data_analysis.ipynb`):

- **H1**: 74 PA characters, more than one PA in 10 games; the result holds with the 44 games that have a male or female PA (14/44, p = 0.011) and with customizable protagonists counted as female (21/60, p = 0.014).
- **H2**: holds for visual (OR = 18.28) and narrative dimensions (OR = 12.15) separately, with standard errors clustered by game, and under the disputed codings in `data/qualitative/verification_flags.csv` (OR between 11.08 and 17.40).
- **H3**: Spearman (rho = -0.20, p = 0.295) and a logistic regression of index > 0 on release year, naive and clustered by game (p = 0.416 and 0.276), agree with the primary test; coding Ciri and Ellie as playable leaves it non-significant (r = -0.19, p = 0.29). The supplementary decline among all female characters (r = -0.22, p = 0.003) is not robust to clustering by game (p = 0.076).
- **Centrality without control**: in the 38 games with both a central (PA or MC) female and male character, 16 make only the male playable and 8 only the female (exact McNemar p = 0.15). Among female characters, sexualization is not associated with protagonism (PA 13.3% vs. other 16.1%, Fisher p = 1.0) or playability (17.2% vs. 15.6%, p = 0.79).
- **Subgroup counts** (Table IV of the paper): female characters, games and sexualized characters by period and genre.

The figures of the paper are `figures/fig_share_by_role.png` (Fig. 1) and `figures/fig_sexualization_year.png` (Fig. 2); `figures/fig_character_counts.png` appeared in an earlier version.

## Qualitative validation file

`data/qualitative/verification_flags.csv` lists the ambiguous codings found in these cases or in the general review of the sample, their potential impact, the sensitivity check and the resolution. Following the paper, the index is used without recoding: flagged scores are kept and their impact is tested.

## Data sources and credits

- Game metadata: [RAWG Video Games Database API](https://rawg.io/apidocs).
- Character coding: Palomar, *Gender Representation in Video Games*, Kaggle, 2022 - <https://www.kaggle.com/datasets/br33sa/gender-representation-in-video-games>.

## License and data attribution

- **Code** (notebooks): [MIT License](LICENSE), Copyright (c) 2026 Marina Ladeira.
- **Data** (`data/`): [Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/), see [`data/LICENSE`](data/LICENSE). The data may be used, shared and adapted **for non-commercial purposes only**, with attribution.

The character data come from *Gender Representation in Video Games* by Palomar (2022), published on [Kaggle](https://www.kaggle.com/datasets/br33sa/gender-representation-in-video-games) under CC BY-NC 4.0. The game metadata come from the [RAWG Video Games Database](https://rawg.io) API and remain subject to the RAWG API terms of use.

**Changes to the original data.** The files `data/raw/kaggle_*.csv` are copies of the Kaggle tables, re-saved without changes to their content. The files in `data/processed/` are adapted versions: the games were filtered to those with a Metacritic score of at least 80 on RAWG, integrated with RAWG metadata by normalized title, deduplicated to one edition per game and supplemented with derived columns; the characters were restricted to the selected games and joined with the sexualization dimensions. All steps are documented in `notebooks/01_data_processing.ipynb`.

## Citation

If you use these data or code, please cite:

```bibtex
@misc{ladeira2026code,
  author       = {Ladeira, Marina},
  title        = {Female characters in critically acclaimed games (2012--2022): Data and code},
  year         = {2026},
  howpublished = {GitHub repository},
  version      = {v1.0},
  url          = {https://github.com/marinaladeira/female-characters-aaa-games-2012-2022}
}
```

The results reported in the paper correspond to release **v1.0** of this repository.
