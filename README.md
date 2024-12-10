# Пояснительная записка
## Кому нужна разрабатываемая программа, как она будет использоваться
Это программа для ведения заметок. Заметки хранятся в программе (её базе данных).
Заметки могут иметь теги и связи с другими заметками. А также существует понятие
пользователя и группы пользователей, которые имеют права на чтение и редактирование
заметок.

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

## БД
### ER
Note(name, text, create_date, modified_date, author_login)

Tag(name, description, note_name)

Link(a_note_name, b_note_name)

Person(login, password)

Group(name, description, person_login)

Permission(note_name, group_name, permission)

Это не нормализованная БД. Такая схема плоха, так как, например, если мы захотим
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

### Ограничения
- Name у note, tag, group уникальны.
- Login у person уникален.