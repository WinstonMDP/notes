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

Taged(note_id (PK, FK), tag_id (PK, FK))

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
    password VARCHAR(255) NOT NULL
);

CREATE TABLE "Group" (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT
);

CREATE TABLE Note (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    text TEXT NOT NULL,
    create_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    modified_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    author_id INTEGER NOT NULL,
    FOREIGN KEY (author_id) REFERENCES Person(id) ON DELETE CASCADE
);

CREATE TABLE Tag (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT
);

CREATE TABLE Member (
    group_id INTEGER NOT NULL,
    person_id INTEGER NOT NULL,
    PRIMARY KEY (group_id, person_id),
    FOREIGN KEY (group_id) REFERENCES "Group"(id) ON DELETE CASCADE,
    FOREIGN KEY (person_id) REFERENCES Person(id) ON DELETE CASCADE
);

CREATE TABLE Taged (
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
    CHECK (a_note_id <> b_note_id) -- Предотвращает связь заметки с самой собой
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
