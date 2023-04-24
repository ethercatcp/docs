# Сообщения
Структура сообщения<br>
Поля помеченные знаком `?` не обязательны для заполнения (зависит от ситуации)
```json5
{
    "id": "string", // Айди сообщения
    "channel_id": "string", // Айди канала
    "author": {
        "id": "string", // Айди пользователя
        "username": "string", // Ник пользователя
        "avatar": "string", // Хэш аватарки
        "discriminator": 1234, // Дискриминатор
    },
    "encryption_meta?": {
      "iv": "324/Rv0eDahUHeMDDaIY2A==", // Вектор инициализации
      "tag": "ZIxE7/fbJqyu1jLNdbHTRA==", // Тег аутентификации (Проверка целостности)
      "files?": [{
        "tag": "ZIxE7/fbJqyu1jLNdbHTRA==", // Тег аутентификации (Проверка целостности)
        "encoded_as": "base64", // В какой кодировке находится зашифрованный файл
      }] /* Если у вас есть другие файлы, то продолжайте массив. Индексы объектов должны быть идентичны индексу файла в названии при передаче через multipart */
    },
    "content": "string", // Контент сообщения (по умолчанию должен быть зашифрован)
    "reply_to?": {
        "channel_id": "string", // Айди канала
        "message_id": "string", // Айди сообщения
    },
    "acknowledged": true, // Если сообщение прочитано
    "timestamp": "string", // Время отправки сообщения
    "edited_timestamp?": "string", // Время редактирования сообщения
    "flags": 0, // BigFlags, Тип сообщения
    "attachments?": [
        {
            "filename": "string", // Имя файла
            "size": 0, // Размер файла
            "mime": "image/png", // MIME тип файла
            "url": "string", // Ссылка на файл на вашей CDN, т.е если у вас домен example.org, то ссылка будет mcp.example.org/attachment/1234567890.ext
        }
    ],
}
```
## Загрузка файлов
Файлы должны быть переданы в формате `multipart/form-data` с ключом `files[n]` и заголовком `Content-Type: multipart/form-data`.<br>
Если вы хотите отправить несколько файлов, то значение n должно быть больше на 1 `files[n]`.<br>
Максимальный размер файла - уточняйте у провайдера, об этом - [здесь](
Для передачи json используйте ключ `payload`.<br>
Пример запроса:
```
--boundary
Content-Disposition: form-data; name="payload"
Content-Type: application/json

{
    "content": "Привет, зацени фотки",
    "flags": 3
}

--boundary
Content-Disposition: form-data; name="files[0]"; filename="image.png";
Content-Type: image/png

[image bytes in base64 format]
--boundary
Content-Disposition: form-data; name="files[1]"; filename="mygif.gif";
Content-Type: image/gif

[image bytes in base64 format]
--boundary--
```
**ПРИ ЭТОМ САМИ ФАЙЛЫ ДОЛЖНЫ БЫТЬ ЗАШИФРОВАНЫ ВАМИ, ТЕМ ЖЕ САМЫМ КЛЮЧОМ И ВЕКТОРОМ КАК ВЫ ШИФРОВАЛИ СООБЩЕНИЯ**<br>
Подробнее о шифровании - [здесь](https://github.com/mooncatcp/docs/tree/main/ru/encryption/encryption.md)<br>
(Если вы не хотите шифровать файлы, то просто не добавляйте их в массив `files` в `encryption_meta`, но они все ещё должны быть переданы в кодировке base64)
## Флаги
| Значение |    Название     |                 Описание                  |
|:--------:|:---------------:|:-----------------------------------------:|
|  1 << 0  |  `TextMessage`  |            Текстовое сообщение            |
|  2 << 0  |   `Encrypted`   |          Зашифрованное сообщение          |
|  4 << 0  | `ECDHHandshake` | сообщение переносит данные ECDH хендшейка |

## HTTP Запросы

##### Получение сообщений в канале (Группа)
`GET /api/v1/channels/{channel_id}/messages`<br>
Возвращает массив сообщений в канале
Query-String параметры:

| Параметр |   Тип    |                      Описание                      |
|:--------:|:--------:|:--------------------------------------------------:|
| `before` | `string` |  Возвращает сообщения до указанного ID сообщения   |
| `after`  | `string` | Возвращает сообщения после указанного ID сообщения |
| `limit`  |  `int`   |     Максимальное количество сообщений (1-100)      |

Ошибки:
- `403` - У вас нет доступа к этому каналу
- `400`
  * Параметр `limit` указан неверно (field limit is invalid)
- `404` - Канал не найден

##### Получение конкретного сообщения (Группа)
`GET /api/v1/channels/{channel_id}/messages/{message_id}`<br>
Возвращает сообщение по его ID

Ошибки:
- `404` - Сообщение не найдено
- `403` - У вас нет доступа к этому сообщению

##### Отправка сообщения (Группа)
`POST /api/v1/channels/{channel_id}/messages`<br>
Отправляет сообщение в канал
Обязательные поля:
- `content` - Контент сообщения
- `flags` - Флаги сообщения

Ошибки:
- `400`
    * Не указан контент сообщения (field content is not specified)
    * Не указаны флаги сообщения (field flags is not specified)
    * Количество аттачментов больше разрешенного (too many attachments)
- `403` - У вас нет доступа к этому каналу
- `404` - Канал не найден
- `413` - Аттачмент слишком большой (attachment files[0] is too large)
- `415` - Неподдерживаемый тип аттачмента

##### Редактирование сообщения (Группа)
`PATCH /api/v1/channels/{channel_id}/messages/{message_id}`<br>
Редактирует контент сообщение по его ID
Обязательные поля:
- `content` - Контент сообщения, должен быть изменен

Ошибки:
- `400` - Не указан контент сообщения (field content is not specified)
- `403`
    * У вас нет доступа к этому каналу (you dont have access to this channel)
    * Вы не являетесь автором сообщения (you are not the author of this message)
- `404` - Сообщение не найдено

##### Удаление сообщения (Группа)
`DELETE /api/v1/channels/{channel_id}/messages/{message_id}`<br>
Удаляет сообщение по его ID

