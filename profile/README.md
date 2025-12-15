<div align="center">

# ⚡ TG-Serverless Platform

**Next-Gen Serverless Infrastructure for Telegram Bots**

[![High Load](https://img.shields.io/badge/Architecture-High_Load-orange?style=for-the-badge&logo=appveyor)](https://github.com/tg-serverless)
[![GitOps](https://img.shields.io/badge/Delivery-GitOps-blueviolet?style=for-the-badge&logo=git)](https://github.com/tg-serverless)
[![Kubernetes](https://img.shields.io/badge/Orchestration-K8s-326CE5?style=for-the-badge&logo=kubernetes)](https://github.com/tg-serverless)

<p align="center">
  <b>TG-Serverless</b> — это облачная PaaS платформа, позволяющая разворачивать, масштабировать и мониторить тысячи Telegram-ботов с единой точки входа.
  <br>
  Мы решаем проблему "шумных соседей", бесконечных ретраев и потери вебхуков.
</p>

</div>

---

## 🏗 Архитектура

Наша платформа построена на принципах **Event-Driven Architecture** и **Hard Multi-tenancy**. Мы отказались от синхронной обработки в пользу асинхронной очереди с гарантиями доставки.
```mermaid
graph LR
    %% Стилизация
    classDef plain fill:#fff,stroke:#333,stroke-width:1px;
    classDef pod fill:#2d2d2d,stroke:#fff,stroke-width:2px,stroke-dasharray: 5 5,color:#fff;
    classDef darkNode fill:#444,stroke:#fff,stroke-width:1px,color:#fff;
    


    %% Основной поток (GW теперь обычный)
    TG[Telegram Webhook] -->|POST /:hash| GW(Gateway Fiber)
    
    %% Кэш и Редис
    GW -->|L1: Memory| CACHE{Shard Config}:::cache
    CACHE -.->|L2: Fallback| REDIS[(Redis)]:::cache

    %% Кафка
    GW -->|Async Push| KAFKA{Kafka Cluster}:::kafka

    %% Воркер (Под)
    subgraph Worker_Pod [Pod: Worker-N]
        direction TB
        %% Определяем узлы
        %% SC теперь использует redNode
        SC[Sidecar Go]
        BOT[User Bot]:::darkNode
        
        %% Хак: Невидимая связь заставляет Бота встать ПОД Сайдкаром
        SC ~~~ BOT
        
        %% Реальная связь (Long Polling)
        BOT -->|Long Polling| SC
    end
    
    %% Применяем стиль к рамке пода
    class Worker_Pod pod

    %% Связь Кафки с сайдкаром
    KAFKA -->|Topic: shard-N| SC
```

---

## 🛠 Технологический стек

Мы используем современные инструменты для обеспечения надежности и производительности:

| Компонент | Технологии | Назначение |
| :--- | :--- | :--- |
| **Ingress** | `Go`, `Fiber`, `Sarama` | **Gateway Service.** Прием вебхуков (10k+ RPS), валидация и шардирование в Kafka. Zero-Allocation. |
| **Compute** | `Go`, `Franz-Go`, `Python` | **Sidecar Proxy.** Адаптер между Kafka и кодом пользователя. Реализует Backpressure и Long Polling API. |
| **Bus** | `Apache Kafka`, `KRaft` | Надежная шина событий. Гарантирует, что ни одно сообщение от Telegram не потеряется. |
| **Orchestration** | `K8s`, `KEDA`, `cdk8s` | Автомасштабирование (Scale-to-Zero) на основе лага очереди. Infrastructure as Code на Go. |
| **Storage** | `Redis`, `ClickHouse` | L2-кэширование конфигураций и аналитика реального времени (Vector pipeline). |

---

## 💎 Ключевые компоненты

### 🛡️ [Gateway Service](https://github.com/tg-serverless/tg-gateway)
Единая точка входа. Реализует паттерн **Fire-and-Forget**.
*   **Smart Sharding:** Детерминированное распределение сообщений по шардам на основе `user_id` для сохранения порядка.
*   **Resilience:** Защита от *Cache Stampede* (Singleflight) и падения брокеров.

### 🏎️ [Sidecar Proxy](https://github.com/tg-serverless/worker-sidecar)
Интеллектуальный "сосед" для контейнера бота.
*   **Protocol Translation:** Превращает Push-поток Kafka в Pull-интерфейс, совместимый с библиотеками (`aiogram`, `telebot`).
*   **Safety:** Изолирует бота от интернета, предоставляя контролируемый прокси для исходящих запросов.

### 🧠 [Control Plane]() (Coming Soon)
API управления платформой.

---

<div align="center">

### 🤝 Join the Development

Мы строим Open Source компоненты для High Load систем.
**[Связаться с Lead Architect](https://t.me/chebnick)**

</div>

---

<div align="center">

### 💎 Support Architecture & Development

Building High Load infrastructure requires coffee and cloud credits.
If you like this project, you can support us via **TON**:

`UQB05cLbbMiz0MX_2ETU6M-tT3tx5VedP7mPqFFfx5d_EMdN`

</div>
