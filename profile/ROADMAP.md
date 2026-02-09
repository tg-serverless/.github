# 🗺️ Platform Roadmap

## 👁️ Phase 1: Architecture & Traffic Control

* [ ] **Shuffle Sharding**
* Делаем shuffle sharding вместо обычной репликации

* [ ] **Egress Gateway (go, fiber)**
* Пишем Gateway для анализа исходящего трафика ботов + передаем в deployments ботов BotID и на Engress Gateway заменяем на telegram bot token

## 🛡️ Phase 2: Zero Trust Security


* [ ] **Network Hardening (Cilium / Calico)**
* Бот не видит соседние поды, не имеет доступа к внутренней сети кластера (`10.0.0.0/8`) и API облака (Metadata service).


* [ ] **Hardened Runtime (gVisor)**
* Внедряем `runsc` (gVisor) для изоляции на уровне ядра.


## 🕵️ Phase 3: Anti-Fraud & Governance

* [ ] **Fraud Detection Engine**
* Анализируем трафик ботов с целью нахождения ботов "нарушающих EULA". 


* [ ] **Scoring Rules**
* **Rate Limiting:** Ловим аномалии (например, резкий скачок >30 RPS на токен).
* **Content Analysis:** Быстрый regex-анализ и эвристики на фишинг и стоп-слова.


## 🔥 Phase 4: Battle Testing

* [ ] **High Load Benchmarks (k6)**



---

## 🏆 Целевой технологический стек

| Слой | Технологии |
| --- | --- |
| **Control Plane** | `Go API`, `ArgoCD` (GitOps), `cdk8s` (IaC), `PostgreSQL` |
| **Data Plane** | `Go Gateway` (Fiber/Fasthttp), `Apache Kafka` (KRaft), `Go Sidecar` (Franz-Go) |
| **Runtime** | `K3s` (Orchestration), `gVisor` (Sandbox), `Docker` |
| **Security** | `Cilium` (Network Policy), `Redis ACL`, `Kafka SASL (Scram)` |
| **Observability** | `ClickHouse` (Logs/Metrics), `Grafana` |
