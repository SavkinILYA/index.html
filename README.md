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


## Задание 2. План вертикального и горизонтального масштабирования + шардинг

```mermaid
graph TD
    A[Приложение] --> B(Proxy/Router);
    B --(Запросы на запись)---> SM(Shard Master);
    B --(Запросы на чтение)---> SR(Shard Replicas);
    B --(Глобальные запросы)---> GR(Global Read-Replicas);

    subgraph "Шарды данных (на основе shop_id)"
        direction LR
        SM1[Shard 1 Master] -- Репликация --> SR1A(Shard 1 Replica A);
        SM1 --> SR1B(Shard 1 Replica B);

        SM2[Shard 2 Master] -- Репликация --> SR2A(Shard 2 Replica A);
        SM2 --> SR2B(Shard 2 Replica B);

        SMN[Shard N Master] -- Репликация --> SRNA(Shard N Replica A);
        SMN --> SRNB(Shard N Replica B);
    end

    subgraph "Глобальный Шард (Справочные данные)"
        direction LR
        GR1[Global Replica 1]
        GR2[Global Replica 2]
        GR3[Global Replica 3]
    end

    classDef masterNode fill:#FFF3CD,stroke:#664D03,stroke-width:2;
    classDef replicaNode fill:#E9ECEF,stroke:#6C757D,stroke-width:2;
    classDef globalNode fill:#DAE5F0,stroke:#0C4F7F,stroke-width:2;
    classDef proxyNode fill:#F8D7DA,stroke:#721C24,stroke-width:2;
    classDef appNode fill:#D1E7DD,stroke:#0A3622,stroke-width:2;

    class A appNode;
    class B proxyNode;
    class SM1,SM2,SMN masterNode;
    class SR1A,SR1B,SR2A,SR2B,SRNA,SRNB replicaNode;
    class GR1,GR2,GR3 globalNode;
```
