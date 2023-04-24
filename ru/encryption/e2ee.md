# End 2 End Encryption
Если на русском - Женя и Саша обмениваются публичными ключами, вычисляют приватный ключ (в случае с ECDH) и шифруют сообщение с помощью AES-256. После чего они отправляют сообщение в сеть. Каждый участник сети может прочитать сообщение, но не может его расшифровать, так как не имеет приватного ключа. Таким образом, сообщение зашифровано от всех, кроме отправителя и получателя, даже от сервера. В базе данных хранится зашифрованный текст.

## Как это работает в MCP
Для личных сообщений используется кривая secp256k1, в случае групп - шифрования нету.<br>
Шифруются такие данные:
- Контент сообщения
- Все аттачменты (картинки, файлы, и т.д)

## Личные сообщения
Вы нашли человека с которым хотите общаться, создаете ключи в кривой secp256k1, сохраняете приватный ключ локально.<br>
Предположим что у нас есть человек - Саша, его ID - `174473846617255485`.<br>
Для начала отправляем незашифрованное сообщение с вашим публичным ключем, алгоритмом и кодировкой, в итоге должен получиться такой запрос<br>
`POST /api/users/174473846617255485/messages`
```json5
{
  content: "ECDH:base64:BBB5nYtgL4ueHvYxJUo6CsebjZQlU75G0ra8qSlpFHQ4bIFFwiV3Tq+6gea8V+nxH9q5y3CTzelsxcw1KsI8vHw=",
  /*
      В случае использования 16-ричной кодировки:
      "content": "ECDH:hex:0410799d8b602f8b9e1ef631254a3a0ac79b8d942553be4..."
  */
  flags: 5 // Подробнее в папке messages, раздел Флаги.
}
```
Ожидаем ответа от Саши, допустим мы получили такой ответ: `ECDH:base64:BCNdoDSve65KemD1sjw+83sphLZYGZNs5Q8NLOS7PWZRB3+HjHwP+oQlJzZ47eSgSfM/htwr6LVWxYJGYibPJTM=`<br>
Теперь нам нужно вычислить секретный ключ, для этого мы делаем:
```js
// Код Node.js
const crypto = require('crypto');

const me = crypto.createECDH('secp256k1');
me.generateKeys();

const receivedPublicKey = 'BCNdoDSve65KemD1sjw+83sphLZYGZNs5Q8NLOS7PWZRB3+HjHwP+oQlJzZ47eSgSfM/htwr6LVWxYJGYibPJTM=';
const secret = me.computeSecret(receivedPublicKey, 'base64');
/* Используем ^^^ для шифрования И СОХРАНЯЕМ ЭТОТ КЛЮЧ ЛОКАЛЬНО (Как именно - забота клиента, но главное чтобы ключ был сохранен, потеря ключа = потеря переписки) */

/* Сам алгоритм шифрования */
const iv = crypto.randomBytes(16);
const cipher = crypto.createCipheriv('aes-256-gcm', secret);
const encrypted = Buffer.concat([cipher.update('Привет!'), cipher.final()]);
const tag = cipher.getAuthTag();
/*
    encrypted - зашифрованный текст
    iv - вектор инициализации
    tag - тег аутентификации (Проверка целостности)
    
    Все это нужно отправить в запросе в формате base64
 */
```
Теперь отправляем зашифрованное сообщение:<br>
`POST /api/users/174473846617255485/messages`
```json5
{
  content: "zyof5D8RuyOk6YGdbQ==", // Зашифрованный текст "Привет!"
  encryption_meta: {
    iv: "324/Rv0eDahUHeMDDaIY2A==", // Вектор инициализации
    tag: "ZIxE7/fbJqyu1jLNdbHTRA==", // Тег аутентификации (Проверка целостности)
  },
  flags: 3 // Подробнее в папке messages, раздел Флаги.
}
```

## Шифрование файлов
Для шифрования файлов используется AES-256-GCM, вектор инициализации берется от сообщения.<br>
Для шифрования используется секретный ключ, который мы получили в прошлом разделе.<br>
После шифрования файла, он отправляется на сервер, где хранится в зашифрованном виде.<br>
Пример кода для шифрования файла:
```js
// Код Node.js
const fs = require('fs');
const crypto = require('crypto');

function encryptFile(filename) {
    return new Promise((resolve) => {
        const secret = new Buffer() /* Секретный ключ */;
        const iv = crypto.randomBytes(16); /* Вектор инициализации, ДОЛЖЕН БЫТЬ ИДЕНТИЧЕН ТОМУ ВЕКТОРУ, КОТОРЫМ ШИФРОВАЛОСЬ СООБЩЕНИЕ */
        const cipher = crypto.createCipheriv('aes-256-gcm', secret, iv);

        const readStream = fs.createReadStream(filename);
        let encryptedData = '';
        readStream.on('data', (chunk) => encryptedData += cipher.update(chunk, 'utf8', 'base64'));
        readStream.on('end', () => {
            encryptedData += cipher.final('base64');
            resolve({
                tag: cipher.getAuthTag(),
                data: encryptedData
            });
        });
    });
}

encryptFile('test.png').then((encrypted) => {
    const jsonPayload = JSON.stringify({
        content: "zyof5D8RuyOk6YGdbQ==", // Зашифрованный текст "Привет!"
        encryption_meta: {
            iv: "324/Rv0eDahUHeMDDaIY2A==", // Вектор инициализации
            tag: "ZIxE7/fbJqyu1jLNdbHTRA==", // Тег аутентификации контента (Проверка целостности)
            files: [{
                tag: encrypted.tag.toString('base64'), // Тег аутентификации (Проверка целостности)
                encoded_as: "base64", // В какой кодировке находится зашифрованный файл
            }]
        },
        flags: 3 // Подробнее в папке messages, раздел Флаги.
    });
    const form = new FormData();
    form.append('payload', jsonPayload);
    form.append('files[0]', encrypted.data);
    
    // Отправляем запрос на сервер
    fetch('https://mcp.example.org/api/users/174473846617255485/messages', {
        method: 'POST',
        headers: {
            Authorization: 'Ваш токен',
            'Content-Type': 'multipart/form-data'
        },
        body: form
    }).then((res) => res.json()).then((json) => {
        console.log(json);
    });
});
```