FROM python:3.8.12-slim

COPY requirements.txt .
# COPY ["*.py", "project_one_model.pkl", "./"]

RUN pip install -r requirements.txt

WORKDIR /app

COPY ["predict.py", "project_one_model.pkl", "./"]

EXPOSE 9696

ENTRYPOINT ["gunicorn", "--bind=0.0.0.0:9696", "predict:app"]