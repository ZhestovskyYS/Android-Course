# Binder

<secondary-label ref="wip"/>

**Binder** - происходит от проекта [OpenBinder](https://en.wikipedia.org/wiki/OpenBinder), спонсируемого Google.
**Binder** представляет собой быстрый и легковесный RPC механизм для общения между процессами и приложениями.

![Схема работы Binder](Binder_work_mechanism.png)

## Обязанности Binder

### 1. Идентификация процесса получателя

При любом взаимодействии процесса с Binder внутри **binder driver-а** регистрируются данные об этом процессе:

![](Binder_process_identification.png)

> TODO list - это список задач к исполнению

### 2. Доставка сообщения

Реализован на основе утилиты `ioctl systemcall`

### 3. Передача FileDescriptors

**FileDescriptor** 