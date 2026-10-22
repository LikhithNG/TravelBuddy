# TravelBuddy Flight Service

A small FastAPI service that collects flight fares and loads them into Snowflake for the main backend to query.

## What It Does

1. Takes a route and date (`source`, `destination`, `date`).
2. Fetches one-way fares from the Skyscanner API through RapidAPI.
3. Uploads two JSON files to S3: `Flightdata/{source}_to_{destination}_{date}_daily.json` and `Flightdata/{source}_to_{destination}_{month}_monthly.json`.
4. Triggers the Airflow DAG `load_to_snowflake`, which loads the files into Snowflake.

The main backend then reads the fares from Snowflake through `/api/flights/search` and `/api/flights/analysis`.

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/flights/` | Run the steps above for the route in the request body |
| GET | `/flights/search?origin_id=BOS&destination_id=MIA&departure_date=2026-12-01` | Same, with query parameters |
| POST | `/flights/test` | Echo the request body to check parsing |
| GET | `/` | Health check |

Interactive docs are at `/docs` while the service is running.

## Settings

Create `flight_service/.env`:

```
RAPID_API_KEY=your-rapidapi-key
AWS_ACCESS_KEY=your-aws-access-key
AWS_SECRET_KEY=your-aws-secret-key
S3_BUCKET_NAME=your-bucket
AIRFLOW_API_URL=http://your-airflow-host:8080/api/v1/dags/load_to_snowflake/dagRuns
AIRFLOW_USERNAME=your-airflow-user
AIRFLOW_PASSWORD=your-airflow-password
```

Set all of these. The defaults in `app/config/settings.py` point at the original team's S3 bucket and Airflow server.

## Run

```bash
cd flight_service
pip install -r requirements.txt
uvicorn app.main:app --port 8001
```

Or with Docker (the container listens on port 8000):

```bash
docker build -t travelbuddy-flights .
docker run -p 8001:8000 --env-file .env travelbuddy-flights
```

## Layout

```
app/
├── main.py                   App setup and health check
├── api/routes/               /flights endpoints
├── controllers/              Fetch → S3 → Airflow flow
├── services/                 Skyscanner, S3 and Airflow clients
├── models/schemas.py         Request and response models
└── config/settings.py        Settings from environment variables
```
