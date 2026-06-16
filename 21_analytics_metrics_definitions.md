# 21. Analytics Metrics Definitions

## Назначение

Этот файл фиксирует определения метрик. Разработчики, админка и аналитика должны считать одинаково.

## Общие правила

Все метрики считаются по серверным данным. В MVP не считаем клики и просмотры экранов. Выручка считается только по transaction, не по ожиданиям.

## Users

### New users

Количество пользователей, созданных за период.

`count(users.id where created_at in period)`

### Profile completed users

Пользователи, у которых `profile_completed_at is not null`.

### Active users 30d

Пользователи с хотя бы одним `attended` событием за последние 30 дней.

### Sleeping users

Пользователи, у которых был хотя бы один attended event, но нет посещений за последние 30 дней.

### First-time attendees

Пользователи, у которых первое attended-событие попало в выбранный период.

### Repeat users

Пользователи, у которых attended events count >= 2.

## Events

### Booked count

Количество bookings со статусом `booked`.

### Spots left

`capacity - booked_count`

### Fill rate

`booked_count / capacity`

### Quorum reached

`booked_count >= min_quorum`

## Attendance

### Attended count

Количество attendance со статусом `attended`.

### No-show count

Количество attendance со статусом `no_show`.

### Attendance rate

`attended_count / booked_count`

### No-show rate

`no_show_count / booked_count`

Denominator — активные записи, не capacity.

## Payments

### Paid users count

Количество payments со статусом `paid`.

### Payment rate

`paid_users_count / booked_count`

### Unpaid amount

Сумма expected payment по unpaid участникам. Это операционная оценка, не долг в бухгалтерском смысле.

## Finance

### Revenue

`sum(amount where transactions.type = 'income')`

### Expenses

`sum(amount where transactions.type = 'expense')`

### Profit

`revenue - expenses`

### Event revenue

Сумма income transactions, связанных с event_id.

### Event expenses

Сумма expense transactions, связанных с event_id.

### Event gross profit

`event_revenue - event_expenses`

### Average check

`revenue / paid_users_count`

### Cashbox balance

`sum(income) - sum(expense)` по cashbox, пока transfer не реализован.

## Retention

### First visit cohort

Пользователи, у которых первое attended-событие было в выбранном событии или периоде.

### Second visit rate

`users_with_2plus_attended / first_time_attendees`

### Third visit rate

`users_with_3plus_attended / first_time_attendees`

### Weekly active attendees

Уникальные пользователи с attended event в выбранную неделю.

### Current week streak

Количество последовательных недель, в которых пользователь имел хотя бы одно attended event.

## Sources

### Users by source

Количество пользователей по `traffic_source`.

### First attendees by source

Первые пришедшие пользователи по `traffic_source`.

### Revenue by source

Income transactions через user_id и user_profiles.traffic_source.

## Activity analytics

Revenue, attendance, fill rate и profit можно группировать по `events.activity_type`.

## Venue analytics

Считать events, revenue, expenses и profit по venue.

## Host analytics

Считать events, attendance, revenue и среднюю оценку, если есть host survey.

## Dashboard MVP

Главная показывает:

- upcoming events;
- booked / capacity;
- quorum status;
- expected revenue;
- revenue week;
- expenses week;
- profit week;
- new users week;
- first-time attendees week;
- repeat attendees week;
- no-show rate week.

## Expected revenue

`booked_count * event.price`

Это только ожидаемая выручка. В UI не называть ее фактической выручкой.

## Actual revenue

Только transactions type `income`.

## Не считаем в MVP

Не считаем LTV, CAC из рекламных кабинетов, ROAS, клики, просмотры событий, view-to-booking conversion, churn prediction, сложные retention triangles, seasonality, cohort heatmaps.

## Предупреждения качества данных

Если явка не отмечена — attendance analytics неполная. Если оплаты не отмечены — revenue analytics неполная. Если расходы не внесены — profit analytics неполная. Поэтому на главной нужны действия по закрытию событий.
