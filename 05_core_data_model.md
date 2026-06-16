# 05. Core Data Model

## Назначение

Этот файл описывает базовые сущности первой версии PlayUp. Модель должна быть простой для MVP, но не тупиковой для будущих подписок, онлайн-оплат, клубных профилей и более глубокой аналитики.

## Главные сущности

В MVP нужны:

- User
- UserProfile
- Venue
- Host
- Event
- Booking
- Attendance
- Payment
- Transaction
- Cashbox
- Category
- Notification
- Survey
- SurveyResponse
- AdminUser

## User

Пользователь клиентской мини-апки.

Поля MVP: `id`, `telegram_id`, `telegram_username`, `created_at`, `updated_at`, `last_seen_at`, `status`.

Статусы: `new`, `active`, `sleeping`, `blocked`.

Важно: главным внешним идентификатором является `telegram_id`, а не username.

## UserProfile

Профиль и данные онбординга.

Поля MVP: `id`, `user_id`, `display_name`, `phone`, `preferred_sports`, `level`, `preferred_area`, `available_times`, `traffic_source`, `notes`, `created_at`, `updated_at`.

Обязательный минимум при первой записи: имя, уровень, интерес/спорт, район или удобная локация, источник привлечения.

## Venue

Площадка.

Поля MVP: `id`, `name`, `address`, `contact_name`, `contact_phone`, `contact_telegram`, `default_cost`, `notes`, `created_at`, `updated_at`.

## Host

Хост события.

Поля MVP: `id`, `name`, `telegram_username`, `phone`, `default_fee`, `notes`, `status`, `created_at`, `updated_at`.

В MVP хост не имеет отдельного кабинета.

## Event

Событие PlayUp.

Поля MVP: `id`, `title`, `activity_type`, `description`, `starts_at`, `ends_at`, `venue_id`, `host_id`, `price`, `capacity`, `min_quorum`, `status`, `level`, `published_at`, `cancelled_at`, `completed_at`, `created_at`, `updated_at`.

Статусы: `draft`, `published`, `cancelled`, `completed`.

## Booking

Запись пользователя на событие.

Поля MVP: `id`, `event_id`, `user_id`, `status`, `created_at`, `cancelled_at`, `cancel_reason`, `source`.

Статусы: `booked`, `cancelled`, `waitlisted`, `admin_removed`.

`waitlisted` можно заложить в модель, но не реализовывать в интерфейсе первой версии.

## Attendance

Факт посещения.

Поля MVP: `id`, `event_id`, `user_id`, `booking_id`, `status`, `marked_by_admin_id`, `marked_at`, `notes`.

Статусы: `unknown`, `attended`, `no_show`, `cancelled_before_event`.

## Payment

Ручная отметка оплаты.

Поля MVP: `id`, `event_id`, `user_id`, `booking_id`, `amount`, `status`, `cashbox_id`, `paid_at`, `marked_by_admin_id`, `notes`.

Статусы: `unpaid`, `paid`, `refunded`, `waived`.

## Transaction

Движение денег.

Поля MVP: `id`, `type`, `amount`, `currency`, `cashbox_id`, `category_id`, `event_id`, `venue_id`, `host_id`, `user_id`, `payment_id`, `description`, `transaction_date`, `created_by_admin_id`, `created_at`, `updated_at`.

Типы: `income`, `expense`, `transfer`.

`Payment` отражает факт оплаты участником. `Transaction` отражает движение денег в кассе.

## Cashbox

Касса или счет.

Поля MVP: `id`, `name`, `type`, `currency`, `is_active`, `created_at`, `updated_at`.

Типы: `cash`, `bank`, `crypto`, `other`.

## Category

Категории доходов и расходов.

Поля MVP: `id`, `name`, `type`, `parent_id`, `is_active`, `created_at`, `updated_at`.

Через `parent_id` можно сделать категории и подкатегории без отдельной таблицы.

## Notification, Survey, SurveyResponse

Notification хранит сообщение, сегмент, статус, текст, событие и дату отправки.

Survey хранит вопрос, тип ответа, привязку к событию и активность.

SurveyResponse хранит ответ пользователя на конкретный опрос.

## Что нельзя смешивать

Нельзя хранить участников массивом внутри Event — нужна Booking.

Нельзя смешивать “записался”, “пришел” и “оплатил” в одно поле — это три разных факта.

Нельзя считать Telegram username надежным ключом.

Нельзя хранить деньги только комментариями — нужна Transaction.
