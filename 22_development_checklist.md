# 22. Development Checklist

## Назначение

Практический чек-лист для старта разработки. Он помогает не расползтись и собрать рабочий MVP.

## До начала разработки

Нужно выбрать:

- где живет backend;
- ORM;
- миграции;
- деплой;
- способ авторизации админов;
- проверку Telegram init data;
- хранение env variables;
- способ отправки Telegram-сообщений.

## Рекомендуемый технический контур

Клиент: Telegram mini app на React / Next.js / Vite.

Админка: веб-приложение на Next.js или React.

Backend: FastAPI, NestJS или backend внутри Next.js, если так быстрее.

База: PostgreSQL.

ORM: Prisma, SQLAlchemy, Drizzle, TypeORM — выбрать то, что быстрее для команды.

## Репозитории

`playup-client-app`:

```text
src/
  api/
  components/
  screens/
  routes/
  types/
  utils/
```

`playup-admin-app`:

```text
src/
  api/
  auth/
  components/
  screens/
  modules/
    users/
    events/
    finance/
    notifications/
    analytics/
  types/
  utils/
```

Если backend внутри admin repo:

```text
server/
  db/
  migrations/
  modules/
    auth/
    users/
    events/
    bookings/
    finance/
    notifications/
    analytics/
  routes/
```

## Milestone 1. End-to-end booking

Готовность:

- создать User через Telegram;
- создать Event в админке;
- опубликовать Event;
- увидеть Event в клиенте;
- записаться;
- увидеть Booking в админке.

## Milestone 2. Провести событие

Готовность:

- открыть участников;
- отметить attendance;
- отметить payment;
- создать transaction;
- увидеть revenue события.

## Milestone 3. Финансы

Готовность:

- добавить cashbox;
- добавить category;
- добавить expense;
- увидеть profit;
- экспортировать CSV.

## Milestone 4. Коммуникации

Готовность:

- отправить ручное сообщение участникам;
- отправить 24h reminder;
- отправить post-event survey;
- сохранить response.

## Milestone 5. Управленческая главная

Готовность:

- upcoming events;
- quorum status;
- actions;
- revenue/expenses/profit;
- no-show;
- repeat users.

## Backend checklist

- Telegram auth;
- admin auth;
- protected routes;
- users CRUD для админки;
- client profile;
- events CRUD;
- publish/cancel/complete event;
- bookings create/cancel;
- admin participants;
- attendance bulk update;
- payments bulk update;
- income transaction on paid;
- cashboxes;
- finance categories;
- transactions;
- dashboard summary;
- CSV export;
- notifications;
- surveys.

## Client checklist

- Telegram auth loading;
- Home;
- Schedule;
- Event detail;
- Booking flow;
- Profile completion step;
- Profile screen;
- Profile edit;
- Survey response;
- Error states;
- Empty states.

## Admin checklist

- Login;
- Dashboard;
- Users list;
- User detail;
- Schedule week view;
- Events list;
- Event create/edit;
- Event detail;
- Participants table;
- Attendance marking;
- Payment marking;
- Venues;
- Hosts;
- Finance transactions;
- Cashboxes;
- Categories;
- Notifications;
- Surveys;
- Analytics;
- CSV export.

## Data quality checklist

- нет duplicate active booking;
- нет published event без capacity;
- нет paid payment без cashbox;
- нет income transaction без amount;
- нет event с ends_at раньше starts_at;
- client не видит draft events;
- revenue не считается из expected revenue.

## Testing checklist

Unit tests: spots_left, booking validation, status transitions, revenue calculation, no-show calculation.

Integration tests: Telegram auth creates user, published event appears in client API, booking appears in admin participants, payment creates transaction, CSV export returns data.

Manual QA: first open, first booking, repeat booking, cancel booking, create event, cancel event, complete event, mark attended, mark paid, add expense, dashboard updates.

## Definition of Done

Фича готова, если есть backend endpoint, UI, запись в базу, связанные данные видны в других модулях, ошибки обработаны, права проверены, есть пустые состояния и фича проходит ручной тест.

## Первый dev sprint

1. Поднять базу и миграции.
2. Создать таблицы users, profiles, venues, hosts, events, bookings.
3. Реализовать client Telegram auth.
4. Реализовать admin auth.
5. Реализовать создание event в админке.
6. Реализовать client schedule.
7. Реализовать booking.
8. Реализовать admin participants list.

После этого появится первая рабочая петля.
