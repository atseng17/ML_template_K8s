FROM python:3.8.12-slim

COPY requirements.txt .

RUN pip install -r requirements.txt

WORKDIR /app

EXPOSE 9696

ENTRYPOINT ["gunicorn", "--bind=0.0.0.0:9696", "predict:app"]