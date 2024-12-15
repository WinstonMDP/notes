# Пояснительная записка
## Кому нужна разрабатываемая программа, как она будет использоваться
Это программа для ведения заметок. Заметки хранятся в программе (её базе данных).
Заметки могут иметь теги и связи с другими заметками. А также существует понятие
пользователя и группы пользователей, которые имеют права на чтение и редактирование
заметок.

Может использоваться, например, it-компанией, в которой имеется несколько команд, 
которые в данной программе моделируются через группы. Также управляющие должности
могут принадлежать нескольким группам, чтобы координировать их работу. Через заметки компания
может вести базу знаний проекта.

## Функциональные требования
Система должна позволять:
1. Регистрацию новых пользователей (ввод логина и пароля).
2. Аутентификацию (вход) существующих пользователей.
3. Добавление новых заметок (с указанием имени и текста).
4. Изменение существующих заметок (только для их автора или имеющих разрешения).
5. Добавление и изменение тэгов (меток).
6. Присвоение/удаление тэгов у заметок.
7. Связывание заметок (создание ассоциативных связей между ними).
8. Создание групп пользователей, добавление/удаление участников групп.
9. Назначение группам прав доступа на заметки:
   - 0 – чтение
   - 1 – запись
10. Получение списка заметок, на которые у пользователя есть доступ.
11. Получение подробной информации о заметке.
12. Возможность выполнять вышеперечисленные операции через командный интерфейс (CLI).

Из данных функциональных требований следует набор операций, который был записан нами в формате CLI-команд
### CLI
Если где-то есть add, то подразумевается и delete.
- Регистрация: ``notes up <login> <password>``
- Вход: ``notes in <login> <password>``
- Добавление новой заметки: ``notes add <note_name> <text>``
- Изменение заметки: ``notes modify <note_name> <text>``
- Добавить тэг: ``notes tag add <name> <description>``
- Изменить тэг: ``notes tag modify <tag_name> <new_tag_name> <new_tag_description>``
- Присвоить тэг: ``notes tag <login> <tag_name>``
- Убрать тэг: ``notes untag <login> <tag_name>``
- Добавить связь: ``notes link add <a_note_name> <b_note_name>``
- Создать группу: ``notes group add <name> <description``.
  Тот, кто создал группу, является её первым членом.
- Изменить группу: ``notes group modify <group_name> <new_group_name> <new_group_description>``
- Добавить человека в группу: ``notes member add <group_name> <login>``.
  Ошибка, если тот, кто исполняет операцию, сам не находится в этой группе.
- Добавить право группе: ``notes permission add <group_name> <permission>``
  Ошибка, если у того, кто добавляет право, ни в одной его группе нет большего или равного права на этот файл.

### Права
- 0 - чтение.
- 1 - запись.

Таким образом, функциональные требования включают:
1. Управление пользователями (регистрация, вход).
2. Управление заметками (создание, редактирование, удаление).
3. Управление тегами (создание, редактирование, присвоение/снятие с заметок).
4. Управление группами (создание, редактирование, добавление участников, выдача прав).
5. Управление связями между заметками (добавление, удаление).

## Нефункциональные требования
- Безопасность: Пароли хранятся в хэширорованном виде (в схеме можно просто хранить text, но предполагается их хэширование на уровне приложения).
- Производительность: Основные операции (добавление заметок, чтение заметок) должны выполняться за O(1) или O(log n).
- Масштабируемость: Возможность добавления большого количества заметок, тегов, пользователей и групп без критического падения производительности.
- Надёжность: Использование транзакций для обеспечения целостности данных.
- Модульность: Логика приложения и БД-часть разделены.
- Расширяемость: Возможность легко добавить новые типы метаданных в будущем (например, прикрепление файлов к заметкам).

## БД
### ER
Note(name, text, create_date, modified_date, author_login)

Tag(name, description, note_name)

Link(a_note_name, b_note_name)

Person(login, password)

Group(name, description, person_login)

Permission(note_name, group_name, permission)

### Ограничения
- Name у note, tag, group уникальны.
- Login у person уникален.
- Note_name у tag может быть 0 или больше.
- Person_login у group может быть 1 или больше.
- Link - это отношение многие ко многим: n\:m.

