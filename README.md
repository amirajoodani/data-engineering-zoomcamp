# data-engineering-zoomcamp
Solution For Course 2026
<b>Module 1 Homework: Docker & SQL </b><br>
Question 1. Understanding Docker images <br>
```bash
docker run -it --entrypoint bash python:3.13
pip --version
```
25.3

Question 2. Understanding Docker networking and docker-compose <br>
Hostname: db <br>
Port: 5432 <br>
db:5432

Question 3. Counting short trips <br>
Question 4. Longest trip for each day <br>
Question 5. Biggest pickup zone <br>
Question 6. Largest tip <br>
solution for 3,4,5,6 : <br>
```python
import pandas as pd

# Load data
trips = pd.read_parquet(
    "https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2025-11.parquet"
)

zones = pd.read_csv(
    "https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv"
)

# Ensure datetime
trips["lpep_pickup_datetime"] = pd.to_datetime(trips["lpep_pickup_datetime"])

# Filter November 2025
nov_trips = trips[
    (trips["lpep_pickup_datetime"] >= "2025-11-01") &
    (trips["lpep_pickup_datetime"] < "2025-12-01")
]

# -----------------------------
# Question 3: Short trips
# -----------------------------
short_trips_count = (nov_trips["trip_distance"] <= 1).sum()
print("Q3:", short_trips_count)

# -----------------------------
# Question 4: Longest trip day
# -----------------------------
valid_trips = nov_trips[nov_trips["trip_distance"] < 100]
valid_trips["pickup_day"] = valid_trips["lpep_pickup_datetime"].dt.date

longest_trip_day = (
    valid_trips.loc[
        valid_trips.groupby("pickup_day")["trip_distance"].idxmax()
    ]
    .sort_values("trip_distance", ascending=False)
    .iloc[0]["pickup_day"]
)

print("Q4:", longest_trip_day)

# -----------------------------
# Question 5: Biggest pickup zone on Nov 18
# -----------------------------
nov_18 = nov_trips[
    nov_trips["lpep_pickup_datetime"].dt.date == pd.to_datetime("2025-11-18").date()
]

pickup_zone_totals = (
    nov_18.groupby("PULocationID")["total_amount"]
    .sum()
    .reset_index()
)

pickup_zone_totals = pickup_zone_totals.merge(
    zones, left_on="PULocationID", right_on="LocationID"
)

biggest_pickup_zone = pickup_zone_totals.sort_values(
    "total_amount", ascending=False
).iloc[0]["Zone"]

print("Q5:", biggest_pickup_zone)

# -----------------------------
# Question 6: Largest tip for East Harlem North pickups
# -----------------------------
ehn_id = zones[zones["Zone"] == "East Harlem North"]["LocationID"].iloc[0]

ehn_trips = nov_trips[nov_trips["PULocationID"] == ehn_id]

tip_by_dropoff = (
    ehn_trips.groupby("DOLocationID")["tip_amount"]
    .sum()
    .reset_index()
)

tip_by_dropoff = tip_by_dropoff.merge(
    zones, left_on="DOLocationID", right_on="LocationID"
)

largest_tip_zone = tip_by_dropoff.sort_values(
    "tip_amount", ascending=False
).iloc[0]["Zone"]

print("Q6:", largest_tip_zone)
```
Question 7. Terraform Workflow <br>
```
Downloading provider plugins & setting up backend
→ terraform init

Generating proposed changes and auto-executing the plan
→ terraform apply -auto-approve
(apply generates the plan and executes it; -auto-approve skips confirmation)

Remove all resources managed by Terraform
→ terraform destroy
```
