# Практические задачи по системному дизайну

В этом документе представлены практические задачи по системному дизайну, которые помогут вам подготовиться к соответствующей части технического интервью в Яндексе и Озоне. Каждая задача включает описание, требования и подсказки для решения.

## Задача 1: Проектирование URL-сокращателя

### Описание
Необходимо спроектировать сервис для сокращения URL, аналогичный bit.ly или TinyURL.

### Требования
- Пользователи могут вводить длинный URL и получать короткий URL
- При переходе по короткому URL пользователь должен быть перенаправлен на оригинальный URL
- Сервис должен поддерживать высокую доступность и низкую латентность
- Система должна масштабироваться до миллиардов URL
- Желательно иметь возможность настраивать короткие URL (кастомные URL)
- Необходимо отслеживать статистику переходов

### Подсказки для решения

**Оценка масштаба:**
- Предположим 100 миллионов новых URL в месяц
- Соотношение чтения к записи: 10:1
- Средний размер URL: 100 байт
- Хранение URL в течение 5 лет

**Генерация коротких URL:**
1. **Хеш-функция**: MD5/SHA256 с последующим усечением до 6-8 символов
   - Плюсы: быстро, детерминированно
   - Минусы: возможны коллизии

2. **Инкрементальные ID с кодированием в base62**:
   - Плюсы: гарантия уникальности, последовательные ID
   - Минусы: централизованный счетчик может стать узким местом

3. **Распределенная генерация ID (например, Snowflake)**:
   - Плюсы: масштабируемость, уникальность
   - Минусы: сложность реализации, синхронизация времени

**Хранение данных:**
- Реляционная БД для соответствия коротких и длинных URL
- NoSQL для масштабируемости (например, DynamoDB, Cassandra)
- Кэширование часто используемых URL в Redis/Memcached

**API и компоненты:**
- REST API для создания и управления URL
- Сервис перенаправления для обработки запросов по коротким URL
- Сервис аналитики для отслеживания статистики
- Распределенный кэш для ускорения доступа

**Обработка коллизий:**
- Проверка существования URL перед вставкой
- Повторная генерация при коллизии
- Использование более длинных ключей при необходимости

**Масштабирование:**
- Шардирование базы данных по диапазону или хешу
- Репликация для высокой доступности
- CDN для глобального распределения

## Задача 2: Проектирование распределенного кэша

### Описание
Необходимо спроектировать распределенный кэш для хранения пар ключ-значение, аналогичный Redis или Memcached, но с возможностью горизонтального масштабирования.

### Требования
- Хранение пар ключ-значение с возможностью задания TTL
- Высокая производительность (низкая латентность)
- Масштабируемость до сотен узлов
- Устойчивость к отказам отдельных узлов
- Консистентность данных при масштабировании
- Поддержка базовых операций: get, set, delete

### Подсказки для решения

**Архитектура:**
- Клиентская библиотека для взаимодействия с кэшем
- Серверные узлы для хранения данных
- Сервис обнаружения для отслеживания доступных узлов
- Механизм шардирования для распределения данных

**Распределение данных:**
1. **Консистентное хеширование**:
   - Распределение ключей по кольцу хешей
   - Минимальное перераспределение при изменении числа узлов
   - Виртуальные узлы для равномерного распределения

2. **Репликация**:
   - Master-Slave для чтения/записи
   - Multi-Master для высокой доступности
   - Quorum-based подход для согласованности

**Согласованность:**
- Eventual consistency для высокой производительности
- Quorum reads/writes для строгой согласованности
- Версионирование данных (vector clocks)

**Обработка отказов:**
- Heartbeat механизм для обнаружения отказов
- Автоматическое перераспределение данных
- Graceful degradation при частичных отказах

**Масштабирование:**
- Горизонтальное масштабирование добавлением узлов
- Автоматическое ребалансирование данных
- Мониторинг нагрузки и автоматическое масштабирование

**Оптимизации:**
- LRU/LFU политики вытеснения
- Сжатие данных
- Батчинг операций
- Предварительное чтение (prefetching)

## Задача 3: Проектирование системы обработки платежей

