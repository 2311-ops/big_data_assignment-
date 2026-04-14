# Anime Genre Prediction Pipeline Summary

This file is a full project walkthrough, from the raw CSV to the final NLP model and outputs. Use it as a speaking guide when someone asks what the project does, how it works, and why each step exists.

## 1. Project Goal

The goal of the project is to predict anime `genres` from the text `synopsis` field, while also using other useful metadata such as `type`, `source`, `rating`, and numeric popularity features.

This is a **multi-label classification** problem because one anime can belong to more than one genre at the same time, for example:

- `Action`
- `Adventure`
- `Fantasy`
- `Drama`

## 2. Dataset

The dataset file is `top_anime_dataset.csv`.

Important columns:

- `title`: anime title
- `type`: TV, Movie, OVA, etc.
- `source`: manga, novel, original, etc.
- `episodes`: number of episodes
- `status`: finished or currently airing
- `airing`: boolean indicator
- `rating`: age rating
- `score`: community score
- `scored_by`: number of users who scored it
- `popularity`: popularity rank
- `members`: members count
- `favorites`: favorites count
- `synopsis`: main text used for NLP
- `year`: release year
- `genres`: target label

Some columns are dropped because they are not useful for prediction or can leak information:

- `mal_id`
- `url`
- `title_english`
- `rank`

## 3. End-to-End Workflow

The pipeline runs in this order:

```text
top_anime_dataset.csv
  -> ingest.py
  -> preprocess.py
  -> analytics.py
  -> visualize.py
  -> cluster.py
  -> output files in results/
```

## 4. Ingestion

### What happens

`ingest.py` reads the raw CSV and saves a copy as `data_raw.csv`.

### Why we do it

- It creates a reproducible raw snapshot of the dataset.
- It separates the original file from later processed versions.
- It gives the pipeline a standard starting point.

### What to say

> We first ingest the raw anime dataset and store it as `data_raw.csv` so the pipeline always starts from a known input.

## 5. Cleaning

Cleaning is the first big part of preprocessing.

### 5.1 Dropping irrelevant columns

We remove:

- `mal_id`
- `url`
- `title_english`
- `rank`

### Why

- `mal_id` is just an ID.
- `url` is not useful for prediction.
- `title_english` is metadata, not a predictive feature.
- `rank` is leakage-prone because it is closely tied to community evaluation.

### What to say

> We drop columns that do not help the model or that could leak the answer, especially `rank`.

### 5.2 Handling missing values

We handle missing data in different ways depending on the column:

- `synopsis`: rows with missing synopsis are dropped
- `episodes`: filled with the median
- `year`: filled with the median
- `rating`: filled with the mode
- `score`: filled with the mean
- `scored_by`: filled with the median

### Why

- `synopsis` is the main NLP input, so missing text cannot be recovered reliably.
- Numeric columns are filled using robust statistics so we do not lose too many rows.
- Categorical columns are filled with the most common value when appropriate.

### What to say

> We drop rows without synopsis because text is the core feature of the task, but we impute missing numeric and categorical values to preserve as much data as possible.

### 5.3 Removing duplicates

We remove duplicate anime entries using `title`.

### Why

- Duplicates can bias training and evaluation.
- A duplicated title would effectively count twice.

### What to say

> We remove duplicate titles so the dataset reflects unique anime entries.

### 5.4 Normalizing genres

The `genres` column is a comma-separated string.

We:

- split each genre label by comma
- strip extra spaces
- standardize the genre list

### Why

- Genre labels must be clean for multi-label encoding.
- Small formatting differences would create inconsistent labels.

### What to say

> We normalize genre strings so every label is consistent before binarizing them.

## 6. Transformation

After cleaning, we transform the data into model-ready features.

### 6.1 Encoding categorical variables

We one-hot encode:

- `type`
- `source`
- `rating`
- `status`
- `airing`

### Why

