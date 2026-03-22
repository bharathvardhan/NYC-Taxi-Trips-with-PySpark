# NYC Taxi Trips with PySpark

This folder contains **Q1** of HW3: work in **`q1.ipynb`** to analyze NYC taxi-style trip data using **PySpark DataFrames** (Docker environment recommended).

## Technology

- **PySpark** (DataFrame API only for your solution code)
- **Docker** (course image with Jupyter + Java + Spark)

## Dataset

- **`yellow_tripdata_2019-01_short.csv`** — modified NYC Green Taxi–style records (pickup/dropoff times, locations, distances, fares, tips, passengers, etc.).


## Docker setup (recommended)

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine).
2. Run Docker from the directory that should appear as `/root` in the container (so Jupyter sees **`q1.ipynb`** and the CSV):

   - **This repo (notebook at repo root):** `cd` into the clone, then use `-v "${PWD}:/root"`.
   - **Course skeleton layout** (a `Q1/` subfolder next to other HW folders): from the **parent** of `Q1/`, use `-v "${PWD}/Q1:/root"`.

```bash
docker container run -d \
  -v "${PWD}:/root" \
  -p 127.0.0.1:6242:8888 \
  --name hw3 \
  polodataclub/cse6242hw3-public:Q1
```

3. Open Jupyter in a browser: **http://localhost:6242**

4. When finished, stop the container:

```bash
docker stop hw3
```

Optional — remove the container:

```bash
docker rm hw3
```

**Apple Silicon (M1/M2/M3):** You may see a platform warning (`linux/amd64` vs `arm64`). The image can still run via emulation; performance may be slower.

## Deliverable

- Submit **`q1.ipynb`** to **Gradescope** (as specified on Canvas).

## Important rules (autograder / grading)

| Rule | Detail |
|------|--------|
| **API** | Use **Spark DataFrame operations** only in your functions. |
| **`sqlContext` in *your* functions** | **Do not** reference `sqlContext` inside the functions you implement. |
| **No raw SQL** | **Do not** use `sqlContext.sql("SELECT ...")` (or equivalent) for the homework logic. |
| **`#export`** | **Do not** remove `#export` comments — the autograder uses them. |
| **Imports** | **Do not** add new cells or extra `import` lines beyond what the skeleton allows. |
| **Debug output** | Remove extra **`print`**, **`display`**, and **`show()`** from graded cells; they can break Gradescope. |
| **Kernel** | If you re-run Spark cells, **restart the kernel** if you see SparkContext / context errors. |
| **Saving work** | If the notebook doesn’t appear in Jupyter, confirm the volume mount path. If you **upload** the notebook in the UI without saving to the mounted folder, use **File → Download as → Notebook (.ipynb)** often so you don’t lose work. |

## Tasks (point breakdown)

### Q1.1 — `clean_data` [4 pts]

Cast columns to the required types:

| Column | Type |
|--------|------|
| `passenger_count` | integer |
| `total_amount` | float |
| `tip_amount` | float |
| `trip_distance` | float |
| `fare_amount` | float |
| `tpep_pickup_datetime` | timestamp |
| `tpep_dropoff_datetime` | timestamp |

### Q1.2 — `common_pair` [6 pts]

- Sum **`passenger_count`** per **(PULocationID, DOLocationID)**.
- Exclude trips where pickup and dropoff are the **same** zone.
- **`per_person_rate`**: average **`total_amount` per passenger** over all trips in that pair (i.e. tied to **`total_amount`** and passenger counts as specified).
- Return **top 10** pairs by **total passengers**; ties broken by **higher `per_person_rate`**.
- Rename the sum column to **`total_passenger_count`**.

### Q1.3 — `distance_with_most_tip` [6 pts]

- Keep trips with **`fare_amount > 2`** and **`trip_distance > 0`**.
- **`tip_percent` = `tip_amount * 100 / fare_amount`** per trip.
- **Ceil** **`trip_distance`** to the next whole mile, then aggregate by that rounded distance.
- Average **`tip_percent`** per rounded distance; sort by **`tip_percent` descending**; take **top 15** rows.
- Columns: **`trip_distance`**, **`tip_percent`**.

### Q1.4 — `time_with_most_traffic` [9 pts]

- **Speed** for trips that **start** in a given hour bucket:  
  **average `trip_distance` ÷ average trip duration (in hours)** — compute **averages first**, then divide (distance per hour).  
  Use **cast to long** on timestamps when computing duration, as suggested in the writeup.
- Split **AM** (hours **0:00–11:59**) vs **PM** (**12:00–23:59**).
- Map to **12-hour “clock positions”** so each row’s **`time_of_day`** lines up with the **1 AM / 1 PM** style example (e.g. one row for “1 o’clock” with **1–2 AM** in **`am_avg_speed`** and **1–2 PM** in **`pm_avg_speed`**).
- Format **`time_of_day`** with **`date_format`** using the pattern that matches the assignment (**sorted 0–11** as in the PDF).
- Rows may be **missing** for hours with no data; **do not** fabricate empty hour rows unless the course says otherwise.
- Columns: **`time_of_day`**, **`am_avg_speed`**, **`pm_avg_speed`**.

## Troubleshooting

| Issue | What to try |
|--------|-------------|
| SparkContext already exists | **Restart Jupyter kernel**, run init cells once. |
| Gradescope “connection refused” / timeout | Simplify code; avoid huge shuffles or Python loops over all rows. |
| Notebook not on disk in Docker | Check **`-v` mount**; download the `.ipynb` from Jupyter if needed. |

## Version

Instructions follow **HW3 Version 0** (see PDF footer). Check Ed / Canvas for updates.
