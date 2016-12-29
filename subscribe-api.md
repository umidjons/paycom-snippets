# Subscribe API
## Описание
Данный API предназначен для интеграции Payme в приложении поставщика. Например, мобильное приложение с возможностью оплатить за услуги через платёжный инструмент Payme.
- Нужно реализовать сервер по протоколу описанной в [документации](http://paycom.uz/developers/merchant/overview).
- Нужно на фронте или в приложении сделать форму для ввода карточных данных и `OTP` (`One Time Password` - `Проверочный код`). Основное требование к форме: элементы для ввода не должны иметь аттрибута `name` и тег `form` не должен иметь атрибут `action`.
- Пользователь на фронте или в приложении вводит свои карточные данные, вводит проверочный код. Вводимые пользователем данные ни в коем случае не отправлять себе на сервер. Все это происходит с помощью API которые описаны ниже. В ответ на запросы от API получаете токен карты, который дальше используете в серверном API.
- Разместить логотип Payme в приложение или в форму на фронте.
- Поставить [оферту Payme](https://cdn.payme.uz/terms/main.html).
- Поставить подсказку или текст, о том что все данные пользователей не передаются поставщику, а сохраняются на сервисе Payme.

### Пример формы
В этой форме можно увидеть логотип, подсказку и ссылку на оферту.
![Пример формы](https://cdn.paycom.uz/documentation_assets/safety_guaranteed.png)

### Логотип
| PNG | SVG |
| ------------- |:-------------:|
| ![Логотип в PNG формате](https://cdn.paycom.uz/documentation_assets/power-by-Payme.png) | ![Логотип в SVG формате](https://cdn.paycom.uz/documentation_assets/power-by-Payme.svg) |
|[Логотип в PNG формате](https://cdn.paycom.uz/documentation_assets/power-by-Payme.png) | [Логотип в SVG формате](https://cdn.paycom.uz/documentation_assets/power-by-Payme.svg) |

## Endpoints

Endpoint для серверного и фронт API один и тот же: `https://checkout.paycom.uz/api`
Для тестов `http://checkout.test.paycom.uz/api`

Для авторизации вам необходимо отправить заголовок `X-Auth`.

| Для Front-End | Для Back-End |
| ------------- |:-------------:|
| `X-Auth: {id}`| `X-Auth: {id}:{key}` |

## Методы для Front-End

API строятся на основе `json-rpc`, поэтому дальнейшие действия будут описаны в этом контексте.

### cards.create

Метод для создания токена карты.

Параметры:

```js
{
    card: {
        number String,
        expire: String
    },
    amount: Number,
    account: Object,
    save: Boolean
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `card`    | Object  | параметры карты |
| `number`  | String  | номер карты |
| `expire`  | String  | срок действия карты |
| `amount`  | Number  | запрашиваемая сумма платежа |
| `account?` | Object  | объект Account, данный параметр не обязательный |
| `save?`    | Boolean | данный параметр не обязателен. Если флаг `true` токен можно будет использовать для дальнейших платежей, если флаг `false` токеном можно будет сделать только 1 оплату, после чего токен будет недоступен. Данное поле должно проставляться пользователем, (например `checkbox` `Запонить карту`, по умолчанию `checkbox` должен быть выключен).|

Результат:
```js
{
    card: {
        number: String,
        expire: String,
        token: String,
        recurrent: Boolean,
        verify: Boolean
    }
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `number` | String | не полный номер карты, данную строку можете сохранить у себя на сервере |
| `expire` | String | срок действия карты |
| `token` | String | токен карты |
| `recurrent`| Boolean | флаг, доступна ли карта для последующих платежей |
| `verify` | Boolean | флаг, верифицирована ли карта |

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc
Cache-Control: no-cache

{
    "id": 123,
    "method": "cards.create",
    "params": {
		"card": { "number": "4444444444444444", "expire": "0918"},
		"amount": 3500,
		"save": true
	}
}
```

Пример ответа:

```json
{
    "jsonrpc": "2.0",
    "id": 123,
    "result": {
        "card": {
            "number": "444444******4444",
            "expire": "09/18",
            "token": "NTg0YTg0ZDYyYWJiNWNhYTMxMDc5OTE0X1VnYU02ME92IUttWHVHRThJODRJNWE0Xl9EYUBPQCZjNSlPRlpLIWNWRz1PNFp6VkIpZU0kQjJkayoyVUVtUuKElmt4JTJYWj9VQGNAQyVqT1pOQ3VXZ2NyajBEMSYkYj0kVj9NXikrJE5HNiN3K25pKHRQOEVwOGpOcUYxQ2dtemk9dDUwKDNATjd2XythbibihJYoJispJUtuREhlaClraGlJWTlLMihrLStlRjd6MFI3VCgjVDlpYjQ1ZThaMiojPVNTZylYJlFWSjlEZGFuSjZDNDJLdlhXP3YmV1B2dkRDa3g5X2l4N28oU0pOVEpSeXZKYnkjK0h3ViZfdmlhUHMp",
            "recurrent": true,
            "verify": false
        }
    }
}
```

### cards.get_verify_code

Метод запрашивает код для верификации карты.

Параметры:

```js
{
    token: String
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `token` | String | токен карты |

Результат:

```js
{
    sent: true,
    phone: String,
    wait: Number
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `sent` | Boolean | флаг результата отправки |
| `phone` | String | не полный номер карты куда был отправлен смс с кодом |
| `wait` | Number | время в миллисекундах которое необходимо выждать перед повторным запросом кода |

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc
Cache-Control: no-cache

{
    "id": 123,
    "method": "cards.get_verify_code",
    "params": {
		"token": "NTg0YTg0ZDYyYWJiNWNhYTMxMDc5OTE0X1VnYU02ME92IUttWHVHRThJODRJNWE0Xl9EYUBPQCZjNSlPRlpLIWNWRz1PNFp6VkIpZU0kQjJkayoyVUVtUuKElmt4JTJYWj9VQGNAQyVqT1pOQ3VXZ2NyajBEMSYkYj0kVj9NXikrJE5HNiN3K25pKHRQOEVwOGpOcUYxQ2dtemk9dDUwKDNATjd2XythbibihJYoJispJUtuREhlaClraGlJWTlLMihrLStlRjd6MFI3VCgjVDlpYjQ1ZThaMiojPVNTZylYJlFWSjlEZGFuSjZDNDJLdlhXP3YmV1B2dkRDa3g5X2l4N28oU0pOVEpSeXZKYnkjK0h3ViZfdmlhUHMp"
	}
}
```

Пример ответа:

```json
{
    "jsonrpc": "2.0",
    "id": 123,
    "result": {
        "sent": true,
        "phone": "99890*****31",
        "wait": 60000
    }
}
```

### cards.verify

Метод верифицирует карту с помощью кода отправленного по СМС.

Параметры:

```js
{
    token: String,
    code: String
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `token` | String | токен карты |
| `code` | String | код для верификации |

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc
Cache-Control: no-cache

{
    "id": 123,
    "method": "cards.verify",
    "params": {
		"token": "NTg0YTg0ZDYyYWJiNWNhYTMxMDc5OTE0X1VnYU02ME92IUttWHVHRThJODRJNWE0Xl9EYUBPQCZjNSlPRlpLIWNWRz1PNFp6VkIpZU0kQjJkayoyVUVtUuKElmt4JTJYWj9VQGNAQyVqT1pOQ3VXZ2NyajBEMSYkYj0kVj9NXikrJE5HNiN3K25pKHRQOEVwOGpOcUYxQ2dtemk9dDUwKDNATjd2XythbibihJYoJispJUtuREhlaClraGlJWTlLMihrLStlRjd6MFI3VCgjVDlpYjQ1ZThaMiojPVNTZylYJlFWSjlEZGFuSjZDNDJLdlhXP3YmV1B2dkRDa3g5X2l4N28oU0pOVEpSeXZKYnkjK0h3ViZfdmlhUHMp",
		"code": "666666"
	}
}
```

Пример ответа:

```json
{
  "jsonrpc": "2.0",
  "id": 123,
  "result": {
    "card": {
      "number": "444444******4444",
      "expire": "09/18",
      "token": "NTg0YTgxZWYyYWJiNWNhYTMxMDc5OTExXyVwOTY4TzI3MTJRQ28lWmsoREEyRClYOCtxZ18kVWRLRm0xP3FucVUzJChZazhFV3I1dmtrQiZUaFU5MzZRdSlGbUJPSEh2K1IoWU0lYSg3ZEYlK1QhTUV4P3pUU+KElkMkXjNuIUR6U19pdjY4b3Ffbkt3ajImZTRhZll0dUptNjBVMUF4KXJKJD0qTlNeQmJ5X2Q3bXZNRnZ2UXhfU25TS0dpcGc9V1doUEZxKSM5R0dJYjA9U2dGX2ReZ3lATeKElj9mZWZJS3MzKVp5MjFeOVY5cE8jZWh6cHZLeWZXKSF2PVBfVVU4ei1Gbj82JkI3YjhuRCFWa1omaDB4JEliQm8h",
      "recurrent": true,
      "verify": true
    }
  }
}
```

## Методы для серверной части

### cards.check

Метод для проверка токена карты.

Параметры:

```js
{
    token: String
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `token` | String | токен карты |

Результат:

```js
{
    card: {
        number: String,
        expire: String,
        token: String,
        recurrent: Boolean,
        verify: Boolean
    }
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `number` | String | не полный номер карты, данную строку можете сохранить у себя на сервере |
| `expire` | String | дата истечения карты |
| `token` | String | токен карты |
| `recurrent` | Boolean | флаг, доступна ли карта для последующих платежей |
| `verify` | Boolean | флаг, верифицирована ли карта |

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc:Rw712wMJspZBczFvrG09?bHkSNxnD4PY0n1C
Content-Type: application/json
Cache-Control: no-cache

{
	"id": 123,
	"method": "cards.check",
	"params": {
		"token": "NTg1Yjc4OWMyYWJiNWNhYTMxMDc5YTE0X3hCJjc/M0NPejR4Jks5JmIxK2QkNCFHJXUqRyplIUB4MHpKVnUxOXZuRHVXK3h3XmVudS1hJFhON01ISSZBUV4jciQ4UD1YdFM4R0F0SmIkK3dfRlXihJYmI2F1MSpYNGNUVFViZkRtekZDNnU3XyElcERtdjRKXmtibWdFYjVpIVF0VW9NZWgzbyN5ZWhGRTdOQkBGU0JhS2ooR1dHZV5pWlJWZCVOekR2VHlJSmh5aSNxdVVXXnp2QUQmanVwb0AxbU1XcEMrcStPRUZQR1ZUTVllVTBeSGNEZkc/OD09JWleVEtqYUE4Y08rJloqVURLcG1rdiZEWCNJUk09dC1KKQ=="
	}
}
```

Пример ответа:

```json
{
    "jsonrpc": "2.0",
    "id": 123,
    "result": {
        "card": {
            "number": "444444******4444",
            "expire": "09/18",
            "token": "NTg1Yjc4OWMyYWJiNWNhYTMxMDc5YTE0X3hCJjc/M0NPejR4Jks5JmIxK2QkNCFHJXUqRyplIUB4MHpKVnUxOXZuRHVXK3h3XmVudS1hJFhON01ISSZBUV4jciQ4UD1YdFM4R0F0SmIkK3dfRlXihJYmI2F1MSpYNGNUVFViZkRtekZDNnU3XyElcERtdjRKXmtibWdFYjVpIVF0VW9NZWgzbyN5ZWhGRTdOQkBGU0JhS2ooR1dHZV5pWlJWZCVOekR2VHlJSmh5aSNxdVVXXnp2QUQmanVwb0AxbU1XcEMrcStPRUZQR1ZUTVllVTBeSGNEZkc/OD09JWleVEtqYUE4Y08rJloqVURLcG1rdiZEWCNJUk09dC1KKQ==",
            "recurrent": true,
            "verify": true
        }
    }
}
```

### cards.remove

Метод для удаления токена.

Параметры:

```js
{
    token: String
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `token` | String | токен карты |

Ответ:
```js
{
    success: Boolean
}
```

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc:Rw712wMJspZBczFvrG09?bHkSNxnD4PY0n1C
Content-Type: application/json
Cache-Control: no-cache

{
	"id": 123,
	"method": "cards.remove",
	"params": {
		"token": "NTg1Yjc4OWMyYWJiNWNhYTMxMDc5YTE0X3hCJjc/M0NPejR4Jks5JmIxK2QkNCFHJXUqRyplIUB4MHpKVnUxOXZuRHVXK3h3XmVudS1hJFhON01ISSZBUV4jciQ4UD1YdFM4R0F0SmIkK3dfRlXihJYmI2F1MSpYNGNUVFViZkRtekZDNnU3XyElcERtdjRKXmtibWdFYjVpIVF0VW9NZWgzbyN5ZWhGRTdOQkBGU0JhS2ooR1dHZV5pWlJWZCVOekR2VHlJSmh5aSNxdVVXXnp2QUQmanVwb0AxbU1XcEMrcStPRUZQR1ZUTVllVTBeSGNEZkc/OD09JWleVEtqYUE4Y08rJloqVURLcG1rdiZEWCNJUk09dC1KKQ=="
	}
}
```

Пример ответа:

```json
{
    "jsonrpc": "2.0",
    "id": 123,
    "result": {
        "success": true
    }
}
```

### receipts.create

Метод создает чек для оплаты.

Параметры:

```js
{
    amount: Number,
    account: Object,
    description: String,
    detail: Object
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `amount` | Number | Сумма платежа в тиинах |
| `account` | Object | Объект `Account` |
| `description?` | String | Необязательный параметр. Описание платежа. |
| `detail?` | Object | Необязательный параметр. Объект детализации платежа. |

Ответ:

```js
{
    result: receipt
}
```

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc:Rw712wMJspZBczFvrG09?bHkSNxnD4PY0n1C
Content-Type: application/json
Cache-Control: no-cache

{
	"id": 123,
	"method": "receipts.create",
	"params": {
		"amount": 2500,
		"account": {
			"order_id": 106
		}
	}
}
```

Пример ответа:

```json
{
  "jsonrpc": "2.0",
  "id": 123,
  "result": {
    "receipt": {
      "_id": "2e0b1bc1f1eb50d487ba268d",
      "create_time": 1481113810044,
      "pay_time": 0,
      "cancel_time": 0,
      "state": 0,
      "type": 1,
      "external": false,
      "operation": -1,
      "category": null,
      "error": null,
      "description": "",
      "detail": null,
      "amount": 2500,
      "commission": 0,
      "account": [
        {
          "name": "order_id",
          "title": "Код заказа",
          "value": "106"
        }
      ],
      "card": null,
      "merchant": {
        "_id": "100fe486b33784292111b7dc",
        "name": "Online Shop LLC",
        "organization": "ЧП «Online Shop»",
        "address": "",
        "epos": {
          "merchantId": "106600000050000",
          "terminalId": "20660000"
        },
        "date": 1480582278779,
        "logo": null,
        "type": "Shop",
        "terms": null
      },
      "meta": null
    }
  }
}
```

### receipts.pay

Метод для оплаты чека.

Параметры:

```js
id: String,
token: String,
payer: {
    id: String,
    phone: String,
    email: String,
    name: String,
    ip: String
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `id` | String | ID Чека |
| `token` | String | Токен карты |
| `payer?` | Object | Дополнительная информация о плательщике, необходима для системы антифрода, все поля данного объекта не обязательны |

Ответ:
```js
{
    result: receipt
}
```

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc:Rw712wMJspZBczFvrG09?bHkSNxnD4PY0n1C
Content-Type: application/json
Cache-Control: no-cache

{
	"id": 123,
	"method": "receipts.pay",
	"params": {
		"id": "2e0b1bc1f1eb50d487ba268d",
		"token": "NTg1Yjc4OWMyYWJiNWNhYTMxMDc5YTE0X3hCJjc/M0NPejR4Jks5JmIxK2QkNCFHJXUqRyplIUB4MHpKVnUxOXZuRHVXK3h3XmVudS1hJFhON01ISSZBUV4jciQ4UD1YdFM4R0F0SmIkK3dfRlXihJYmI2F1MSpYNGNUVFViZkRtekZDNnU3XyElcERtdjRKXmtibWdFYjVpIVF0VW9NZWgzbyN5ZWhGRTdOQkBGU0JhS2ooR1dHZV5pWlJWZCVOekR2VHlJSmh5aSNxdVVXXnp2QUQmanVwb0AxbU1XcEMrcStPRUZQR1ZUTVllVTBeSGNEZkc/OD09JWleVEtqYUE4Y08rJloqVURLcG1rdiZEWCNJUk09dC1KKQ==",
		"payer": {
			"phone": "998901304527"
		}
	}
}
```

Пример ответа:

```json
{
  "jsonrpc": "2.0",
  "id": null,
  "result": {
    "receipt": {
      "_id": "2e0b1bc1f1eb50d487ba268d",
      "create_time": 1481113810044,
      "pay_time": 1481113810265,
      "cancel_time": 0,
      "state": 4,
      "type": 1,
      "external": false,
      "operation": -1,
      "category": null,
      "error": null,
      "description": "",
      "detail": null,
      "amount": 3500,
      "commission": 0,
      "account": [
        {
          "name": "order_id",
          "title": "Код заказа",
          "value": "5"
        }
      ],
      "card": {
        "number": "444444******4444",
        "expire": "1809"
      },
      "merchant": {
        "_id": "100fe486b33784292111b7dc",
        "name": "Online Shop LLC",
        "organization": "ЧП «Online Shop»",
        "address": "",
        "epos": {
          "merchantId": "106600000050000",
          "terminalId": "20660000"
        },
        "date": 1480582278779,
        "logo": null,
        "type": "Shop",
        "terms": null
      },
      "meta": null
    }
  }
}
```

### receipts.cancel

Метод ставит в очередь на отмену оплаченный чек.

Параметры:

```js
{
    id: String,
    partially: {
        receivers: Array<{
            id: String,
            hold: Number
        }>,
        description: String,
        detail: Object
    }
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `id`       | String | ID Чека          |
| `partially?`| Object | Необязательный параметр. При указании данного параметра, чек будет отменен не полностью, а только получатели из переданного списка `receivers`. При этом текущий чек будет помечен как отмененный, и вернется новый чек с перещитанной суммой. |
| `receivers` | Array<Object> | Список получателей которых необходимо исключить из оригинального чека |
| `receivers.id` | String | `id` получателя которого необходимо исключить из чека |
| `hold?`      | Number | Необязательный параметр. Сумму которую нужно удержать на счету получателя. Данный параметр временно не работает. |
| `description?` | String | Необязательный параметр. Описание для новго чека. Если не указать данный параметр, он будет скопирован из оригинального чека. |
| `detail?` | Object | Необязательный параметр. Детализация нового чека. Если не указать данный параметр, он будет скопирован из оригинального чека. |

Ответ:
```js
{
    result: receipt
}
```

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc:Rw712wMJspZBczFvrG09?bHkSNxnD4PY0n1C
Content-Type: application/json
Cache-Control: no-cache

{
	"id": 123,
	"method": "receipts.cancel",
	"params": {
		"id": "2e0b1bc1f1eb50d487ba268d"
	}
}
```

Пример ответа:

```json
{
    "jsonrpc": "2.0",
    "id": 123,
    "result": {
        "receipt": {
            "_id": "2e0b1bc1f1eb50d487ba268d",
            "create_time": 1482823890336,
            "pay_time": 1482823890564,
            "cancel_time": 0,
            "state": 21,
            "type": 1,
            "external": false,
            "operation": -1,
            "category": null,
            "error": null,
            "description": "",
            "detail": null,
            "amount": 2000,
            "commission": 0,
            "account": [
                {
                    "name": "order_id",
                    "title": "Код заказа",
                    "value": "124"
                }
            ],
            "card": null,
            "merchant": {
                "_id": "100fe486b33784292111b7dc",
                "name": "Online Shop LLC",
                "organization": "ЧП «Online Shop»",
                "address": "",
                "epos": {
                    "merchantId": "106600000050000",
                    "terminalId": "20660000"
                },
                "date": 1480582278779,
                "logo": null,
                "type": "Shop",
                "terms": null
            },
            "meta": null
        }
    }
}
```

### receipts.check

Метод для проверки статуса чека.

Параметры:

```js
{
    id: String
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `id` | String | ID Чека |

Ответ:
```js
{
    state: Number
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `state` | Number | Состояние чека |

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc:Rw712wMJspZBczFvrG09?bHkSNxnD4PY0n1C
Content-Type: application/json
Cache-Control: no-cache

{
	"id": 123,
	"method": "receipts.check",
	"params": {
		"id": "2e0b1bc1f1eb50d487ba268d"
	}
}
```

Пример ответа:

```json
{
    "jsonrpc": "2.0",
    "id": 123,
    "result": {
        "state": 4
    }
}
```

### receipts.get

Метод возвращает полную информацияю по чеку.

Параметры:

```js
{
    id: String
}
```

| Параметр  | Тип     | Описание |
|-----------|---------|----------|
| `id` | String | ID Чека |

Пример запроса:

```http
POST /api HTTP/1.1
Host: checkout.test.paycom.uz
X-Auth: 100fe486b33784292111b7dc:Rw712wMJspZBczFvrG09?bHkSNxnD4PY0n1C
Content-Type: application/json
Cache-Control: no-cache

{
	"id": 123,
	"method": "receipts.get",
	"params": {
		"id": "2e0b1bc1f1eb50d487ba268d"
	}
}
```

Пример ответа:

```json
{
    "jsonrpc": "2.0",
    "id": 123,
    "result": {
        "receipt": {
            "_id": "2e0b1bc1f1eb50d487ba268d",
            "create_time": 1482823890336,
            "pay_time": 1482823890564,
            "cancel_time": 0,
            "state": 4,
            "type": 1,
            "external": false,
            "operation": -1,
            "category": null,
            "error": null,
            "description": "",
            "detail": null,
            "amount": 2000,
            "commission": 0,
            "account": [
                {
                    "name": "order_id",
                    "title": "Код заказа",
                    "value": "124"
                }
            ],
            "card": null,
            "merchant": {
                "_id": "100fe486b33784292111b7dc",
                "name": "Online Shop LLC",
                "organization": "ЧП «Online Shop»",
                "address": "",
                "epos": {
                    "merchantId": "106600000050000",
                    "terminalId": "20660000"
                },
                "date": 1480582278779,
                "logo": null,
                "type": "Shop",
                "terms": null
            },
            "meta": null
        }
    }
}
```

### Состояния чека

| Код | Описание |
|-----------|------------------|
| `0` | чек создан и ожидает подтверждение оплаты |
| `1` | запущена оплата, идет первая стадия проверок, создание транзакции в билинге поставщика |
| `2` | снятие средств с карты |
| `3` | закрытие транзакции в билинге поставщика |
| `4` | чек полностью проведен |
| `20` | чек стоит на паузе для ручного вмешательства |
| `21` | чек стоит в очереди на отмену |
| `30` | чек стоит в очереди на закрытие транзакции в билинге поставщика |
| `50` | чек отменен |
