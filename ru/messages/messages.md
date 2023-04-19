# Сообщения
Структура сообщения
```json5
{
    "id": "string", // Айди сообщения
    "channel_id": "string", // Айди канала
    "author": {
        "id": "string", // Айди пользователя
        "username": "string", // Ник пользователя
        "avatar": "string", // Хэш аватарки
    },
    "encryption_meta": {
      "iv": "324/Rv0eDahUHeMDDaIY2A==", // Вектор инициализации
      "tag": "ZIxE7/fbJqyu1jLNdbHTRA==", // Тег аутентификации (Проверка целостности)
      "files": [{
        "tag": "ZIxE7/fbJqyu1jLNdbHTRA==", // Тег аутентификации (Проверка целостности)
        "encoded_as": "base64", // В какой кодировке находится зашифрованный файл
      }] /* Если у вас есть другие файлы, то продолжайте массив. Индексы объектов должны быть идентичны индексу файла в названии при передаче через multipart */
    },
    "content": "string", // Контент сообщения (по умолчанию должен быть зашифрован)
    "reply_to": {
        "channel_id": "string", // Айди канала
        "message_id": "string", // Айди сообщения
    },
    "acknowledged": true, // Если сообщение прочитано
    "timestamp": "string", // Время отправки сообщения
    "edited_timestamp": "string", // Время редактирования сообщения
    "flags": 0, // BigFlags, Тип сообщения
    "attachments": [
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
Подробнее о шифровании - [здесь](https://github.com/mooncatcp/docs/tree/main/ru/encryption/encryption.md)
## Флаги
| Значение |    Название     |                 Описание                  |
|:--------:|:---------------:|:-----------------------------------------:|
|  1 << 0  |  `TextMessage`  |            Текстовое сообщение            |
|  2 << 0  |   `Encrypted`   |          Зашифрованное сообщение          |
|  4 << 0  | `ECDHHandshake` | сообщение переносит данные ECDH хендшейка |

