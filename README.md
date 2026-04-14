# Anime NLP Pipeline (Big Data Assignment 1)

End-to-end data pipeline for anime analytics and multi-label genre prediction using the MyAnimeList dataset.

## Team

- Youssef Hassan Younis (ID: 231001243)
- Khaled Mohamed Yehia (ID: 231001055)

## Project Structure

```text
.
|-- Anime-pipeline/
|   |-- ingest.py
|   |-- preprocess.py
|   |-- analytics.py
|   |-- visualize.py
|   |-- cluster.py
|   |-- pipeline_utils.py
|   |-- Dockerfile
|   |-- summary.sh
|   |-- top_anime_dataset.csv
|   `-- results/
|-- summary.md
`-- README.md
```

## Pipeline Flow

1. `ingest.py` loads the dataset and writes `data_raw.csv`
2. `preprocess.py` cleans/transforms data and writes `data_preprocessed.csv`
3. `analytics.py` creates `insight1.txt`, `insight2.txt`, `insight3.txt`
4. `visualize.py` creates `summary_plot.png`
5. `cluster.py` creates `clusters.txt`

## Run With Docker

```bash
cd Anime-pipeline
docker build -t anime-pipeline .
docker run -it --name anime-run anime-pipeline
```

Inside the container:

```bash
python ingest.py top_anime_dataset.csv
```

On the host, copy outputs from the container and clean up:

```bash
bash summary.sh
```

All generated artifacts are copied into `Anime-pipeline/results/`.

## Main Outputs

- `data_raw.csv`
- `data_preprocessed.csv`
- `insight1.txt`
- `insight2.txt`
- `insight3.txt`
- `summary_plot.png`
- `clusters.txt`

