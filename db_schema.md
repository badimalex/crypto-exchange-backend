# Архитектура криптообменника

## 1. Таблицы базы данных

### users
- id: BIGINT PRIMARY KEY AUTO_INCREMENT
- email: VARCHAR(255) UNIQUE NOT NULL
- created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- processed: BOOLEAN DEFAULT FALSE - обработан фоновым процессом

### currencies
- id: INT PRIMARY KEY AUTO_INCREMENT
- code: VARCHAR(10) UNIQUE NOT NULL - код валюты (BTC, USDT, ETH)
- name: VARCHAR(100) NOT NULL - полное название
- blockchain: VARCHAR(50) NOT NULL - блокчейн (bitcoin, ethereum, omni)
- decimals: TINYINT NOT NULL - количество знаков после запятой
- min_amount: DECIMAL(20,8) NOT NULL - минимальная сумма для обмена
- max_amount: DECIMAL(20,8) NOT NULL - максимальная сумма для обмена
- network_fee: DECIMAL(20,8) NOT NULL - сетевая комиссия
- is_active: BOOLEAN DEFAULT TRUE
- created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- processed: BOOLEAN DEFAULT FALSE - обработана фоновым процессом
- updated_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

### exchange_rates
- id: BIGINT PRIMARY KEY AUTO_INCREMENT
- from_currency_id: INT NOT NULL - FOREIGN KEY currencies(id)
- to_currency_id: INT NOT NULL - FOREIGN KEY currencies(id)
- rate: DECIMAL(20,8) NOT NULL - курс обмена
- created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- INDEX idx_currencies_created (from_currency_id, to_currency_id, created_at)

### orders
- id: BIGINT PRIMARY KEY AUTO_INCREMENT
- user_id: BIGINT NOT NULL - FOREIGN KEY users(id)
- from_currency_id: INT NOT NULL - FOREIGN KEY currencies(id)
- to_currency_id: INT NOT NULL - FOREIGN KEY currencies(id)
- from_amount: DECIMAL(20,8) NOT NULL - сумма для отправки
- to_amount: DECIMAL(20,8) NOT NULL - сумма для получения
- exchange_rate: DECIMAL(20,8) NOT NULL - курс на момент создания
- exchange_fee: DECIMAL(20,8) NOT NULL - комиссия обменника
- network_fee: DECIMAL(20,8) NOT NULL - сетевая комиссия
- recipient_address: VARCHAR(255) NOT NULL - адрес получателя
- deposit_address: VARCHAR(255) NOT NULL - адрес для депозита
- is_completed: BOOLEAN DEFAULT FALSE - заказ выполнен
- is_failed: BOOLEAN DEFAULT FALSE - заказ провален
- is_expired: BOOLEAN DEFAULT FALSE - заказ истек
- expires_at: TIMESTAMP NOT NULL - время истечения заказа
- created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- updated_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
- INDEX idx_user_completed (user_id, is_completed)
- INDEX idx_completed_created (is_completed, created_at)

### transactions
- id: BIGINT PRIMARY KEY AUTO_INCREMENT
- order_id: BIGINT NOT NULL - FOREIGN KEY orders(id)
- type: ENUM('deposit', 'withdrawal') NOT NULL
- currency_id: INT NOT NULL - FOREIGN KEY currencies(id)
- amount: DECIMAL(20,8) NOT NULL
- address: VARCHAR(255) NOT NULL
- tx_hash: VARCHAR(255) UNIQUE - хеш транзакции в блокчейне
- confirmations: INT DEFAULT 0 - количество подтверждений
- required_confirmations: INT DEFAULT 1 - требуемое количество подтверждений
- is_confirmed: BOOLEAN DEFAULT FALSE - транзакция подтверждена
- is_failed: BOOLEAN DEFAULT FALSE - транзакция провалена
- created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- updated_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
- INDEX idx_order_type (order_id, type)
- INDEX idx_tx_hash (tx_hash)

### wallets
- id: BIGINT PRIMARY KEY AUTO_INCREMENT
- currency_id: INT NOT NULL - FOREIGN KEY currencies(id)
- address: VARCHAR(255) UNIQUE NOT NULL
- private_key: TEXT NOT NULL - зашифрованный приватный ключ
- balance: DECIMAL(20,8) DEFAULT 0 - баланс кошелька
- is_active: BOOLEAN DEFAULT TRUE
- created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- updated_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
- INDEX idx_currency_active (currency_id, is_active)

### processing_queue
- id: BIGINT PRIMARY KEY AUTO_INCREMENT
- table_name: VARCHAR(50) NOT NULL - имя таблицы (orders, transactions)
- record_id: BIGINT NOT NULL - ID записи
- event_type: VARCHAR(100) NOT NULL - тип события
- processed: BOOLEAN DEFAULT FALSE
- created_at: TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- processed_at: TIMESTAMP NULL
- INDEX idx_processed_created (processed, created_at)
- INDEX idx_table_record (table_name, record_id)
