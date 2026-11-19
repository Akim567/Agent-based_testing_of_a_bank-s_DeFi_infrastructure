# Жизненный цикл банковской криптооперации

Схема показывает, через какие проверки проходит операция внутри стенда и в
каких точках она может быть отклонена или отправлена на ручную проверку.

```mermaid
%%{init: {"themeVariables": {"fontSize": "22px"}, "flowchart": {"nodeSpacing": 60, "rankSpacing": 70, "diagramPadding": 20}}}%%
flowchart TB
    request(["Создание операции"])
    risk["Проверка происхождения средств"]
    riskDecision{"Риск допустим?"}
    policy["Проверка прав и лимитов"]
    approvalDecision{"Согласования получены?"}
    signing["Подписание транзакции"]
    wallet["Выбор кошелька"]
    broadcast["Отправка в блокчейн"]
    confirmation{"Транзакция подтверждена?"}
    ledger["Обновление внутреннего учёта"]
    audit["Запись в аудит"]
    completed(["Операция завершена"])
    review["Ручная проверка"]
    rejected(["Операция отклонена"])
    pending(["Ожидание подтверждения"])

    request --> risk
    risk --> riskDecision
    riskDecision -->|"Да"| policy
    riskDecision -->|"Нет"| review
    review -->|"Не разрешена"| rejected
    review -->|"Разрешена"| policy
    policy --> approvalDecision
    approvalDecision -->|"Да"| signing
    approvalDecision -->|"Нет"| rejected
    signing --> wallet
    wallet --> broadcast
    broadcast --> confirmation
    confirmation -->|"Да"| ledger
    confirmation -->|"Нет"| pending
    pending -.->|"Повторная проверка"| confirmation
    ledger --> audit
    audit --> completed

    style riskDecision fill:#FFECBD,stroke:#FFC943
    style approvalDecision fill:#FFECBD,stroke:#FFC943
    style confirmation fill:#FFECBD,stroke:#FFC943
    style rejected fill:#FFCDC2,stroke:#FF7556
    style completed fill:#CDF4D3,stroke:#66D575
```

## Короткое описание

Операция последовательно проходит проверку риска, правил и необходимых
согласований. Только после этого система создаёт подпись и отправляет
транзакцию в блокчейн. Внутренний баланс изменяется после подтверждения
транзакции, а все действия сохраняются в журнале аудита. Агент будет пытаться
найти последовательность действий, при которой один из этих этапов работает
неправильно или обходится.
