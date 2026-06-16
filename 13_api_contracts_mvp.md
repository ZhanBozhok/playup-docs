# 13. API Contracts MVP

## Назначение

Этот файл фиксирует минимальные API-контракты для клиентской мини-апки и админки. Его цель — чтобы два репозитория работали с одной логикой и одной базой, а не придумывали разные версии событий, записей, оплат и пользователей.

## Общие правила

API единый, но доступы разные: клиент видит только свои данные и опубликованные события, админ видит операционные данные. Все ответы — JSON. Все даты — ISO 8601. Все деньги передаются с `amount` и `currency`. По умолчанию валюта MVP — `RSD`, но поле валюты должно быть всегда.

Клиент не должен получать внутренние заметки, расходы, чужие данные, внутренние ID админов и неопубликованные события.

## Авторизация клиента

`POST /api/client/auth/telegram`

Request:

```json
{
  "init_data": "telegram_init_data_string"
}
```

Response:

```json
{
  "token": "client_session_token",
  "user": {
    "id": "uuid",
    "telegram_username": "username",
    "profile_completed": false
  }
}
```

Backend должен валидировать Telegram init data, найти или создать пользователя по `telegram_id`, обновить `last_seen_at` и вернуть токен.

## Авторизация админа

`POST /api/admin/auth/login`

Request:

```json
{
  "email": "admin@playup.club",
  "password": "password"
}
```

Response:

```json
{
  "token": "admin_session_token",
  "admin": {
    "id": "uuid",
    "name": "Admin",
    "role": "owner"
  }
}
```

## Client API

### Главная клиента

`GET /api/client/home`

Response:

```json
{
  "user": {
    "id": "uuid",
    "display_name": "Jean",
    "profile_completed": true
  },
  "upcoming_bookings": [
    {
      "booking_id": "uuid",
      "booking_status": "booked",
      "event": {
        "id": "uuid",
        "title": "Sunday Football",
        "activity_type": "football",
        "starts_at": "2026-06-21T18:00:00+02:00",
        "ends_at": "2026-06-21T19:30:00+02:00",
        "venue_name": "Sportski Selo",
        "status": "published"
      }
    }
  ],
  "suggested_events": []
}
```

### Расписание клиента

`GET /api/client/events?from=2026-06-16&to=2026-07-16&activity_type=football`

Response:

```json
{
  "events": [
    {
      "id": "uuid",
      "title": "Sunday Football",
      "activity_type": "football",
      "starts_at": "2026-06-21T18:00:00+02:00",
      "ends_at": "2026-06-21T19:30:00+02:00",
      "venue": {
        "id": "uuid",
        "name": "Sportski Selo",
        "address": "Belgrade"
      },
      "price": 1200,
      "currency": "RSD",
      "capacity": 18,
      "booked_count": 11,
      "spots_left": 7,
      "level": "mixed",
      "status": "published",
      "user_booking_status": null
    }
  ]
}
```

Клиентский список возвращает только `published` будущие события.

### Карточка события клиента

`GET /api/client/events/{event_id}`

Response:

```json
{
  "id": "uuid",
  "title": "Sunday Football",
  "activity_type": "football",
  "description": "Friendly football game with host and coffee after.",
  "starts_at": "2026-06-21T18:00:00+02:00",
  "ends_at": "2026-06-21T19:30:00+02:00",
  "venue": {
    "id": "uuid",
    "name": "Sportski Selo",
    "address": "Belgrade",
    "map_url": null
  },
  "host": {
    "name": "Vlad"
  },
  "price": 1200,
  "currency": "RSD",
  "capacity": 18,
  "booked_count": 11,
  "spots_left": 7,
  "min_quorum": 10,
  "level": "mixed",
  "status": "published",
  "user_booking_status": null,
  "requires_profile_completion": true
}
```

### Запись на событие

`POST /api/client/events/{event_id}/book`

Request для первой записи:

```json
{
  "profile": {
    "display_name": "Jean",
    "level": "amateur",
    "preferred_sports": ["football"],
    "preferred_area": "Vracar",
    "traffic_source": "telegram_chat"
  }
}
```

Request для повторной записи:

```json
{}
```

Response:

```json
{
  "booking": {
    "id": "uuid",
    "status": "booked"
  },
  "event": {
    "id": "uuid",
    "spots_left": 6,
    "user_booking_status": "booked"
  }
}
```

Backend проверяет: событие опубликовано, не отменено, еще не началось, есть места, пользователь еще не записан.

### Отмена записи

`POST /api/client/bookings/{booking_id}/cancel`

Request:

```json
{
  "reason": "plans_changed"
}
```

Response:

```json
{
  "booking": {
    "id": "uuid",
    "status": "cancelled"
  }
}
```

