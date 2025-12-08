# Генерация RSA ключей для Anonymizer

## Обзор

Для безопасной авторизации между сервисами Mindbox и Anonymizer необходимо сгенерировать пару RSA ключей. Приватный ключ будет использоваться в Anonymizer, а публичный ключ нужно передать менеджеру Mindbox.

## Генерация ключей

### 1. Генерация приватного ключа

```bash
# Генерация приватного ключа RSA 4096 бит
openssl genrsa -out anonymizer_private_key.pem 4096
```

### 2. Извлечение публичного ключа

```bash
# Извлечение публичного ключа из приватного
openssl rsa -in anonymizer_private_key.pem -pubout -out anonymizer_public_key.pem
```

### 3. Конвертация в формат для Kubernetes Secret

```bash
# Конвертация приватного ключа в base64 для Kubernetes Secret
base64 -i anonymizer_private_key.pem | tr -d '\n' > anonymizer_private_key_base64.txt
```

## Использование ключей

### 1. Приватный ключ в Kubernetes Secret

Скопируйте содержимое файла `anonymizer_private_key_base64.txt` в поле `MindboxAuth__AnonymizerPrivateKey` в секрете `manual-secrets`.

### 2. Публичный ключ для Mindbox

Отправьте содержимое файла `anonymizer_public_key.pem` менеджеру Mindbox для настройки авторизации.

## Проверка ключей

### Проверка приватного ключа

```bash
# Проверка формата приватного ключа
openssl rsa -in anonymizer_private_key.pem -check -noout
```

### Проверка публичного ключа

```bash
# Проверка формата публичного ключа
openssl rsa -pubin -in anonymizer_public_key.pem -text -noout
```

### Проверка соответствия ключей

```bash
# Проверка что публичный ключ соответствует приватному
openssl rsa -in anonymizer_private_key.pem -pubout | diff - anonymizer_public_key.pem
```

## Удалите временные файлы после использования:

```bash
# Очистка временных файлов
rm anonymizer_private_key.pem anonymizer_public_key.pem anonymizer_private_key_base64.txt
```
