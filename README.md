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
    %% Клиенты и прокси
    A[Приложение / API] --> B[Proxy / Router\n(Vitess / ProxySQL / MySQL Router)]

    %% Шарды
    subgraph Shard_1 [Шард 1 (shop_id % 8 = 0,1)]
        M1[(Master-1\nserver-id=101)]
        R1a[(Replica-1)]
        R1b[(Replica-2)]
        R1c[(Replica-3)]
        M1 --> R1a & R1b & R1c
    end

    subgraph Shard_2 [Шард 2 (shop_id % 8 = 2,3)]
        M2[(Master-2\nserver-id=102)]
        R2a[(Replica-1)]
        R2b[(Replica-2)]
        M2 --> R2a & R2b
    end

    subgraph Shard_N [Шард N (shop_id % 8 = 6,7)]
        MN[(Master-N\nserver-id=10N)]
        RNa[(Replica-1)]
        RNb[(Replica-2)]
        MN --> RNa & RNb
    end

    subgraph Global_Shard [Global Shard — справочные данные]
        GR1[(Read-Replica-1)]
        GR2[(Read-Replica-2)]
        GR3[(Read-Replica-3)]
    end

    %% Связи
    B --> Shard_1
    B --> Shard_2
    B --> Shard_N
    B --> Global_Shard

    %% Стили
    classDef master fill:#f46666,stroke:#333,color:white
    classDef replica fill:#66b3ff,stroke:#333,color:white
    classDef proxy fill:#ffcc00,stroke:#333,color:black
    classDef global fill:#95e1d3,stroke:#333,color:black

    class M1,M2,MN master
    class R1a,R1b,R1c,R2a,R2b,RNa,RNb,GR1,GR2,GR3 replica
    class B proxy
    class Global_Shard global
```
