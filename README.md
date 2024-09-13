# Krasyov Pavel

# 253502

---

# Тема: система управления медицинского учреждения "VitaCare"

---

## Функциональные требования

Система управления предназначена для администрирования медицинских услуг, управления расписанием врачей, больничными листами, а также для контроля проведения приёмов пациентов, выписки рецептов, выставление счетов за услуги.

Система включает роли администратора, врача и пациента.
Для доступа к системе пользователи проходят аутентификации и авторизации.

- Администратор управляет всеми данными, добавляет новых врачей, лекарства, подтверждает оплату приёмов и может просматривать журнал действий пользователей.
- Врачи после входа в систему могут управлять пациентами: проводить приёмы, составлять больничные листы, выписывать рецепты на лекарства. _Счёт будет выставляться автоматически после завершения приема, его сумма будет зависеть от базовой стоимости консультации специалиста, его категории и стоимости препаратов._
- Пациенты после входа/регистрации могут записаться на приём к врачу, оплачивать счета за приём и просматривать свою историю болезни(формируется из больничных листов пациента).

---

## Основные сущности

> Роль:  
> id - уникальный индентификатор  
> name - название роли

```
CREATE TABLE "roles" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "name" varchar(255) UNIQUE NOT NULL
);
```

> Пользователь:  
> (связь многие к одному с сущностью 'Роль')  
> id - уникальный идентификатор  
> name - имя пользователя  
> email - электронная почта пользователя для связи  
> password - пароль от аккаунта  
> role - идентификатор роли пользователя

```
CREATE TABLE "users" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "name" varchar(255) NOT NULL,
  "email" varchar(255) UNIQUE NOT NULL,
  "password" text NOT NULL,
  "role" integer NOT NULL DEFAULT -1 REFERENCES "roles" ("id") ON DELETE SET DEFAULT ON UPDATE CASCADE
);
```

> Запись действия пользователя:  
> (связь многие к одному с сущностью 'Пользователь')  
> id - уникальный идентификатор  
> type - тип действия (Вход в аккаунт, Запись на приём, и т.д.)  
> description - описание действия  
> user_id - идентификатор пользователя

```
CREATE TABLE "log_actions" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "type" varchar(255) NOT NULL,
  "description" text NOT NULL,
  "user_id" integer NOT NULL REFERENCES "users" ("id") ON DELETE CASCADE ON UPDATE CASCADE
);
```

> Пациент:  
> (связь один к одному с сущностью 'Пользователь',  
> связь многие ко многим с сущностью 'Доктор' через сущность 'Приём')  
> id - уникальный идентификатор  
> birhday - день рождения пациента  
> gender - пол пациента  
> user_id - идентификатор пользователя

```
CREATE TABLE "patients" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "birthday" date NOT NULL,
  "gender" varchar(1) NOT NULL,
  "user_id" integer NOT NULL REFERENCES "users" ("id") ON DELETE CASCADE ON UPDATE CASCADE
);
```

> Доктор:  
> (связь один к одному с сущностью 'Пользователь',  
> связь многие к одному с сущностью 'Специализация врача',  
> связь многие ко многим с сущностью 'Пациент' через сущность 'Приём')  
> id - уникальный идентификатор  
> qualification - категория врача (2-я, 1-я, высшая)  
> user_id - идентификатор пользователя  
> specialization_id - идентификатор специализации врача

```
CREATE TABLE "doctors" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "qualification" integer NOT NULL DEFAULT 2,
  "user_id" integer NOT NULL REFERENCES "users" ("id") ON DELETE CASCADE ON UPDATE CASCADE,
  "specialization_id" integer NOT NULL DEFAULT -1 REFERENCES "specializations" ("id") ON DELETE SET DEFAULT ON UPDATE CASCADE
);
```

> Специализация врача:  
> id - уникальный идентификатор  
> name - название специализации  
> description - описание специализации  
> service_fee - плата за приём

```
CREATE TABLE "specializations" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "name" varchar(255) UNIQUE NOT NULL,
  "description" text NOT NULL,
  "service_fee" decimal(19,2) NOT NULL DEFAULT 0
);
```

