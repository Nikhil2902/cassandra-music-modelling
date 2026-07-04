# NoSQL Data Modelling with Apache Cassandra

![Apache Cassandra](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apachecassandra&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat&logo=jupyter&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)

> Designed and implemented a NoSQL database using Apache Cassandra for a music streaming platform, applying query-first data modelling principles to solve three analytical queries efficiently.

---

## Overview

A music streaming platform collects user activity data — which songs users play, when, and in what order — but stores it as flat CSV files, making analytical queries slow and difficult. This project migrates that data into **Apache Cassandra**, a distributed NoSQL database optimised for high-speed reads on large datasets.

The key challenge in Cassandra is **query-first data modelling**: unlike relational databases where you normalise data and join tables at query time, in Cassandra each table is designed specifically for one query. This project demonstrates that pattern across three tables.

---

## The Three Queries

| Query | Question | Table |
|---|---|---|
| 1 | What song was played during session 338 at position 4? | `song_history` |
| 2 | What songs did user 10 listen to in session 182, in order? | `user_session_songs` |
| 3 | Who listened to the song 'All Hands Against His Own'? | `song_listeners` |

---

## Data Modelling Decisions

### Table 1 — song_history
```sql
PRIMARY KEY (session_number, item_in_session_number)
```
- **Partition Key:** `session_number` groups all events in a session on the same node
- **Clustering Key:** `item_in_session_number` enables filtering by exact position within a session
- Optimised for: WHERE session_number = ? AND item_in_session_number = ?

### Table 2 — user_session_songs
```sql
PRIMARY KEY ((user_id, session_number), item_in_session_number)
```
- **Composite Partition Key:** `(user_id, session_number)` uniquely identifies a user's session
- **Clustering Key:** `item_in_session_number` enables ORDER BY sorting at read time at no extra cost
- Optimised for: WHERE user_id = ? AND session_number = ? ORDER BY item_in_session_number

### Table 3 — song_listeners
```sql
PRIMARY KEY (song_title, user_id)
```
- **Partition Key:** `song_title` stores all listeners of a song together in one partition
- **Clustering Key:** `user_id` ensures each user appears only once per song
- Optimised for: WHERE song_title = ?

---

## Query Results

**Query 1** — Session 338, Position 4:
```
Artist: Faithless | Song: Music Matters (Mark Knight Dub) | Length: 495.31 seconds
```

**Query 2** — User 10, Session 182 (sorted by position):
```
Item 0 | Down To The Bone    | Keep On Keepin' On | Sylvie Cruz
Item 1 | Three Drives        | Greece 2000        | Sylvie Cruz
Item 2 | Sebastien Tellier   | Kilometer          | Sylvie Cruz
Item 3 | Lonnie Gordon       | Catch You Baby     | Sylvie Cruz
```

**Query 3** — Listeners of 'All Hands Against His Own':
```
Jacqueline Lynch
Tegan Levine
Sara Johnson
```

---

## Project Structure

```
cassandra-music-modelling/
│
├── README.md                        <- You are here
├── cassandra_music_modelling.ipynb  <- Main notebook (all 3 queries)
└── event_data.csv                   <- Source dataset (not included — see below)
```

---

## Dataset

The `event_data.csv` file contains music streaming event logs with the following fields:

| Index | Field | Description |
|---|---|---|
| 0 | artist | Artist name |
| 1 | firstName | User first name |
| 3 | itemInSession | Position of song in session |
| 4 | lastName | User last name |
| 5 | length | Song length in seconds |
| 8 | sessionId | Session identifier |
| 9 | song | Song title |
| 10 | userId | User identifier |

---

## Getting Started

### Prerequisites
```bash
pip install cassandra-driver pandas numpy jupyter
```

### Run Cassandra Locally
```bash
# Using Docker (recommended)
docker run --name cassandra -p 9042:9042 -d cassandra:latest

# Or install directly
# https://cassandra.apache.org/download/
```

### Run the Notebook
```bash
jupyter notebook cassandra_music_modelling.ipynb
```

Run cells in order — the notebook connects to Cassandra, creates the keyspace, builds each table, inserts data, executes queries, and cleans up.

---

## Tech Stack

- **Database:** Apache Cassandra
- **Language:** Python 3.x
- **Libraries:** cassandra-driver · pandas · numpy
- **Environment:** Jupyter Notebook
- **Data:** Music streaming event logs (CSV)

---

## Related Projects

- [Global Airport Intelligence — PySpark Pipeline](https://github.com/Nikhil2902/global-airport-intelligence)

---

*All code is original. Query results validated against the source dataset.*