### Описание
Необходимо спроектировать систему обработки платежей для крупного e-commerce сайта, которая будет обрабатывать различные типы платежей (кредитные карты, электронные кошельки, банковские переводы).

### Требования
- Обработка различных типов платежей
- Высокая надежность и отказоустойчивость
- Безопасность и соответствие стандартам (PCI DSS)
- Масштабируемость до миллионов транзакций в день
- Поддержка возвратов и отмены платежей
- Интеграция с внешними платежными системами
- Отчетность и аналитика

### Подсказки для решения

**Архитектура:**
- API Gateway для приема запросов на платежи
- Сервис платежей для обработки транзакций
- Сервис проверки мошенничества
- Сервис учета (ledger) для финансовых записей
- Сервис интеграции с внешними платежными системами
- Сервис отчетности и аналитики

**Обработка платежей:**
1. **Процесс авторизации**:
   - Валидация платежной информации
   - Проверка на мошенничество
   - Резервирование средств
   - Подтверждение платежа

2. **Состояния транзакции**:
   - Создана
   - Авторизована
   - Захвачена (средства списаны)
   - Отменена
   - Возвращена

**Безопасность:**
- Шифрование данных карт (в покое и при передаче)
- Токенизация платежных данных
- Сегментация сети и контроль доступа
- Аудит и логирование

**Надежность:**
- Асинхронная обработка с очередями сообщений
- Идемпотентность операций
- Транзакционная согласованность
- Резервное копирование и восстановление

**Масштабирование:**
- Горизонтальное масштабирование сервисов
- Шардирование базы данных
- Кэширование для часто запрашиваемых данных
- Асинхронная обработка для неблокирующих операций

**Интеграции:**
- Адаптеры для различных платежных систем
- Стандартизированный внутренний API
- Механизмы повторных попыток и обработки ошибок
- Мониторинг состояния внешних систем

## Задача 4: Проектирование системы рекомендаций

### Описание
Необходимо спроектировать систему рекомендаций для онлайн-магазина, которая будет предлагать пользователям товары на основе их предпочтений и поведения.

### Требования
- Персонализированные рекомендации товаров
- Обработка данных о поведении пользователей в реальном времени
- Масштабируемость до миллионов пользователей и товаров
- Низкая латентность при генерации рекомендаций
- Возможность A/B тестирования различных алгоритмов
- Аналитика эффективности рекомендаций

### Подсказки для решения

**Архитектура:**
- Сервис сбора данных о пользовательском поведении
- Система обработки данных (batch и streaming)
- Сервис машинного обучения
- Сервис рекомендаций
- Сервис A/B тестирования
- Сервис аналитики

**Алгоритмы рекомендаций:**
1. **Коллаборативная фильтрация**:
   - User-based: рекомендации на основе похожих пользователей
   - Item-based: рекомендации на основе похожих товаров
   - Matrix factorization: выявление скрытых факторов

2. **Контентная фильтрация**:
   - На основе характеристик товаров
   - На основе профиля пользователя
   - Гибридные подходы

**Обработка данных:**
- Batch-обработка для обучения моделей
- Stream-обработка для реального времени
- Feature engineering и нормализация
- Хранение в колоночных БД (ClickHouse, BigQuery)

**Генерация рекомендаций:**
- Предварительное вычисление для популярных сценариев
- Кэширование персонализированных рекомендаций
- Fallback на популярные товары при отсутствии данных
- Контекстные рекомендации (время, местоположение)

**Масштабирование:**
- Распределенное обучение моделей
- Шардирование данных пользователей и товаров
- Асинхронное обновление моделей
- Репликация сервисов рекомендаций

**Оценка эффективности:**
- Метрики: CTR, конверсия, выручка
- A/B тестирование различных алгоритмов
- Мониторинг разнообразия рекомендаций
- Анализ долгосрочного влияния на пользовательское поведение

## Задача 5: Проектирование системы обмена сообщениями (чат)

### Описание
Необходимо спроектировать масштабируемую систему обмена сообщениями, поддерживающую личные и групповые чаты, с доставкой сообщений в реальном времени.

