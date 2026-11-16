# Тест-кейсы платёжного процесса

## E2E-тесты

| Название | Тип | Компоненты | Предусловия |
|---|---|---|---|
| Успешный платёж | E2E | Orchestrator, Payment Service, FraudCheck Service, Notification Service | У пользователя должно быть достаточно средств на счёте. Антифрод-сервис должен быть доступен. |
| Отказ антифрод-сервиса с компенсацией | E2E | Orchestrator, Payment Service, FraudCheck Service, Notification Service | У пользователя должно быть достаточно средств. Антифрод-сервис возвращает запрет. |
| Ручная проверка с одобрением оператора | E2E | Orchestrator, Payment Service, FraudCheck Service, Notification Service | Антифрод-сервис возвращает решение "ручная проверка". Оператор доступен. |
| Ручная проверка с отказом оператора | E2E | Orchestrator, Payment Service, FraudCheck Service, Notification Service | Антифрод-сервис возвращает решение "ручная проверка". Оператор отклоняет транзакцию. |
| Срабатывание cut-off-time при ручной проверке | E2E | Orchestrator, Payment Service, FraudCheck Service, Notification Service | Антифрод-сервис возвращает решение "ручная проверка". Оператор не принимает решение в течение 20 минут. |
| Ошибка списания средств | E2E | Orchestrator, Payment Service | У пользователя недостаточно средств на счёте. |

## Интеграционные тесты

### Orchestrator и Payment Service

| Название | Тип | Компоненты | Предусловия |
|---|---|---|---|
| Создание платежа и переход в статус "в обработке" | Интеграционный | Orchestrator, Payment Service, PostgreSQL | БД доступна. Данные платежа валидны. |
| Списание средств со счёта пользователя | Интеграционный | Orchestrator, Payment Service, PostgreSQL | Платёж в статусе CREATED. У пользователя достаточно средств. |
| Возврат средств при компенсации | Интеграционный | Orchestrator, Payment Service, PostgreSQL | Платёж в статусе REJECTED. Средства ранее списаны. |
| Отмена платежа при компенсации | Интеграционный | Orchestrator, Payment Service, PostgreSQL | Средства возвращены. Платёж в статусе REFUNDING. |
| Перевод средств контрагенту | Интеграционный | Orchestrator, Payment Service, PostgreSQL | Платёж в статусе APPROVED. |

### Orchestrator и FraudCheck Service

| Название | Тип | Компоненты | Предусловия |
|---|---|---|---|
| Антифрод-проверка: разрешение | Интеграционный | Orchestrator, FraudCheck Service | Платёж в статусе FUNDS_DEBITED. Антифрод-сервис доступен. |
| Антифрод-проверка: запрет | Интеграционный | Orchestrator, FraudCheck Service | Платёж в статусе FUNDS_DEBITED. Антифрод-сервис возвращает запрет. |
| Антифрод-проверка: направление на ручную проверку | Интеграционный | Orchestrator, FraudCheck Service | Платёж в статусе FUNDS_DEBITED. Антифрод-сервис возвращает решение "ручная проверка". |
| Таймаут ответа антифрод-сервиса | Интеграционный | Orchestrator, FraudCheck Service | Платёж в статусе FUNDS_DEBITED. Антифрод-сервис не отвечает в установленное время. |

### Orchestrator и Notification Service

| Название | Тип | Компоненты | Предусловия |
|---|---|---|---|
| Уведомление клиента об успешном платеже | Интеграционный | Orchestrator, Notification Service | Платёж в статусе FUNDS_TRANSFERRED. |
| Уведомление клиента об отмене платежа | Интеграционный | Orchestrator, Notification Service | Платёж в статусе CANCELLED. Компенсация выполнена. |
| Уведомление службы безопасности при отказе антифрода | Интеграционный | Orchestrator, Notification Service | Антифрод-сервис вернул запрет. Компенсация выполнена. |
| Повторная отправка уведомления при сбое | Интеграционный | Orchestrator, Notification Service | Notification Service временно недоступен. |

### Saga-оркестрация (Camunda)

| Название | Тип | Компоненты | Предусловия |
|---|---|---|---|
| Корректное сохранение состояния Saga в БД | Интеграционный | Orchestrator, PostgreSQL | Процесс запущен. БД доступна. |
| Восстановление процесса после перезапуска оркестратора | Интеграционный | Orchestrator, Camunda, PostgreSQL | Saga в промежуточном состоянии. Оркестратор перезапущен. |
| Компенсация в обратном порядке при сбое после списания | Интеграционный | Orchestrator, Payment Service, PostgreSQL | Средства списаны. Антифрод-сервис вернул запрет. |
| Идемпотентность повторного вызова DEBIT_FUNDS | Интеграционный | Orchestrator, Payment Service, PostgreSQL | Платёж в статусе CREATED. Вызов DEBIT_FUNDS выполняется повторно с тем же ключом идемпотентности. |
| Идемпотентность повторного вызова TRANSFER_FUNDS | Интеграционный | Orchestrator, Payment Service, PostgreSQL | Платёж в статусе APPROVED. Вызов TRANSFER_FUNDS выполняется повторно с тем же ключом идемпотентности. |
| Retry при временном сбое TRANSFER_FUNDS | Интеграционный | Orchestrator, Payment Service | Платёж в статусе APPROVED. Первый вызов TRANSFER_FUNDS возвращает ошибку, повторный завершается успехом. |
| Таймер cut-off-time запускается при переходе в MANUAL_REVIEW | Интеграционный | Orchestrator, Camunda | Антифрод-сервис вернул решение "ручная проверка". Таймер настроен на 20 минут. |
| Параллельное выполнение нескольких Saga без взаимного влияния | Интеграционный | Orchestrator, Payment Service, FraudCheck Service, PostgreSQL | Два платежа инициированы одновременно. |
