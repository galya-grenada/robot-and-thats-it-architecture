# 02. Информационные потоки

Колонки: контейнер-источник, контейнер-приёмник, передаваемая информация, протокол.

## A. Люди и приложения

| # | Источник | Приёмник | Информация | Протокол |
|---|---|---|---|---|
| A1 | Посетитель | Web, Mobile, Terminal | Заказ | UI |
| A2 | Order Tableau | Посетитель | Готовность заказа | UI |
| A3 | Технолог | Service Web | Технологические карты | UI |
| A4 | Менеджер по ассортименту | Service Web | Настройка меню, акций | UI |
| A5 | Менеджер по гостеприимству | Service Web | Отмена заказа | UI |
| A6 | Администратор | Service Web | Права доступа | UI |
| A7 | Notification Service | Посетитель | Уведомления о статусе заказа | Push / SMS / E-mail |
| A8 | Notification Service | Менеджер по гостеприимству | Сбои | Push / SMS / E-mail |

## B. Клиентские приложения → API Gateway

| # | Источник | Приёмник | Информация | Протокол |
|---|---|---|---|---|
| B1 | Web, Mobile, Terminal | API Gateway | Запрос меню, создание заказа, статус заказа, логин, профиль | HTTPS/JSON (REST) |
| B2 | Service Web | API Gateway | Правка меню и техкарт, отмена заказа, заказ поставки, управление правами | HTTPS/JSON (REST) |
| B3 | Order Tableau | API Gateway | Подписка на статусы заказов | WSS |

## C. API Gateway → сервисы

| # | Источник | Приёмник | Информация | Протокол |
|---|---|---|---|---|
| C1 | API Gateway | IAM Service | Логин, проверка прав (Auth/Login) | REST |
| C2 | API Gateway | Menu Service | Запрос меню (Get Menu), изменение меню (Update Menu) | REST |
| C3 | API Gateway | Orders Service | Создание заказа (состав, сумма), отмена (ID заказа), статус заказа; в ответе на создание результат оплаты | REST |
| C4 | API Gateway | Clients Service | Профиль посетителя (Get Profile) | REST |
| C5 | API Gateway | Supply Service | Ручной заказ продукции (позиции, количество) | REST |
| C6 | API Gateway | Tableau Service | Подписка табло на статусы (Subscribe Status) | WSS |

## D. Событийный обмен через Kafka

Публикация (сервис → Kafka):

| # | Источник | Приёмник | Информация | Протокол |
|---|---|---|---|---|
| D1 | Orders Service | Message Broker | `OrderCancelled`: ID заказа | Kafka |
| D2 | Payment Service | Message Broker | `OrderPaid`: ID заказа, ID клиента, сумма | Kafka |
| D3 | Robots Service | Message Broker | `OrderReady`: ID заказа | Kafka |
| D4 | Robots Service | Message Broker | `StockUpdated`: ингредиент, количество | Kafka |
| D5 | Robots Service | Message Broker | `RobotFailed`: описание сбоя | Kafka |
| D6 | IAM Service | Message Broker | `UserRegistered`: ID клиента, контакты | Kafka |
| D7 | Menu Service | Message Broker | `MenuUpdated`: изменённые позиции и цены | Kafka |

Подписка (Kafka → сервис):

| # | Источник | Приёмник | Информация | Протокол |
|---|---|---|---|---|
| D8 | Message Broker | Payment Service | `OrderCancelled` | Kafka |
| D9 | Message Broker | Robots Service | `OrderPaid` | Kafka |
| D10 | Message Broker | Orders Service | `OrderReady`, `MenuUpdated` | Kafka |
| D11 | Message Broker | Clients Service | `UserRegistered`, `OrderPaid` | Kafka |
| D12 | Message Broker | Supply Service | `StockUpdated` | Kafka |
| D13 | Message Broker | Tableau Service | `OrderReady` | Kafka |
| D14 | Message Broker | Notification Service | `OrderReady`, `OrderCancelled`, `UserRegistered`, `RobotFailed` | Kafka |

## E. Сервис → сервис (синхронно)

| # | Источник | Приёмник | Информация | Протокол |
|---|---|---|---|---|
| E1 | Orders Service | Payment Service | Оплата заказа (Pay Order): ID заказа, ID клиента, сумма; в ответе результат оплаты | REST |
| E2 | Robots Service | Orders Service | Запрос заказа (Get Order): ID заказа; в ответе состав заказа | REST |
| E3 | Notification Service | Clients Service | Контакты посетителя (Get Contacts) | REST |
| E4 | Notification Service | IAM Service | Контакты сотрудника (Get Staff Contacts) | REST |

## F. Сервисы ↔ внешние системы

| # | Источник | Приёмник | Информация | Протокол |
|---|---|---|---|---|
| F1 | Payment Service | Платёжный сервис | Оплата и возврат (ID заказа, сумма); в ответе результат | HTTPS/JSON |
| F2 | Robots Service | Роботы | Заказ на приготовление (Cook Order): состав заказа | HTTPS/JSON |
| F3 | Роботы | Robots Service | Остатки ингредиентов (Push Stock) | HTTPS/JSON |
| F4 | Supply Service | Склад | Заказ продукции (позиции, количество) | HTTPS/JSON |
| F5 | Clients Service | Система лояльности | Данные посетителя, сумма заказа; в ответе баланс бонусов | HTTPS/JSON |

## G. Сервисы → базы данных

| # | Источник | Приёмник | Информация | Протокол |
|---|---|---|---|---|
| G1 | IAM Service | IAM DB | Учётные записи, роли, права | JDBC |
| G2 | Menu Service | Menu DB | Меню, техкарты, акции | JDBC |
| G3 | Orders Service | Orders DB | Заказы, статусы, копия цен | JDBC |
| G4 | Clients Service | Clients DB | Профили клиентов | JDBC |
| G5 | Payment Service | Payment DB | Платёжные транзакции | JDBC |
| G6 | Supply Service | Supply DB | Остатки, пороги, заказы на склад | JDBC |
