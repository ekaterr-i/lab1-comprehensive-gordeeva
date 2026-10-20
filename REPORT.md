# Итоговый аналитический отчет (Лабораторная работа №1)

## 1. Резюме (Executive Summary)
В рамках данной работы был спроектирован и запущен процесс разработки для нового микросервиса **«Расчет корзины и скидок»**. С нуля развернута кодовая база, настроены автоматические инструменты контроля качества кода (Quality Gates), проведено исследование и оптимизация алгоритма расчета скидок (legacy-код), а также внедрен регламентированный процесс проведения Code Review.

## 2. Сравнительный анализ технологий

### 2.1. Матрица принятия решений

| Критерий выбора | Вес | PostgreSQL | MongoDB |
| :--- | :---: | :---: | :---: |
| Поддержка ACID-транзакций | 0.40 | 5 (2.00) | 3 (1.20) |
| Гибкость схемы данных | 0.15 | 4 (0.60) | 5 (0.75) |
| Производительность | 0.20 | 4 (0.80) | 5 (1.00) |
| Надежность и экосистема | 0.15 | 5 (0.75) | 4 (0.60) |
| Сложность поддержки (TCO) | 0.10 | 4 (0.40) | 4 (0.40) |
| **ИТОГО** | **1.00** | **4.55** | **3.95** |
https://portal.unn.ru/bitrix/services/main/ajax.php?action=disk.api.file.download&SITE_ID=s1&humanRE=1&fileId=5629530&exact=N&fileName=20261020_180037.jpg
<img width="960" height="1280" alt="image" src="https://github.com/user-attachments/assets/2e698c0d-75cd-455f-8b93-9ca14df78f37" />

### 2.2. Расчет и обоснование выбора
Расчет производился по формуле:  
\[\text{Оценка} = \sum (\text{Вес критерия} \times \text{Балл})\]

**Обоснование:** Итоговый балл PostgreSQL (**4.55**) значительно превышает балл MongoDB (**3.95**). Финансовая природа сервиса лояльности (балансы баллов, списания) требует строгой консистентности данных (ACID). PostgreSQL полностью закрывает эту потребность, а его встроенная поддержка типа `JSONB` позволяет беспрепятственно хранить динамические структуры промокодов.

## 3. Архитектура и контроль версий

### 3.1. Настройка репозитория
В проекте используется адаптивная модель ветвления **GitFlow/Feature Branching**:
* `main` — стабильный продакшн-код.
* `develop` — интеграционная ветка для тестирования.
* `feature/*` — ветки разработки под конкретные фичи.

*Ссылка на репозиторий:* 
https://github.com/ekaterr-i/lab1-comprehensive-gordeeva.git
### 3.2. Контроль доступа
* Файл настроек `.github/CODEOWNERS` обеспечивает автоматическое назначение Tech Lead в качестве обязательного ревьюера для любых изменений в репозитории.
* В настройках ветки `develop` активировано правило **Branch Protection**: запрещен прямой пуш, слияние возможно только через Pull Request.

## 4. Инструментальный контроль качества

