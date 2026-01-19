# Module 1 Homework: Docker & SQL

In this homework we'll prepare the environment and practice
Docker and SQL

When submitting your homework, you will also need to include
a link to your GitHub repository or other public code-hosting
site.

This repository should contain the code for solving the homework.

When your solution has SQL or shell commands and not code
(e.g. python files) file format, include them directly in
the README file of your repository.


## Question 1. Understanding Docker images

Run docker with the `python:3.13` image. Use an entrypoint `bash` to interact with the container.

What's the version of `pip` in the image?

```bash
micha@PCtour:~/learn/my-zoomcamp-2026/cohorts/2026/01-docker-terraform/pipeline$ docker run -it --rm python:3.13 bash
root@2c6281e29111:/# pip --version
pip 25.3 from /usr/local/lib/python3.13/site-packages/pip (python 3.13)
```

- 25.3


## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that pgadmin should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

```Text
Two solutions are correct:
- db:5432 (prefered one for me)
- postgres:5432

mapping: HOST_PORT:CONTAINER_PORT
postgres and pgadmin are container orchestrated by the same docker compose so they can communicate via their internal container ports (5432 for postgres)
db : service name
postgres : container name
Both resolves well internaly
```



## Prepare the Data

Download the green taxi trips data for November 2025:

```bash
wget https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2025-11.parquet
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

I used a notebook to ingest green_taxi_data (parquet) and taxi_zones (csv)

## Question 3. Counting short trips

For the trips in November 2025 (lpep_pickup_datetime between '2025-11-01' and '2025-12-01', exclusive of the upper bound), how many trips had a `trip_distance` of less than or equal to 1 mile?

```SQL
SELECT COUNT(*)
 FROM green_taxi_data 
 WHERE (lpep_pickup_datetime between '2025-11-01' and '2025-12-01' )
 AND trip_distance <= 1
 ORDER BY lpep_pickup_datetime ASC;
```

- 8,007



## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance? Only consider trips with `trip_distance` less than 100 miles (to exclude data errors).

Use the pick up time for your calculations.

```SQL
SELECT 
	lpep_pickup_datetime, 
	MAX(trip_distance) max_dist
FROM green_taxi_data 
WHERE trip_distance <= 100
GROUP BY lpep_pickup_datetime
ORDER BY max_dist DESC
LIMIT 1;
```

- 2025-11-14


## Question 5. Biggest pickup zone

Which was the pickup zone with the largest `total_amount` (sum of all trips) on November 18th, 2025?


```SQL
SELECT 
	z."Zone",
	COUNT(*) sum_trips
FROM green_taxi_data g
LEFT JOIN zones z
ON g."PULocationID" = z."LocationID"
GROUP BY z."Zone"
ORDER BY sum_trips DESC
LIMIT 1;
```
- East Harlem North


## Question 6. Largest tip

For the passengers picked up in the zone named "East Harlem North" in November 2025, which was the drop off zone that had the largest tip?

Note: it's `tip` , not `trip`. We need the name of the zone, not the ID.

```SQL
SELECT 
	zDO."Zone",
  MAX(g.tip_amount) max_tip
FROM green_taxi_data g
LEFT JOIN zones zpu ON g."PULocationID" = zpu."LocationID"
LEFT JOIN zones zdo ON g."DOLocationID" = zdo."LocationID"
WHERE zPU."Zone" = 'East Harlem North'
GROUP BY zDO."Zone"
ORDER BY max_tip DESC
LIMIT 1;
```

- Yorkville West


## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform.
Copy the files from the course repo
[here](../../../01-docker/terraform/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.


## Question 7. Terraform Workflow

Which of the following sequences, respectively, describes the workflow for:
1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform`

Answer:
- terraform init, terraform apply -auto-approve, terraform destroy


## Submitting the solutions

* Form for submitting: https://courses.datatalks.club/de-zoomcamp-2026/homework/hw1


## Learning in Public

We encourage everyone to share what they learned. This is called "learning in public".

### Why learn in public?

- Accountability: Sharing your progress creates commitment and motivation to continue
- Feedback: The community can provide valuable suggestions and corrections
- Networking: You'll connect with like-minded people and potential collaborators
- Documentation: Your posts become a learning journal you can reference later
- Opportunities: Employers and clients often discover talent through public learning

You can read more about the benefits [here](https://alexeyondata.substack.com/p/benefits-of-learning-in-public-and).

Don't worry about being perfect. Everyone starts somewhere, and people love following genuine learning journeys!

### Example post for LinkedIn

```
🚀 Week 1 of Data Engineering Zoomcamp by @DataTalksClub complete!

Just finished Module 1 - Docker & Terraform. Learned how to:

✅ Containerize applications with Docker and Docker Compose
✅ Set up PostgreSQL databases and write SQL queries
✅ Build data pipelines to ingest NYC taxi data
✅ Provision cloud infrastructure with Terraform

Here's my homework solution: <LINK>

Following along with this amazing free course - who else is learning data engineering?

You can sign up here: https://github.com/DataTalksClub/data-engineering-zoomcamp/
```

### Example post for Twitter/X


```
🐳 Module 1 of Data Engineering Zoomcamp done!

- Docker containers
- Postgres & SQL
- Terraform & GCP
- NYC taxi data pipeline

My solution: https://github.com/MichaelG-create/01-docker-terraform

Free course by @DataTalksClub: https://github.com/DataTalksClub/data-engineering-zoomcamp/
#dezoomcamp
```


