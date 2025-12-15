# 🗺️ Platform Roadmap

Ниже представлен план развития платформы от MVP до состояния **Enterprise PaaS**. Основной фокус — на наблюдаемости, безопасности выполнения недоверенного кода и устойчивости к высоким нагрузкам.

## 👁️ Phase 1: Observability & Logging
**Цель:** Обеспечить полную прозрачность работы платформы и пользовательских ботов в реальном времени с минимальным потреблением ресурсов.

- [ ] **ClickHouse Data Warehouse**
    - Развертывание кластера через Altinity Operator.
    - Проектирование схемы данных: `logs` (сырые данные с TTL 7 дней) и `metrics_1m` (агрегаты для дашбордов).
- [ ] **Structured Logging**
    - Внедрение `uber/zap` во все системные компоненты (Gateway, Sidecar, API).
    - Стандартизация формата логов для пользовательских раннеров (JSON Output).

## 🛡️ Phase 2: Security & Isolation
**Цель:** Реализация концепции "Defense in Depth". Полная изоляция пользовательского кода от инфраструктуры кластера.

- [ ] **Network Hardening (Cilium/Calico)**
    - Политика **Default Deny** для всех подов в неймспейсе `bots`.
    - Блокировка доступа к внутренней сети кластера (`10.0.0.0/8`) и Metadata-сервисам облака.
- [ ] **Runtime Isolation (gVisor)**
    - Внедрение `runsc` (gVisor) на рабочих нодах для изоляции ядра Linux.
    - Настройка `RuntimeClass` в Kubernetes для запуска ботов в песочнице.

## 🕵️ Phase 3: Anti-Fraud System
**Цель:** Автоматическое обнаружение и блокировка вредоносной активности (спам, фишинг) в реальном времени.

- [ ] **Fraud Detection Engine**
    - Асинхронный стриминг исходящих запросов из Sidecar в топик Kafka `fraud-stream`.
    - Реализация аналитического воркера на Go.
- [ ] **Scoring Rules**
    - **Rate Limiting:** Детекция аномалий (например, >30 msg/sec на токен).
    - **Content Analysis:** Regex-анализ на наличие фишинговых ссылок и стоп-слов.
- [ ] **Automated Kill Switch**
    - Интеграция с Redis для мгновенной блокировки трафика на уровне Sidecar.
    - API для автоматического удаления деплойментов нарушителей.

## 🔥 Phase 4: High Load Testing & Chaos Engineering
**Цель:** Валидация архитектурных пределов и сценариев восстановления.

- [ ] **Load Testing Suite (k6 / Locust)**
    - Разработка сценариев: эмуляция вебхуков, задержки обработки, массовые рассылки.
    - Бенчмаркинг вертикального (1 тяжелый бот) и горизонтального (1000 легких ботов) масштабирования.
- [ ] **Chaos Engineering**
    - Тестирование отказа компонентов (Kafka Brokers, Redis Master) под нагрузкой.
    - Проверка работы буферизации (Backpressure) при разрывах сети.
    - Валидация механизма **Scale-to-Zero** и холодного старта.

---

## 🏆 Target Technology Stack

К моменту завершения Roadmap платформа будет базироваться на следующих технологиях:

| Уровень | Компоненты |
| :--- | :--- |
| **Control Plane** | `Go API`, `ArgoCD` (GitOps), `cdk8s`, `PostgreSQL` |
| **Data Plane** | `Go Gateway` (Fiber/Fasthttp), `Apache Kafka` (KRaft), `Go Sidecar` (Franz-Go) |
| **Runtime** | `K3s`, `gVisor` (Sandbox), `Docker` |
| **Network Security** | `Cilium`/`Calico`, `iptables hijacking` |
| **Auth & Governance** | `Redis ACL`, `Kafka SASL`, `Anti-Fraud Worker` |
| **Observability** | `ClickHouse`, `Grafana` |
