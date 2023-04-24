# Группа
Структура группы:
```json5
{
  id: "string", // Айди группы
  name: "string", // Название группы
  icon: "string", // Хэш иконки группы
  channels: [Channel], // Массив каналов группы
  roles: [Role], // Массив ролей группы
  members: [Member], // Массив участников группы
  owner_id: "string", // Айди владельца группы
  emotes: [Emote], // Массив эмоций группы
  stickers: [Sticker] // Массив стикеров группы
}
```