# Робот и точка — микросервисная архитектура (C4, уровни 1–2)

Учебный проект (OTUS, «Системный аналитик. Advanced», ДЗ 10): архитектура системы заказа еды
на точке с роботизированным приготовлением. Монолитная C4-диаграмма переведена на модель
**«Микросервисы»** и описана таблицами компонентов и информационных потоков.

## Диаграммы

**Уровень 1. Контекст**: кто и какие внешние системы взаимодействуют с системой.

![C4 level 1](diagrams/c4-level1-context.png)

**Уровень 2. Контейнеры (микросервисы)**: клиентские приложения, API Gateway, сервисы, базы данных, Kafka.

![C4 level 2](diagrams/c4-level2-containers-microservices.png)

Исходник диаграммы: [`diagrams/c4-microservices.drawio`](diagrams/) (открывать в [draw.io](https://app.diagrams.net)).
Файл `legacy-c4-monolith.drawio` — исходная монолитная версия до перевода на микросервисы.

## Главный сценарий: заказ → оплата → готовка → табло

```mermaid
sequenceDiagram
    actor V as Посетитель
    participant T as Terminal
    participant G as API Gateway
    participant O as Orders
    participant K as Kafka
    participant P as Payment
    participant PS as Платёжный сервис
    participant R as Robots
    participant RB as Роботы
    participant TS as Tableau Service
    participant TB as Order Tableau

    V->>T: Заказ
    T->>G: Создать заказ (REST)
    G->>O: Create Order (gRPC)
    O-->>K: OrderCreated
    K-->>P: OrderCreated
    P->>PS: Create Payment (HTTPS)
    PS->>P: Payment Result (HTTPS)
    P-->>K: OrderPaid
    K-->>O: OrderPaid (обновить статус)
    K-->>R: OrderPaid
    R->>RB: Cooking Order (HTTPS)
    R-->>K: OrderReady
    K-->>TS: OrderReady
    TS->>TB: Статус «готов» (WSS)
```

## Стек и протоколы

| Связь | Протокол |
|---|---|
| Клиентские приложения → API Gateway | HTTPS/JSON (REST) |
| Order Tableau ↔ API Gateway ↔ Tableau Service | WSS (WebSocket) |
| API Gateway → сервисы | gRPC |
| Сервис ↔ сервис (синхронно) | gRPC |
| Сервис ↔ сервис (события) | Apache Kafka |
| Сервис → своя БД | JDBC (PostgreSQL) |
| Сервис ↔ внешняя система | HTTPS/JSON (допущение, API внешних систем неизвестен) |

## Документация

| Документ | Содержание |
|---|---|
| [01. Компоненты](docs/01-components.md) | Люди, внешние системы, контейнеры: назначение и взаимодействия |
| [02. Информационные потоки](docs/02-information-flows.md) | Источник, приёмник, данные, протокол |
| [03. События Kafka](docs/03-kafka-events.md) | Кто публикует, кто читает, что внутри события |
| [04. Архитектурные решения](docs/04-design-decisions.md) | Почему сделано именно так, принятые допущения |