### Профиль клиента

`GET /api/client/profile`

`PATCH /api/client/profile`

Поля для редактирования: `display_name`, `level`, `preferred_sports`, `preferred_area`, `available_times`, `phone`.

Источник привлечения лучше не редактировать в клиенте после первого заполнения.

### Ответ на опрос

`POST /api/client/surveys/{survey_id}/responses`

Request:

```json
{
  "event_id": "uuid",
  "answer_value": 5,
  "answer_text": "Great host, good game."
}
```

## Admin API

### Главная админки

`GET /api/admin/dashboard`

Response:

```json
{
  "upcoming_events": [
    {
      "id": "uuid",
      "title": "Sunday Football",
      "starts_at": "2026-06-21T18:00:00+02:00",
      "venue_name": "Sportski Selo",
      "host_name": "Vlad",
      "capacity": 18,
      "booked_count": 11,
      "min_quorum": 10,
      "risk_status": "ok",
      "expected_revenue": 13200
    }
  ],
  "actions": [
    {
      "type": "mark_attendance",
      "event_id": "uuid",
      "label": "Mark attendance for Sunday Football"
    }
  ],
  "metrics": {
    "revenue_week": 42000,
    "expenses_week": 18000,
    "profit_week": 24000,
    "new_users_week": 18,
    "repeat_users_week": 6,
    "no_show_rate_week": 0.08
  }
}
```

### Пользователи

`GET /api/admin/users?search=&status=&source=&activity_type=`

`GET /api/admin/users/{user_id}`

`PATCH /api/admin/users/{user_id}/profile`

Карточка пользователя должна возвращать: базовые данные, профиль, записи, посещения, оплаты, ответы на опросы и заметки.

### События

`GET /api/admin/events?from=&to=&status=&activity_type=&venue_id=&host_id=`

`POST /api/admin/events`

`GET /api/admin/events/{event_id}`

`PATCH /api/admin/events/{event_id}`

`POST /api/admin/events/{event_id}/publish`

`POST /api/admin/events/{event_id}/cancel`

`POST /api/admin/events/{event_id}/complete`

### Участники события

`GET /api/admin/events/{event_id}/participants`

Response:

```json
{
  "participants": [
    {
      "user_id": "uuid",
      "booking_id": "uuid",
      "display_name": "Jean",
      "telegram_username": "username",
      "booking_status": "booked",
      "attendance_status": "unknown",
      "payment_status": "unpaid",
      "payment_amount": 1200
    }
  ]
}
```

Добавить участника вручную:

`POST /api/admin/events/{event_id}/participants`

Отметить явку массово:

`POST /api/admin/events/{event_id}/attendance/bulk`

Отметить оплату массово:

`POST /api/admin/events/{event_id}/payments/bulk`

При отметке `paid` backend создает income transaction.

### Площадки и хосты

`GET /api/admin/venues`

`POST /api/admin/venues`

`PATCH /api/admin/venues/{venue_id}`

`GET /api/admin/hosts`

`POST /api/admin/hosts`

`PATCH /api/admin/hosts/{host_id}`

### Финансы

`GET /api/admin/cashboxes`

`POST /api/admin/cashboxes`

`GET /api/admin/finance/categories`

`POST /api/admin/finance/categories`

`GET /api/admin/transactions?from=&to=&type=&cashbox_id=&event_id=&category_id=`

`POST /api/admin/transactions`

### Пуши и опросы

`POST /api/admin/notifications/send`

`GET /api/admin/notifications`

`POST /api/admin/surveys`

`GET /api/admin/surveys/{survey_id}/responses`

### CSV

`GET /api/admin/export/users.csv`

`GET /api/admin/export/events.csv`

`GET /api/admin/export/bookings.csv`

`GET /api/admin/export/attendance.csv`

`GET /api/admin/export/payments.csv`

`GET /api/admin/export/transactions.csv`

## Стандарт ошибок

```json
{
  "error": {
    "code": "EVENT_FULL",
    "message": "No spots left for this event.",
    "details": {}
  }
}
```

Коды ошибок MVP:

- `UNAUTHORIZED`
- `FORBIDDEN`
- `NOT_FOUND`
- `VALIDATION_ERROR`
- `EVENT_NOT_PUBLISHED`
- `EVENT_CANCELLED`
- `EVENT_ALREADY_STARTED`
- `EVENT_FULL`
- `ALREADY_BOOKED`
- `BOOKING_NOT_FOUND`
- `PROFILE_REQUIRED`
- `INVALID_STATUS_TRANSITION`

## Не делаем в API MVP

Не делаем GraphQL, публичный API для партнеров, webhooks, сложную версионизацию, полноценный rate limiting, idempotency для всех операций и API для нативных приложений.
