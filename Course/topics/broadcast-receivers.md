# Broadcast Receivers

**BroadcastReceiver** — это компонент Android, предназначенный для получения и обработки широковещательных сообщений (
broadcasts). Эти сообщения могут отправляться либо операционной системой (например, о состоянии батареи или подключении
к сети), либо другим приложением.

Примеры широковещательных сообщений:

- Изменение состояния батареи
- Уведомления о подключении или отключении сети
- События, связанные с загрузкой устройства
- Пользовательские сообщения, отправленные внутри приложения

> Когда приложение регистрирует BroadcastReceiver, оно указывает, на какие события или сообщения оно будет реагировать.

## Типы сообщений

{id="messages-types"}

- Системные
  Android отправляет сообщения о событиях системы, таких как:
    - Подключение к сети: `android.net.conn.CONNECTIVITY_CHANGE`
    - Завершение загрузки устройства: `android.intent.action.BOOT_COMPLETED`
    - Изменение уровня заряда батареи: `android.intent.action.BATTERY_CHANGED`

- Пользовательские
  Разработчики могут отправлять свои собственные широковещательные сообщения, которые могут быть приняты другими
  компонентами внутри приложения или другими приложениями.

Пример отправки пользовательского сообщения:

```kotlin
val intent = Intent("com.example.CUSTOM_ACTION")
sendBroadcast(intent)
```

## Как создать BroadcastReceiver?

{id="how-to-create-br"}

Создание `BroadcastReceiver` включает два основных шага:

1. Реализация класса, наследующего `BroadcastReceiver`.
2. Регистрация `BroadcastReceiver` (статически в манифесте или динамически в коде).

Пример `BroadcastReceiver`:

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        val action = intent?.action
        if (action == Intent.ACTION_BATTERY_CHANGED) {
            // Обработка изменения состояния батареи
        }
    }
}
```

## Регистрация BroadcastReceiver

{id="br-registration"}

- Статическая регистрация
  Регистрируется в `AndroidManifest.xml`. Это позволяет получать сообщения даже тогда, когда приложение неактивно.
  Пример:
  ```xml
  <receiver android:name=".MyBroadcastReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
  </receiver>
  ```

  Особенности статической регистрации:
    - Используется для системных сообщений, таких как BOOT_COMPLETED или CONNECTIVITY_CHANGE.
    - После Android 8 (Oreo) некоторые сообщения могут быть зарегистрированы только динамически, например,
      `CONNECTIVITY_CHANGE`.

- Динамическая регистрация
  Регистрация выполняется в коде (например, в `Activity` или `Service`). Используется для сообщений, которые должны
  обрабатываться только при активном приложении.
  Пример:
  ```kotlin
  val receiver = MyBroadcastReceiver()
  val intentFilter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)
  registerReceiver(receiver, intentFilter)
  ``` 

  Для отмены регистрации:
  ```kotlin
  unregisterReceiver(receiver)
  ```
  Особенности динамической регистрации:
    - Используется для сообщений, которые актуальны только при активном приложении.
    - Требует явной отмены регистрации через unregisterReceiver(), чтобы избежать утечек памяти.

## Отправка широковещательных сообщений

Приложение может отправлять Broadcast, которые могут быть приняты другими компонентами. Для этого используется
`sendBroadcast()` или `sendOrderedBroadcast()`.

Пример отправки Broadcast:

```kotlin
val intent = Intent("com.example.MY_CUSTOM_ACTION")
sendBroadcast(intent)
```

Пример защищенного Broadcast:

Для отправки сообщений только авторизованным получателям можно использовать `sendBroadcast(intent, permission)`.

```kotlin
sendBroadcast(intent, "com.example.permission.CUSTOM_PERMISSION")
```

## Обработка упорядоченных широковещательных сообщений

Упорядоченные Broadcast (Ordered Broadcasts) обрабатываются получателями в определенной последовательности. Это
позволяет приоритетным получателям обработать сообщение раньше остальных.

Пример отправки упорядоченного Broadcast:

```kotlin
val intent = Intent("com.example.ORDERED_ACTION")
sendOrderedBroadcast(intent, null)
```

Пример получения упорядоченного Broadcast:

```XML

<receiver android:name=".OrderedReceiver">
    <intent-filter android:priority="1">
        <action android:name="com.example.ORDERED_ACTION"/>
    </intent-filter>
</receiver>
```

## Особенности работы BroadcastReceiver

- Ограничения в Android 8 (Oreo) и выше:

    - Для оптимизации работы системы, многие системные Broadcast больше не могут быть зарегистрированы статически в
      манифесте.
      Например:
        - CONNECTIVITY_CHANGE
        - ACTION_PACKAGE_ADDED

    - Для обработки таких событий необходимо использовать динамическую регистрацию или специализированные API (например,
      JobScheduler или WorkManager).

- Время выполнения:

    - Код в onReceive() выполняется в основном потоке приложения, поэтому выполнение длительных операций (например,
      обращение к сети) может привести к блокировке интерфейса. Для долгих операций следует использовать IntentService
      или WorkManager.

- Приоритеты Intent-фильтров:

    - Приоритет для получения Broadcast задается с помощью атрибута android:priority в IntentFilter.
    - Чем выше значение, тем раньше этот BroadcastReceiver обработает сообщение.

## Пример использования BroadcastReceiver

{id="br-example"}

Обработка изменения состояния батареи:

```kotlin
class BatteryReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        val level = intent?.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) ?: -1
        println("Battery Level: $level%")
    }
}

// Регистрация Receiver в коде
val batteryReceiver = BatteryReceiver()
val intentFilter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)
context.registerReceiver(batteryReceiver, intentFilter)
```

<seealso>
  <category ref="src">
    <a href="https://developer.alexanderklimov.ru/android/broadcast.php">Klimov A. | Broadcast (Широковещательные сообщения)</a>
  </category>
</seealso>