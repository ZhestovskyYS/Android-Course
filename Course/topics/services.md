# Services

<secondary-label ref="wip"/>

Service в Android ключевой механизм для выполнения длительных операций в Android,
но их использование требует понимания их типов, жизненного цикла и ограничений.

## Когда использовать Service {id="service-usage"}

- Непрерывные задачи (аудиоплеер, запись с микрофона)
- Обмен данными между приложениями через IPC
- Работа с hardware (Bluetooth, камера) в фоне

## Когда избегать {id="service-not-recommended"}

- Короткие фоновые задачи → Используйте WorkManager или ThreadPool
- Отложенные операции → JobScheduler/WorkManager с условиями (сеть, зарядка)
- Периодические задачи → AlarmManager для точного времени

## Ограничение

**Невозможность запустить сервис из фона**

**Android 12+**: Если ваше приложение находится в фоне, вызов `startService()` приведёт к ошибке
`IllegalStateException`:
система не даст сервису запуститься. Чтобы «поднять» сервис из фона, нужно вместо `startService()` вызвать
`startForegroundService()`, а в течение 5 секунд в самом сервисе вызвать `startForeground()` с уведомлением. Иначе
сервис
остановят автоматически

**Сервис без действий**

Сервисы, запущенные как обычные (**background**) и не переведённые в **foreground**, будут остановлены системой через
короткий промежуток (несколько минут) после того, как приложение уйдёт в фон.

Это не значит, что все фоновые сервисы обязаны стать **foreground**, а то, что длительные или прерываемые задачи в фоне
уже не могут работать как раньше.

Если задача заметна пользователю (музыка, локация, запись), её нужно оформить как **foreground**-сервис с уведомлением;
если нет —
переносить **в планировщик (JobScheduler/WorkManager)**.

**Запуск Activity из сервиса**

**Android 10+**: прямой вызов startActivity из фона блокируется. Используйте уведомление с PendingIntent.

**Параллельные вызовы**

Несколько последовательных startService: все идут в onStartCommand по мере поступления. Используйте startId, чтобы
остановить только нужный инстанс.

## Типы Service-ов {id="service-types"}

### Started Service

Запускается через startService(), работает независимо от UI. Используется для:

- Однократных задач (например, загрузка файла).
- Длительных операций без привязки к компонентам.

**Жизненный цикл**

1. **onCreate()** — создаётся экземпляр.
2. **onStartCommand(intent, flags, startId)** — вызывается при каждом startService.
3. **Работает** пока явно не вызвать stopSelf(startId) или stopService().
4. **onDestroy()** — перед уничтожением.

Пример создания:

```kotlin
class MyService : Service() {
    override fun onCreate() { /* инициализация */
    }

    override fun onStartCommand(
        intent: Intent?,
        flags: Int,
        startId: Int
    ): Int {
        // Запускаем работу в фоне
        Thread {
            // тяжелая операция
            stopSelf(startId)  // останавливаем себя
        }.start()
        return START_NOT_STICKY
    }

    override fun onBind(intent: Intent?) = null
}
```

Запуск:

```kotlin
startService(Intent(this, MyService::class.java))
```

> **onStartCommand(...)** выполняется на UI-потоке.
> Всегда выносите тяжелую работу в другие потоки (Thread, Executor, Coroutines)
> {style="warning"}

**Режимы запуска:**

- **START_NOT_STICKY** — не перезапускать после убийства.
- **START_STICKY** — перезапустить с intent=null.
- **START_REDELIVER_INTENT** — перезапустить с последним intent.

> С **Android8+** фоновые сервисы убиваются системой через
> минуту–две, если не поднять их в foreground или не использовать _JobScheduler/WorkManager_
> {style="note"}

**~~IntentService~~ (устарел)**:

Внутри создаёт рабочий поток и обрабатывает Intent последовательно,
затем сам останавливается. На **Android8+** часто не запускается в фоне
— используйте вместо него WorkManager или JobIntentService.

