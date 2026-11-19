# Цикл агентного тестирования

Схема показывает не бизнес-процесс банка, а ход одного эксперимента: как агент
получает состояние, действует, проверяет результат и строит следующий шаг с
учётом предыдущих наблюдений.

```mermaid
%%{init: {"themeVariables": {"fontSize": "22px"}, "flowchart": {"nodeSpacing": 60, "rankSpacing": 70, "diagramPadding": 20}}}%%
flowchart TB
    setup["Подготовка состояния стенда"]
    observe["Наблюдение состояния"]
    choose["Выбор следующего действия"]
    execute["Выполнение на стенде"]
    record["Запись действий и состояния"]
    check["Проверка контрольных правил"]
    violation{"Нарушение найдено?"}
    budget{"Лимит шагов исчерпан?"}
    save["Сохранение сценария"]
    replay["Повторный запуск"]
    reproducible{"Ошибка повторилась?"}
    regression["Регрессионный тест"]
    finish(["Эксперимент завершён"])

    setup --> observe
    observe --> choose
    choose --> execute
    execute --> record
    record --> check
    check --> violation
    violation -->|"Нет"| budget
    budget -->|"Нет"| observe
    budget -->|"Да"| finish
    violation -->|"Да"| save
    save --> replay
    replay --> reproducible
    reproducible -->|"Да"| regression
    reproducible -->|"Нет"| finish
    regression --> finish

    style violation fill:#FFECBD,stroke:#FFC943
    style budget fill:#FFECBD,stroke:#FFC943
    style reproducible fill:#FFECBD,stroke:#FFC943
    style regression fill:#CDF4D3,stroke:#66D575
```

## Короткое описание

Агент видит только разрешённую часть состояния стенда и выбирает очередное
действие. После выполнения независимый модуль проверяет контрольные правила:
например, отсутствие двойного исполнения или соответствие внутреннего баланса
состоянию блокчейна. Если нарушение найдено, вся последовательность сохраняется
и запускается повторно. В результат эксперимента попадают только
воспроизводимые нарушения; для них формируется регрессионный тест.
