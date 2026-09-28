# Anonymizer Deployment Guide

## Предварительные требования

### 1. Готовый кластер Kubernetes

Включая:

- **Prometheus-like monitoring system** с CRD `monitoring.coreos.com/v1`
- **Loki-like log aggregation system** (любой, логи приложения пишутся в stdout)
- **Ingress с TLS termination**

### 2. HA PostgreSQL кластер версии 17+

⚠️ **Важно**: Вся настройка и поддержка, включая бекапы, на вашей стороне, поскольку кластер содержит чувствительные данные и у компании Mindbox не должно быть никакого административного доступа к нему.

### 3. S3 хранилище

Для импортов и экспортов. Anonymizer использует два бакета — `private` и `public`.

Названия `public` и `private` описывают, какие системы работают с бакетом:

- `private` — с этим бакетом работают только сам anonymizer и вы (клиент). Микросервисы Mindbox к нему не обращаются.
- `public` — с этим бакетом, помимо anonymizer и вас, работают ещё и микросервисы контура Mindbox. Все обращения Mindbox к нему идут по подписанным (presigned) ссылкам; ключи от бакета остаются только у вас и никуда не передаются.

Слово `public` в названии иногда ошибочно понимают как «бакет должен быть открыт для всех, включая неавторизованных пользователей из интернета». В действительности `public` требует лишь сетевой доступности бакета из интернета — Mindbox является внешней по отношению к вашему кластеру системой и должен иметь возможность до него достучаться. Само же обращение к бакету всегда остаётся авторизованным по подписанным ссылкам. Анонимный доступ на чтение и просмотр списка объектов (`public-read`, открытый листинг, доступ без подписи) для работы anonymizer и Mindbox не требуется — ни один из потребителей на него не рассчитывает.

### 4. FTP хранилище (опционально)

Если планируется использование FTP-интеграций.

## Инструкция по установке fluxcd



## 1. Bootstrap FluxCD

Забутстрапь fluxcd в своем кластере по официальной инструкции - [https://fluxcd.io/flux/installation/bootstrap/generic-git-server/](https://fluxcd.io/flux/installation/bootstrap/generic-git-server/). 

В команду `flux bootstrap` добавь флаг `--components-extra='image-reflector-controller,image-automation-controller'`. Команда идемпотентная, если забыл флаг можно применить еще раз с флагом.

### Пример использованный для данного репозитория:

```bash
flux bootstrap git \
  --url=ssh://git@github.com/mindbox-cloud/fluxcd-anonymizer-example-public \
  --branch=main \
  --private-key-file=~/.ssh/id_rsa \
  --components-extra='image-reflector-controller,image-automation-controller' \
  --path=clusters/anonymizer-test-stand
```



## 3. Продолжи по инструкции установки анонимайзера

[ТЫК](./clusters/anonymizer-test-stand/README.md)

## Мониторинг

Дашборды Grafana для слежения за метриками анонимайзера — [docs/dashboards](./docs/dashboards/README.md)

### Алерты

Чарт создаёт в namespace релиза `PrometheusRule` (или `VMRule` при `victoriametrics.enabled: true`) с алертами и `ServiceMonitor`/`VMServiceScrape` для сбора метрик. Чтобы алерты заработали:

- **Ваш Prometheus должен подхватывать эти ресурсы.** Чарт вешает на них лейблы из `prometheus.labels` (по умолчанию `mindbox/prometheus: common`) или `victoriametrics.labels`. Замените их на те, что ждут `ruleSelector` и `serviceMonitorSelector` вашего Prometheus. Например, kube-prometheus-stack по умолчанию выбирает ресурсы с лейблом `release: <имя релиза kube-prometheus-stack>`.
- **Нужен kube-state-metrics** с `job="kube-state-metrics"`. На нём работают алерты о подах и ресурсах (`DeploymentPodsInsufficient`, `AnonymizerRoleHasZeroPods`, `MemoryRequestsNotEqualMemoryLimits`).
- **Окно `rate` должно покрывать хотя бы 4 интервала скрейпа.** Оно задаётся значением `alertsRateWindow` (по умолчанию `1m`, подходит для скрейпа раз в 15s). При скрейпе раз в 30s поставьте `2m`, раз в 60s — `4m`, иначе алерты по метрикам запросов могут не срабатывать.
- Ссылки `runbook_url` и `dashboard_url` в аннотациях ведут в Notion и Grafana Mindbox. Для своих дежурных используйте дашборды из [docs/dashboards](./docs/dashboards/README.md).
