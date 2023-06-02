FROM python:3.9
LABEL description="Kittygram backend" \
      version="1.0" \
      maintainer="Yandex Practicum" \
      deployer="https://github.com/Altair21817"
RUN pip install --upgrade pip
RUN pip install gunicorn==20.1.0
WORKDIR /kittygram
COPY . .
RUN pip install -r requirements.txt
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "kittygram_backend.wsgi"]
