# 💳 Telegram Payment Handler

> Type-safe библиотека для обработки платежей в Telegram (ЮKassa integration)

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)
![MyPy](https://img.shields.io/badge/MyPy-Strict-blue)
![Tests](https://img.shields.io/badge/Tests-Passing-success)

---

## 📋 О Проекте

**Telegram Payment Handler** — type-safe библиотека для интеграции платёжной системы ЮKassa с Telegram ботами. Обеспечивает автоматическую обработку webhook'ов, валидацию платежей и управление статусами.

### Ключевые Особенности

✅ **Type-Safe**: MyPy strict mode, Pydantic schemas  
✅ **Async/Sync**: Поддержка обоих режимов  
✅ **Webhook Handling**: Автоматическая обработка уведомлений  
✅ **Payment Tracking**: Отслеживание статусов платежей  
✅ **Error Handling**: Детальная обработка ошибок  

---

## 🚀 Установка

```bash
pip install telegram-payment-handler
```

### Требования

- Python 3.11+
- ЮKassa API ключи

---

## 📖 Быстрый Старт

### 1. Базовое Использование

```python
from telegram_payment_handler import PaymentHandler, PaymentConfig

# Конфигурация
config = PaymentConfig(
    shop_id="YOUR_SHOP_ID",
    secret_key="YOUR_SECRET_KEY"
)

# Создание обработчика
handler = PaymentHandler(config)

# Создание платежа
payment = await handler.create_payment(
    amount=1000.00,  # в рублях
    description="Подписка на 1 месяц",
    user_id=123456789,  # Telegram user ID
    return_url="https://t.me/your_bot"
)

print(f"Ссылка для оплаты: {payment.confirmation_url}")
```

### 2. Обработка Webhook

```python
from flask import Flask, request
from telegram_payment_handler import PaymentHandler

app = Flask(__name__)
handler = PaymentHandler(config)

@app.route('/webhook', methods=['POST'])
async def webhook():
    # Валидация и обработка webhook
    event = await handler.handle_webhook(request.json)
    
    if event.status == "succeeded":
        # Активировать подписку
        await activate_subscription(event.user_id)
    
    return {"status": "ok"}
```

### 3. Проверка Статуса Платежа

```python
# Получить статус платежа
payment_status = await handler.get_payment_status(payment_id)

if payment_status.is_paid:
    print("Платёж успешно завершён!")
```

---

## 🛠️ API Документация

### PaymentHandler

#### `create_payment()`
Создание нового платежа

**Параметры**:
- `amount` (float): Сумма в рублях
- `description` (str): Описание платежа
- `user_id` (int): Telegram user ID
- `return_url` (str): URL для возврата после оплаты

**Возвращает**: `Payment` объект

```python
payment = await handler.create_payment(
    amount=500.00,
    description="Тестовый платёж",
    user_id=123456789,
    return_url="https://t.me/bot"
)
```

#### `handle_webhook()`
Обработка webhook от ЮKassa

**Параметры**:
- `webhook_data` (dict): JSON данные от ЮKassa

**Возвращает**: `WebhookEvent` объект

```python
event = await handler.handle_webhook(request.json)
```

#### `get_payment_status()`
Получение статуса платежа

**Параметры**:
- `payment_id` (str): ID платежа

**Возвращает**: `PaymentStatus` объект

```python
status = await handler.get_payment_status("payment_id")
```

---

## 📦 Pydantic Schemas

### Payment
```python
class Payment(BaseModel):
    id: str
    status: str
    amount: float
    description: str
    user_id: int
    confirmation_url: str
    created_at: datetime
```

### WebhookEvent
```python
class WebhookEvent(BaseModel):
    payment_id: str
    status: str  # "pending", "succeeded", "canceled"
    user_id: int
    amount: float
    timestamp: datetime
```

### PaymentStatus
```python
class PaymentStatus(BaseModel):
    payment_id: str
    status: str
    is_paid: bool
    amount: float
```

---

## 🔧 Продвинутое Использование

### Кастомная Обработка Ошибок

```python
from telegram_payment_handler import PaymentError

try:
    payment = await handler.create_payment(...)
except PaymentError as e:
    print(f"Ошибка создания платежа: {e.message}")
    print(f"Код ошибки: {e.code}")
```

### Логирование

```python
import logging

# Включить debug логи
logging.basicConfig(level=logging.DEBUG)

handler = PaymentHandler(config, logger=logging.getLogger(__name__))
```

### Retry Механизм

```python
from telegram_payment_handler import RetryConfig

retry_config = RetryConfig(
    max_retries=3,
    backoff_factor=2.0
)

handler = PaymentHandler(config, retry_config=retry_config)
```

---

## 🧪 Тестирование

```bash
# Установить dev зависимости
pip install -e ".[dev]"

# Запустить тесты
pytest

# С покрытием
pytest --cov=telegram_payment_handler
```

---

## 📚 Примеры

### Пример 1: Telegram Bot с Платежами

```python
from telebot.async_telebot import AsyncTeleBot
from telegram_payment_handler import PaymentHandler

bot = AsyncTeleBot(TOKEN)
payment_handler = PaymentHandler(config)

@bot.message_handler(commands=['subscribe'])
async def subscribe(message):
    payment = await payment_handler.create_payment(
        amount=1000.00,
        description="Подписка на 1 месяц",
        user_id=message.from_user.id,
        return_url=f"https://t.me/{bot.get_me().username}"
    )
    
    await bot.send_message(
        message.chat.id,
        f"Оплатите подписку: {payment.confirmation_url}"
    )
```

### Пример 2: Flask Webhook Handler

```python
from flask import Flask, request
from telegram_payment_handler import PaymentHandler

app = Flask(__name__)
handler = PaymentHandler(config)

@app.route('/yookassa/webhook', methods=['POST'])
async def yookassa_webhook():
    event = await handler.handle_webhook(request.json)
    
    if event.status == "succeeded":
        # Активировать подписку в БД
        db.activate_subscription(event.user_id)
        
        # Отправить уведомление в Telegram
        await bot.send_message(
            event.user_id,
            "✅ Платёж успешно завершён!"
        )
    
    return {"status": "ok"}
```

---

## 🔐 Безопасность

### Валидация Webhook

Библиотека автоматически валидирует webhook'и через HMAC-SHA256:

```python
# Автоматическая валидация
event = await handler.handle_webhook(
    webhook_data=request.json,
    signature=request.headers.get('X-Yookassa-Signature')
)
```

---

## 📖 Связанные Проекты

- [QentVPN Case Study](https://github.com/kets-kets/portfolio-case-studies/blob/main/qentvpn-architecture.md) — Production использование этой библиотеки
- [telegram-subscription-manager](https://github.com/kets-kets/telegram-subscription-manager) — Демо-проект с интеграцией

---

## 📄 Лицензия

MIT License - см. [LICENSE](LICENSE)

---

## 👤 Автор

**kets**  
- GitHub: [@kets-kets](https://github.com/kets-kets)  
- Telegram: [@ketsdpt](https://t.me/ketsdpt)

---

## 📦 О Библиотеке

Библиотека выделена из production проекта QentVPN и адаптирована для публичного использования.
