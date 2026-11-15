# 03. События Kafka

Модель — хореография: общего «дирижёра» нет, каждый сервис сам реагирует на события.
События называются в прошедшем времени (факт, а не команда).

| Событие | Публикует | Читают | Что в событии | Реакция |
|---|---|---|---|---|
| `OrderCreated` | Orders | Payment | ID заказа, состав, сумма | Запросить оплату |
| `OrderCancelled` | Orders | Payment, Notification | ID заказа | Возврат; уведомление |
| `OrderPaid` | Payment | Robots, Orders, Clients | ID заказа, сумма | Начать готовку; обновить статус; начислить бонусы |
| `PaymentFailed` | Payment | Orders | ID заказа | Обновить статус |
| `OrderReady` | Robots | Tableau Service, Notification, Orders | ID заказа | Показать на табло; уведомить; обновить статус |
| `StockUpdated` | Robots | Supply | Ингредиент, количество | Сохранить остаток, при необходимости заказать на склад |
| `RobotFailed` | Robots | Notification | Описание сбоя | Уведомить менеджера по гостеприимству |
| `UserRegistered` | IAM | Clients, Notification | ID клиента, контакты | Создать профиль; приветствие |
| `MenuUpdated` | Menu | Orders | Изменённые позиции и цены | Обновить копию цен |

## Цепочка главного сценария

```
Orders --OrderCreated--> Payment --OrderPaid--> Robots --OrderReady--> Tableau Service, Notification
                                         └─────> Orders (статус), Clients (бонусы)
```
