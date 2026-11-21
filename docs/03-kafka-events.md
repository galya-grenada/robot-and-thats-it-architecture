# 03. События Kafka

Модель — хореография: общего «дирижёра» нет, каждый сервис сам реагирует на события.
События называются в прошедшем времени (факт, а не команда).

| Событие | Публикует | Читают | Что в событии | Реакция |
|---|---|---|---|---|
| `OrderPaid` | Payment | Robots, Clients | ID заказа, ID клиента, сумма | Robots: запросить состав у Orders и начать готовку; Clients: начислить бонусы |
| `OrderCancelled` | Orders | Payment, Notification | ID заказа | Возврат; уведомление |
| `OrderReady` | Robots | Tableau Service, Notification, Orders | ID заказа | Показать на табло; уведомить; обновить статус |
| `StockUpdated` | Robots | Supply | Ингредиент, количество | Сохранить остаток, при необходимости заказать на склад (Robots формирует его из `StockReport` от Роботов) |
| `RobotFailed` | Robots | Notification | Описание сбоя | Уведомить менеджера по гостеприимству |
| `UserRegistered` | IAM | Clients, Notification | ID клиента, контакты | Создать профиль; приветствие |
| `MenuUpdated` | Menu | Orders | Изменённые позиции и цены | Обновить копию цен |

Оплата в Kafka не участвует: Orders вызывает Payment синхронно (`Pay Order`, REST) и получает результат в ответе.
После успешной оплаты Payment публикует `OrderPaid` для остальных сервисов. В событиях только идентификаторы и ключевые значения;
состав заказа Robots запрашивает у Orders (`Get Order`, REST).

## Обмен с внешними системами (Роботы, Склад)

Заказы роботам и на склад идут через брокер, а не по REST: при перегрузке сообщения не теряются, повторные попытки обеспечивает Kafka.
Брокер один; для внешних систем созданы отдельные топики с правами доступа.

| Топик | Кто пишет | Кто читает | Сообщение | Что внутри |
|---|---|---|---|---|
| `robots.cook-orders` | Robots Service | Роботы | `CookOrder` | ID заказа, состав |
| `robots.cook-results` | Роботы | Robots Service | `CookResult` | ID заказа, результат (готов или сбой) |
| `robots.stock` | Роботы | Robots Service | `StockReport` | Ингредиент, количество |
| `supply.orders` | Supply Service | Склад | `SupplyOrder` | Позиции, количество |

Права доступа: Роботы читают `robots.cook-orders`, пишут `robots.cook-results` и `robots.stock`; Склад читает `supply.orders`.
К внутренним событиям (`OrderPaid`, `OrderReady` и остальным) у внешних систем доступа нет.
Допущение: внешние системы умеют работать с Kafka.

## Цепочка главного сценария

```
Orders --REST: Pay Order--> Payment --(результат в ответе)--> Orders --> клиент
Payment --OrderPaid--> Robots, Clients
Robots --REST: Get Order--> Orders              (состав заказа)
Robots --CookOrder--> Роботы --CookResult--> Robots
Robots --OrderReady--> Tableau Service, Notification, Orders
Роботы --StockReport--> Robots --StockUpdated--> Supply --SupplyOrder--> Склад
```
