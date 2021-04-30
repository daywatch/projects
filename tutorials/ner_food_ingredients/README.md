<a href="https://www.youtube.com/watch?v=59BKHO_xBPA" target="_blank"><img src="https://user-images.githubusercontent.com/13643239/76709989-cd96db00-6703-11ea-927a-456200f74f45.png" width="300" height="auto" align="right" /></a>

<!-- SPACY PROJECT: AUTO-GENERATED DOCS START (do not remove) -->

# 🪐 spaCy Project: Analyzing how mentions of ingredients change over time (Named Entity Recognition)

**This project was created as part of a [step-by-step video tutorial](https://www.youtube.com/watch?v=59BKHO_xBPA).** It uses [`sense2vec`](https://github.com/explosion/sense2vec) and [Prodigy](https://prodi.gy) to bootstrap an NER model to detect ingredients [Reddit comments](https://files.pushshift.io/reddit/comments/) and to calculate how mentions change over time. The results were then used to create a [bar chart race visualization](https://public.flourish.studio/visualisation/1532208/) of selected ingredients.

## 📋 project.yml

The [`project.yml`](project.yml) defines the data assets required by the
project, as well as the available commands and workflows. For details, see the
[spaCy projects documentation](https://spacy.io/usage/projects).

### 🗂 Assets

The following assets are defined by the project. They can
be fetched by running [`spacy project assets`](https://spacy.io/api/cli#project-assets)
in the project directory.

| File | Source | Description |
| --- | --- | --- |
| `assets/reddit_r_cooking.jsonl` | URL | Extracted data from r/cooking on Reddit from 2012-2019 (2,174,118 comments) |
| [`assets/reddit_r_cooking_sample.jsonl`](assets/reddit_r_cooking_sample.jsonl) | Local | Randomly selected sample from the r/cooking data for annotation and testing (10,000 comments) |
| [`assets/food_patterns.jsonl`](assets/food_patterns.jsonl) | Local | Match patterns created with `sense2vec.teach` and `terms.to-patterns`, used to bootstrap annotation or at runtime via spaCy's `EntityRuler` (209 patterns) |
| [`assets/food_data.jsonl`](assets/food_data.jsonl) | Local | Full dataset annotated with `INGRED` (ingredient) entities (949 examples) |
| [`assets/counts.csv`](assets/counts.csv) | Local | Full entity counts by month, generated by running the trained model over the whole Reddit data. |
| `assets/tok2vec_cd8_model289.bin` | URL | Pretrained tok2vec weights to initialize the model |
| `assets/s2v_reddit_2015_md.tar.gz` | URL | sense2vec vectors trained on Reddit comments of 2015. Used to bootstrap the terminology list |

<!-- SPACY PROJECT: AUTO-GENERATED DOCS END (do not remove) -->

## 🧮 Results

| Model                                                                                                                         |   F-Score | Precision | Recall | # Examples |
| ----------------------------------------------------------------------------------------------------------------------------- | --------: | --------: | -----: | ---------: |
| **[spaCy](https://spacy.io)**<br />blank                                                                                      |     80.85 |     82.13 |  79.61 |        760 |
| **[spaCy](https://spacy.io)**<br /> [`en_vectors_web_lg`](https://spacy.io/models/en#en_vectors_web_lg) + tok2vec<sup>1</sup> | **84.95** |     87.28 |  82.74 |        760 |

1. Representations trained on 1 billion words from Reddit comments using
   [`spacy pretrain`](https://spacy.io/api/cli#pretrain) predicting the
   `en_vectors_web_lg` vectors (~8 hours on GPU). Download:
   [`tok2vec_cd8_model289.bin`](https://github.com/explosion/projects/releases/download/tok2vec/tok2vec_cd8_model289.bin)

## 📚 Data

Labelling the data took about 2.5 hours and was done manually using the patterns
(first half) and pretrained NER model (second half) to pre-highlight
suggestions. For more details, see the
[data creation and training workflow](#data-creation-and-training-workflow)
below. The raw text was sourced from the from the
[r/Cooking](https://www.reddit.com/r/Cooking/) subreddit.

### Raw data

| File                                                                                                                              |     Count | Description                                          |
| --------------------------------------------------------------------------------------------------------------------------------- | --------: | ---------------------------------------------------- |
| [`reddit_r_cooking.jsonl`](https://github.com/explosion/projects/releases/download/reddit_r_cooking.jsonl/reddit_r_cooking.jsonl) | 2,174,118 | Extracted data from Reddit (2012-2019).              |
| [`reddit_r_cooking_sample.jsonl`](reddit_r_cooking_sample.jsonl)                                                                  |    10,000 | Randomly selected sample for annotation and testing. |

### Labelled data, patterns and artifacts

| File                                         |   Count | Description                                                                                                                                                                                                                                                                                                                                                                                               |
| -------------------------------------------- | ------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`food_patterns.jsonl`](food_patterns.jsonl) |     209 | Match patterns created with [`sense2vec.teach`](https://github.com/explosion/sense2vec/tree/master#recipe-sense2vecteach) and [`terms.to-patterns`](https://prodi.gy/docs/recipes#terms-to-patterns). Can be used to bootstrap annotation with [`ner.manual`](https://prodi.gy/docs/recipes#ner-manual) or at runtime via spaCy's [`EntityRuler`](https://spacy.io/usage/rule-based-matching#entityruler) |
| [`food_data.jsonl`](food_data.jsonl)         |     949 | Full dataset annotated with `INGRED` (ingredient) entities.                                                                                                                                                                                                                                                                                                                                               |
| [`counts.csv`](counts.csv)                   | 113,577 | Full `INGRED` entity counts by month, generated by running the trained model over `reddit_r_cooking.jsonl`. See the [notebooks](#-notebooks) section for the code.                                                                                                                                                                                                                                        |

### Data format

The datasets are distributed in Prodigy's simple JSONL (newline-delimited JSON)
format. Each entry contains a `"text"` and a list of `"spans"` with the
`"start"` and `"end"` character offsets and the `"label"` of the annotated
entities. The data also includes the tokenization and meta information (name of
the subreddit and UTC timestamp). Here's a simplified example entry:

```json
{
  "text": "Where do you get the mock duck?",
  "meta": { "section": "Cooking", "utc": "1364690064" },
  "tokens": [
    { "text": "Where", "start": 0, "end": 5, "id": 0 },
    { "text": "do", "start": 6, "end": 8, "id": 1 },
    { "text": "you", "start": 9, "end": 12, "id": 2 },
    { "text": "get", "start": 13, "end": 16, "id": 3 },
    { "text": "the", "start": 17, "end": 20, "id": 4 },
    { "text": "mock", "start": 21, "end": 25, "id": 5 },
    { "text": "duck", "start": 26, "end": 30, "id": 6 },
    { "text": "?", "start": 30, "end": 31, "id": 7 }
  ],
  "spans": [
    {
      "start": 21,
      "end": 30,
      "token_start": 5,
      "token_end": 6,
      "label": "INGRED"
    }
  ],
  "answer": "accept",
  "_input_hash": -768143800,
  "_task_hash": -1459747024
}
```

### Data creation and training workflow

1. Create a phrase list using seed terms. Requires
   [`sense2vec`](https://github.com/explosion/sense2vec/tree/master#recipe-sense2vecteach)
   and a vectors package.

```bash
prodigy sense2vec.teach food_terms ./s2v_reddit_2015_md --seeds "garlic, avocado, cottage cheese, olive oil, cumin, chicken breast, beef, iceberg lettuce"
```

2. Convert the phrase list to a match patterns file.

```bash
prodigy terms.to-patterns food_terms --label INGRED --spacy-model blank:en > ./food_patterns.jsonl
```

3. Label data manually with help from the patterns.

```bash
prodigy ner.manual food_data blank:en ./reddit_r_cooking_sample.jsonl --label INGRED --patterns food_patterns.jsonl
```

4. Pretrain a model to check results and to improve later on. Uses the
   pretrained tok2vec weights
   [`tok2vec_cd8_model289.bin`](https://github.com/explosion/projects/releases/download/tok2vec/tok2vec_cd8_model289.bin).

```bash
prodigy train ner food_data en_vectors_web_lg --init-tok2vec ./tok2vec_cd8_model289.bin --output ./tmp_model --eval-split 0.2
```

4. Label more data by correcting the model's predictions.

```bash
prodigy ner.correct food_data_correct ./tmp_model ./reddit_r_cooking_sample.jsonl --label INGRED --exclude food_data
```

5. Train the final model.

```bash
prodigy train ner food_data,food_data_correct en_vectors_web_lg --init-tok2vec ./tok2vec_cd8_model289.bin --output ./food_model --eval-split 0.2 --n-iter 20
```

## 📓 Notebooks

The notebooks include the code used to preprocess the raw Reddit data, run the
trained model over the data and create the final counts.

| File                                                                   | Description                                                          |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------- |
| [`01_Preprocess_Reddit.ipynb`](notebooks/01_Preprocess_Reddit.ipynb)             | Preprocess and clean Reddit data and export JSONL for annotation.    |
| [`02_Process_text_and_counts.ipynb`](notebooks/02_Process_text_and_counts.ipynb) | Use the trained model to count entities.                             |
| [`03_Generate_counts.ipynb`](notebooks/03_Generate_counts.ipynb)                 | Generate the final counts for the visualization and select examples. |