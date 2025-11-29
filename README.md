# Домашнее задание к занятию «Репликация и масштабирование. Часть 2» - Савкин И.Н.

## Задание 1. Преимущества разных схем репликации

### 1. Активный Master + пассивный Slave (hot standby) Slave
**Основные преимущества:**
- Высокая доступность записи: при падении Master можно за секунды–минуты переключить приложение на Slave (failover).
- Нулевая или почти нулевая потеря данных при использовании semi-sync replication.
- Slave можно использовать для бэкапов, аналитики и тяжёлых отчётов без нагрузки на основной сервер.
- Простая и быстрая настройка автоматического переключения (MHA, Orchestrator, ProxySQL).

### 2. Один Master + несколько Slave-серверов
**Основные преимущества:**
- Масштабирование чтения: каждый новый Slave увеличивает пропускную способность SELECT-запросов.
- Геораспределение: можно разместить реплики в разных регионах → низкая задержка для пользователей.
- Разделение нагрузки: отдельные Slave под аналитику, поиск, отчёты, бэкапы.
- Лёгкое горизонтальное масштабирование: добавил ещё один Slave — получил +20–50 % производительности на чтение.


## Задание 2. Блок-схема архитектуры с шардингом

## Задание 2. Блок-схема архитектуры с шардингом

```mermaid
flowchart TD
    %% Клиенты
    App[Приложение / API] --> Proxy[Proxy / Router<br/>(Vitess / ProxySQL / MySQL Router)]

    %% Шарды
    subgraph Shard1 ["Шард 1 (shop_id % 8 = 0–1)"]
        direction TB
        Master1[(Master-1<br/>server-id=101)]:::master
        Replica1a[(Replica)]:::replica
        Replica1b[(Replica)]:::replica
        Replica1c[(Replica)]:::replica
        Master1 --> Replica1a & Replica1b & Replica1c
    end

    subgraph Shard2 ["Шард 2 (shop_id % 8 = 2–3)"]
        direction TB
        Master2[(Master-2<br/>server-id=102)]:::master
        Replica2a[(Replica)]:::replica
        Replica2b[(Replica)]:::replica
        Master2 --> Replica2a & Replica2b
    end

    subgraph ShardN ["Шард N (shop_id % 8 = 6–7)"]
        direction TB
        MasterN[(Master-N)]:::master
        ReplicaNa[(Replica)]:::replica
        ReplicaNb[(Replica)]:::replica
        MasterN --> ReplicaNa & ReplicaNb
    end

    subgraph Global ["Global Shard<br/>(справочные данные)"]
        direction TB
        GReplica1[(Read-Replica)]:::global
        GReplica2[(Read-Replica)]:::global
    end

    %% Связи
    Proxy --> Shard1
    Proxy --> Shard2
    Proxy --> ShardN
    Proxy --> Global

    %% Стили
    classDef master fill:#f44336,stroke:#900,color:#fff
    classDef replica fill:#2196F3,stroke:#1976D2,color:#fff
    classDef global fill:#4CAF50,stroke:#388E3C,color:#fff
    classDef proxy fill:#FF9800,stroke:#F57C00,color:#000
    class Proxy proxy
```
