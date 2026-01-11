Лекция композ:

https://docs.yandex.ru/docs/view?url=ya-disk-public%3A%2F%2FNg6MvFe%2BW5j82svZBBw09ounATBi2KpKrI0L3TMIXSALrTOdzVd8PiRqBQF5W80Vq%2FJ6bpmRyOJonT3VoXnDag%3D%3D%3A%2F%D0%9B%D0%B5%D0%BA%D1%86%D0%B8%D0%B8%2F05-%20Docker%202-3%20(net%2Bcompose).pdf&name=05-%20Docker%202-3%20(net%2Bcompose).pdf&nosw=1 

Лаба композ:

https://docs.yandex.ru/docs/view?url=ya-disk-public%3A%2F%2FNg6MvFe%2BW5j82svZBBw09ounATBi2KpKrI0L3TMIXSALrTOdzVd8PiRqBQF5W80Vq%2FJ6bpmRyOJonT3VoXnDag%3D%3D%3A%2F%D0%9F%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D0%BA%D0%B8%2F08%20-%20Docker_5.docx&name=08%20-%20Docker_5.docx

Вся теория и лабы:

https://vk.cc/cPCso8

 
##ОСНОВНЫЕ КОМАНДЫ
```bash
docker compose build	#Собрать образ по докер файлу
docker compose up          # запуск
docker compose up -d       # в фоне
docker compose down        # остановка
docker compose ps          # список контейнеров
docker compose logs        # логи
docker compose logs -f     # логи в реальном времени
docker exec -it containername bash(sh у альпина
docker container stop name/id
docker container start name/id
docker container rm --force name
docker network ls
docker image ls
docker volume ls
dolume volume create vol-1 = создание вольюма
dolume volume inspect vol-1 = просмотр характеристик вольюма
```

##ШАБЛОН
```yaml
version: "3.9"   # версия синтаксиса (часто можно опустить)
services:        # контейнеры
  service1:
    image: ubuntu:22.04
    build: (вместо image)
      context: .
      dockerfile: Dockerfile
    extends:
      file: example.yml
      service: servicename
    command: sleep infinity
    command: bash -c "apt update && apt install -y iputils-ping curl && sleep infinity" (если нужен набор команд)
    container_name:
    ports:
      - “8080:80”
    secrets:
      - db_password
    environment:
      - POSTGRES_DB=app_db
      - POSTGRES_USER=app_user
      - POSTGRES_PASSWORD=secret
    environment: (или)
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/app_db
      SPRING_DATASOURCE_USERNAME: app_user
      SPRING_DATASOURCE_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    depends_on:
      - service2
      service2: (если хотим завязать запуск на хелфчеке другого сервиса)
        condition: service_healthy
    networks:
      - front-net
    healthcheck:
      test: ["CMD", "команда", "аргумент"]
      test: ["CMD", "curl", "-f", "http://localhost"]
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 3
  service2:
    ...
volumes:         # тома (если нужны)
  pgdata:
networks:        # сети (если нужны)
  front-net:
secrets:
  db_password:
    file: ./db_password.txt
```
##ШАБЛОНЫ
##UBUNTUx3
```yaml
version: "3.9"

services:
  u1:
    image: ubuntu:22.04
    command: sleep infinity
    volumes:
      - shared:/data

  u2:
    image: ubuntu:22.04
    command: sleep infinity
    volumes:
      - shared:/data

  u3:
    image: ubuntu:22.04
    command: sleep infinity
    volumes:
      - shared:/data

volumes:
  shared:
```

```bash
docker exec -it ubuntu1 bash
apt update
apt install -y iputils-ping
ping ubuntu2
echo "hello" > /data/test.txt
```

##ELK
```yaml
version: "3.7"

services:
  elasticsearch:
    image: elasticsearch:8.4.3
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - xpack.security.enabled=false
      - xpack.security.authc.api_key.enabled=false
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      - "9300:9300"
    healthcheck:
      test: ["CMD-SHELL", "curl --silent --fail localhost:9200/_cluster/health || exit 1"]
      interval: 10s
      timeout: 10s
      retries: 3
    networks:
      - elastic

  logstash:
    image: logstash:8.4.3
    container_name: logstash
    environment:
      discovery.seed_hosts: logstash
      LS_JAVA_OPTS: "-Xms512m -Xmx512m"
    volumes:
      - ./logstash/logstash.conf:/usr/share/logstash/logstash.conf
      - ../../logs/service-logs.log:/var/log/service-logs.log
    command: logstash -f /usr/share/logstash/logstash.conf
    ports:
      - "5000:5000/tcp"
      - "5000:5000/udp"
      - "5044:5044"
      - "9600:9600"
      - "4560:4560"
    depends_on:
      - elasticsearch
    networks:
      - elastic

  kibana:
    image: kibana:8.4.3
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    networks:
      - elastic

networks:
  elastic:
    driver: bridge
```
https://github.com/BlackCoffeeCoding/Journal-Project/tree/main/docker/elastic 

