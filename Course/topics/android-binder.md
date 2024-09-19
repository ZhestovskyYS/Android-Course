# Binder

**Binder** - происходит от проекта [OpenBinder](https://en.wikipedia.org/wiki/OpenBinder), спонсируемого Google.
**Binder** представляет собой быстрый и легковесный RPC механизм для общения между процессами и приложениями.

![Схема работы Binder](Binder_work_mechanism.png)

## Возможности и особенности

### Идентификация процесса получателя

При любом взаимодействии процесса с Binder внутри **binder driver-а** регистрируются данные об этом процессе:

![](Binder_process_identification.png)

> TODO list - это список задач к исполнению

### Доставка сообщения

Реализован на основе утилиты `ioctl systemcall`

### Передача FileDescriptors

**FileDescriptor** - это целое число, идентифицирующее файл, открытый для данного процесса. Пока у процесса есть
**FileDescriptor** как бы не менялись права доступа на файл, процесс не потеряет доступ с теми правами, с которыми был
открыт файл до всех прочих изменений.

Это позволяет доверенному процессу открыть сокет для клиентского приложения и передать дескриптор на доступ к
определенному файлу.

### Поддержка _credentials_

**Binder** передает в сообщении UID и Process ID получателя и отправителя, с помощью которых можно идентифицировать
безопасность транзакции

### Death notification

![](binder_death_notify.png)

### Thread Pool Management

Взаимодействие с внешним сервисом происходит на потоках Binder-а (Binder Threads), максимальное кол-во которых 16.
Binder самостоятельно контролирует кол-во потоков исходя из собственной текущей загруженности. Со старта приложения,
для него доступно 4 потока Binder-а.

![binder_thread_pool_management.png](binder_thread_pool_management.png)

### Клиент не знает, что он делает IPC запрос

Запрос к внешнему сервису происходит прозрачно, как будто исполнение происходит в локальном компоненте приложения. Для
этого поток, который делает запрос, блокируется до момента окончания выполнения транзакции.

### Передача ссылок во внешний сервис

Ссылки могут передаваться многократно и вести все на один и тот же объект

## Реализация

Реализация класса Binder скрывается Android фреймворком за интерфейсом IBinder.
Клиент со своей стороны реализует
метод [`transact`](https://developer.android.com/reference/kotlin/android/os/IBinder#transact):

```Kotlin
abstract fun transact(
    code: Int, 
    data: Parcel, 
    reply: Parcel?, 
    flags: Int
): Boolean
```

Этот метод возвращает результат выполнения метода `onTransact`, обладающего идентичной сигнатурой, со стороны сервера
Параметры этих методов это:

- `code` - идентификатор метода который нужно вызвать
- `data` - `Parcel` который нужно передать
- `reply` - `Parcel` в который нужно записать ответ
- `falgs` - [дополнительный аргумент](https://developer.android.com/reference/kotlin/android/os/IBinder#constants) для
  маркировки транзакции

### Parcel

![binder_parcelable.png](binder_parcelable.png)

Это специализированный класс данных в Android, используемый при передаче данных между процессами. Чтобы сделать
собственный класс данных допустимым для передачи между процессами, используется
интерфейсы [`Parcelable`](https://developer.android.com/reference/android/os/Parcelable)
и [`Parcelable.Creator`](https://developer.android.com/reference/android/os/Parcelable.Creator):

```Kotlin
 class MyParcelable private constructor(`in`: Parcel) : Parcelable {
     private val mData: Int = `in`.readInt()

     override fun describeContents(): Int {
         return 0
     }

     override fun writeToParcel(out: Parcel, flags: Int) {
         out.writeInt(mData)
     }

     companion object CREATOR: Parcelable.Creator<MyParcelable?> {
         override fun createFromParcel(`in`: Parcel): MyParcelable? {
             return MyParcelable(`in`)
         }

         override fun newArray(size: Int): Array<MyParcelable?> {
             return arrayOfNulls(size)
         }
     }
 }
```

### Работа с Binder. AIDL

Для реализации собственного RPC существует [**AIDL (Android Interface Definition Language)**](https://developer.android.com/develop/background-work/services/aidl?hl=ru)

Для создания собственного интерфейса с помощью **AIDL** необходимо создать файл с расширением `.aidl` в каталоге `aidl` 
внутри директории `main` и описать этот интерфейс с использованием `Java`-подобного синтаксиса.
Поддерживаемые типы данных:
- `boolean`, `boolean[]`, `byte`, `byte[]`, `char[]`, `int`, `int[]`, `long`, `long[]`, `float`, `float[]`, `double`, `double[]`
- `java.lang.CharSequence`, `java.lang.String`
- `java.io.FileDescriptor`
- `java.io.Serializable`
- `android.os.Bundle`
- `java.util.List`
- `android.util.SparseArray`, `android.util.SparseBooleanArray`
- `android.os.IBinder`, `android.os.IInterface`
- `android.os.Parcelable`

С помощью этого интерфейса будут сгенерированы классы **Proxy**, предоставляющий доступ к реализации интерфейса клиенту
и **Stub**, который нужно реализовать на стороне сервера.

<seealso>
    <category ref="src">
        <a href="https://developer.android.com/reference/kotlin/android/os/IBinder">Android Developers | IBinder</a>
        <a href="https://www.youtube.com/watch?v=yyaw0C6oA5k">Android Broadcast youtube | Binder </a>
    </category>
</seealso>