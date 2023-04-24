# Ошибки
Здесь рассказано о структуре объекте ошибки.<br>
Сам код ошибки вы берете из HTTP статуса, объект ошибки переносит только описание и какие именно ошибки были допущены
```json5
{
  message: "string", // Сообщение об ошибке
  errors: ["field content is not specified", "channel not found", "etc"], // Массив ошибок
}
```

## Общие ошибки
###### `401` - Не указан токен авторизации, или он невалидный
```json5
{
  message: "Unauthorized",
  errors: ["token is not specified", "token is invalid"],
}
```
###### `429` - Вы попали под рейтлимит
```json5
{
  message: "Too Many Requests",
  errors: ["you are being rate limited"],
}
```