### Задание 1

**Напишите ответ в свободной форме, не больше одного абзаца текста.**

Установите Docker Compose и опишите, для чего он нужен и как может улучшить лично вашу жизнь.

### Решение

Docker-compose устанавливался на операционную систему `Astra Linux` с помощью команды `apt install docker-compose`

> ![img](img/img_1.png)

Docker-compose позволяет запускать множество контейнеров за раз с помощью файла конфигурации `yaml`, в котором можно указать различные настройки для запускаемых контейнеров и таким образом сразу развернуть работающий проект. Альтернативой ему является docker run, который может запускать за раз только один контейнер и работает через командную строку, либо через `Dockerfile`, который только собирает образ.

**Docker-compose облегчает жизнь в принципе любым работникам IT-инфраструктуры, позволяя за короткое время создать и развернуть полноценный проект из множества различных программных решений включа базы данных, системы мониторинга серверов и т.д.**

---

### Задание 2 

**Выполните действия и приложите текст конфига на этом этапе.** 

Создайте файл docker-compose.yml и внесите туда первичные настройки: 

 * version;
 * services;
 * volumes;
 * networks.

При выполнении задания используйте подсеть 10.5.0.0/16.
Ваша подсеть должна называться: <ваши фамилия и инициалы>-my-netology-hw.
Все приложения из последующих заданий должны находиться в этой конфигурации.

### Решение

**<ins>Далее в работе будет использоваться файл</ins> `compose.yaml` <ins> так как он более предпочтителен для Docker-compose</ins>**

Создаю файл `compose.yaml` в директории пользователя и прописываю конфиг:

```
version: '3'

services:

volumes:

networks:
  smirnov_vv-my-netology-hw:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1
```

---

### Задание 3 

**Выполните действия:** 