### Причины, по которым не нормализованная БД - это плохо
Такая схема плоха, так как, например, если мы захотим
изменить имя записки, придётся изменять почти всю БД, и мы вообще можем где-нибудь
забыть поменять имя, и образуется аномалия.

### Нормализованная ER
Note(id (PK), name, text, create_date, modified_date, author_id (FK))

Tag(id (PK), name, description)

Tagged(note_id (PK, FK), tag_id (PK, FK))

Link(a_note_id (PK, FK), b_note_id (PK, FK))

Person(id (PK), login, password)

Group(id (PK), name, description)

Member(group_id (PK, FK), person_id (PK, FK))

Permission(note_id (PK, FK), group_id (PK, FK), permission)

### DDL
```sql
CREATE TABLE Person (
    id SERIAL PRIMARY KEY,
    login VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(50) NOT NULL
);

CREATE TABLE "Group" (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    description TEXT
);

CREATE TABLE Note (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    text TEXT,
    create_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    modified_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    author_id INTEGER NOT NULL,
    FOREIGN KEY (author_id) REFERENCES Person(id) ON DELETE CASCADE
);

CREATE TABLE Tag (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT
);

CREATE TABLE Tagged (
    note_id INTEGER NOT NULL,
    tag_id INTEGER NOT NULL,
    PRIMARY KEY (note_id, tag_id),
    FOREIGN KEY (note_id) REFERENCES Note(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES Tag(id) ON DELETE CASCADE
);

CREATE TABLE Link (
    a_note_id INTEGER NOT NULL,
    b_note_id INTEGER NOT NULL,
    PRIMARY KEY (a_note_id, b_note_id),
    FOREIGN KEY (a_note_id) REFERENCES Note(id) ON DELETE CASCADE,
    FOREIGN KEY (b_note_id) REFERENCES Note(id) ON DELETE CASCADE,
    CHECK (a_note_id <> b_note_id)
);

CREATE TABLE Member (
    group_id INTEGER NOT NULL,
    person_id INTEGER NOT NULL,
    PRIMARY KEY (group_id, person_id),
    FOREIGN KEY (group_id) REFERENCES "Group"(id) ON DELETE CASCADE,
    FOREIGN KEY (person_id) REFERENCES Person(id) ON DELETE CASCADE
);

CREATE TABLE Permission (
    note_id INTEGER NOT NULL,
    group_id INTEGER NOT NULL,
    permission INTEGER NOT NULL CHECK (permission IN (0, 1)),
    PRIMARY KEY (note_id, group_id),
    FOREIGN KEY (note_id) REFERENCES Note(id) ON DELETE CASCADE,
    FOREIGN KEY (group_id) REFERENCES "Group"(id) ON DELETE CASCADE
);
```

### SQL-запросы
``notes up <login> <password>``:
```sql
INSERT INTO Person (login, password)
SELECT '<login>', '<password>'
WHERE NOT EXISTS (
    SELECT 1 FROM Person WHERE login = '<login>'
);
```

``notes down <login> <password>``:
```sql
DELETE FROM Person WHERE login = '<login>';
```

``notes in <login> <password>``:
```sql
SELECT id FROM Person
WHERE login = '<login>' AND password = '<password>';
```

``notes add <note_name> <text>``:
```sql
INSERT INTO Note (name, text, author_id)
SELECT '<note_name>', '<text>', (
    SELECT id FROM Person WHERE login = '<login>'
)
WHERE NOT EXISTS (
    SELECT 1 FROM Note WHERE name = '<note_name>'
);
```

``notes delete <note_name> <text>``:
```sql
DELETE FROM Note WHERE name = '<note_name>';
```

``notes modify <note_name> <text>``:
```sql
UPDATE Note
SET text = '<text>', modified_date = CURRENT_TIMESTAMP
WHERE name = '<note_name>';
```

``notes tag add <name> <description>``:
```sql
INSERT INTO Tag (name, description)
VALUES ('<name>', '<description>')
ON CONFLICT (name) DO NOTHING;
```

``notes tag delete <name> <description>``:
```sql
DELETE FROM Tag WHERE name = '<name>';
```

