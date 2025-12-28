### <a name="_b7urdng99y53"></a>**Название: Проектирование мультитенантной SaaS-платформы для свиноферм**
### <a name="_hjk0fkfyohdk"></a>**Автор: Терехин Алексей**
### <a name="_uanumrh8zrui"></a>**Дата: 27.12.2025**

Контекст и проблема
После успешного внедрения MVP на собственных фермах «АгроТех Х» планирует вывести решение на рынок как коммерческую SaaS-платформу. Это требует преобразования внутренней системы в мультитенантную архитектуру с поддержкой:

- Полной изоляции данных между клиентами
- Гибкой системы подписок и биллинга
- API для интеграции клиентов
- Панели самообслуживания
- Масштабирования по мере роста числа клиентов

Основная проблема: Выбор стратегии мультитенантности, найти баланс между изоляцией данных, стоимостью и сложностью эксплуатации.

Задача 1: Проектирование мультитенантной архитектуры
Варианты изоляции данных
Вариант 1: Изоляция на уровне схем БД (Shared Database, Separate Schemas)
[C3_db_schema_isolation.puml](https://github.com/gremrok/MSA-AgroTech/blob/main/Task%205/C3_db_schema_isolation.puml)

Вариант 2: Изоляция на уровне отдельных БД (Database per Tenant)
[C3_db_isolation.puml](https://github.com/gremrok/MSA-AgroTech/blob/main/Task%205/C3_db_isolation.puml)

Вариант 3: Изоляция на уровне инстансов (Instance per Tenant)
[C3_instance.puml](https://github.com/gremrok/MSA-AgroTech/blob/main/Task%205/C3_instance.puml)

Сравнение вариантов изоляции
|Критерий|	Вариант 1 (Схемы)|	Вариант 2 (БД на клиента)|	Вариант 3 (Инстансы)|
| :-: | :- | :- | :- |
|Изоляция данных|	Средняя (логическая)|	Высокая (физическая)|	Максимальная (полная)|
|Безопасность| 	Средняя (риск SQL injection между схемами)	|Высокая|	Максимальная|
|Стоимость эксплуатации	|Низкая (1 БД)|	Средняя (пул БД)|	Высокая (много инстансов)|
|Масштабируемость|	Ограничена одним инстансом БД|	Хорошая (шардинг по БД)|	Отличная (горизонтальное масштабирование)|
|Сложность разработки	|Низкая (tenant_id в каждом запросе)|	Средняя (роутинг соединений)|	Высокая (управление множеством инстансов)|
|Производительность	|Риск contention при росте|	Хорошая (раздельные ресурсы)|	Отличная (выделенные ресурсы)|
|Резервное копирование	|Простое (одна БД)|	Сложное (много БД)	|Очень сложное (много инстансов)|
|Подходит для|	Стартапы, малый бизнес	|Средний бизнес, регулируемые отрасли|	Крупные предприятия, высокие требования безопасности|


Решение по мультитенантности
Выбран гибридный подход:

Для SMB-клиентов (до 10 ферм): Database per Tenant (Вариант 2)

Для Enterprise-клиентов (10+ ферм): Instance per Tenant (Вариант 3)

Обоснование:

- Баланс между изоляцией и стоимостью
- Возможность предлагать разные тарифные планы

Задача 2: Система биллинга и монетизации
Архитектура системы биллинга
[C2_billing.puml](https://github.com/gremrok/MSA-AgroTech/blob/main/Task%205/C2_billing.puml)

Модель тарифных планов:
```
tariff_plans:
  starter:
    name: "Стартовый"
    price: 9900 ₽/мес
    limits:
      farms: 3
      cameras_per_farm: 10
      storage_days: 30
      users: 5
    features:
      - basic_analytics
      - email_alerts
      - mobile_app
  
  professional:
    name: "Профессиональный"
    price: 29900 ₽/мес
    limits:
      farms: 10
      cameras_per_farm: 30
      storage_days: 90
      users: 20
    features:
      - advanced_analytics
      - sms_alerts
      - api_access
      - custom_metrics
  
  enterprise:
    name: "Корпоративный"
    price: "Индивидуально"
    limits: custom
    features:
      - dedicated_instance
      - sla_99_9
      - on_premise_option
      - custom_integrations
```

Задача 3: Интеграции с клиентами
API Gateway и Dev Portal
[C2_api_integration.puml](https://github.com/gremrok/MSA-AgroTech/blob/main/Task%205/C2_api_integration.puml)

Scope доступного функционала через API
```
yaml
api_scopes:
  read_only:
    endpoints:
      - GET /v1/herd/animals
      - GET /v1/health/metrics
      - GET /v1/events
    use_case: "Интеграция с BI-системами"
  
  basic_write:
    endpoints:
      - все read_only
      - POST /v1/herd/animals/{id}/events
      - PUT /v1/feeding/schedules
    use_case: "Базовая интеграция с ERP"
  
  full_access:
    endpoints:
      - все endpoints
      - WebSocket /v1/realtime
      - Webhooks configuration
    use_case: "Продвинутые интеграции"
  
  partner_access:
    endpoints:
      - специфичные партнерские endpoints
      - доступ к агрегированным анонимным данным
    use_case: "Партнеры для аналитики/ML"
```

Задача 4: Финальная архитектура To-Be
Диаграмма C1: SaaS Platform
[C1_saas.puml](https://github.com/gremrok/MSA-AgroTech/blob/main/Task%205/C1_saas.puml)

Диаграмма C2: Пример интеграции с «АгроТех Х» как клиентом
[C2_saas_agrotech_integration.puml](https://github.com/gremrok/MSA-AgroTech/blob/main/Task%205/C1_saas_agrotech_integration.puml)
