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

```mermaid
flowchart TD
    A[Приложение
или API] --> B[Proxy / Router
Vitess │ ProxySQL │ MySQL Router]

    subgraph Shard1 ["Шард 1
(shop_id % 8 = 0–1)"]
        direction TB
        M1[(Master-1)]:::master
        R1a[(Replica)]:::replica
        R1b[(Replica)]:::replica
        R1c[(Replica)]:::replica
        M1 --> R1a & R1b & R1c
    end

    subgraph Shard2 ["Шард 2
(shop_id % 8 = 2–3)"]


        direction TB
        M2[(Master-2)]:::master
        R2a[(Replica)]:::replica
        R2b[(Replica)]:::replica
        M2 --> R2a & R2b
    end

    subgraph ShardN ["Шард N
(shop_id % 8 = 6–7)"]


        direction TB
        MN[(Master-N)]:::master
        RNa[(Replica)]:::replica
        RNb[(Replica)]:::replica
        MN --> RNa & RNb
    end

    subgraph Global ["Global Shard
справочные данные
категории, страны и т.д."]


        direction TB
        GR1[(Read-Replica)]:::global
        GR2[(Read-Replica)]:::global
    end

    B --> Shard1
    B --> Shard2
    B --> ShardN
    B --> Global

    classDef master fill:#e74c3c,stroke:#c0392b,color:white
    classDef replica fill:#3498db,stroke:#2980b9,color:white
    classDef global fill:#27ae60,stroke:#1e8449,color:white
    classDef proxy fill:#f39c12,stroke:#e67e22,color:black
    class B proxy
```