### Требования
- Поддержка личных и групповых чатов
- Доставка сообщений в реальном времени
- Отображение статуса пользователей (онлайн/офлайн)
- Хранение истории сообщений
- Поддержка медиа-файлов (изображения, видео)
- Уведомления о новых сообщениях
- Масштабируемость до миллионов пользователей

### Подсказки для решения

**Архитектура:**
- Сервис аутентификации и авторизации
- Сервис сообщений для обработки и маршрутизации
- Сервис присутствия для отслеживания статуса
- Сервис хранения для истории сообщений
- Сервис уведомлений для push-нотификаций
- Сервис медиа для обработки файлов

**Протоколы реального времени:**
1. **WebSocket**:
   - Двунаправленная связь
   - Низкая задержка
   - Поддержка большинством браузеров

2. **Альтернативы**:
   - Long Polling
   - Server-Sent Events (SSE)
   - MQTT для мобильных устройств

**Хранение данных:**
- NoSQL БД для сообщений (Cassandra, MongoDB)
- Реляционная БД для метаданных и отношений
- Объектное хранилище для медиа-файлов
- Redis для кэширования и управления присутствием

**Доставка сообщений:**
- Очереди сообщений для асинхронной обработки
- Публикация/подписка для групповых чатов
- Подтверждения доставки и прочтения
- Обработка офлайн-пользователей

**Масштабирование:**
- Шардирование по пользователям или чатам
- Федерация серверов для географического распределения
- Оптимизация для горячих пользователей
- Балансировка нагрузки WebSocket-соединений

**Оптимизации:**
- Сжатие сообщений
- Батчинг для групповых чатов
- Ограничение частоты сообщений
- Кэширование последних сообщений

## Задача 6: Проектирование системы хранения и обработки логов

### Описание
Необходимо спроектировать систему для сбора, хранения и анализа логов с тысяч серверов, обрабатывающую терабайты данных ежедневно.

### Требования
- Сбор логов с различных источников (серверы, приложения, сетевые устройства)
- Обработка и индексация логов в реальном времени
- Хранение логов с возможностью быстрого поиска
- Масштабируемость до петабайт данных
- Визуализация и аналитика
- Настраиваемые алерты на основе паттернов в логах
- Управление доступом и безопасность

### Подсказки для решения

**Архитектура:**
- Агенты сбора логов на источниках
- Брокеры сообщений для буферизации
- Обработчики для парсинга и обогащения
- Хранилище для индексированных логов
- Поисковый движок для запросов
- Сервис визуализации и дашбордов
- Сервис алертов и нотификаций

**Сбор логов:**
1. **Агенты**:
   - Filebeat, Fluentd, Logstash
   - Минимальное влияние на производительность
   - Буферизация при сетевых проблемах

2. **Транспорт**:
   - Kafka для высокой пропускной способности
   - NATS для низкой латентности
   - Сжатие и батчинг для оптимизации

**Обработка логов:**
- Парсинг структурированных и неструктурированных логов
- Нормализация форматов
- Обогащение дополнительной информацией
- Фильтрация и агрегация

**Хранение и индексация:**
- Elasticsearch для полнотекстового поиска
- ClickHouse для аналитических запросов
- Политики хранения и архивирования
- Шардирование по времени или источнику

**Масштабирование:**
- Горизонтальное масштабирование всех компонентов
- Автоматическое шардирование индексов
- Балансировка нагрузки на брокеры
- Распределенная обработка запросов

**Визуализация и алерты:**
- Дашборды с ключевыми метриками
- Поиск и фильтрация логов
- Алерты на основе пороговых значений или аномалий
- Интеграция с системами мониторинга

## Задача 7: Проектирование системы потоковой обработки данных

### Описание
Необходимо спроектировать систему для обработки потоковых данных в реальном времени, например, для анализа активности пользователей на веб-сайте или обработки данных с IoT-устройств.

### Требования
- Прием и обработка миллионов событий в секунду
- Минимальная задержка обработки (< 1 секунды)
- Масштабируемость и отказоустойчивость
- Поддержка сложных операций (агрегация, объединение потоков)
- Интеграция с системами хранения для долгосрочного анализа
- Мониторинг и управление потоками данных