### Bound Service

Связывается с компонентами (Activity, Fragment) через bindService().
Подходит для:

- Межпроцессного взаимодействия (IPC).
- Постоянного обмена данными (например, стриминг музыки).

**Жизненный цикл**

1. **onCreate()** — при первом bind или start
2. **onBind(intent)** — возвращает IBinder
3. Работает, пока есть хотя бы один подключенный клиент
4. **onRebind(), onUnbind()** — при повторных bind/unbind
5. **onDestroy()** — когда нет ни одного bind и нет вызова start

Пример создания:

```kotlin
class MyService : Service() {
    private val binder = object : Binder() {
        fun getService() = this@MyService
    }

    override fun onBind(intent: Intent?) = binder

    fun computeSomething(x: Int): Int { /* ... */
    }
}
```

Запуск:

```kotlin
private var svc: MyService? = null
private val conn = object : ServiceConnection {
    override fun onServiceConnected(
        name: ComponentName,
        binder: IBinder
    ) {
        svc = (binder as MyService.Binder).getService()
        val result = svc?.computeSomething(42)
    }
    override fun onServiceDisconnected(name: ComponentName) {
        svc = null
    }
}

fun startBinding() {
    bindService(
        Intent(this, MyService::class.java),
        conn,
        BIND_AUTO_CREATE
    )
}
fun stopBinding() {
    unbindService(conn)
}
```

> - **Работа в UI‑потоке**: методы сервиса по-прежнему выполняются в основном потоке — при тяжёлых вычислениях
    организуйте свои потоки внутри сервиса
> - **Несколько клиентов**: сервис остаётся жив пока хотя бы один клиент привязан
> - **Комбинация start+bind**: если вы и запускаете через startService, и привязываетесь, то сервис завершится только
    когда будет и остановлен, и все привязки отпадут
> - **IPC через AIDL**: для межпроцессного доступа вместо локального Binder используйте AIDL-файлы

### Foreground Service

Обязательно показывает уведомление.
Используется для:

- Задач, заметных пользователю (музыка, GPS-трекинг).
- Длительных операций, которые нельзя прервать (например, синхронизация данных).

**Жизненный цикл**

1. **onCreate()**
2. **onStartCommand()**
3. **startForeground(id, notification)** — переводит в foreground.
4. Выполняется, пока не вызвать **stopForeground(true) + stopSelf()**.
5. **onDestroy()**.

Пример создания:

```kotlin
class MyFgService : Service() {
    override fun onCreate() {
        val notif = NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle("Task running")
            .setSmallIcon(R.drawable.ic_service)
            .build()
        startForeground(1, notif)
    }

    override fun onStartCommand(
        intent: Intent?,
        flags: Int,
        startId: Int
    ): Int {
        CoroutineScope(Dispatchers.IO).launch {
            // длительная работа
            stopSelf(startId)
        }
        return START_NOT_STICKY
    }
    override fun onBind(intent: Intent?) = null
}
```

Запуск:

```kotlin
ContextCompat.startForegroundService(
    this,
    Intent(this, MyFgService::class.java)
)
```

**Short-service**

если в манифесте указать тип shortService,
система ограничит работу ~3 минутами и после вызовет onTimeout()

> - **Уведомление** ForegroundService должно появиться немедленно, иначе система бросит исключение
> - С **Android 10+** в манифесте обязателен атрибут `android:foregroundServiceType`,
    например:
> ```xml
> <service android:name=".MyFgService"
>          android:foregroundServiceType="mediaPlayback" />
> ```
> - С **Android 13+** нужна разрешение POST_NOTIFICATIONS, иначе нотификация скрывается и сервис сбрасывается
> - С **Android 14+** для каждого типа foreground‑сервиса требуется своё permission (например,
    FOREGROUND_SERVICE_MEDIA_PLAYBACK)