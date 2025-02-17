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

## Question 1. Understanding docker first run

Run docker with the `python:3.12.8` image in an interactive mode, use the entrypoint `bash`.

What's the version of `pip` in the image?

- 24.3.1
- 24.2.1
- 23.3.1
- 23.2.1

```
(base) hubert@Huberts-MacBook-Pro 3_homework % docker run -it test
root@ae3a902396ef:/app# pip --version
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
root@ae3a902396ef:/app#

24.3.1
```

## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
      POSTGRES_DB: "ny_taxi"
    ports:
      - "5433:5432"
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

- postgres:5433
- localhost:5432
- db:5433
- postgres:5432
- db:5432

If there are more than one answers, select only one of them

```
db:5432
```

## Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from October 2019:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Download this data and put it into Postgres.

You can use the code from the course. It's up to you whether
you want to use Jupyter or a python script.

## Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:

1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles

Answers:

- 104,802; 197,670; 110,612; 27,831; 35,281
- 104,802; 198,924; 109,603; 27,678; 35,189
- 104,793; 201,407; 110,612; 27,831; 35,281
- 104,793; 202,661; 109,603; 27,678; 35,189
- 104,838; 199,013; 109,645; 27,688; 35,202

```
select
	count(1)
from
	green_taxi_trips gtt
where
	gtt."trip_distance" <= 1
	and Cast(gtt."lpep_pickup_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_pickup_datetime" as date) < DATE '2019-11-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) < DATE '2019-11-1'

return 104802

select
	count(1)
from
	green_taxi_trips gtt
where
	gtt."trip_distance" > 1
	and gtt."trip_distance" <= 3
	and Cast(gtt."lpep_pickup_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_pickup_datetime" as date) < DATE '2019-11-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) < DATE '2019-11-1'

return 198924

select
	count(1)
from
	green_taxi_trips gtt
where
	gtt."trip_distance" > 3
	and gtt."trip_distance" <= 7
	and Cast(gtt."lpep_pickup_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_pickup_datetime" as date) < DATE '2019-11-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) < DATE '2019-11-1'

retrun 109603

select
	count(1)
from
	green_taxi_trips gtt
where
	gtt."trip_distance" > 7
	and gtt."trip_distance" <= 10
	and Cast(gtt."lpep_pickup_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_pickup_datetime" as date) < DATE '2019-11-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) < DATE '2019-11-1'

return 27678

select
	count(1)
from
	green_taxi_trips gtt
where
	gtt."trip_distance" > 10
	and Cast(gtt."lpep_pickup_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_pickup_datetime" as date) < DATE '2019-11-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_dropoff_datetime" as date) < DATE '2019-11-1'

return 35189

104802, 198924, 109603, 27678, 35189
```

## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance.

- 2019-10-11
- 2019-10-24
- 2019-10-26
- 2019-10-31

```
select
  Cast(gtt."lpep_pickup_datetime" as date),
  max(trip_distance)
from
  green_taxi_trips gtt
where
  Cast(gtt."lpep_pickup_datetime" as date) in (DATE '2019-10-11', DATE '2019-10-24', DATE '2019-10-26', DATE '2019-10-31')
group by
  Cast(gtt."lpep_pickup_datetime" as date)
order by
  max(trip_distance) desc
limit 1;

return 2019-10-31 515.89
```

## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in
`total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.

- East Harlem North, East Harlem South, Morningside Heights
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park

```
select
  zpu."Zone",
	sum(gtt."total_amount")
from
  green_taxi_trips gtt
	join zones zpu on gtt."PULocationID" = zpu."LocationID"
where
  Cast(gtt."lpep_pickup_datetime" as date) = DATE '2019-10-18'
group by
	zpu."Zone"
having
	sum(gtt."total_amount") > 13000
order by
	sum(gtt."total_amount") desc
limit
	3

return
East Harlem North, 18686.68
East Harlem South, 16797.26
Morningside Heights 13029.79
```

## Question 6. Largest tip

For the passengers picked up in October 2019 in the zone
name "East Harlem North" which was the drop off zone that had
the largest tip?

Note: it's `tip` , not `trip`

We need the name of the zone, not the ID.

- Yorkville West
- JFK Airport
- East Harlem North
- East Harlem South

```
select
  zdo."Zone",
	max(gtt."tip_amount")
from
  green_taxi_trips gtt
	join zones zpu on gtt."PULocationID" = zpu."LocationID"
	join zones zdo on gtt."DOLocationID" = zdo."LocationID"
where
  Cast(gtt."lpep_pickup_datetime" as date) >= DATE '2019-10-1'
	and Cast(gtt."lpep_pickup_datetime" as date) <= DATE '2019-10-31'
	and zpu."Zone" = 'East Harlem North'
group by
	zdo."Zone"
order by
	max(gtt."tip_amount") desc
limit
	1

return JFK Airport, 87.3
```

## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform.
Copy the files from the course repo
[here](../../../01-docker-terraform/1_terraform_gcp/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.

## Question 7. Terraform Workflow

Which of the following sequences, **respectively**, describes the workflow for:

1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform`

Answers:

- terraform import, terraform apply -y, terraform destroy
- teraform init, terraform plan -auto-apply, terraform rm
- terraform init, terraform run -auto-approve, terraform destroy
- terraform init, terraform apply -auto-approve, terraform destroy
- terraform import, terraform apply -y, terraform rm

```
terraform init, terraform apply -auto-approve, terraform destroy
```

## Submitting the solutions

- Form for submitting: https://courses.datatalks.club/de-zoomcamp-2025/homework/hw1

```

```
