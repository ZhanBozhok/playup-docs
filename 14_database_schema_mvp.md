# 14. Database Schema MVP

## Назначение

Этот файл описывает стартовую схему PostgreSQL. Это не финальная миграция один-в-один, но достаточно точная база для разработки.

## Общие правила

Все таблицы имеют `id uuid primary key`, `created_at`, `updated_at`. Все критичные связи задаются foreign key. Ключевые сущности не удаляются физически, а переводятся в статусы.

Подключить расширение:

```sql
create extension if not exists "pgcrypto";
```

## admin_users

```sql
create table admin_users (
  id uuid primary key default gen_random_uuid(),
  email text not null unique,
  password_hash text,
  name text not null,
  role text not null default 'admin',
  status text not null default 'active',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

Роли: `owner`, `admin`, `viewer`. Для MVP достаточно `owner` и `admin`.

## users

```sql
create table users (
  id uuid primary key default gen_random_uuid(),
  telegram_id text unique,
  telegram_username text,
  status text not null default 'new',
  first_seen_at timestamptz not null default now(),
  last_seen_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index idx_users_telegram_username on users (telegram_username);
create index idx_users_status on users (status);
create index idx_users_created_at on users (created_at);
```

`telegram_id` — главный внешний идентификатор. `telegram_username` не считается надежным ключом.

## user_profiles

```sql
create table user_profiles (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null unique references users(id),
  display_name text,
  phone text,
  preferred_sports text[] not null default '{}',
  level text,
  preferred_area text,
  available_times text[] not null default '{}',
  traffic_source text,
  notes text,
  profile_completed_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

## venues

```sql
create table venues (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  address text,
  map_url text,
  contact_name text,
  contact_phone text,
  contact_telegram text,
  default_cost numeric(12,2),
  currency text not null default 'RSD',
  status text not null default 'active',
  notes text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

## hosts

```sql
create table hosts (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  telegram_username text,
  phone text,
  default_fee numeric(12,2),
  currency text not null default 'RSD',
  status text not null default 'active',
  notes text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

## events

```sql
create table events (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  activity_type text not null,
  description text,
  starts_at timestamptz not null,
  ends_at timestamptz not null,
  venue_id uuid references venues(id),
  host_id uuid references hosts(id),
  price numeric(12,2) not null default 0,
  currency text not null default 'RSD',
  capacity integer not null,
  min_quorum integer,
  level text,
  status text not null default 'draft',
  published_at timestamptz,
  cancelled_at timestamptz,
  cancellation_reason text,
  completed_at timestamptz,
  internal_notes text,
  created_by_admin_id uuid references admin_users(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint events_capacity_positive check (capacity > 0),
  constraint events_time_valid check (ends_at > starts_at),
  constraint events_quorum_valid check (min_quorum is null or min_quorum <= capacity)
);

create index idx_events_starts_at on events (starts_at);
create index idx_events_status on events (status);
create index idx_events_activity_type on events (activity_type);
create index idx_events_venue_id on events (venue_id);
create index idx_events_host_id on events (host_id);
```

## bookings

```sql
create table bookings (
  id uuid primary key default gen_random_uuid(),
  event_id uuid not null references events(id),
  user_id uuid not null references users(id),
  status text not null default 'booked',
  source text not null default 'client_app',
  cancelled_at timestamptz,
  cancel_reason text,
  created_by_admin_id uuid references admin_users(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create unique index uniq_active_booking_per_event_user
on bookings(event_id, user_id)
where status = 'booked';

create index idx_bookings_event_id on bookings (event_id);
create index idx_bookings_user_id on bookings (user_id);
create index idx_bookings_status on bookings (status);
```

## attendance

```sql
create table attendance (
  id uuid primary key default gen_random_uuid(),
  event_id uuid not null references events(id),
  user_id uuid not null references users(id),
  booking_id uuid references bookings(id),
  status text not null default 'unknown',
  marked_by_admin_id uuid references admin_users(id),
  marked_at timestamptz,
  notes text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint uniq_attendance_event_user unique (event_id, user_id)
);
```

## cashboxes

```sql
create table cashboxes (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  type text not null,
  currency text not null default 'RSD',
  is_active boolean not null default true,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

## finance_categories

```sql
create table finance_categories (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  type text not null,
  parent_id uuid references finance_categories(id),
  is_active boolean not null default true,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

## payments

```sql
create table payments (
  id uuid primary key default gen_random_uuid(),
  event_id uuid not null references events(id),
  user_id uuid not null references users(id),
  booking_id uuid references bookings(id),
  amount numeric(12,2) not null default 0,
  currency text not null default 'RSD',
  status text not null default 'unpaid',
  cashbox_id uuid references cashboxes(id),
  paid_at timestamptz,
  marked_by_admin_id uuid references admin_users(id),
  notes text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint uniq_payment_event_user unique (event_id, user_id)
);
```

## transactions

```sql
create table transactions (
  id uuid primary key default gen_random_uuid(),
  type text not null,
  amount numeric(12,2) not null,
  currency text not null default 'RSD',
  cashbox_id uuid references cashboxes(id),
  category_id uuid references finance_categories(id),
  event_id uuid references events(id),
  venue_id uuid references venues(id),
  host_id uuid references hosts(id),
  user_id uuid references users(id),
  payment_id uuid references payments(id),
  description text,
  transaction_date date not null default current_date,
  created_by_admin_id uuid references admin_users(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint transactions_amount_not_zero check (amount <> 0)
);

create index idx_transactions_type on transactions (type);
create index idx_transactions_date on transactions (transaction_date);
create index idx_transactions_cashbox_id on transactions (cashbox_id);
create index idx_transactions_event_id on transactions (event_id);
```

## notifications

```sql
create table notifications (
  id uuid primary key default gen_random_uuid(),
  type text not null,
  status text not null default 'draft',
  target_type text not null,
  target_filter jsonb,
  event_id uuid references events(id),
  title text,
  message text not null,
  scheduled_at timestamptz,
  sent_at timestamptz,
  created_by_admin_id uuid references admin_users(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

## notification_recipients

```sql
create table notification_recipients (
  id uuid primary key default gen_random_uuid(),
  notification_id uuid not null references notifications(id),
  user_id uuid not null references users(id),
  status text not null default 'pending',
  sent_at timestamptz,
  failed_at timestamptz,
  error_message text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  constraint uniq_notification_user unique (notification_id, user_id)
);
```

## surveys и survey_responses

```sql
create table surveys (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  question text not null,
  answer_type text not null,
  event_id uuid references events(id),
  options jsonb,
  is_active boolean not null default true,
  created_by_admin_id uuid references admin_users(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table survey_responses (
  id uuid primary key default gen_random_uuid(),
  survey_id uuid not null references surveys(id),
  user_id uuid not null references users(id),
  event_id uuid references events(id),
  answer_value numeric(12,2),
  answer_text text,
  created_at timestamptz not null default now(),
  constraint uniq_survey_response_user unique (survey_id, user_id)
);
```

## admin_notes

```sql
create table admin_notes (
  id uuid primary key default gen_random_uuid(),
  entity_type text not null,
  entity_id uuid not null,
  note text not null,
  created_by_admin_id uuid references admin_users(id),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

## Seed MVP

Создать минимум: кассы `Cash RSD`, `Bank RSD`; категории доходов `Event payment`, `Corporate`, `Other income`; категории расходов `Venue`, `Host`, `Water`, `Equipment`, `Ads`, `Content`, `Software`, `Refund`, `Other expense`; площадки `Sportski Selo`, `Prime Padel Club`; хосты `Jean`, `Vlad`.

## Важные запреты

Нельзя хранить участников массивом внутри `events`. Нельзя считать выручку через `capacity * price`. Нельзя считать Telegram username уникальным ключом. Нельзя смешивать запись, явку и оплату в одно поле.