``notes tag modify <tag_name> <new_tag_name> <new_tag_description>``:
```sql
UPDATE Tag
SET name = '<new_tag_name>', description = '<new_tag_description>'
WHERE name = '<tag_name>';
```

``notes tag <note_name> <tag_name>``:
```sql
INSERT INTO Tagged (note_id, tag_id)
SELECT n.id, t.id
FROM Note n
JOIN Tag t ON t.name = '<tag_name>'
WHERE n.name = '<note_name>'
  AND n.author_id = (
      SELECT id FROM Person WHERE login = '<login>'
  )
ON CONFLICT DO NOTHING;
```

``notes untag <note_name> <tag_name>``:
```sql
DELETE FROM Tagged
USING Note n, Tag t
WHERE Tagged.note_id = n.id
  AND Tagged.tag_id = t.id
  AND n.name = '<note_name>'
  AND t.name = '<tag_name>';
```

``notes link add <a_note_name> <b_note_name>``:
```sql
INSERT INTO Link (a_note_id, b_note_id)
SELECT n1.id, n2.id
FROM Note n1
JOIN Note n2 ON n1.name = '<a_note_name>' AND n2.name = '<b_note_name>'
WHERE n1.id <> n2.id
ON CONFLICT DO NOTHING;
```

``notes link delete <a_note_name> <b_note_name>``:
```sql
DELETE FROM Link
WHERE a_note_id = (
    SELECT id FROM Note WHERE name = '<a_note_name>'
)
AND b_note_id = (
    SELECT id FROM Note WHERE name = '<b_note_name>'
);
```

``notes group add <name> <description>``:
```sql
WITH new_group AS (
    INSERT INTO "Group" (name, description)
    VALUES ('<name>', '<description>')
    RETURNING id
)
INSERT INTO Member (group_id, person_id)
SELECT new_group.id, p.id
FROM new_group, Person p
WHERE p.login = '<creator_login>';
```

``notes group delete <name> <description>``:
```sql
DELETE FROM "Group" WHERE name = '<name>';
```

``notes group modify <group_name> <new_group_name> <new_group_description>``:
```sql
UPDATE "Group"
SET name = '<new_group_name>', description = '<new_group_description>'
WHERE name = '<group_name>';
```

``notes member add <group_name> <login>``:
```sql
INSERT INTO Member (group_id, person_id)
SELECT g.id, p.id
FROM "Group" g
JOIN Person p ON p.login = '<login>'
WHERE g.name = '<group_name>'
ON CONFLICT DO NOTHING;
```

``notes member delete <group_name> <login>``:
```sql
DELETE FROM Member
WHERE group_id = (
    SELECT id FROM "Group" WHERE name = '<group_name>'
)
AND person_id = (
    SELECT id FROM Person WHERE login = '<login>'
);
```

``notes permission add <note_name> <group_name> <permission>``:
```sql
INSERT INTO Permission (note_id, group_id, permission)
SELECT n.id, g.id, <permission>
FROM Note n
JOIN "Group" g ON g.name = '<group_name>'
WHERE n.name = '<note_name>'
ON CONFLICT (note_id, group_id) DO UPDATE
SET permission = <permission>;
```

``notes permission delete <note_name> <group_name> <permission>``:
```sql
DELETE FROM Permission
WHERE note_id = (
    SELECT id FROM Note WHERE name = '<note_name>'
)
AND group_id = (
    SELECT id FROM "Group" WHERE name = '<group_name>'
);
```

### Группировка запросов в транзакции

Приложение и БД спроектированы так, чтобы операции были атомарными, и выполнялись независимо друг от друга, поэтому в большинстве случаев группировки запросов в транзакции не требуется. Но при удалении пользователя может возникнуть ситуация, что он был единственным членом каких-то групп (то есть они были созданы им самим, и никого кроме него там нет). В таком случае после удаления пользователя необходимо удалить еще и группы, которые остались пустыми. Сделать это можно с помощью такой транзакции:

```sql
BEGIN;

DELETE FROM Person
WHERE login = '<login>';

DELETE FROM "Group"
WHERE id NOT IN (
    SELECT DISTINCT group_id FROM Member
);

COMMIT;
```
