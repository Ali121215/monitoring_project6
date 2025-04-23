# monitoring_project6
Практическая работа №6
Мониторинг и логирование в DevOps
📌 Цель работы:
Настроить мониторинг и логирование работы контейнеризированного приложения с использованием Prometheus, Grafana и Loki.
📌 Задачи:
1. Настроить логирование в Docker-контейнере с помощью Loki.
2. Настроить Prometheus для сбора метрик.
3. Подключить Grafana для визуализации логов и метрик.
4. (Дополнительно) Настроить уведомления в Telegram или Slack при ошибках.
1. Подготовка проекта
🔹 Создание каталога проекта:
$ mkdir monitoring_project && cd monitoring_project
🔹 Создание файлов проекта:
$ touch app.py requirements.txt Dockerfile
🔹 Код `app.py`:
from flask import Flask
import logging

app = Flask(__name__)

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.route('/')
def home():
    logger.info('Received request on home page')
    return 'Hello, Monitoring!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
🔹 Файл зависимостей `requirements.txt`:
flask
2. Создание Dockerfile
🔹 Код `Dockerfile`:
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt requirements.txt
COPY app.py app.py
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000
CMD ['python', 'app.py']
🔹 Сборка и запуск контейнера:
$ docker build -t flask-monitoring .
$ docker run -p 5000:5000 flask-monitoring
3. Настройка мониторинга с Prometheus
🔹 Файл `prometheus.yml`:
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'flask-app'
    static_configs:
      - targets: ['flask-app:5000']
4. Настройка `docker-compose.yml`
🔹 Добавление Prometheus, Grafana и Loki:
version: '3.8'
services:
  flask-app:
    build: .
    ports:
      - '5000:5000'
    restart: always

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - '9090:9090'
    restart: always

  grafana:
    image: grafana/grafana
    ports:
      - '3000:3000'
    restart: always

  loki:
    image: grafana/loki:latest
    ports:
      - '3100:3100'
    restart: always
🔹 Запуск контейнеров:
$ docker-compose up -d
5. Настройка логирования с Loki
🔹 Файл `loki-config.yml`:
auth_enabled: false
server:
  http_listen_port: 3100

ingester:
  lifecycler:
    address: 127.0.0.1
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
6. Настройка Grafana
🔹 Открываем Grafana: http://localhost:3000
🔹 Добавляем источники данных:
- Prometheus (URL: `http://prometheus:9090`)
- Loki (URL: `http://loki:3100`)
7. Уведомления при ошибках в Telegram/Slack (Дополнительно)
🔹 Настройка Telegram Alerting:
1. Получаем Bot API Token через `@BotFather`.
2. Получаем Chat ID через `@userinfobot`.
3. Добавляем в Grafana Alerting.
8. Ожидаемый результат
✅ Запущено контейнеризированное приложение с логированием и мониторингом
✅ Prometheus собирает метрики, Grafana визуализирует данные
✅ Loki хранит логи приложения
✅ (Дополнительно) Уведомления отправляются в Telegram/Slack
9. Формат сдачи работы
1. Ссылка на GitHub-репозиторий с `Dockerfile`, `docker-compose.yml`, `prometheus.yml`, `loki-config.yml`.
2. Скриншоты работающих панелей в Grafana (метрики, логи).
3. (Дополнительно) Скриншот уведомлений в Telegram/Slack.
