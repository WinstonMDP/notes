# notes
## ER
Note(id, name, text, create_date, modified_date, author_id)

Tag(id, name, description)

Taged(note_id, tag_id)

Link(a_note_id, b_note_id)

Person(id, login, password)

Group(id, name, description)

Member(group_id, person_id)

Permission(note_id, group_id, permission)


## CLI
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

## Права
- 0 - чтение.
- 1 - запись.
