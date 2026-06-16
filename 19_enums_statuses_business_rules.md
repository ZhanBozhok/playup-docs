# 19. Enums, Statuses, Business Rules

## Назначение

Этот файл фиксирует статусы, справочники и бизнес-правила MVP. Frontend, backend и аналитика должны использовать одни значения.

## Event status

`draft` — создано, не видно клиентам.

`published` — опубликовано и доступно для записи.

`cancelled` — отменено.

`completed` — проведено и закрыто.

Событие можно опубликовать, если заполнены title, activity_type, starts_at, ends_at, venue_id, price, currency, capacity, min_quorum. `ends_at` должен быть позже `starts_at`. `min_quorum` не должен быть больше `capacity`.

## Activity type

Стартовые значения:

- `football`
- `padel`
- `run`
- `yoga`
- `recovery`
- `social`
- `other`

На уровне компании PlayUp не равен одному спорту, но каждое событие имеет activity_type.

## Level

- `beginner`
- `amateur`
- `confident`
- `advanced`
- `mixed`

Для первых событий чаще всего использовать `mixed`.

## Booking status

`booked` — пользователь записан и занимает место.

`cancelled` — пользователь отменил запись.

`waitlisted` — лист ожидания, в MVP можно заложить, но не показывать.

`admin_removed` — админ удалил участника.

Один пользователь не может иметь две активные записи `booked` на одно событие.

## Attendance status

`unknown` — явка еще не отмечена.

`attended` — пользователь пришел.

`no_show` — пользователь был записан, но не пришел.

`cancelled_before_event` — пользователь отменил до события.

До события статус `unknown`. После события админ меняет его вручную.

## Payment status

`unpaid` — оплата не получена.

`paid` — оплата получена.

`refunded` — деньги возвращены.

`waived` — оплата списана или участие бесплатное.

Оплата не равна посещению. При `paid` нужны amount, currency, cashbox_id, paid_at.

## Transaction type

`income` — деньги пришли.

`expense` — деньги ушли.

`transfer` — деньги переложили между кассами. Можно не реализовывать в UI MVP.

## Cashbox type

- `cash`
- `bank`
- `crypto`
- `other`

## User status

`new` — создан, но еще не посещал события.

`active` — посещал событие за последние 30 дней.

`sleeping` — был на событии, но не посещал 30+ дней.

`blocked` — заблокирован вручную.

## Traffic source

- `instagram`
- `threads`
- `telegram_chat`
- `friend`
- `host_invite`
- `ads`
- `offline`
- `other`

## Notification type

- `manual`
- `event_reminder_24h`
- `event_reminder_3h`
- `post_event_survey`
- `reactivation`

В MVP достаточно первых четырех.

## Notification status

- `draft`
- `scheduled`
- `sending`
- `sent`
- `failed`
- `cancelled`

## Notification target_type

- `all_users`
- `event_participants`
- `event_attended`
- `event_no_show`
- `new_users`
- `active_users`
- `sleeping_users`
- `activity_type`
- `traffic_source`

## Survey answer_type

- `rating_1_5`
- `rating_1_10`
- `text`
- `single_choice`

## Risk status for events

`below_quorum`: booked_count < min_quorum.

`ok`: booked_count >= min_quorum and spots_left > 2.

`almost_full`: spots_left is 1 or 2.

`full`: spots_left = 0.

`needs_closing`: event ended but status is not completed.

## Финансовые правила

При отметке Payment `paid` создается income transaction.

При добавлении расхода создается expense transaction.

Баланс кассы = income - expense, пока transfer не реализован.

## Аналитические правила

Booked count считается только по bookings `booked`.

Attendance считается только по attendance `attended`.

No-show считается только по attendance `no_show`.

Revenue считается по transactions `income`, а не по booked_count * price.

Profit = income - expense.

Repeat user — пользователь с 2+ attended events.

## Важные запреты

Нельзя считать выручку по capacity * price. Нельзя считать пришедших по оплатившим. Нельзя показывать draft event клиенту. Нельзя считать пользователя уникальным по Telegram username.
