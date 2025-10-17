# Anonymizer Test Stand

## Зависимости

Данный вариант развертывания использует CR `monitoring.coreos.com/v1:PrometheusRule`.

## Инструкция по развертыванию

### 1. Копирование конфигурации

Скопируй себе в получившийся fluxcd репозиторий фолдер `anonymizer`.

### 2. Проверка Helm values

Убедись что правильно указаны Values для хельм релиза. Основные values есть в примере, но если чего-то не хватает можно проверить весь чарт с вальюсами вот так:

```bash
helm pull oci://cr.yandex/crp2cvbrp76d7dmfegco/helm-charts/anonymizer-app
```

### 3. Деплой и настройка секретов

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
  name: db-anonymizer-anonymizer-user-password
  namespace: anonymizer
data:
  password: <example>==
  username: <example>==
type: kubernetes.io/basic-auth
```

### Описание секретов

- **manual-secrets** - секреты для авторизации между сервисами Mindbox и anonymizer. Это две пары ключей, паблик от пары ключей Mindbox вам предоставит менеджер, вашу пару необходимо сгенерировать по инструкции [docs/key-generation.md](../../docs/key-generation.md), приватную часть положить в секрет, а публичную передать менеджеру Mindbox.
- **db-anonymizer-anonymizer-user-password** - логин и пароль пользователя PostgreSQL

### 4. Завершение деплоя

После того как секреты были добавлены, деплой должен успешно завершиться.

### 5. Настройка ингресса

Настройка ингресса остается за вами, в данном примере использован external NLB yandex-cloud, но использовать его в продакшен окружении крайне не рекомендуется, т.к. он не имеет функционала терминации TLS