# Часть 1. (v5) Как гипервизор относится к виртуализации? Какие IT Tower можно виртуализовать (Вычисления, Хранилища, Сеть, Облачные Сервисы, ...?)

## Про гипервизор
Гипервизор - это основа всей виртуализации. Он берет физическое железо и делает из него виртуальные машины. Как это работает:

- Создает изолированные окружения для разных систем
- Распределяет ресурсы между виртуалками
- Управляет всеми процессами виртуализации

Бывают два типа гипервизоров:
1. Работают напрямую с железом (ESXi, Hyper-V)
2. Запускаются поверх обычной ОС (VirtualBox, VMware)

## Что можно виртуализировать

### Вычисления
- v-CPU
- v-ROM
- Изолированные ресурсы для VM

### Хранилища
- Виртуальные диски
- Распределенные хранилища
- Системы хранения данных

### Сети
- Виртуальные сетевые карты
- Виртуальные коммутаторы (типа hyperV)
- Программно-определяемые сети

### Облачные сервисы
- IaaS
- IDEs
- Готовые облачные приложения (YaCloudApps)

# Часть 2. (v8) Ваша команда разрабатывает новый Нетфликс. Какие сервисы вы будете использовать?

## Инфра

### Базовые сервисы
- Kubernetes
- AWS ECS или Google GKE

### Масштабирование
- ASG
- Elastic Load Balancer для распределения нагрузки
- Amazon RDS Multi-AZ развертывание для отказоустойчивости

## Система хранения

### Контент и файлы
- S3 или Google Cloud Storage для хранения видео
- CloudFront CDN для доставки
- Route 53 для маршрутизации

### Базы данных
- Cassandra как основная БД (прямо как у нетфликса)
- Redis для кэширования
- ElastiCache для метаданных
- DynamoDB для профилей

## Аналитика + эмэль

### Обработка данных
- AppMetrica потому что
- Kinesis для потоковых данных
- Elasticsearch как поисковый движок

### Крутые фичи
- SageMaker для рекомендательной системы
- Personalize для персонализации контента
- Comprehend для анализа отзывов

## Безопасность

### Аутентификация и доступ
- Предпочел бы проприетарное решение

### Защита
- Cloudflare для защиты от 99% пакостей


## Итоги (+ немного конкретики и обобщений)
- Глобальная инфра чтоб не лагало
- Умная система рекомендаций
- HLS streaming (придется у эпл закрытые библиотеки свизлить и вообще круто)
- Защита от пиратства (как вариант нанять полноценный отдел СБ)
- Мультиплатформ (при нативе🥀)