### Подсказки для решения

**Архитектура:**
- Источники данных (producers)
- Брокеры сообщений
- Обработчики потоков
- Хранилище состояний
- Приемники данных (consumers)
- Система мониторинга и управления

**Компоненты потоковой обработки:**
1. **Брокеры сообщений**:
   - Kafka для высокой пропускной способности
   - Pulsar для гибкой маршрутизации
   - Kinesis для AWS-интеграции

2. **Фреймворки обработки**:
   - Flink для сложной обработки с состоянием
   - Spark Streaming для интеграции с ML
   - Samza для интеграции с Kafka

**Модели обработки:**
- Windowing (скользящие окна, сессии)
- Stateful processing (агрегация, join)
- CEP (Complex Event Processing)
- Exactly-once семантика

**Хранение состояний:**
- Локальное хранилище с чекпоинтами
- Распределенные хранилища (RocksDB)
- Снапшоты для восстановления
- Репликация для надежности

**Масштабирование:**
- Параллельная обработка с партиционированием
- Динамическое масштабирование обработчиков
- Балансировка нагрузки между партициями
- Автоматическое восстановление при сбоях

**Оптимизации:**
- Локальность данных
- Минимизация сетевого взаимодействия
- Батчинг для эффективной обработки
- Адаптивное управление ресурсами

## Задача 8: Проектирование распределенной поисковой системы

### Описание
Необходимо спроектировать распределенную поисковую систему, способную индексировать и искать по миллиардам документов с низкой латентностью.

### Требования
- Индексация миллиардов документов
- Поиск по ключевым словам с ранжированием результатов
- Поддержка фильтрации и фасетного поиска
- Низкая латентность поиска (< 200 мс)
- Масштабируемость и отказоустойчивость
- Инкрементальное обновление индекса
- Поддержка сложных поисковых запросов (фразы, операторы)

### Подсказки для решения

**Архитектура:**
- Краулеры для сбора данных
- Индексаторы для обработки документов
- Хранилище индекса
- Поисковые узлы для обработки запросов
- Ранжировщики для сортировки результатов
- Сервис кэширования для популярных запросов

**Индексация:**
1. **Инвертированный индекс**:
   - Отображение термов на документы
   - Позиции термов для фразового поиска
   - Метаданные для ранжирования

2. **Процесс индексации**:
   - Извлечение текста и метаданных
   - Токенизация и нормализация
   - Построение индекса
   - Оптимизация и сжатие

**Распределение данных:**
- Шардирование индекса по документам
- Репликация для надежности и производительности
- Партиционирование по термам для параллельного поиска
- Балансировка нагрузки между узлами

**Поиск и ранжирование:**
- Разбор и оптимизация запросов
- Параллельный поиск по шардам
- Объединение и ранжирование результатов
- Релевантность на основе TF-IDF, BM25, ML-моделей

**Масштабирование:**
- Горизонтальное масштабирование всех компонентов
- Инкрементальное обновление индекса
- Кэширование популярных запросов и результатов
- Распределенное выполнение сложных запросов

**Оптимизации:**
- Сжатие индекса
- Ранняя остановка при достаточном количестве результатов
- Предварительное вычисление для частых запросов
- Адаптивное ранжирование на основе обратной связи

## Задача 9: Проектирование системы обработки заказов для e-commerce

### Описание
Необходимо спроектировать систему обработки заказов для крупного e-commerce сайта, которая будет обрабатывать весь жизненный цикл заказа от создания до доставки.

### Требования
- Создание и управление заказами
- Проверка наличия товаров на складе
- Обработка платежей
- Управление доставкой
- Обработка возвратов и отмен
- Масштабируемость до миллионов заказов
- Устойчивость к пиковым нагрузкам (распродажи)

### Подсказки для решения

**Архитектура:**
- Сервис заказов для управления жизненным циклом
- Сервис инвентаря для проверки наличия товаров
- Сервис платежей для обработки транзакций
- Сервис доставки для управления логистикой
- Сервис уведомлений для информирования клиентов
- Сервис аналитики для бизнес-метрик

