# [инфо] 🌐

[![Symfony](https://img.shields.io/badge/Symfony-6.4%20%2F%208.1-black?style=flat-square&logo=symfony)](https://symfony.com/)
[![PHP](https://img.shields.io/badge/PHP-8.4%2B-777BB4?style=flat-square&logo=php)](https://www.php.net/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql)](https://www.postgresql.org/)
[![Bootstrap](https://img.shields.io/badge/Bootstrap-5.3-7952B3?style=flat-square&logo=bootstrap)](https://getbootstrap.com/)
[![Cloudinary](https://img.shields.io/badge/Storage-Cloudinary-3448C5?style=flat-square&logo=cloudinary)](https://cloudinary.com/)
[![Yandex Maps](https://img.shields.io/badge/Maps-Yandex_Maps_API-red?style=flat-square&logo=yandex)](https://yandex.ru/dev/maps/)
[![License](https://img.shields.io/badge/License-Proprietary-gray?style=flat-square)](#)

> **[инфо]** — современная масштабируемая веб-платформа для управления проектами и подбора специалистов, обеспечивающая полный цикл взаимодействия: от публикации задачи до согласования результата и уведомления участников.

---

## 📑 Содержание

- [О проекте](#-о-проекте)
- [Стек технологий](#-стек-технологий)
- [Основные функции](#-основные-функции)
  - [1. Профили и проекты](#1-профили-и-проекты)
  - [2. Медиа и документы](#2-медиа-и-документы)
  - [3. Безопасность и доступ](#3-безопасность-и-доступ)
  - [4. Уведомления и коммуникации](#4-уведомления-и-коммуникации)
  - [5. Интеграции и аналитика](#5-интеграции-и-аналитика)
- [Архитектура и структура кода](#-архитектура-и-структура-кода)
- [Потоки данных (Data Flows)](#-потоки-данных-data-flows)
  - [Сценарий 1: загрузка файла через presigned URL](#сценарий-1-загрузка-файла-через-presigned-url)
  - [Сценарий 2: создание проекта и асинхронный экспорт](#сценарий-2-создание-проекта-и-асинхронный-экспорт)
- [Установка и запуск](#-установка-и-запуск)
  - [Предварительные требования](#предварительные-требования)
  - [Пошаговая инструкция](#пошаговая-инструкция)
- [Конфигурация окружения (.env)](#-конфигурация-окружения-env)
- [Тестовые пользователи (Fixtures)](#-тестовые-пользователи-fixtures)
- [Команды Makefile и консоли](#-команды-makefile-и-консоли)
- [Тестирование](#-тестирование)
- [Лицензия](#-лицензия)

---

## 🎯 О проекте

Платформа **[инфо]** решает проблему разрозненного поиска исполнителей и контроля проектных задач, предоставляя заказчикам единое пространство для публикации проектов, сравнения откликов, обмена файлами и отслеживания этапов работы.

Ключевые инженерные преимущества платформы:
- **Масштабируемость:** stateless HTTP-узлы, Redis-кэш и Symfony Messenger позволяют горизонтально добавлять workers и web-ноды.
- **Безопасная работа с файлами:** браузер загружает объекты напрямую в S3-compatible storage по короткоживущему presigned URL.
- **Разделение ответственности:** слоистая архитектура Controller/Service/Repository/DTO упрощает тестирование и развитие домена.
- **Надёжность:** транзакции PostgreSQL, идемпотентные сообщения очереди и повторная доставка webhook-событий.

---

## 🛠 Стек технологий

### Backend
- **Язык:** PHP 8.4+
- **Фреймворк:** Symfony 6.4 LTS / Symfony 8.1 components
- **ORM / База данных:** Doctrine ORM 3.x, PostgreSQL 16 (полнотекстовый поиск и JSONB)
- **Асинхронность и очереди:** Symfony Messenger, Redis Streams, Symfony Scheduler
- **Безопасность:** Symfony Security, RateLimiter, Argon2id, JWT для API
- **Авторизация:** OAuth2 (Google, GitHub), email verification

### Frontend
- **Шаблонизатор:** Twig
- **UI Framework:** Bootstrap 5.3
- **Клиентский стек:** Stimulus + Turbo, TypeScript для интерактивных модулей
- **Работа с картами:** Yandex Maps API (поиск и геокодирование)
- **Работа с медиа:** Cropper.js, Dropzone.js

### Интеграции и внешние сервисы
- **Cloudinary / S3:** хранение, ресайз и CDN-доставка медиа
- **Maps API:** геокодирование адресов и отображение проектов на карте
- **OAuth 2.0:** Google и GitHub
- **Почта:** SMTP через Symfony Mailer; события доставляются через очередь
- **Наблюдаемость:** Monolog, Sentry, Prometheus metrics

Пример маршрута и сообщения Messenger:

```yaml
# config/packages/messenger.yaml
framework:
  messenger:
    transports:
      async: '%env(MESSENGER_TRANSPORT_DSN)%'
    routing:
      'App\Message\GenerateProjectExport': async
```

```php
final readonly class GenerateProjectExport
{
    public function __construct(public int $projectId, public string $requestedBy) {}
}
```

---

## 🌟 Основные функции

### 1. Профили и проекты
- Профиль специалиста с навыками, ставкой, портфолио и подтверждёнными компетенциями.
- Создание проекта с бюджетом, сроками, географией и приватностью.
- Фильтрация и полнотекстовый поиск по PostgreSQL; отклики и согласование статуса проекта.

### 2. Медиа и документы
- Загрузка аватаров, брифов и результатов через presigned URL без проксирования больших файлов приложением.
- Проверка MIME-типа и размера, антивирусная проверка worker-ом, автоматические превью через Cloudinary.
- Версионирование документов, права доступа на уровне проекта и журнал скачиваний.

### 3. Безопасность и доступ
- RBAC-роли `ROLE_ADMIN`, `ROLE_MANAGER`, `ROLE_SPECIALIST`, `ROLE_CLIENT`.
- OAuth2, подтверждение email, восстановление пароля и JWT access/refresh tokens для API.
- Rate limiting, CSRF, аудит входов и блокировка подозрительных сессий.

### 4. Уведомления и коммуникации
- Центр уведомлений в интерфейсе, email и web-push о новых откликах, сообщениях и изменениях статуса.
- Комментарии к проекту и real-time обновления через Mercure.
- Настройки частоты уведомлений и quiet hours в профиле пользователя.

### 5. Интеграции и аналитика
- Геокодирование адреса проекта и карта ближайших специалистов через Yandex Maps API.
- Экспорт проектов и финансовых отчётов в CSV/XLSX асинхронно.
- Webhook-интеграции с CRM с подписью HMAC и повтором неуспешных доставок.

---

## 📂 Архитектура и структура кода

Проект следует принципам чистой слоистой архитектуры (Layered Architecture / SOLID):

```text
[инфо]/
├── assets/                          # Фронтенд-ресурсы
│   ├── app.ts                       # Главная точка входа
│   ├── controllers/                 # Stimulus-контроллеры
│   ├── js/
│   └── styles/
├── bin/
│   └── console
├── config/                          # Конфигурация Symfony
├── migrations/                      # Миграции БД
├── src/
│   ├── Controller/                  # HTTP-контроллеры и API endpoints
│   ├── DTO/                         # Валидируемые Data Transfer Objects
│   ├── DataFixtures/                # Тестовые данные
│   ├── Entity/                      # Сущности Doctrine ORM
│   ├── Enum/                        # Перечисления домена
│   ├── Message/                     # Команды и события очереди
│   ├── Repository/                  # Запросы и репозитории
│   ├── Security/                    # Аутентификация, voters, OAuth
│   └── Service/                     # Бизнес-логика и интеграции
├── templates/                       # Twig-шаблоны
├── tests/                           # Unit/Functional тесты
├── compose.yaml                     # Docker Compose
├── Makefile
└── composer.json
```

`Controller` принимает запрос и формирует DTO, `Service` выполняет бизнес-операцию в транзакции, `Repository` инкапсулирует запросы Doctrine, а `Entity` не содержит инфраструктурной логики. Доступ проверяется через Voter до вызова сервиса.

---

## 🔄 Потоки данных (Data Flows)

### Сценарий 1: загрузка файла через presigned URL

```text
Browser --POST /api/uploads/presign--> UploadController
       <--{uploadId, url, headers}----
Browser --PUT binary------------------> S3/Cloudinary
       --POST /api/uploads/{id}/complete--> UploadController
       <--202 Accepted----------------
Messenger --MediaUploaded-----------> MediaProcessor
MediaProcessor --scan/resize---------> Storage
MediaProcessor --UPDATE media---------> PostgreSQL
```

1. Сервер проверяет роль пользователя, расширение, размер и принадлежность `projectId`, затем создаёт `Media` со статусом `pending`.
2. `StorageService` выдаёт URL с TTL 10 минут и ключом `projects/{projectId}/{uuid}`; секретные ключи браузеру не передаются.
3. После успешного PUT клиент вызывает `complete`. Сервер проверяет наличие объекта через HEAD и публикует `MediaUploaded`.
4. Worker сканирует объект, генерирует превью и меняет статус на `ready` либо `rejected`. Повторная обработка безопасна по `uploadId`.

### Сценарий 2: создание проекта и асинхронный экспорт

```text
Client --POST /projects----------------> ProjectController
       --ProjectCreateData-------------> ProjectService
ProjectService --transaction-----------> PostgreSQL
ProjectService --address---------------> Yandex Geocoder
ProjectService --ProjectCreated--------> Messenger/Redis
ExportController --POST /exports-------> ExportService
ExportService --GenerateProjectExport-> queue
ExportWorker --read/stream-------------> PostgreSQL
ExportWorker --write-------------------> S3
Browser <--notification + signed URL--- NotificationHandler
```

1. `ProjectCreateData` валидирует бюджет, даты и адрес. `ProjectService` сохраняет проект и outbox-событие в одной транзакции.
2. `GeocodingService` нормализует адрес и координаты; временная ошибка API не отменяет создание проекта, а ставит геокодирование в retry-очередь.
3. Запрос экспорта возвращает `202` и `exportId`. Worker читает данные пакетами, формирует CSV/XLSX и загружает результат в приватное хранилище.
4. После завершения публикуется уведомление. Клиент получает одноразовый signed URL с TTL 15 минут; неуспешные сообщения повторяются с backoff.

---

## 🚀 Установка и запуск

### Предварительные требования

- Docker Compose plugin 2.20+ и Docker Engine 24+.
- PHP 8.4+, Composer 2.7+, Node.js 22+ и npm 10+ для локального запуска без Docker.
- PostgreSQL 16, Redis 7 и S3-compatible storage (для production).
- Ключи Yandex Maps и OAuth-провайдеров для полного набора интеграционных сценариев.

### Пошаговая инструкция

```bash
git clone <repository-url> platform
cd platform
cp .env.example .env.local
docker compose up -d postgres redis
composer install
php bin/console secrets:generate-keys
php bin/console doctrine:migrations:migrate --no-interaction
php bin/console doctrine:fixtures:load --no-interaction
npm ci
npm run build
symfony server:start -d
php bin/console messenger:consume async -vv
```

Для Docker-профиля:

```bash
docker compose --profile app up -d --build
docker compose exec php bin/console doctrine:migrations:migrate --no-interaction
docker compose exec php bin/console messenger:consume async --time-limit=3600
```

Приложение доступно по адресу `http://localhost:8000`, health-check — `GET /health`.

---

## 🔐 Конфигурация окружения (.env)

Секреты не коммитятся в репозиторий. Минимальный набор для `.env.local`:

```dotenv
APP_ENV=dev
APP_SECRET=change-me
APP_URL=http://localhost:8000

DATABASE_URL="postgresql://platform:platform@127.0.0.1:5432/platform?serverVersion=16&charset=utf8"
REDIS_URL=redis://127.0.0.1:6379
MESSENGER_TRANSPORT_DSN=redis://127.0.0.1:6379/messages

AWS_ACCESS_KEY_ID=local-access-key
AWS_SECRET_ACCESS_KEY=local-secret-key
AWS_DEFAULT_REGION=ru-central1
AWS_BUCKET=platform-media
AWS_ENDPOINT=https://storage.yandexcloud.net
AWS_USE_PATH_STYLE_ENDPOINT=false

CLOUDINARY_URL=cloudinary://api-key:api-secret@cloud-name
YANDEX_MAPS_API_KEY=your-yandex-maps-key

OAUTH_GOOGLE_CLIENT_ID=google-client-id
OAUTH_GOOGLE_CLIENT_SECRET=google-client-secret
OAUTH_GITHUB_CLIENT_ID=github-client-id
OAUTH_GITHUB_CLIENT_SECRET=github-client-secret

MAILER_DSN=smtp://user:password@smtp.example.com:587
MAIL_FROM_ADDRESS=no-reply@example.com
MAIL_FROM_NAME="[инфо]"

MERCURE_URL=http://mercure/.well-known/mercure
MERCURE_PUBLIC_URL=http://localhost:3000/.well-known/mercure
MERCURE_JWT_SECRET=change-me-too
SENTRY_DSN=
```

В production значения хранятся в secret manager, `APP_DEBUG=0`, а bucket и database доступны только из private network.

---

## 👥 Тестовые пользователи (Fixtures)

После `doctrine:fixtures:load` доступны следующие аккаунты (пароль для всех: `ChangeMe123!`):

| Роль | Email | Возможности |
|---|---|---|
| Администратор | `admin@info.test` | пользователи, роли, аудит и системные настройки |
| Менеджер | `manager@info.test` | проекты команды, отклики, экспорт и уведомления |
| Заказчик | `client@info.test` | создание проектов, выбор специалиста и документы |
| Специалист | `specialist@info.test` | профиль, портфолио, отклики и рабочие файлы |
| Гость | `guest@info.test` | только публичный каталог без изменения данных |

Тестовые пользователи предназначены только для dev/test окружений и не должны использоваться в production.

---

## 🧰 Команды Makefile и консоли

```bash
make install       # установка PHP/JS зависимостей и подготовка окружения
make up            # запуск инфраструктуры Docker
make down          # остановка контейнеров
make migrate       # применение миграций Doctrine
make fixtures      # загрузка тестовых данных
make cs-fix        # форматирование PHP через PHP-CS-Fixer
make lint          # Symfony lint:container и проверка Twig
make test          # PHPUnit и функциональные тесты
make queue         # запуск async worker
```

Эквивалентные команды Symfony:

```bash
php bin/console cache:clear
php bin/console debug:router
php bin/console doctrine:migrations:diff
php bin/console messenger:failed:show
php bin/console messenger:failed:retry --force
php bin/console app:projects:archive --before="30 days ago"
```

---

## 🧪 Тестирование

```bash
APP_ENV=test php bin/console doctrine:database:create --if-not-exists
APP_ENV=test php bin/console doctrine:migrations:migrate --no-interaction
vendor/bin/phpunit --testsuite unit
vendor/bin/phpunit --testsuite functional
vendor/bin/phpunit --coverage-text
```

Unit-тесты проверяют сервисы и политики доступа без сети; functional-тесты используют изолированную PostgreSQL и Redis. Интеграционные тесты внешних API запускаются только при наличии соответствующих ключей и помечаются `@group integration`.

---

## 📄 Лицензия

**[инфо]** распространяется по проприетарной лицензии. Исходный код, документация и медиаматериалы не могут копироваться, изменяться или распространяться без письменного разрешения правообладателя.