- Machine learning models need numeric input.
- One-hot encoding turns each category into a binary feature.

### What to say

> We convert categorical fields into one-hot vectors so the model can use them numerically.

### 6.2 Scaling numeric variables

We standardize these numeric columns:

- `score`
- `episodes`
- `members`
- `favorites`
- `popularity`
- `year`
- `scored_by`

### Why

- Different numeric fields have very different ranges.
- Standardization puts them on the same scale.
- This helps linear models and clustering behave more consistently.

### What to say

> We scale the numeric features so large-value columns do not dominate smaller ones.

### 6.3 NLP preprocessing on synopsis

We clean the synopsis text by:

- converting to lowercase
- removing punctuation and special characters
- removing stopwords
- applying TF-IDF vectorization

We keep the top TF-IDF features, up to 500 in the final model stage.

### Why

- Lowercasing reduces duplicate token variants.
- Removing punctuation simplifies tokenization.
- Stopwords remove common words with little meaning.
- TF-IDF converts text into weighted numeric features.

### What to say

> We turn synopsis text into TF-IDF features so the model can learn which words and phrases are important for each genre.

### 6.4 Target binarization

The target `genres` is transformed into a binary matrix using `MultiLabelBinarizer`.

Example:

If an anime has:

```text
Action, Adventure, Fantasy
```

then the output target becomes:

```text
Action = 1
Adventure = 1
Fantasy = 1
Comedy = 0
Drama = 0
```

### Why

- This is a multi-label problem, not single-class classification.
- The model must predict several genres at once.

### What to say

> We use multi-label binarization because each anime can belong to multiple genres simultaneously.

## 7. Dimensionality Reduction

We also reduce unnecessary or noisy features.

### 7.1 Low-variance filtering

After one-hot encoding, we drop low-variance columns using a threshold of `0.01`.

### Why

- Features with almost no variation do not add useful information.
- Removing them reduces noise and model size.

### What to say

> We remove nearly constant encoded columns because they do not help the model learn anything meaningful.

### 7.2 Dropping leakage-prone columns

We drop `rank`.

### Why

- It is highly derived from score-related data.
- Keeping it would make the model unrealistically strong and less honest.

### What to say

> We avoid label leakage by removing `rank`, since it is too closely related to outcome information.

### 7.3 TF-IDF feature selection

TF-IDF can create many text features, so we keep the strongest ones by variance.

### Why

- High-variance features are usually more informative.
- Keeping only the top features reduces dimensionality.

### What to say

> We keep the most informative TF-IDF tokens rather than using every possible token.

## 8. Discretization

We also create bucketed versions of some numeric features.

### 8.1 Score bins

`score` is converted into quartile categories:

- `low`
- `average`
- `good`
- `excellent`

### Why

- It helps us describe anime quality in simpler terms.
- It is useful for interpretation and analysis.

### What to say

> We bin scores into quartiles so we can compare low-rated and high-rated anime more clearly.

### 8.2 Year bins

`year` is converted into decade groups:

- `pre-2000`
- `2000s`
- `2010s`
- `2020s`

### Why

- This makes temporal analysis easier.
- It gives a simpler view of historical anime trends.

### What to say

> We group years into decades so we can analyze trends across time more clearly.

## 9. NLP Model

The main model is built in `analytics.py`.

### Model type

- `TfidfVectorizer`
- `OneVsRestClassifier(LogisticRegression)`

### Why this model

- TF-IDF is a strong baseline for text classification.
- Logistic Regression is simple, fast, and interpretable.
- One-vs-rest allows multi-label prediction.

### How it works

1. Clean synopsis text
2. Convert synopsis into TF-IDF vectors
3. Train a binary classifier for each genre
4. Predict multiple genres for each anime

### Training and evaluation

We split the data into train and test sets and evaluate using:

- Micro F1
- Macro F1
- Hamming loss

### What these metrics mean

