# Ревью итераций 0-5

Дата: 2026-06-16.

Scope: `playup-admin-app` и `playup-client-app`, итерации `0-5` из `26_build_roadmap.md`.
Незакоммиченные изменения по итерации 6 в admin-репо не ревьюились как часть scope.

Проверено:

- сверка с `13`, `17`, `19`, `21`, `22`, `23`;
- статический проход по API, Prisma-схеме, бизнес-логике, admin/client UI;
- `npm run typecheck` в обоих app-репозиториях;
- `npm run build` в обоих app-репозиториях.

Результат по сборке: оба приложения собираются и проходят typecheck.

## Findings

### 1. Повторная запись после paid-cancel ломает Payment/Transaction

Severity: high.

Где:

- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/bookings.ts:131`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/bookings.ts:133`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:84`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:112`

Сценарий:

1. Пользователь записался.
2. Админ отметил оплату `paid`, создалась income `Transaction`.
3. Пользователь отменил запись. Это ок: возврат не автоматический по `17`.
4. Пользователь снова записался на то же событие.

Что происходит: `createBooking` делает `payment.upsert` и переводит существующий Payment обратно в `unpaid`, но связанную income-транзакцию не удаляет и не корректирует. Логика согласования Payment/Transaction живет только в `setPaymentsBulk`, а booking-flow ее обходит.

Итог: в карточке участника/пользователя оплата может быть `unpaid`, а revenue события и финансы продолжают учитывать старую income-транзакцию. Это прямой рассинхрон Payment и Transaction.

### 2. Attendance/Payment можно создать для пользователя без Booking

Severity: high.

Где:

- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:25`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:28`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:60`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:81`

`setAttendanceBulk` и `setPaymentsBulk` принимают произвольный `user_id` и делают `upsert` по `(eventId, userId)`, не проверяя, что у пользователя есть активный Booking на это событие. При `create` также не проставляется `bookingId`.

Итог: API может создать явку, оплату и даже доходную транзакцию для человека, который не является участником события через Booking. Это нарушает инвариант: участники события только через Booking.

### 3. `paid` Payment допускается без cashbox

Severity: high.

Где:

- `/Users/zhanbozhok/Desktop/playup-admin-app/src/app/api/admin/events/[id]/payments/bulk/route.ts:12`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/app/api/admin/events/[id]/payments/bulk/route.ts:14`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:66`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:99`

Контракт и правила говорят, что при `paid` нужны amount, currency, `cashbox_id`, `paid_at` (`19`, data-quality из `22`). Сейчас endpoint принимает любой string-статус и nullable/optional `cashbox_id`; бизнес-логика спокойно создает `paid` Payment и income Transaction с `cashboxId = null`.

Итог: можно получить `paid payment without cashbox`, что прямо запрещено data-quality checklist.

### 4. Записанный клиент не видит отмененное событие

Severity: medium.

Где:

- `/Users/zhanbozhok/Desktop/playup-admin-app/src/app/api/client/events/[id]/route.ts:12`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/app/api/client/home/route.ts:18`

`17` Flow 7 говорит: клиент видит отмену, если был записан. Сейчас клиентская карточка события возвращает 404 для всего, что не `published`, а Home показывает только booked-записи на `published` события.

Итог: после отмены события записанный пользователь не увидит это событие ни на Главной, ни в карточке. В client UI даже есть ветка для `event.status === "cancelled"`, но backend не дает ей сработать.

### 5. Refund стирает income вместо корректирующего движения

Severity: medium.

Где:

- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:110`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/payments.ts:112`

В `08` указано, что при `refunded` в финансах должна появиться корректирующая транзакция: отрицательный income или expense с категорией refund. Сейчас при переводе оплаты из `paid` в `refunded` связанная income-транзакция удаляется.

Итоговые суммы становятся правильными, но история движения денег теряется. Для простого MVP это не ломает profit, но противоречит описанному поведению refund и делает аудит возвратов слабым.

### 6. Capacity можно переполнить при конкурентных записях

Severity: medium.

Где:

- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/bookings.ts:117`
- `/Users/zhanbozhok/Desktop/playup-admin-app/src/lib/bookings.ts:120`

Проверка мест делается через `count(booked)` внутри транзакции, но без блокировки события/слота или serializable isolation. Partial unique защищает только от дубля одного пользователя, но не от двух разных пользователей, которые одновременно видят последнее свободное место.

Итог: ручной тест `EVENT_FULL` проходит последовательно, но при реальной одновременной записи можно получить `booked_count > capacity`.

## Low-priority notes

- `requireAdmin` отсекает только роль `client`; если когда-нибудь появится admin user с ролью `viewer`, backend даст ему write-доступ. В `18` viewer заложен на будущее, поэтому сейчас это не blocker, но лучше не забыть перед добавлением viewer.
- `POST /api/admin/transactions` уже принимает `type = transfer`, хотя UI transfer не реализует, а баланс касс его игнорирует. Пока UI не дает создать transfer, риск низкий.
