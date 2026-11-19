FROM python:3.14.8-slim AS builder

WORKDIR /install

COPY requirements.txt requirements-prod.txt ./
RUN pip install --no-cache-dir --prefix=/install -r requirements-prod.txt

FROM python:3.14.8-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY --from=builder /install /usr/local

COPY . .

RUN addgroup --system app && adduser --system --ingroup app app \
    && chmod +x /app/entrypoint.sh \
    && chown -R app:app /app

USER app

EXPOSE 8000

ENTRYPOINT ["/app/entrypoint.sh"]