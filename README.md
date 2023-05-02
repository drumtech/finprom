# finprom

## Задание выполнял на своих VM в разных DC

##### **Архитектура**

*node-host:*
1. docker-compose - поднимает cadvisor, mysql, wordpress, mysqld_exporter, всё в контейнерах потому что 2 из них по заданию докером поднимать остальные два... ну на мой взгляд так проще обновлять и держать в консистентном состоянии
2. node_exporter - установлен на сам хост, без контейниризации, ибо кмк так лучше мониторить хост, как минимум меньше граблей с мониторингом дисковой подсистемы
3. IP - 34.118.87.190

*prom-host*
1. prometheus - развёрнут на хосте без докера, пока выполнял задания в практикуме разворачивал промик и в докере и без, мне на хост машине понравилось больше.
2. IP - 5.189.201.76

##### **prometheus.yml**

```
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 5.189.201.76:9093
rule_files:
  - "alerts.yml"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["5.189.201.76:9090"]
  - job_name: "node_exporters"
    scrape_interval: 30s
    static_configs:
      - targets: ["34.118.87.190:9100"]
  - job_name: "mysqld_exporters"
    scrape_interval: 10s
    static_configs:
      - targets: ["34.118.87.190:9104"]
  - job_name: "cadvisor"
    scrape_interval: 20s
    static_configs:
      - targets: ["34.118.87.190:8080"]

```

##### **docker-compose.yml**

```
version: "3.8"

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: SuPer_Pupper_ROOT_password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: SuPer_Pupper_wordpress_password
  mysql_exporter:
    image: prom/mysqld-exporter
    ports:
      - "9104:9104"
    environment:
      DATA_SOURCE_NAME: "root:SuPer_Pupper_ROOT_password@(db:3306)/"
    depends_on:
      - db
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: SuPer_Pupper_wordpress_password
      WORDPRESS_DB_NAME: wordpress
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
    - db

volumes:
  db_data: {}
  wordpress_data: {}

```


![Лог запуска compose без mysql exporter](/images/log_compose_without_ms_exporter.png)

![Лог запуска mysql exporter](/images/log_with_ms_exporter.png)

![Статус nodeexporter](/images/log_node_exp.png)

![Cadvisor's Dashboard](/images/grafana_cadvisor.png)

![MYSQL's Dashboard](/images/grafana_ms.png)

![Nodeexporter's Dashboard](/images/grafana_nose_exp.png)
