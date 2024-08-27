# План курса

Каждая тема содержит теоретический материал с ссылками на источники,
некоторые из тем содержат практические задания для закрепления материала.

## Список тем:

1. Android Base
    1. Android Internals
        1. Intent
        2. Activity
        3. Services
        4. Content Providers
        5. Broadcast Receivers
        6. ViewModel
        7. Zygote
        8. IPC
    2. View
        1. Views
        2. Canvas
        3. RecyclerView
        4. Fragment
        5. Animations
    3. Compose
        1. Введение. Что такое функциональное программирование
        2. Слот машина
        3. Работа 'remember,' 'mutableState,' 'CompositionLocal'
        4. Side Effects
        5. Принципы работы с рекомпозицией
        6. Modifiers
        7. LazyColumn, LazyList, LazyRow
        8. Animations
    4. Навигация
        1. Место навигации в прхитектуре проекта
        2. Основные библиотеки и способы навигации:
            1. Activity
            2. Fragment:
                1. Fragment Manager
                2. Cicerone
                3. Navigation Component
            3. Compose:
                1. Compose Navigation Component
                2. Manual Switch structure
                3. Appyx
                4. Compose Destinations
                5. Voyager
    5. UI Profiling
    6. Persistent Storage libraries
        1. SQLite
        2. Room
        3. SQLDelight
        4. SharedPreferences
        5. DataStore
        6. KStore
    7. Network libraries
        1. WebSockets
        2. OkHttp
        3. Retrofit
        4. Ktor
    8. Dependency Injection:
        1. Теория DI. Отличия Dependency Injection и Service Locator. Manual DI
        2. Dagger2
        3. Hilt
        4. Koin
        5. Kodein
    9. Unit Testing
        1. JUint4
        2. JUint5
        3. MockK
        4. Strikt
        5. Coroutines Test
        6. Turbine
    10. UI tests
        1. Espresso
        2. Kspresso
    11. Build Process
        1. Dex, MultiDex
        2. R8 и D8
        3. Proguard
        4. Android Lint
        5. Gradle API
            1. Основной синтаксис конфигурации проекта
            2. Version Catalog
            3. API и ABI совместимость
            4. Conventions Plugins
            5. KMP конфигурация
        6. Gradle Optimizations
        7. Кодогенерация
    12. App publishing and monitoring
        1. Google Play Console
        2. RuStore Console
        3. Аналитика, метрики для ведения аналитики
        4. Крашлитика
2. [Архитектура](architecture.md)
    1. Принципы ООП
    2. SOLID
    3. Clean Architecture
    4. KISS
    5. DRY
    6. Основные паттерны проектирования:
        1. Фасад
        2. Строитель
        3. Наблюдатель
        4. Декоратор
        5. Сингтон
        6. Шина
        7. Стратегия
    7. Архитектурные паттерны внешнего слоя приложения:
        1. MVC
        2. MVP
        3. MVVM
        4. MVI
    8. UDF архитектуры
    9. Multi-module project
    10. Multiplatform project
    11. Single Activity
3. CI/CD
    1. GitLab CI
    2. Github CI
    3. Jenkins
    4. Docker
4. Безопасность
    1. Атака Main in the middle
    2. Симметричное и несимметричное шифрование
    3. RSA, AES, Sha, MD5
    4. Salt
    5. Необратимое шифрование
    6. Подписи
    7. Основные уязвимости Android
5. Компиляция
    1. AOT
    2. JIT
    3. Kotlin Multiplatform
6. Память
    1. Куча и стэк
    2. Сборщик мусора - принцип работы и виды
    3. Утечки памяти
    4. Типы ссылок
    5. Виды физической памяти и способы обращения к ней
7. Сеть
    1. Сетевая модель OSI
    2. Сетевая модель TCP/IP
    3. Виды API
        1. REST. RESTful
        2. GraphQL
        3. SOAP
    4. RPC
8. Базы данных
    1. Терминология
    2. СУБД
    3. Транзакция
    4. Коллизия в БД
    5. ACID
    6. ORM
    7. Миграция данных
9. Asynchronous programming
    1. Java Threads
    2. Coroutines
    3. Flow
    4. RxJava
    5. Handler / Looper
    6. Основные проблемы асинхронного кода