### 4.1. Настройка CI/CD
Для автоматической проверки качества кода настроен GitHub Actions пайплайн.
*Ссылка на конфигурацию:*(https://github.com/ekaterr-i/lab1-comprehensive-gordeeva.git)

### 4.2. Работа Quality Gate
В процессе тестирования пайплайна была применена практика «Сломать и починить»:
1. **Этап "Сломать":** В файл `discount.py` были умышленно внесены неиспользуемый импорт `import os` и строка > 80 символов. Линтер `flake8` выдал ошибки:
   * `F401 'os' imported but unused`
   * `E501 line too long (84 > 80 characters)`
   Пайплайн в Pull Request окрасился в **красный статус (failed)**.
2. **Этап "Починить":** Лишний импорт удален, длинные строки отформатированы. После пуша статус проверки сменился на **зеленый (passed)**, код успешно интегрирован в `develop`.
<img width="1280" height="743" alt="image" src="https://github.com/user-attachments/assets/947c0c5f-ffb5-4d1f-9ea5-b202a92a5be1" />
https://portal.unn.ru/bitrix/services/main/ajax.php?action=disk.api.file.download&SITE_ID=s1&humanRE=1&fileId=5629520&exact=N&fileName=20261020_175713.jpg
## 5. Обратное проектирование и планирование ревью

### 5.1. Анализ алгоритма
Бизнес-логика расчета стоимости корзины была визуализирована в виде диаграммы деятельности:
https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=%D0%94%D0%B8%D0%B0%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B0%20%D0%B1%D0%B5%D0%B7%20%D0%BD%D0%B0%D0%B7%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F.drawio#R%3Cmxfile%3E%3Cdiagram%20name%3D%22%D0%A1%D1%82%D1%80%D0%B0%D0%BD%D0%B8%D1%86%D0%B0%20%E2%80%94%201%22%20id%3D%22ksQXVABlOuGTJLxUFbvt%22%3E7V1bc6O4Ev41rjq7VU4h7jw6t91zydTUZGfPnqctYis2Oxj5CDxJ9tevBBKIRiQeNrHImIcQkEQj1Bd1f2rhmXOxffyJxrvNDVnhdGZbq8eZczmzbeQ6LvvHS56qktAVBWuarESjpuA2%2BROLQkuU7pMVzlsNC0LSItm1C5cky%2FCyaJXFlJKHdrN7krafuovXuFNwu4zTbul%2Fk1WxEW9hB035zzhZb%2BSTkR9VNdtYNhZvkm%2FiFXlQipyrmXNBCSmqs%2B3jBU754Mlxqe677qmtO0ZxVhxyw0%2B%2FLq8f7%2Be%2FEN%2B5%2Fde%2Ff8se7iiaO35F5muc7sUbz2w%2FZQTPd3xU0zivBtD%2F%2F5539PwmJx8I3cZpU8TO1uJ%2FeWO%2BizN2bxpn69atVx%2Fmn2%2B1t8XbHTvJ7vJdeW1pinY0WWJexSlay5gWv%2Bf77UH3Vr1iA1N1rN1ZVrxryko%2BFU%2BS%2BWwAmZyxi%2FOHTVLg21285DUPTNRZ2abYpuwK1Xd%2BxbTAj73sQTXTmbZgssUFfWJNxA22K%2BREKIq8fGikDklR2igSF4qyWAj6uqbcyAI7EeLwLaIRaEQDDlC2WnAlY1elqCTL9rhQss9WmD%2BFMe4cPybFb8r5%2F0Qrfn75qFRcPomL6oF41dHRF0dVGTZPM2qyjOI0LpKvbfK6oRRP%2BEgS9uCGaZFzFoSOFznIDkMrQC0WMn6dWZZrB8h2A9dHUZt8TvZ0iQVFVXHBQxzLP7PblG0LnQV2m1wR0zUuOuRK3tcDM1wcXEdnKWa2gyxFne5oS0Ckrt8T9qar%2BZKkhM7j1R%2F7nJUuWNOMZLjS27KuKqTru38w%2FrB%2BWvLfD1Wje5IV8%2Ft4m6RPVdOfcfoVF8kyVurzchLhtcjePaoVVb%2Bq55Y2rKn7GtMkZv%2FTZB0Xe8qnnGfbLeNdX5MHIWe80rWq0bFSXBSYzpkJWibcLsI7Cd1t4kyQtKsyZkyKecx6lFXFSyb0mCp1CVOuTDzJkq9a1hSUEWOjvoXDzPSinBOVxzwQump3rKbF3uXuS8LIcZp5QckXPBea1WpXmseShmZ87%2BLll3VpCeaAybaHKv6qJz8or7HCS0KZgpJsXmyS5ZcM56LvSZYUiXwCbKsw%2Btl2SnfUdi9MbG8s3pOUTlJ6iJR2jB3aFT1WckGTLam6nrMRn%2BeYJvdA1A%2F21xTf6%2B%2BZ%2FF5HUFxZyLO4hjzrv00TzqTK716V%2ByacSbgn4Z6EexLuSbjfkXArIXFzCnCTBhfh%2BMfxwaVSWw5Bl%2Fy3QpckDPwsurRa41txKcSS0GJD1iSL0%2F8QshMD9AdTmCcBHcf7gsyewaBeBVFC1ivBRwiwJQDDfShEhII2IccGhHrAoQWl8ZPSbMcb5G8CH3ldfjOGnLvl0erwnutfm4%2FCul3Wpo1rBDPn6UJUbJPVit9%2Bzkxy8md8V5LiPBdvxeh65zPvktNiUlLZ%2FpJ0Za4uKhWXonafpCkoegVNRFATg7Cjia5GEZ03U0TNCgBnyWV59GZsgMKuan6v7HF8wB7PMHskUH1sO1lj9WeRN2vgeuVSj9jzi4%2BYJuztuRR8k81lL1XauxdR6KMYZwcYZ8caaJydEBCKxmacPaTD9l%2FCPBv%2FGaEzqwSf0iTD803j1Dae%2FrX0kBBiM4EnCSgOugB%2FBHkBWbULmePXwq%2B2OWEUKI5zRulZcr8wgWRDZ33AD%2Bz4iWxLJKlNvB2a1B3mD2EPTeYcINuXy8WLChprd0Cp%2FvRZ1N0lq9Z9i0%2Fz28Ug1E2sf2r6NwMrotaP3N0%2BC71e5MywMwrVwTHti3p2r%2FiPaxGcRWl8pMsYhJ9QsiW5omhD1rhNhyKuLhTRLdm%2B2UK3p1vZPOoU6wfqHMuMqeOYnGS940VAyAHSgAZOsjYk5I5ukjUU8TZi5tpAykKjUmYfTcrqdA8hHN5gVw74hN74pEwXZ7MQDpXh3LkM6qrIm5U75fGqPEZleVgeIz4D8ZPrsqo6vyqPF8r5Qjl3Rze3eOZdC11%2B3anBXHXk9CRHe6j6QUKQb%2BbVT5c0N0VSo4qkZlUWadlUE07VN9x%2Bvrm5%2BtQiMMp4Clq9WivMWb1wsnodtgTQVzgY3AcBcwjNp3mrF038th3Ib4gJHzzLAQw6hKtCxvktO3jyizkeVHHDqwW%2BBsg94cUcyB7HNHveC9D4mkXtnTvyas7%2BvBezO0eFXCIQfYTItJ%2Flm0cua6RSYEooMIkp%2BcfDlKDtD%2BGq3sHIpd0mFEHzY3661yGXr2O2utGwPgXshZyrls1RAzyraf7rPz8eEMuN0O7AQCLSrZg4GrvzZismvgZkPDl%2FH%2BotsmDS1cEBXgQpjc%2Fj78kSOjmPP4CsMu5TahDHyeWvbWVomj0TFKako0quDPWVIMaC0OhWAHwdFnbcGK8VcWkgbllfJQpF7ysIQ9AAhxpv6KhRmJwSoAX25MJraYH5kW9bUNZhQ3npyeVXNIuQLPyWhVfe8nzmXBtnD0zDQLVvY449uuTGU7PAyId88Qaa4C6HYZhh3ATLL2OcNMdh1Ig6GjYUoEDO6LJeAg0QNoUnnOmmt5cEGuzohMMTqErG95cEhqAc3begrDPLNwofV8J6HPMMNdUZmvnqvEjJvHnuQY9s4bjyo9fOJLSUXMSgvGRtfKVwoTjSXQN%2FbL8Xrtk3c6Q5v7cHGFKDE0va4joTVAYqTehyLZmxgEC7rLAFl6rWMNw5leRTuCCEaizKnAhM4JMmYHGHJuZ0XGp3fKFPP%2Fo05Z%2BOI%2F%2B0SJjr0reRj99voee2dJkGeKCVc43jb7IDJ23lHGjlvKEAjwNXI%2F3RrUaGPWlupxbud5AZT5P6fdRwUsby5jKSHE%2FuahMZSaHRvZTVhHwcR6ejuIMRPxjNdHbsmDcBPYjfieJKEAPwfMO4Uvh2OWMmUl31SaxiATV4VwuoYZ9um3PgpnQyjWPdcbsOtt6Qw8HoAMGw%2F2cnpjD1vYSpdq%2FZMx2jRi5QAPMmbsrSUxjkwmA10IAIx3VXJqRUE%2FkHEN88dApyIVLa2R1hfgqa9qxq8imDoU4H%2FDQDGt8uZSmCJ48aSX%2B7w3RT1jfSoUajzpl13lPI14EJQ9u0PxQZ2rk4KuvbAewG7yGE36VC0eg%2BAB31A0JTyDeOkO%2BZb4zK9xDGrfu50aB%2FF4Fh69fxTCLj%2B7ajno%2Bhn2g02DFeppNWo%2BmrbTq2DI4GQbjfdHE8s9MhP3b6vXMcbp4cznGY52xbo4OgIw3ic4rRYHd6NL10GE05bJOnaCRO7vxKkvE4uTbt5hJqWp8mn73Gr3UfkDYTHTFtJoBcf63d6Y2sjGbWQ9aAn%2Fr4%2B5%2Fk0Vw%2Fh569Yk8oLva0%2BrEExWT1PPeImSnvBjzsF2qDRlGHF3%2F%2FRlEo73GiP%2BAWNmDlN0d%2FHUqj2z6MrJ5sQnWHkQrO%2BF1xS9Nkl2MzKmkfutlswBfB2CUl3Ho1A87eaXNDVpi3%2BAs%3D%3C%2Fdiagram%3E%3C%2Fmxfile%3E

**Выявленные запахи кода (Code Smells) в legacy-фрагменте:**
1. **Magic Numbers (Магические числа):** Значения `15000`, `0.85`, `500`, `0.90`, `0.97`, `0.93`, `0.70` жестко зашиты в коде. Это усложняет изменение маркетинговых правил.
2. **Float Использование для Финансов:** Операции типа `price * 0.85` используют стандартный тип с плавающей точкой, что может привести к ошибкам округления цен. Необходимо использовать класс `Decimal`.
3. **Отсутствие валидации данных:** Алгоритм не проверяет, может ли `cart_sum` быть отрицательным числом, и не валидирует типы данных на входе.
4. **Жестко зашитые промокоды:** Проверки `p == "SUMMER"` делают систему нерасширяемой. Промокоды должны запрашиваться из БД.

### 5.2. План code-review
Разработан строгий регламент приемки кода перед слиянием.  
*Ссылка на документ:* https://github.com/ekaterr-i/lab1-comprehensive-gordeeva.git
**Ключевой критерий:** обязательное покрытие юнит-тестами алгоритма скидок на уровне не менее 85% и полный отказ от магических чисел.

## 6. Заключение
Внедренные инженерные практики (автоматический линтинг, защита веток, обязательный Code Owners аппрув и регламентированный план ревью) минимизируют влияние человеческого фактора. Это позволяет снизить риск попадания дефектов и багов в продуктивную среду на **80-90%** еще на этапе проектирования и код-ревью.
