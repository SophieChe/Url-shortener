# URL Shortener Service

## Please find eng description below
**Описание проекта:**  
Высокопроизводительный сервис сокращения URL, который преобразует длинные ссылки в короткие, удобные для обмена. Гарантируется уникальность ссылок, быстрый поиск и минимальное количество коллизий.

---

## Архитектура

### 1. Сервис URL (`UrlService`)
- Создаёт короткие URL и возвращает оригинальные.
- Использует `HashCache` для получения заранее сгенерированных уникальных хэшей.
- Сохраняет сущности `Url` в **PostgreSQL**.
- Кэширует отображения URL → оригинальный URL в **Redis** для быстрой работы.

### 2. Генерация и кэш хэшей
- `HashGenerator` и `SqidsEncoder` создают уникальные хэши на основе последовательных чисел.
- `HashCache` хранит готовые хэши в памяти, чтобы ускорить обработку и избежать коллизий.
- `LocalCache` управляет извлечением хэшей из базы и генерирует новые при необходимости.
- `HashCacheWarmupRunner` загружает партию хэшей при запуске приложения.

### 3. Слой хранения данных
- **PostgreSQL** хранит сущности `Url` и `Hash`.
- `UrlRepository` и `HashRepository` управляют операциями с базой.
- `UrlCacheRepository` (Redis) обеспечивает быстрый доступ к отображению короткий URL → длинный URL.

### 4. Планировщик (`CleanerScheduler`)
- Периодически очищает устаревшие URL и возвращает их хэши обратно в пул.

### 5. Кодирование хэшей
- Используется библиотека **Sqids** для генерации компактных, безопасных от коллизий кодов.
- Минимальная длина хэша настраивается (по умолчанию 6 символов).

### 6. API (`UrlController`)
- `POST /api/v1/short` → Принимает длинный URL и возвращает короткий.
- `GET /api/v1/{hash}` → Перенаправляет пользователя на оригинальный URL, связанный с хэшем hash. Возвращает HTTP 302 с заголовком Location.
- Требуется заголовок `x-user-id` для идентификации пользователя.

---

## Ключевые преимущества
- Предварительно сгенерированный пул хэшей обеспечивает высокую производительность с минимальными коллизиями.
- Кэш Redis снижает нагрузку на базу данных.
- Транзакционная безопасность гарантирует согласованность между кэшем и базой.
- Расширяемая архитектура: легко добавить аналитику, управление пользователями или кастомные политики истечения ссылок.

_____________

  # 🇬🇧 English Version# 

**Project Description:**  
A high-performance URL shortening service that converts long URLs into short, shareable links. Ensures uniqueness, fast lookups, and minimal collisions.

---

## Architecture

### 1. URL Service (`UrlService`)
- Creates short URLs and retrieves original URLs.
- Uses `HashCache` to get pre-generated unique hashes.
- Stores `Url` entities in **PostgreSQL**.
- Caches URL → original URL mappings in **Redis** for fast retrieval.

### 2. Hash Generation and Caching
- `HashGenerator` and `SqidsEncoder` generate unique hashes from sequential numbers.
- `HashCache` keeps pre-generated hashes in memory to speed up processing and prevent collisions.
- `LocalCache` manages fetching hashes from the database and generating new ones as needed.
- `HashCacheWarmupRunner` preloads a batch of hashes at application startup.

### 3. Data Storage Layer
- **PostgreSQL** stores `Url` and `Hash` entities.
- `UrlRepository` and `HashRepository` handle database operations.
- `UrlCacheRepository` (Redis) provides fast URL → original URL lookups.

### 4. Scheduler (`CleanerScheduler`)
- Periodically deletes expired URLs and returns their hashes back to the pool.

### 5. Hash Encoding
- Uses **Sqids** library to generate compact, collision-resistant codes.
- Minimum hash length is configurable (default: 6 characters).

### 6. API (`UrlController`)
- `POST /api/v1/short` → Accepts a long URL and returns a short URL.
- `GET /api/v1/{hash}` → Redirects the user to the original URL associated with the hash. Returns HTTP 302 with a Location header.
- Requires `x-user-id` header for user identification.

---

## Key Advantages
- Pre-generated hash pool ensures high performance with minimal collisions.
- Redis caching reduces database load.
- Transactional safety guarantees consistency between cache and database.
- Extensible architecture: easy to add analytics, user management, or custom URL expiration policies.


