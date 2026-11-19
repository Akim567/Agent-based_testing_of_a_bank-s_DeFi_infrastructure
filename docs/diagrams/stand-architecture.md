# Архитектура испытательного стенда

Схема показывает состав стенда и границу между тестирующим агентом, упрощённой
банковской системой, блокчейном и независимой проверкой результата.

```mermaid
%%{init: {"themeVariables": {"fontSize": "22px"}, "flowchart": {"nodeSpacing": 60, "rankSpacing": 70, "diagramPadding": 20}}}%%
flowchart TB
    subgraph testing ["Контур тестирования"]
        direction LR
        agent["AI-агент"]
        runner["Исполнитель сценариев"]
    end

    subgraph banking ["Эмулятор банка"]
        direction TB
        bankApi["Интерфейс банка"]
        policy["Права, лимиты и согласования"]
        ledger[("Внутренний учёт")]
        signing["Модуль подписи"]
        eventLog[("Журнал событий")]
    end

    subgraph chain ["Блокчейн-контур"]
        blockchain[("Тестовый блокчейн")]
    end

    subgraph verification ["Независимая проверка"]
        direction TB
        oracle["Модуль проверки"]
        result[("Результаты экспериментов")]
    end

    agent -->|"Выбирает действие"| runner
    runner -->|"Отправляет запрос"| bankApi
    bankApi --> policy
    policy -->|"Разрешает операцию"| ledger
    ledger --> signing
    signing -->|"Публикует транзакцию"| blockchain

    bankApi -.->|"Записывает событие"| eventLog
    policy -.->|"Записывает решение"| eventLog
    signing -.->|"Записывает подпись"| eventLog

    ledger -->|"Передаёт состояние"| oracle
    blockchain -->|"Передаёт состояние"| oracle
    eventLog -->|"Передаёт историю"| oracle
    oracle -->|"Фиксирует проверку"| result
    result -.->|"Возвращает наблюдение"| agent

    style testing fill:#DCCCFF,stroke:#874FFF
    style banking fill:#C2E5FF,stroke:#3DADFF
    style chain fill:#C6FAF6,stroke:#5AD8CC
    style verification fill:#FFECBD,stroke:#FFC943
```

## Как читать схему

1. Агент не обращается к базе данных или блокчейну напрямую: он действует через интерфейс банка.
2. Эмулятор банка проверяет права, лимиты и согласования, меняет внутренний учёт и подписывает разрешённую транзакцию.
3. Независимый модуль сопоставляет внутренний учёт, состояние блокчейна и журнал событий.
4. Результат проверки становится новым наблюдением агента и сохраняется для последующего анализа.

Реальные HSM, AML-сервисы и холодное хранилище в первой версии заменяются
программными моделями. Схема описывает исследовательский стенд, а не
инфраструктуру конкретного банка.
