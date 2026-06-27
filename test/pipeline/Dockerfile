FROM python:3.13.11-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin

WORKDIR /code

COPY pyproject.toml .python-version uv.lock ./
RUN python -m pip install --no-cache-dir --upgrade pip \
    && python -m pip install --no-cache-dir click pandas psycopg2-binary pyarrow sqlalchemy tqdm

COPY . .

ENTRYPOINT ["python", "/code/ingest_data.py"]