##NGINX+UBUNTU
```yaml
version: "3.9"

services:
  nginx:
    image: nginx:latest
    container_name: web
    ports:
      - "8080:80"

  client:
    image: ubuntu:22.04
    container_name: client
    command: sleep infinity
```

```bash
docker exec -it client bash
apt update && apt install -y curl
curl http://nginx
```

##POSTGRES+PGADMIN+BACK+FRONT
```yaml
version: "3.9"

services:
  db:
    image: postgres:15
    container_name: postgres-db
    environment:
      POSTGRES_DB: app_db
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
  pgadmin:
    image: dpage/pgadmin4:9.8
    ports:
      - 5050:80
    environment:
      PGADMIN_DEFAULT_EMAIL: demo@example.com
      PGADMIN_DEFAULT_PASSWORD: secret
      PGADMIN_CONFIG_SERVER_MODE: 'False'
      PGADMIN_CONFIG_MASTER_PASSWORD_REQUIRED: 'False'
  backend:
    build:
      context: ./backend
    container_name: app-backend
    ports:
      - "8081:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/app_db
      SPRING_DATASOURCE_USERNAME: app_user
      SPRING_DATASOURCE_PASSWORD: secret
    depends_on:
      - db

  frontend:
    image: nginx
    container_name: app-frontend
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  pgdata:
```

1. Переключиться на пользователя postgres  su - postgres
2. Подключиться к PostgreSQL  psql
-- Показать все базы данных \l  или  \list
-- Показать все таблицы в текущей базе данных \dt
-- Показать все схемы \dn
-- Показать все пользователей/роли \du
-- Выйти из psql \q

##PROMETHEUS+GRAFANA
```yaml
version: "3"
services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus
    links:
      - Prometheus

prometheus.yml
scrape_configs:
  - job_name: 'journal-project'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```

##ПРИМЕРS DOCKERFILE
```dockerfile
FROM ubuntu
LABEL maintainer="RUT-USER"
RUN apt-get update && apt-get install -y nginx && rm –rf /var/lib/apt/lists/*
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]
EXPOSE 80
```
```dockerfile
FROM busybox
LABEL description="ENTRYPOINT vs CMD demo" maintainer="RUT-MIIT"
ENTRYPOINT ["ping", "-c", "4"]
CMD ["ya.ru"]
```
serviceone
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /app
EXPOSE 80
COPY *.csproj ./
RUN dotnet restore "serviceone.csproj"
COPY . ./
RUN dotnet publish serviceone.csproj -c Release -o out
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS runtime
WORKDIR /app
COPY --from=build /app/out .
ENTRYPOINT ["dotnet", "serviceone.dll"]
```

servicetwo
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /app
EXPOSE 80
COPY *.csproj ./
RUN dotnet restore "servicetwo.csproj"
COPY . ./
RUN dotnet publish servicetwo.csproj -c Release -o out
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS runtime
WORKDIR /app
COPY --from=build /app/out .
ENTRYPOINT ["dotnet", "servicetwo.dll"]
```

FROM — задаёт базовый (родительский) образ.
LABEL — описывает метаданные. Например — сведения о том, кто создал и поддерживает образ.
ENV — устанавливает постоянные переменные среды.
RUN — выполняет команду и создаёт слой образа. Используется для установки в контейнер пакетов.
COPY — копирует в контейнер файлы и папки.
ADD — копирует файлы и папки в контейнер, может распаковывать локальные .tar-файлы.
CMD — описывает команду с аргументами, которую нужно выполнить когда контейнер будет запущен. Аргументы могут быть переопределены при запуске контейнера. В файле может присутствовать лишь одна инструкция CMD.
WORKDIR — задаёт рабочую директорию для следующей инструкции.
ARG — задаёт переменные для передачи Docker во время сборки образа.
ENTRYPOINT — предоставляет команду с аргументами для вызова во время выполнения контейнера. Аргументы не переопределяются.
EXPOSE — указывает на необходимость открыть порт.
VOLUME — создаёт точку монтирования для работы с постоянным хранилищем.


