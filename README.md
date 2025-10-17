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

Для импортов и экспортов.

### 4. FTP хранилище (опционально)

Если планируется использование FTP-интеграций.

## Инструкция по установке anonymizer

## 1. Bootstrap FluxCD

Забутстрапь fluxcd в своем кластере по официальной инструкции - https://fluxcd.io/flux/installation/bootstrap/generic-git-server/. 

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

## 2. Выбери подходящий вариант

- **a. [`clusters/anonymizer-test-stand`](clusters/anonymizer-test-stand/README.md)** - рекомендованный вариант без установки PostgreSQL
- **b. [`clusters/anonymizer-test-stand-w-pg`](clusters/anonymizer-test-stand-w-pg/README.md)** - НЕ рекомендованный вариант с установкой PostgreSQL через CNPG

### ⚠️ Важное замечание

- Если у вас есть опыт поддержки высоконагруженных баз PostgreSQL - разворачивайте как умеете
- Если опыта нет, настоятельно рекомендуем пользоваться managed PostgreSQL облачных провайдеров
- Если опыта нет и возможности использовать managed PG тоже нет, тогда можете воспользоваться вариантом b, но учитывайте что Mindbox не поможет вам в настройке/поддержке PG

## 3. Продолжи по инструкции

Продолжи по инструкции в README выбранного варианта.
