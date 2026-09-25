# Дашборды Grafana для слежения за метриками анонимайзера

Ниже представлены несколько дашбордов для системы мониторинга и аналитики **Grafana**. Дашборды сохранены в формате JSON-файлов, которые можно импортировать в Grafana, указав источник данных **Prometheus**.

| Дашборд | Файл |
|---|---|
| Anonymizer - Main Metrics | [Anonymizer_-_Main_Metrics.json](./Anonymizer_-_Main_Metrics.json) |
| Anonymizer - Basic Service Metrics | [Anonymizer_-_Basic_Service_Metrics.json](./Anonymizer_-_Basic_Service_Metrics.json) |
| Anonymizer - Instances Services Metrics | [Anonymizer_-_Instances_Services_Metrics.json](./Anonymizer_-_Instances_Services_Metrics.json) |

Запросы опираются на лейблы, которые добавляет `ServiceMonitor`/`VMServiceScrape` из чарта `anonymizer-app`: `namespace`, `service`, `pod` и `exported_endpoint` (лейбл `endpoint` из метрик приложения переименовывается при скрейпе, так как конфликтует с одноимённым таргет-лейблом). Если метрики собираются не через `ServiceMonitor`/`VMServiceScrape` из чарта, имена этих лейблов могут отличаться, и запросы придётся поправить.

Переменные дашбордов:

| Переменная | Назначение |
|---|---|
| `cluster` | Опциональная. Нужна, если один Prometheus/VictoriaMetrics собирает метрики нескольких кластеров. Если лейбла `cluster` нет, остаётся `All` и фильтр ничего не отсекает. Если кластер помечается другим лейблом, замените `cluster` в запросах дашборда. |
| `namespace` | Namespace, в который установлен чарт. По умолчанию `anonymizer`. |
| `service` | Сервис анонимайзера (роль `regular`, `important`, lrt, gate). |
| `pod`, `route`, `endpoint`, `method` | Дополнительные фильтры на отдельных дашбордах. |

Окна `rate`/`increase` заданы через `$__rate_interval`, поэтому дашборды работают при любом `scrape_interval`. Чтобы Grafana считала окно правильно, в настройках источника данных должен быть указан `Scrape interval`, совпадающий с реальным.

## Anonymizer - Main Metrics

**Назначение:** основной мониторинг сервиса Anonymizer с метриками производительности.

**Содержит панели:**

- RPS
- Среднее время выполнения запроса
- Коды ответов MindboxApi и Anonymizer
- Время выполнения синхронных и асинхронных операций
- Отсутствие необходимых данных для деанонимизации
- Конфликты при вставке персональных данных (`anonymizer_insert_conflict_events_total`)
- Импорт/экспорт файлов: начатые, успешные, упавшие и пропущенные файлы, причины ошибок, время обработки файла (`anonymizer_files_*`, `anonymizer_file_transfer_duration_seconds`)
- Аутентификация через IdP: логины, ошибки колбэков, время входа, активные сессии, фоновые задачи очистки, ротация SAML-сертификата (`anonymizer_auth_*`)

**Использование:** главный дашборд для отслеживания базовой производительности сервиса и обнаружения аномалий в обработке запросов.

## Anonymizer - Basic Service Metrics

**Назначение:** базовые метрики API-сервисов Anonymizer для общего мониторинга.

**Использование:** простой дашборд с основными метриками, предназначен для быстрой оценки состояния сервиса.

## Anonymizer - Instances Services Metrics

**Назначение:** мониторинг метрик по отдельным инстансам (подам) сервиса Anonymizer.

**Содержит панели:**

- Requests by pod
- HTTP requests in progress (average by endpoint) — запросы в обработке в среднем на каждый endpoint
- HTTP requests in progress (by pod) — запросы в обработке по каждому поду
- Requests per Second — количество запросов в секунду по подам
- Раздел «Requests in progress» — текущие запросы в обработке

**Использование:** дашборд для углублённого анализа производительности на уровне отдельных инстансов, диагностики дисбаланса нагрузки между подами.
