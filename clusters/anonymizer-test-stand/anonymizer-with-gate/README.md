# Anonymizer + anonymizer-gate

Альтернатива фолдеру [`../anonymizer`](../anonymizer) — та же установка anonymizer, но дополнительно
включает `anonymizer-gate`. Используйте этот пример вместо `anonymizer`, если вам нужен gate;
не разворачивайте оба фолдера одновременно — они используют одни и те же `HelmRelease` и namespace.

## Что такое anonymizer-gate

`anonymizer-gate` — отдельный компонент чарта, который принимает клиентские write-запросы от
`client-gate` Mindbox (адрес выдаёт менеджер). Это не заменяет основной anonymizer (`services` /
`lrt`), который продолжает обрабатывать обычные запросы от `anon-api-gate`, а работает как
дополнительный сервис со своими подами `gate-service` и `gate-lrt`.

Начиная с версии чарта `3.21.0` сам `gate-service` (`gate.services`), как и основной anonymizer,
по умолчанию делится на роли `regular` и `important` — два независимых деплоймента со своим
`GATE_ROLE`. При этом `gate.lrt` (фоновый воркер) остаётся общим — единственная роль `default`,
так же как у основного `lrt` — но настраивается на **два** адреса client-gate Mindbox,
`gate.clientGateRegularUrl` и `gate.clientGateImportantUrl` (вместо одного `gate.clientGateUrl`),
и сам выбирает нужный при исходящей доставке в зависимости от роли конкретного сообщения.

## Отличия от базового примера

В `helm-release.yaml` добавлены секции:

```yaml
gate:
  enabled: true
  clientGateRegularUrl: "https://anon-client-gate-regular.a.mindbox.ru"     # адреса выдаёт
  clientGateImportantUrl: "https://anon-client-gate-important.a.mindbox.ru" # менеджер Mindbox
  # ... resources/replicas для gate.lrt и gate.services

anonymizerGateMandatoryEnv:
  clientGateAuth:
    clientGatePublicKey:
      valueFrom:
        secretKeyRef:
          name: client-gate-auth
          key: ClientGatePublicKey
```

В `loadbalancer-service.yaml` добавлены ещё два `LoadBalancer`-сервиса — `anonymizer-gate-regular-loadbalancer`
и `anonymizer-gate-important-loadbalancer`, — каждый таргетит свою роль подов `gate-service`
(аналогично тому, как это сделано для основных `regular`/`important`).

## Инструкция по установке

Установка полностью повторяет [инструкцию для базового примера](../README.md), с двумя дополнениями:

### 1. Секрет `client-gate-auth`

Помимо секретов, описанных в [базовой инструкции](../README.md#3-деплой-и-настройка-секретов),
приложению также нужен публичный ключ client-gate Mindbox:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: client-gate-auth
  namespace: anonymizer
data:
  ClientGatePublicKey: <example>==
type: Opaque
```

Значение `ClientGatePublicKey` предоставляет менеджер Mindbox.

### 2. Адреса gate

После деплоя `anonymizer-gate-regular-loadbalancer` и `anonymizer-gate-important-loadbalancer`
получат внешние адреса — так же, как основные `regular`/`important` LoadBalancer'ы. Оба адреса
нужно передать менеджеру Mindbox для настройки `client-gate`.

> Как и с основным анонимайзером, external NLB Yandex Cloud использован здесь только в качестве
> примера для тестового стенда и не рекомендуется для продакшена — настройте TLS-terminating ingress
> самостоятельно (см. [`gate.services.ingressRoute`](https://github.com/mindbox-cloud/helm-charts-public/tree/main/charts/anonymizer-app)
> в чарте, если у вас есть Traefik).
