
## 2. Эндпоинты API

### Публичные эндпоинты

#### GET /api/currencies
- Описание: Получить список поддерживаемых валют
- Параметры: нет
- Ответ:
  - currencies: Array<Object>
    - id: Integer
    - code: String
    - name: String
    - blockchain: String
    - min_amount: String
    - max_amount: String
    - network_fee: String

#### GET /api/exchange-rate
- Описание: Получить курс обмена
- Параметры:
  - from: String - код исходной валюты
  - to: String - код целевой валюты
  - amount: String - сумма для обмена
- Ответ:
  - rate: String - курс обмена
  - from_amount: String - сумма отправки
  - to_amount: String - сумма получения
  - exchange_fee: String - комиссия обменника
  - network_fee: String - сетевая комиссия

#### POST /api/orders
- Описание: Создать новый заказ на обмен
- Параметры:
  - email: String - email пользователя
  - from_currency: String - код исходной валюты
  - to_currency: String - код целевой валюты
  - from_amount: String - сумма для отправки
  - recipient_address: String - адрес получателя
  - terms_accepted: Boolean - согласие с условиями
- Ответ:
  - order_id: String - ID заказа
  - deposit_address: String - адрес для депозита
  - from_amount: String - сумма для отправки
  - to_amount: String - сумма получения
  - expires_at: String - время истечения заказа
  - estimated_arrival: String - оценочное время прибытия

#### GET /api/orders/{id}
- Описание: Получить информацию о заказе
- Параметры:
  - id: String - ID заказа
- Ответ:
  - order: Object
    - id: String
    - status: String - статус заказа (pending/completed/failed/expired)
    - from_currency: String
    - to_currency: String
    - from_amount: String
    - to_amount: String
    - deposit_address: String
    - recipient_address: String
    - expires_at: String
    - created_at: String

### Административные эндпоинты

#### GET /api/admin/orders
- Описание: Получить список всех заказов
- Параметры:
  - page: Integer - номер страницы (по умолчанию 1)
  - limit: Integer - количество записей на странице (по умолчанию 20)
  - status: String - фильтр по статусу: completed, failed, expired, pending (опционально)
- Ответ:
  - orders: Array<Object>
    - id: String
    - email: String
    - status: String
    - from_currency: String
    - to_currency: String
    - from_amount: String
    - to_amount: String
    - created_at: String
  - total: Integer - общее количество заказов
  - page: Integer - текущая страница
  - pages: Integer - общее количество страниц

#### GET /api/admin/wallets
- Описание: Получить балансы всех кошельков
- Параметры: нет
- Ответ:
  - wallets: Array<Object>
    - currency: String - код валюты
    - balance: String - баланс
    - addresses: Array<Object>
      - address: String
      - balance: String
      - is_active: Boolean

#### POST /api/admin/orders/{id}/process
- Описание: Обработать заказ вручную
- Параметры:
  - id: String - ID заказа
  - action: String - действие (complete, fail, refund)
- Ответ:
  - success: Boolean
  - message: String

#### GET /api/admin/transactions
- Описание: Получить список транзакций
- Параметры:
  - page: Integer - номер страницы
  - limit: Integer - количество записей на странице
  - type: String - тип транзакции (deposit, withdrawal)
  - status: String - статус транзакции: pending, confirmed, failed
- Ответ:
  - transactions: Array<Object>
    - id: String
    - order_id: String
    - type: String
    - currency: String
    - amount: String
    - address: String
    - tx_hash: String
    - status: String
    - created_at: String
  - total: Integer
  - page: Integer
  - pages: Integer