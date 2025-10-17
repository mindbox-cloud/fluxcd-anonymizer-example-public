# Anonymizer Test Stand with PostgreSQL

## Зависимости

Данный вариант развертывания использует:
- CR `monitoring.coreos.com/v1:PrometheusRule`
- `cert-manager.io/v1:Certificate`

## Инструкция по развертыванию

### 1. Развертывание CloudNative-PG

Скопируй себе в получившийся fluxcd репозиторий фолдер `cloudnative-pg`.

Запушь получившийся код в main, убедись что cloudnative-pg успешно развернулся.

### 2. Развертывание Anonymizer

Скопируй себе в получившийся fluxcd репозиторий фолдер `anonymizer`.

### 3. Проверка Helm values

Убедись что правильно указаны Values для хельм релиза. Основные values есть в примере, но если чего-то не хватает можно проверить весь чарт с вальюсами вот так:

```bash
helm pull oci://cr.yandex/crp2cvbrp76d7dmfegco/helm-charts/anonymizer-app
```

### 4. Деплой и настройка секретов

Запушь получившийся код в main, fluxcd должен подхватить и попытаться задеплоить приложение, однако повиснет на отсутствующих секретах. Задеплой эти секреты любым удобным способом.

Приложение ожидает увидеть следующие секреты:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: manual-secrets
  namespace: anonymizer
data:
  MindboxAuth__AnonymizerPrivateKey: >-
    <example>==
  MindboxAuth__MindboxPublicKey: >-
    <example>==
type: Opaque
---
apiVersion: v1
kind: Secret
metadata:
  name: sa-creds-anonymizer-test-stand-pg-bucket
  namespace: anonymizer
data:
  attribute.access_key: <example>==
  attribute.secret_key: <example>==
type: Opaque
```

### Описание секретов

- **manual-secrets** - секреты для авторизации между сервисами Mindbox и anonymizer. Это две пары ключей, паблик от пары ключей Mindbox вам предоставит менеджер, вашу пару необходимо сгенерировать по инструкции [docs/key-generation.md](../../docs/key-generation.md), приватную часть положить в секрет, а публичную передать менеджеру Mindbox.
- **sa-creds-anonymizer-test-stand-pg-bucket** - статичные ключи к S3 хранилищу для бекапов PostgreSQL

### 5. Завершение деплоя

После того как секреты были добавлены, деплой должен успешно завершиться.

### 6. Настройка ингресса

Настройка ингресса остается за вами, в данном примере использован external NLB yandex-cloud, но использовать его в продакшен окружении крайне не рекомендуется, т.к. он не имеет функционала терминации TLS