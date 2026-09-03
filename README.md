# Football Analytics on the Cloud

A big-data analytics project on European football data — player stats, club performance, and market values — processed with Apache Spark over a Hadoop (HDFS) cluster, deployed on a Google Cloud Platform VM. Built as an individual project for INFS3208 (Cloud Computing) at the University of Queensland.

## What it does

The analysis notebook (`football_analysis.ipynb`) runs Spark SQL queries against ~600MB of match, player, and club data (9 CSV files — appearances, games, clubs, competitions, player valuations, and more) to answer questions like:

- Which club won each competition, by season?
- Who was the top goal scorer each season?
- How do two clubs compare head-to-head in a given season (e.g. Real Madrid vs. Barcelona, 2022)?
- Who was the highest-paid player each season, over the last 5 seasons?
- How has a player's market value changed over their last 10 seasons (e.g. Messi vs. Mbappé)?

Each query is written as parameterised Spark SQL over temp views, run against DataFrames loaded from HDFS, with results visualised via Matplotlib and printed as formatted tables.

## Architecture

```
Kaggle dataset (Transfermarkt player-scores data)
        │  downloaded via opendatasets
        ▼
HDFS (3 datanodes, replication factor 3)
        │
Spark cluster (1 master, 2 workers) ── Spark SQL ──▶ Jupyter Notebook (PySpark, Matplotlib, Pandas)
```

The whole stack — HDFS namenode/datanodes/resourcemanager, Spark master/workers, and a Spark-enabled Jupyter notebook — runs as a multi-container Docker Compose deployment on a GCP Compute Engine VM.

## Dataset

[Football Data from Transfermarkt](https://www.kaggle.com/datasets/davidcariboo/player-scores) (Kaggle, by David Cariboo) — appearances, clubs, competitions, games, game events/lineups, player valuations, and player records, well over the individual project's 10,000-record minimum.

## Tech stack

Apache Spark (Spark SQL), Hadoop/HDFS, Docker Compose, Google Cloud Platform (Compute Engine), Jupyter Notebook, PySpark, Pandas, Matplotlib.

## Running it

The Docker Compose setup (HDFS + Spark cluster configuration) is adapted from a course-provided template; the dataset loading workflow and all of the analysis in the notebook are original.

1. Clone this repo and bring up the cluster:
   ```bash
   sudo chmod -R 777 nbs/
   docker-compose up -d
   ```
2. Open the Jupyter Notebook at `http://<host>:8888`, the Spark master UI at `http://<host>:8080`, and the HDFS health page at `http://<host>:80/dfshealth.html`.
3. Get a Kaggle API token from [kaggle.com/settings](https://www.kaggle.com/settings) and download the dataset from within the notebook:
   ```python
   !pip install opendatasets pandas
   import opendatasets as od
   od.download("https://www.kaggle.com/datasets/davidcariboo/player-scores", data_dir="/home/jovyan/work/nbs")
   ```
   (You'll be prompted for your Kaggle username and API key — never commit `kaggle.json` to the repo.)
4. Create a `/dataset` directory in HDFS and load each CSV in:
   ```bash
   docker exec <namenode_container_id> hdfs dfs -put /home/jovyan/work/nbs/player-scores/<file>.csv /dataset/<file>.csv
   ```
5. Open `football_analysis.ipynb` and run the cells — it reads directly from `hdfs://namenode:9000/dataset/`.

## Results

**Champions by season** — total goals scored by the winning club each season.

<img width="712" height="424" alt="cell7" src="https://github.com/user-attachments/assets/1623fb52-3c11-4e7d-82ac-3560b23d3fb9" />

**Highest goal scorers by season** — top scorer per season, stacked across years.

<img width="856" height="424" alt="cell11" src="https://github.com/user-attachments/assets/befb708a-1161-44a4-a457-5e7765dde310" />

**Market value over time: Messi vs. Mbappé** — tracking both players' valuations across their last 10 seasons.

<img width="855" height="424" alt="cell15" src="https://github.com/user-attachments/assets/9b4f3deb-a019-4566-af7b-0742bf8d14c5" />

## Notes

This was built for a Cloud Computing course's "Big Data query and analytics" project track, so the multi-node HDFS/Spark cluster configuration and GCP deployment were graded requirements, not just implementation choices. The Docker Compose infrastructure is adapted from the course's base template; the analytical queries, dataset integration, and visualisations in the notebook are my own work.