- **Micro F1**: overall performance across all labels
- **Macro F1**: average performance across genres, including rare ones
- **Hamming loss**: fraction of incorrect label decisions

### What to say

> We train a multi-label logistic regression model on TF-IDF synopsis features and evaluate it with micro F1, macro F1, and hamming loss.

## 10. Analytics Outputs

The project generates three insight files.

### `insight1.txt`

Shows the top genres by frequency.

### `insight2.txt`

Shows average score per genre.

### `insight3.txt`

Shows NLP model performance:

- micro F1
- macro F1
- hamming loss
- best predicted genres
- hardest predicted genres
- full classification report

### What to say

> The analytics step gives both descriptive insights about genre distribution and predictive insights about model performance.

## 11. Visualization

`visualize.py` creates `summary_plot.png` with three subplots:

1. Genre frequency bar chart
2. Score distribution by anime type
3. Correlation heatmap of numeric engagement features

### Why

- The genre chart shows imbalance.
- The type boxplot shows how scores vary across formats.
- The correlation heatmap shows how numeric popularity measures relate to each other.

### What to say

> We visualize the data to understand imbalance, score differences across anime types, and correlations among popularity metrics.

## 12. Clustering

`cluster.py` runs K-Means on:

- `score`
- `members`
- `favorites`
- `popularity`
- `episodes`

### Why

- Clustering helps identify groups of anime with similar engagement patterns.
- It gives an unsupervised view of the data.

### Output

`clusters.txt` lists:

- cluster number
- number of samples in each cluster
- centroid interpretation in z-score form

### What to say

> Besides classification, we also cluster anime by numeric engagement features to find groups with similar popularity and score patterns.

## 13. Final Outputs

After the full pipeline, we get:

- `data_raw.csv`
- `data_preprocessed.csv`
- `insight1.txt`
- `insight2.txt`
- `insight3.txt`
- `summary_plot.png`
- `clusters.txt`

## 14. Short Explanation You Can Say in Class

> We built an end-to-end Docker-based big data pipeline for anime genre prediction. First we ingest the raw dataset, then we clean missing values, remove duplicates, normalize genres, and drop irrelevant or leakage-prone columns. After that we transform categorical fields with one-hot encoding, scale the numeric features, clean synopsis text, and convert text into TF-IDF vectors. We bin the multi-label genre target with `MultiLabelBinarizer`, train a `OneVsRestClassifier(LogisticRegression)` model, evaluate it with micro and macro F1, generate descriptive insights and plots, and finally run K-Means clustering on engagement features. The whole process is reproducible inside Docker.

## 15. If You Are Asked "Why This Design?"

Answer like this:

- We used Docker so the environment is reproducible.
- We used TF-IDF because it is a strong, simple baseline for text.
- We used logistic regression because it works well for sparse high-dimensional features.
- We used one-vs-rest because the task is multi-label.
- We used cleaning and scaling because raw data is messy and mixed in type.
- We used clustering and plots to provide analysis beyond just prediction.

## 16. Common Questions and Answers

### Q: Why is this a multi-label problem?

A: Because one anime can have more than one genre at the same time.

### Q: Why did you remove `rank`?

A: Because it is too closely tied to quality and can leak information into the model.

### Q: Why did you drop rows without synopsis?

A: Because synopsis is the core input to the NLP model and cannot be reliably reconstructed.

### Q: Why did you use TF-IDF instead of embeddings?

A: TF-IDF is simpler, fast, and a strong baseline for a course project.

### Q: Why logistic regression?

A: It is efficient, interpretable, and works well with sparse text features.

### Q: Why K-Means?

A: To find hidden groups of anime based on numeric engagement behavior.

## 17. Final Confidence Line

If you want one strong final sentence:

> This project takes raw anime metadata and synopsis text, cleans and transforms it, trains a multi-label NLP classifier to predict genres, and produces both analytical insights and visual summaries, all in a reproducible Docker pipeline.

