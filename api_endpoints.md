# API Endpoints для криптообменника

## Публичные эндпоинты

### 1. Получить список валют

**GET** `/api/currencies`

**Описание:** Получить список поддерживаемых валют

**Параметры:** отсутствуют

**Ответ:**
```json
{
  "currencies": [
    {
      "id": 1,
      "code": "BTC",
      "name": "Bitcoin",
      "blockchain": "bitcoin",
      "min_amount": "0.001",
      "max_amount": "10.0",
      "network_fee": "0.0005"
    }
  ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `currencies` | Array | Массив валют |
| `currencies[].id` | Integer | ID валюты |
| `currencies[].code` | String | Код валюты |
| `currencies[].name` | String | Полное название валюты |
| `currencies[].blockchain` | String | Название блокчейна |
| `currencies[].min_amount` | String | Минимальная сумма для обмена |
| `currencies[].max_amount` | String | Максимальная сумма для обмена |
| `currencies[].network_fee` | String | Сетевая комиссия |

---

### 2. Получить курс обмена

**GET** `/api/exchange-rate`

**Описание:** Получить курс обмена и рассчитать суммы

**Параметры:**

| Параметр | Тип | Описание |
|----------|-----|----------|
| `from` | String | Код исходной валюты |
| `to` | String | Код целевой валюты |
| `amount` | String | Сумма для обмена |

**Ответ:**
```json
{
  "rate": "0.00002151",
  "from_amount": "1000.0",
  "to_amount": "0.02096215",
  "exchange_fee": "0.00005379",
  "network_fee": "0.0005"
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `rate` | String | Курс обмена |
| `from_amount` | String | Сумма отправки |
| `to_amount` | String | Сумма получения |
| `exchange_fee` | String | Комиссия обменника |
| `network_fee` | String | Сетевая комиссия |

---

### 3. Создать заказ на обмен

**POST** `/api/orders`

**Описание:** Создать новый заказ на обмен валют

**Параметры:**

| Параметр | Тип | Описание |
|----------|-----|----------|
| `email` | String | Email пользователя |
| `from_currency` | String | Код исходной валюты |
| `to_currency` | String | Код целевой валюты |
| `from_amount` | String | Сумма для отправки |
| `recipient_address` | String | Адрес получателя |
| `terms_accepted` | Boolean | Согласие с условиями |

**Ответ:**
```json
{
  "order_id": "12345",
  "deposit_address": "1A1zP1eP5QGefi2DMPTfTL...",
  "from_amount": "1000.0",
  "to_amount": "0.02096215",
  "expires_at": "2025-07-10T14:30:00Z",
  "estimated_arrival": "5-30 minutes"
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `order_id` | String | ID созданного заказа |
| `deposit_address` | String | Адрес для депозита |
| `from_amount` | String | Сумма для отправки |
| `to_amount` | String | Сумма получения |
| `expires_at` | String | Время истечения заказа (ISO 8601) |
| `estimated_arrival` | String | Оценочное время прибытия |

---

### 4. Получить информацию о заказе

**GET** `/api/orders/{id}`

**Описание:** Получить детальную информацию о заказе

**Параметры:**

| Параметр | Тип | Описание |
|----------|-----|----------|
| `id` | String | ID заказа (в URL) |

**Ответ:**
```json
{
  "order": {
    "id": "12345",
    "status": "pending",
    "from_currency": "USDT",
    "to_currency": "BTC",
    "from_amount": "1000.0",
    "to_amount": "0.02096215",
    "deposit_address": "1A1zP1eP5QGefi2DMPTfTL...",
    "recipient_address": "1KYDrglejbHICE...",
    "expires_at": "2025-07-10T14:30:00Z",
    "created_at": "2025-07-10T12:30:00Z"
  }
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `order` | Object | Объект заказа |
| `order.id` | String | ID заказа |
| `order.status` | String | Статус заказа (`pending`, `completed`, `failed`, `expired`) |
| `order.from_currency` | String | Код исходной валюты |
| `order.to_currency` | String | Код целевой валюты |
| `order.from_amount` | String | Сумма отправки |
| `order.to_amount` | String | Сумма получения |
| `order.deposit_address` | String | Адрес для депозита |
| `order.recipient_address` | String | Адрес получателя |
| `order.expires_at` | String | Время истечения заказа (ISO 8601) |
| `order.created_at` | String | Время создания заказа (ISO 8601) |

---

## Административные эндпоинты

### 1. Получить список заказов

**GET** `/api/admin/orders`

**Описание:** Получить список всех заказов с фильтрацией и пагинацией

**Параметры:**

| Параметр | Тип | Описание |
|----------|-----|----------|
| `page` | Integer | Номер страницы (по умолчанию: 1) |
| `limit` | Integer | Количество записей на странице (по умолчанию: 20) |
| `status` | String | Фильтр по статусу: `completed`, `failed`, `expired`, `pending` (опционально) |

**Ответ:**
```json
{
  "orders": [
    {
      "id": "12345",
      "email": "user@example.com",
      "status": "completed",
      "from_currency": "USDT",
      "to_currency": "BTC",
      "from_amount": "1000.0",
      "to_amount": "0.02096215",
      "created_at": "2025-07-10T12:30:00Z"
    }
  ],
  "total": 150,
  "page": 1,
  "pages": 8
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `orders` | Array | Массив заказов |
| `orders[].id` | String | ID заказа |
| `orders[].email` | String | Email пользователя |
| `orders[].status` | String | Статус заказа |
| `orders[].from_currency` | String | Код исходной валюты |
| `orders[].to_currency` | String | Код целевой валюты |
| `orders[].from_amount` | String | Сумма отправки |
| `orders[].to_amount` | String | Сумма получения |
| `orders[].created_at` | String | Время создания заказа (ISO 8601) |
| `total` | Integer | Общее количество заказов |
| `page` | Integer | Текущая страница |
| `pages` | Integer | Общее количество страниц |

---

### 2. Получить балансы кошельков

**GET** `/api/admin/wallets`

**Описание:** Получить балансы всех кошельков системы

**Параметры:** отсутствуют

**Ответ:**
```json
{
  "wallets": [
    {
      "currency": "BTC",
      "balance": "5.5",
      "addresses": [
        {
          "address": "1CK6KHY6MHgYvmRQ4PAaf...",
          "balance": "2.3",
          "active": true
        },
        {
          "address": "1A1zP1eP5QGefi2DMPTfTL...",
          "balance": "3.2",
          "active": true
        }
      ]
    }
  ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `wallets` | Array | Массив кошельков |
| `wallets[].currency` | String | Код валюты |
| `wallets[].balance` | String | Общий баланс по валюте |
| `wallets[].addresses` | Array | Массив адресов |
| `wallets[].addresses[].address` | String | Адрес кошелька |
| `wallets[].addresses[].balance` | String | Баланс конкретного адреса |
| `wallets[].addresses[].active` | Boolean | Активен ли адрес |

---

### 3. Обработать заказ вручную

**POST** `/api/admin/orders/{id}/process`

**Описание:** Обработать заказ вручную (завершить, отклонить, возвратить)

**Параметры:**

| Параметр | Тип | Описание |
|----------|-----|----------|
| `id` | String | ID заказа (в URL) |
| `action` | String | Действие: `complete`, `fail`, `refund` |

**Ответ:**
```json
{
  "success": true,
  "message": "Заказ успешно завершен"
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `success` | Boolean | Результат операции |
| `message` | String | Сообщение о результате |

---

### 4. Получить список транзакций

**GET** `/api/admin/transactions`

**Описание:** Получить список всех транзакций с фильтрацией

**Параметры:**

| Параметр | Тип | Описание |
|----------|-----|----------|
| `page` | Integer | Номер страницы (по умолчанию: 1) |
| `limit` | Integer | Количество записей на странице (по умолчанию: 20) |
| `type` | String | Тип транзакции: `deposit`, `withdrawal` (опционально) |
| `status` | String | Статус транзакции: `pending`, `confirmed`, `failed` (опционально) |

**Ответ:**
```json
{
  "transactions": [
    {
      "id": "67890",
      "order_id": "12345",
      "type": "deposit",
      "currency": "USDT",
      "amount": "1000.0",
      "address": "1A1zP1eP5QGefi2DMPTfTL...",
      "tx_hash": "abc123def456...",
      "status": "confirmed",
      "created_at": "2025-07-10T12:35:00Z"
    }
  ],
  "total": 75,
  "page": 1,
  "pages": 4
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `transactions` | Array | Массив транзакций |
| `transactions[].id` | String | ID транзакции |
| `transactions[].order_id` | String | ID связанного заказа |
| `transactions[].type` | String | Тип транзакции (`deposit`, `withdrawal`) |
| `transactions[].currency` | String | Код валюты |
| `transactions[].amount` | String | Сумма транзакции |
| `transactions[].address` | String | Адрес транзакции |
| `transactions[].tx_hash` | String | Хеш транзакции в блокчейне |
| `transactions[].status` | String | Статус транзакции |
| `transactions[].created_at` | String | Время создания транзакции (ISO 8601) |
| `total` | Integer | Общее количество транзакций |
| `page` | Integer | Текущая страница |
| `pages` | Integer | Общее количество страниц |