# telegraf-influxdb-grafana
Репозиторий с примером деплоя системы мониторинга сервера
В docker-compose в дальнейшем вместо образа `osll/telegraf-cuda` следует использовать `mconcas/telegraf-nvidia:latest` либо `telegraf:1.20.4`

## Описание
Подробности гуглятся по запросам `Telegraf + InfluxDB + Grafana`. В двух словах
- Telegraf -- Занимается сбором различных параметров системы. Использование ЦП, памяти, ...
- InfluxDB -- База данных
- Grafana -- Средство визуализации данных. Альтернатива стандартным средствам InfluxDB и без доступа на запись к самой БД.


## Развертывание
### Выбор паролей и ключей
1. Выбрать/сгенерировать пароль и токен для `influxdb`. Указать его в `docker-compose.yaml` для сервиса `influxdb` в переменных `DOCKER_INFLUXDB_INIT_PASSWORD` и `INFLUX_TOKEN` соответственно. Они будут использоваться для доступа в БД через веб интерфейс и API.
1. Их же указать в `grafana/provisioning/datasources/datasource.yml` в параметрах `password` и `token`. 
1. Указать токен в параметре `token` блока `outputs.influxdb_v2` в файле `telegraf.conf`
1. Выбрать/сгенерировать пароль для Grafana. Указать его в `docker-compose.yaml` для сервиса `grafana` в переменной `GF_SECURITY_ADMIN_PASSWORD`. Он будет использовать для доступа к веб-интерфейсу.

### Запуск
Запуск выполняется следущей командой:  
`DOCKER_GROUP=$(stat -c '%g' /var/run/docker.sock) docker-compose up -d`  
Требуется для того, чтобы в контейнере ID группы пользователя совпадал с ID группы docker на хост машине.

## Интерфейсы
- Порт `8086` -- `InfluxDB`
- Порт `3000` -- `Grafana`

## Важные моменты конфиг-файлов
### docker-compose.yaml
- В случае использования в сети с прокси, для telegraf можно указать в переменных окружения прокси по примеру: `http_proxy=http://10.128.0.90:8080`.
- Для сервиса `telegraf` можно использовать стандартный образ `telegraf`, если не требуется мониторинг ГПУ.
- Аналогично, без мониторинга ГПУ строку `/usr/bin/nvidia-smi:/usr/bin/nvidia-smi:ro` можно закомментировать
- `DOCKER_INFLUXDB_INIT_RETENTION=1w` -- определяет срок хранения записей в БД. `1w` -- одна неделя.
### grafana/provisioning/datasources/datasource.yml
В основном содержит данные для авторизации в БД
### telegraf.conf
- Описывает что трекать. Поддерживает много плагинов. 
- `inputs.docker` позволяет отслеживать использование ресурсов по контейнерам
- Содержит данные для авторизации в БД
### Nvidia GPU
Telegraf умеет в помощью плагина собирать данные и с ГПУ Nvidia. Если на сервере ГПУ нет или нет необходимости их отслеживать, требуется закомментировать в `telegraf.conf` строчку `[[inputs.nvidia_smi]]`  
А в `docker-compose.yaml` строчки `runtime: nvidia` и `NVIDIA_VISIBLE_DEVICES: all`.


