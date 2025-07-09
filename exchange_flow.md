# Схема обмена с примерами запросов

## Предварительные данные в БД

```sql
-- Валюты уже настроены
INSERT INTO currencies (code, name, blockchain, decimals, min_amount, max_amount, network_fee)
VALUES 
('BTC', 'Bitcoin', 'bitcoin', 8, 0.001, 10.0, 0.0005),
('USDT', 'Tether', 'omni', 6, 50.0, 100000.0, 25.0);

-- Актуальные курсы
INSERT INTO exchange_rates (from_currency_id, to_currency_id, rate)
VALUES (2, 1, 0.00002151); -- 1 USDT = 0.00002151 BTC

-- Кошельки системы
INSERT INTO wallets (currency_id, address, balance)
VALUES 
(1, '1CK6KHY6MHgYvmRQ4PAaf...', 5.5),
(2, '1A1zP1eP5QGefi2DMPTfTL...', 50000.0);
```

## Шаг 1: Пользователь заходит на сайт и смотрит курсы

**Что делает**: Открывает главную страницу, видит форму обмена  
**Запросы к БД**:
```sql
-- Получить валюты
SELECT code, name, blockchain, min_amount, max_amount, network_fee 
FROM currencies WHERE is_active = true;

-- Получить курс USDT → BTC
SELECT rate FROM exchange_rates 
WHERE from_currency_id = 2 AND to_currency_id = 1 
ORDER BY created_at DESC LIMIT 1;
```

## Шаг 2: Пользователь жмет кнопку "Создать заказ"

**Что делает**: Заполнил форму (1000 USDT → BTC, адрес, email), нажал "Next step"  
**Что получает**: Адрес для отправки USDT и инструкцию "Отправьте 1000 USDT на адрес..."  
**Запросы к БД**:
```sql
-- Создать пользователя (если новый)
INSERT INTO users (email) VALUES ('user@example.com');

-- Создать заказ
INSERT INTO orders (user_id, from_currency_id, to_currency_id, from_amount, to_amount, 
                   exchange_rate, exchange_fee, network_fee, recipient_address, deposit_address, expires_at)
VALUES (1, 2, 1, 1000.0, 0.02096215, 0.00002151, 0.00005379, 0.0005, 
        '1KYDrglejbHICE...', '1A1zP1eP5QGefi2DMPTfTL...', NOW() + INTERVAL 2 HOUR);
```

## Шаг 3: Пользователь переводит деньги через свой кошелек

**Что делает**: Открывает свой кошелек, отправляет 1000 USDT на адрес 1A1zP1eP5QGefi2DMPTfTL...  
**Что происходит**: Крон-задача `blockchain-monitor` (каждые 30 сек) сканирует блокчейн и находит транзакцию  
**Запросы к БД** (когда `blockchain-monitor` обнаружил транзакцию):
```sql
-- Зафиксировать входящую транзакцию
INSERT INTO transactions (order_id, type, currency_id, amount, address, tx_hash, confirmations)
VALUES (1, 'deposit', 2, 1000.0, '1A1zP1eP5QGefi2DMPTfTL...', 'abc123def456...', 0);

-- Обновить баланс кошелька
UPDATE wallets SET balance = balance + 1000.0 WHERE address = '1A1zP1eP5QGefi2DMPTfTL...';
```

## Шаг 4: Система дожидается подтверждений

**Что делает**: Тот же `blockchain-monitor` проверяет количество подтверждений транзакции  
**Запросы к БД** (когда подтверждений стало достаточно):
```sql
-- Подтвердить транзакцию
UPDATE transactions SET confirmations = 1, is_confirmed = true 
WHERE tx_hash = 'abc123def456...';
```

## Шаг 5: Система автоматически отправляет BTC

**Что делает**: Крон-задача `payment-processor` (каждые 10 сек) находит подтвержденные депозиты и отправляет BTC  
**Запросы к БД**:
```sql
-- Создать исходящую транзакцию
INSERT INTO transactions (order_id, type, currency_id, amount, address, tx_hash)
VALUES (1, 'withdrawal', 1, 0.02096215, '1KYDrglejbHICE...', 'def789abc123...');

-- Обновить баланс кошелька
UPDATE wallets SET balance = balance - 0.02096215 WHERE currency_id = 1;

-- Завершить заказ
UPDATE orders SET is_completed = true WHERE id = 1;
```

## Шаг 6: Админ смотрит заказы и балансы

**Что делает**: Заходит в админку, проверяет статистику  
**Запросы к БД**:
```sql
-- Список заказов
SELECT o.id, u.email, o.from_amount, o.to_amount, o.is_completed, o.created_at
FROM orders o JOIN users u ON o.user_id = u.id
ORDER BY o.created_at DESC LIMIT 20;

-- Балансы кошельков
SELECT c.code, w.address, w.balance
FROM wallets w JOIN currencies c ON w.currency_id = c.id
WHERE w.is_active = true;
```

## Фоновые процессы (отдельные бинарники)

**1. `blockchain-monitor`** (каждые 30 сек):
```sql
-- Сканирует блокчейн на предмет новых транзакций на наши адреса
-- Обновляет confirmations для существующих транзакций
```

**2. `payment-processor`** (каждые 10 сек):
```sql
-- Ищет подтвержденные депозиты и отправляет соответствующие выплаты
SELECT * FROM orders WHERE is_completed = false AND EXISTS (
    SELECT 1 FROM transactions WHERE order_id = orders.id AND type = 'deposit' AND is_confirmed = true
);
```

**3. `notification-service`** (каждые 30 сек):
```sql
-- Найти новые заказы для уведомлений
INSERT INTO processing_queue (table_name, record_id, event_type)
SELECT 'orders', id, 'order_created' FROM orders 
WHERE processed = false AND created_at > NOW() - INTERVAL 1 MINUTE;

-- Отправить email/webhook уведомления
```

**4. `order-expirer`** (каждые 5 минут):
```sql
-- Закрыть просроченные заказы
UPDATE orders SET is_expired = true WHERE expires_at < NOW() AND is_completed = false;
```