1. Создайте конфигурацию docker-compose для Prometheus с именем контейнера <ваши фамилия и инициалы>-netology-prometheus. 
2. Добавьте необходимые тома с данными и конфигурацией (конфигурация лежит в репозитории в директории [6-04/prometheus](https://github.com/netology-code/sdvps-homeworks/tree/main/lecture_demos/6-04/prometheus) ).
3. Обеспечьте внешний доступ к порту 9090 c докер-сервера.

## Решение

Редактирую созданный ранее файл `compose.yaml` и добавляю в него следующие строки:

```
version: '3'

services:
  prometheus:
    image: prom/prometheus:v2.47.2
    container_name: smirnov_vv-netology-prometheus
    command: --web.enable-lifecycle  --config.file=/etc/prometheus/prometheus.yaml
    ports:
      - 9090:9090
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    networks:
      - smirnov_vv-my-netology-hw
    restart: always

volumes:
  prometheus-data:

networks:
  smirnov_vv-my-netology-hw:
    driver: bridge
    ipam:
      config:
      - subnet: 10.5.0.0/16
        gateway: 10.5.0.1
```
В директории пользователя в папке `./prometheus` создаю файл `prometheus.yaml` и конфигурирую его следующим образом:

```
# my global config
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
           # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
```


---

### Задание 4 

**Выполните действия:**

1. Создайте конфигурацию docker-compose для Pushgateway с именем контейнера <ваши фамилия и инициалы>-netology-pushgateway. 
2. Обеспечьте внешний доступ к порту 9091 c докер-сервера.

## Решение

Редактирую файл `compose.yaml`:

```
version: '3'

services:
  prometheus:
    image: prom/prometheus:v2.47.2
    container_name: smirnov_vv-netology-prometheus
    command: --web.enable-lifecycle  --config.file=/etc/prometheus/prometheus.yaml
    ports:
      - 9090:9090
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    networks:
      - smirnov_vv-my-netology-hw
    restart: always

  pushgateway:
    image: prom/pushgateway:v1.6.2
    container_name: smirnov_vv-netology-pushgateway
    ports:
      - 9091:9091
    networks:
      - smirnov_vv-my-netology-hw
    depends_on:
      - prometheus
    restart: unless-stopped
```

Добавляю в конфиг `prometheus.yaml` следющие строки:

```
# my global config
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
           # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: 'pushgateway'
    honor_labels: true
    static_configs:
      - targets: ['192.168.53.131:9091']
```
---

### Задание 5 

**Выполните действия:** 

1. Создайте конфигурацию docker-compose для Grafana с именем контейнера <ваши фамилия и инициалы>-netology-grafana. 
2. Добавьте необходимые тома с данными и конфигурацией (конфигурация лежит в репозитории в директории [6-04/grafana](https://github.com/netology-code/sdvps-homeworks/blob/main/lecture_demos/6-04/grafana/custom.ini).
3. Добавьте переменную окружения с путем до файла с кастомными настройками (должен быть в томе), в самом файле пропишите логин=<ваши фамилия и инициалы> пароль=netology.
4. Обеспечьте внешний доступ к порту 3000 c порта 80 докер-сервера.

## Решение

В файл `compose.yaml` добавляю строки с grafana:

```
grafana:
    image: grafana/grafana
    container_name: smirnov_vv-netology-grafana
    environment:
      GF_PATHS_CONFIG: /etc/grafana/grafana.ini
    ports:
      - 80:3000
    volumes:
      - ./grafana:/etc/grafana
      - grafana-data:/var/lib/grafana
    networks:
      - smirnov_vv-my-netology-hw
    depends_on:
      - prometheus
    restart: unless-stopped
```
Файл конфигруации `grafana.ini` размещаю в папке `./grafana` и вставляю следующие строки:

```
[security]

admin_user = smirnov_vv
admin_password = netology
```

---

### Задание 6 

**Выполните действия.**

1. Настройте поочередность запуска контейнеров.
2. Настройте режимы перезапуска для контейнеров.
3. Настройте использование контейнерами одной сети.
4. Запустите сценарий в detached режиме.

## Решение

1. Поочередность запуска контейнеров настроена последовательно с помощью директивы `depends_on: - prometheus`, которая гарантирует, что первым будет запущен `prometheus`, а следом за ним остальные.
2. Режим перезапусков контейнеров настроен следующим образом: \
   2.1. Prometheus - `always` \
   2.2. Pushgateway - `unless-stopped` \
   2.3. Node-exporter - `unless-stopped` \
   2.4. Grafana - `unless-stopped` 
3. Используется подсеть `smirnov_vv-my-netology-hw` определённая в конце файла конфигурации
4. Сценарий запуска контейнеров в detached режиме осуществляется через команду \
`docker-compose up -d`

---

### Задание 7 

**Выполните действия.**
1. Выполните запрос в Pushgateway для помещения метрики <ваши фамилия и инициалы> со значением 5 в Prometheus: ```echo "<ваши фамилия и инициалы> 5" | curl --data-binary @- http://localhost:9091/metrics/job/netology```.
2. Залогиньтесь в Grafana с помощью логина и пароля из предыдущего задания.
3. Cоздайте Data Source Prometheus (Home -> Connections -> Data sources -> Add data source -> Prometheus -> указать "Prometheus server URL = http://prometheus:9090" -> Save & Test).
4. Создайте график на основе добавленной в пункте 5 метрики (Build a dashboard -> Add visualization -> Prometheus -> Select metric -> Metric explorer -> <ваши фамилия и инициалы -> Apply.

В качестве решения приложите:

* docker-compose.yml **целиком**;
* скриншот команды docker ps после запуске docker-compose.yml;
* скриншот графика, постоенного на основе вашей метрики.

## Решение 

* `compose.yaml` целиком:

```

version: '3'

services:

  prometheus:
    image: prom/prometheus:v2.47.2
    container_name: smirnov_vv-netology-prometheus
    command: --web.enable-lifecycle  --config.file=/etc/prometheus/prometheus.yaml
    ports:
      - 9090:9090
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    networks:
      - smirnov_vv-my-netology-hw
    restart: always

  pushgateway:
    image: prom/pushgateway:v1.6.2
    container_name: smirnov_vv-netology-pushgateway
    ports:
      - 9091:9091
    networks:
      - smirnov_vv-my-netology-hw
    depends_on:
      - prometheus
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter
    container_name: smirnov_vv-netology-node_exporter
    ports:
      - 9100:9100
    networks:
      - smirnov_vv-my-netology-hw
    depends_on:
      - prometheus
    restart: unless-stopped

  grafana:
    image: grafana/grafana
    container_name: smirnov_vv-netology-grafana
    environment:
      GF_PATHS_CONFIG: /etc/grafana/grafana.ini
    ports:
      - 80:3000
    volumes:
      - ./grafana:/etc/grafana
      - grafana-data:/var/lib/grafana
    networks:
      - smirnov_vv-my-netology-hw
    depends_on:
      - prometheus
    restart: unless-stopped

volumes:
  prometheus-data:
  grafana-data:

networks:
  smirnov_vv-my-netology-hw:
    driver: bridge
    ipam:
      config:
      - subnet: 10.5.0.0/16
        gateway: 10.5.0.1
```

* 



---

### Задание 8

**Выполните действия:** 

1. Остановите и удалите все контейнеры одной командой.

В качестве решения приложите скриншот консоли с проделанными действиями.

## Решение

Для остановки и удаления всех `docker-compose` контейнеров используется команда `docker-compose down`, которая останавливает и удаляет все контейнеры, включая сети и подключения, описанные в `compose.yaml`.

---

## Дополнительные задания* (со звёздочкой)

Их выполнение необязательное и не влияет на получение зачёта по домашнему заданию. Можете их решить, если хотите лучше разобраться в материале.

---

### Задание 9* 

**Выполните действия:** 

1. Создайте конфигурацию docker-compose для Alertmanager с именем контейнера <ваши фамилия и инициалы>-netology-alertmanager. 
2. Добавьте необходимые тома с данными и [конфигурацией](https://github.com/netology-code/sdvps-homeworks/tree/main/6-04/alertmanager), сеть, режим и очередность запуска.
3. Обновите конфигурацию Prometheus (необходимые изменения ищите в презентации или документации) и перезапустите его. 
4. Обеспечьте внешний доступ к порту 9093 c докер-сервера.

В качестве решения приложите скриншот с событием из Alertmanager.

---

### Задание 10* 

Запустите свой сценарий на чистом железе без предзагруженных образов.

**Ответьте на вопросы в свободной форме:**

1. Опишите выполненный вами процесс развертывания сценария.
2. Как вы думаете зачем может понадобиться такой способ развертывания?
