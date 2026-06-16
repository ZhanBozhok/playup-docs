# 23. Seed Data and Test Cases

## Назначение

Этот файл описывает стартовые тестовые данные и ручные сценарии проверки. Он нужен, чтобы быстро поднять локальную среду и проверить MVP end-to-end.

## Seed: Admin users

```json
[
  {
    "email": "owner@playup.club",
    "name": "Owner",
    "role": "owner",
    "status": "active"
  },
  {
    "email": "admin@playup.club",
    "name": "Admin",
    "role": "admin",
    "status": "active"
  }
]
```

## Seed: Cashboxes

```json
[
  { "name": "Cash RSD", "type": "cash", "currency": "RSD", "is_active": true },
  { "name": "Bank RSD", "type": "bank", "currency": "RSD", "is_active": true }
]
```

## Seed: Finance categories

Income: Event payment, Corporate, Other income.

Expense: Venue, Host, Water, Equipment, Ads, Content, Software, Refund, Other expense.

## Seed: Venues

```json
[
  {
    "name": "Sportski Selo",
    "address": "Belgrade",
    "default_cost": 6000,
    "currency": "RSD",
    "status": "active"
  },
  {
    "name": "Prime Padel Club",
    "address": "Belgrade",
    "default_cost": 8000,
    "currency": "RSD",
    "status": "active"
  }
]
```

## Seed: Hosts

```json
[
  {
    "name": "Vlad",
    "telegram_username": "vlad",
    "default_fee": 2000,
    "currency": "RSD",
    "status": "active"
  },
  {
    "name": "Jean",
    "telegram_username": "jean",
    "default_fee": 0,
    "currency": "RSD",
    "status": "active"
  }
]
```

## Seed: Events

Published football event:

```json
{
  "title": "Sunday Football",
  "activity_type": "football",
  "description": "Friendly football game with host and coffee after.",
  "starts_at": "next_sunday_18_00",
  "ends_at": "next_sunday_19_30",
  "venue": "Sportski Selo",
  "host": "Vlad",
  "price": 1200,
  "currency": "RSD",
  "capacity": 18,
  "min_quorum": 10,
  "level": "mixed",
  "status": "published"
}
```

Draft padel event:

```json
{
  "title": "Evening Padel",
  "activity_type": "padel",
  "description": "Open padel game for amateurs.",
  "starts_at": "next_wednesday_20_00",
  "ends_at": "next_wednesday_21_30",
  "venue": "Prime Padel Club",
  "host": "Jean",
  "price": 1800,
  "currency": "RSD",
  "capacity": 8,
  "min_quorum": 4,
  "level": "amateur",
  "status": "draft"
}
```

## Manual test 1. Первый вход

Открыть mini app новым Telegram user.

Ожидание: создан users row, профиль не заполнен, расписание доступно.

## Manual test 2. Published visible, draft hidden

Создать одно published и одно draft событие. Открыть client schedule.

Ожидание: published видно, draft скрыт.

## Manual test 3. Первая запись с профилем

Открыть событие, нажать “Записаться”, заполнить профиль.

Ожидание: user_profiles updated, booking `booked`, attendance `unknown`, payment `unpaid`, spots_left уменьшился, событие появилось на Главной.

## Manual test 4. Duplicate booking

Попробовать записаться второй раз на то же событие.

Ожидание: ошибка `ALREADY_BOOKED`, второй booking не создан.

## Manual test 5. Cancel booking

Отменить запись.

Ожидание: booking `cancelled`, attendance `cancelled_before_event`, spots_left увеличился.

## Manual test 6. Event full

Поставить capacity = 1. User A записался, User B пытается записаться.

Ожидание: User B получает `EVENT_FULL`, spots_left = 0.

## Manual test 7. Attendance

Админ отмечает одного attended, другого no_show.

Ожидание: attendance rows updated, карточка пользователя и no-show rate обновились.

## Manual test 8. Payment

Админ отмечает пользователя paid, выбирает cashbox.

Ожидание: payment `paid`, income transaction создана, event revenue и cashbox balance обновились.

## Manual test 9. Expense

Админ добавляет venue expense.

Ожидание: expense transaction создана, event expenses и profit обновились.

## Manual test 10. Complete event

Админ отметил явку, оплаты, расходы и нажал complete.

Ожидание: event `completed`, dashboard больше не показывает его как upcoming.

## Manual test 11. Notification

Админ отправляет сообщение участникам события.

Ожидание: notification создана, recipients созданы, статусы отправки сохранены.

## Manual test 12. Survey

Админ создает опрос, пользователь отвечает.

Ожидание: survey_response создан, ответ виден в событии и карточке пользователя.

## Manual test 13. Dashboard actions

Создать событие ниже кворума, completed event без явки и unpaid participants.

Ожидание: dashboard показывает все нужные действия.

## Manual test 14. CSV export

Скачать users, events, transactions.

Ожидание: CSV открывается, колонки читаемые, данные совпадают с базой.

## Regression checks before release

- client schedule loads;
- booking works;
- admin event create works;
- participants list works;
- attendance marking works;
- payment creates transaction;
- finance summary matches transactions;
- draft events hidden from client;
- CSV export works.