> Лекарство:  
> id - уникальный идентификатор  
> name - название лекарства  
> description - описание лекарства  
> price - цена лекарства

```
CREATE TABLE "medications" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "name" varchar(255) UNIQUE NOT NULL,
  "description" text NOT NULL,
  "price" decimal(19,2) NOT NULL DEFAULT 0
);
```

> Рецепт:  
> (связь многие к одному с сущностью 'Лекарство',  
> связь многие к одному с сущностью 'Больничный лист')  
> id - уникальный идентификатор  
> dosage - доза лекарства в мг  
> issue_date - дата выписки рецепта  
> medication_id - идентификатор лекарства  
> medical_record_id - идентификатор больничного листа

```
CREATE TABLE "prescriptions" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "dosage" decimal(6,2) NOT NULL,
  "issue_date" date NOT NULL DEFAULT (current_date),
  "medication_id" integer NOT NULL DEFAULT -1 REFERENCES "medications" ("id") ON DELETE SET DEFAULT ON UPDATE CASCADE,
  "medical_record_id" integer NOT NULL DEFAULT -1 REFERENCES "medical_records" ("id") ON DELETE CASCADE ON UPDATE CASCADE
);
```

> Больничный лист:  
> (связь многие к одному с сущностью 'Пациент',  
> связь многие к одному с сущностью 'Доктор')  
> id - уникальный идентификатор  
> diagnosis - диагноз пациента  
> conclusion_date - дата заключения  
> doctor_id - идентификатор врача  
> patient_id - идентификатор пациента

```
CREATE TABLE "medical_records" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "diagnosis" text NOT NULL,
  "conclusion_date" date NOT NULL DEFAULT (current_date),
  "doctor_id" integer NOT NULL DEFAULT -1 REFERENCES "doctors" ("id") ON DELETE SET DEFAULT ON UPDATE CASCADE,
  "patient_id" integer NOT NULL DEFAULT -1 REFERENCES "patients" ("id") ON DELETE CASCADE ON UPDATE CASCADE
);
```

> Приём:  
> (связь многие к одному с сущностью 'Пациент',  
> связь многие к одному с сущностью 'Доктор')  
> id - уникальный идентификатор  
> date - дата и время приёма  
> doctor_id - идентификатор врача  
> patient_id - идентификатор пациента

```
CREATE TABLE "appointments" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "date" timestamp NOT NULL,
  "status" varchar(1) NOT NULL DEFAULT '0',
  "doctor_id" integer NOT NULL DEFAULT -1 REFERENCES "doctors" ("id") ON DELETE CASCADE ON UPDATE CASCADE,
  "patient_id" integer NOT NULL DEFAULT -1 REFERENCES "patients" ("id") ON DELETE CASCADE ON UPDATE CASCADE
);
```

> График работы врача:  
> (связь один к одному с сущностью 'Доктор')  
> id - уникальный идентификатор  
> skip_days - выходные дни  
> work_time - время работы  
> doctor_id - идентификатор врача

```
CREATE TABLE "doctors_schedules" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "skip_days" varchar(255) NOT NULL DEFAULT '6,7',
  "work_time" varchar(255) NOT NULL DEFAULT '8-16',
  "doctor_id" integer NOT NULL REFERENCES "doctors" ("id") ON DELETE CASCADE ON UPDATE CASCADE
);
```

> Счёт за приём:  
> (связь многие к одному с сущностью 'Пациент')  
> id - уникальный идентификатор  
> value - сумма счёта  
> invoice_date - дата выставления счёта  
> payment_status - статус оплаты (не оплачен, ожидает подтверждения, оплачен)  
> patient_id - идентификатор пациента

```
CREATE TABLE "bills" (
  "id" INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  "value" decimal(19,2) NOT NULL,
  "invoice_date" date NOT NULL DEFAULT (current_date),
  "payment_status" varchar(1) NOT NULL DEFAULT '0',
  "patient_id" integer NOT NULL REFERENCES "patients" ("id") ON DELETE CASCADE ON UPDATE CASCADE
);
```

---

## Схема базы данных

![Database.](/VitaCare.png "Database schema")
