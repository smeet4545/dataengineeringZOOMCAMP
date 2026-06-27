# Why the container was not working earlier

The ingestion image failed because the container entrypoint was starting the Python interpreter without the required runtime packages available in that environment.

## Root cause
- The script in ingest_data.py imports packages such as click, pandas, sqlalchemy, and tqdm.
- The Docker image was not reliably installing those dependencies into the interpreter that actually launched the script.
- An earlier virtual environment setup inside the image also produced a broken Python symlink, which caused the container to fail before the script could start.

## Why it worked after the fix
- The Dockerfile now installs the required packages into the active Python environment used by the container.
- The image entrypoint now launches the correct interpreter, so the script can start normally.

## Practical lesson
When a Python script is run inside Docker, the container must have the dependencies installed in the same interpreter that runs the script. If the entrypoint points to a different Python executable or a broken virtualenv, the script will fail even if the code itself is correct.

## Commands to reproduce and run

Build the image:
```bash
cd test/pipeline
docker build -t taxi_ingest:v001 .
```

Run the ingestion container:
```bash
docker run --rm --network=pg-network taxi_ingest:v001 \
  --pg-user=root \
  --pg-pass=root \
  --pg-host=pgdatabase \
  --pg-port=5432 \
  --pg-db=ny_taxi \
  --target-table=yellow_taxi_trips_2021_1 \
  --chunksize=100000
```

If you want to check the script entrypoint manually:
```bash
docker run --rm taxi_ingest:v001 --help
```