**Жизненный цикл заказа:**
1. **Состояния заказа**:
   - Создан
   - Подтвержден
   - Оплачен
   - Собран
   - Отправлен
   - Доставлен
   - Отменен
   - Возвращен

2. **Переходы между состояниями**:
   - Валидация при каждом переходе
   - Атомарные операции
   - Аудит изменений

**Обработка данных:**
- Транзакционная согласованность для критичных операций
- Eventual consistency для некритичных обновлений
- CQRS для разделения чтения и записи
- Event sourcing для аудита и восстановления

**Интеграции:**
- Платежные системы
- Системы управления складом
- Логистические сервисы
- CRM-системы
- Системы аналитики

**Масштабирование:**
- Шардирование по ID заказа или клиента
- Асинхронная обработка с очередями сообщений
- Кэширование часто запрашиваемых данных
- Репликация для чтения

**Устойчивость к пиковым нагрузкам:**
- Очереди для сглаживания пиков
- Автомасштабирование сервисов
- Деградация функциональности при необходимости
- Приоритизация критичных операций

## Задача 10: Проектирование системы мониторинга и алертинга

### Описание
Необходимо спроектировать систему мониторинга и алертинга для крупной распределенной инфраструктуры, включающей тысячи серверов и сотни сервисов.

### Требования
- Сбор метрик с серверов и приложений
- Визуализация метрик в реальном времени
- Настраиваемые алерты на основе пороговых значений и аномалий
- Агрегация и коррелирование алертов
- Масштабируемость до миллионов метрик
- Минимальное влияние на мониторируемые системы
- Интеграция с системами управления инцидентами

### Подсказки для решения

**Архитектура:**
- Агенты сбора метрик на серверах
- Агрегаторы для предварительной обработки
- Временные ряды для хранения метрик
- Сервис алертинга для обнаружения проблем
- Сервис нотификаций для уведомлений
- Сервис визуализации для дашбордов

**Сбор метрик:**
1. **Типы метрик**:
   - Системные (CPU, память, диск, сеть)
   - Приложения (latency, throughput, error rate)
   - Бизнес-метрики (конверсии, транзакции)
   - Синтетические проверки (проверки доступности)

2. **Методы сбора**:
   - Pull (Prometheus)
   - Push (StatsD, Telegraf)
   - Гибридные подходы

**Хранение метрик:**
- Временные ряды (Prometheus, InfluxDB, TimescaleDB)
- Даунсэмплинг для долгосрочного хранения
- Компрессия данных
- Шардирование по времени

**Алертинг:**
- Пороговые алерты (статические правила)
- Динамические пороги (на основе исторических данных)
- Обнаружение аномалий (ML-подходы)
- Снижение шума (дедупликация, группировка)

**Масштабирование:**
- Федерация для распределенного сбора
- Иерархическая агрегация
- Горизонтальное масштабирование хранилища
- Оптимизация запросов для визуализации

**Интеграции:**
- Системы управления инцидентами (PagerDuty, OpsGenie)
- Чаты и мессенджеры (Slack, Telegram)
- Системы тикетов (Jira, ServiceNow)
- Системы автоматизации для самовосстановления

## Заключение

В этом документе мы рассмотрели 10 практических задач по системному дизайну, которые часто встречаются на технических интервью в компаниях уровня Яндекс и Озон. Для каждой задачи были представлены описание, требования и подсказки для решения.

При решении задач по системному дизайну на интервью важно:

1. **Структурированно подходить к решению**:
   - Уточнить требования и ограничения
   - Оценить масштаб системы
   - Предложить высокоуровневый дизайн
   - Углубиться в детали ключевых компонентов
   - Обсудить компромиссы и альтернативы

2. **Демонстрировать понимание ключевых концепций**:
   - Масштабируемость и производительность
   - Надежность и отказоустойчивость
   - Согласованность данных
   - Безопасность и приватность

3. **Адаптировать решение под конкретные требования**:
   - Учитывать специфику задачи
   - Предлагать решения, соответствующие масштабу
   - Обосновывать архитектурные решения

Практика решения подобных задач поможет вам развить системное мышление и уверенно пройти техническое интервью в компаниях уровня Яндекс и Озон.
