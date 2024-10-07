# Activity

<primary-label ref="main"/> 
<secondary-label ref="wip"/>

**[Activity](https://developer.android.com/reference/android/app/Activity)** - точка входа для графического приложения
в Android

## Концепция

Activity выполняет роль графической среды приложения, предоставляя ему API для работы с системой. Activity может
вмещать в себя как все окна приложения, так и единственное. Это полезно, когда вы предоставляете возможность другим
приложением запускать часть вашего приложения, а не все целиком, в таких сценариях, как:

- предоставление обработчика электронных покупок
- предоставление возможности быстрого набора и отправки сообщения по электронной почте или SMS и т.п.
- предоставление возможности быстрой записи видео / фото
- и т.д.

## Жизненный цикл

Запуск и контроль графического окружения приложения в ОС важная задача, так как пользователи оценивают приложение по
его скорости отклика, удобству и привлекательности. Для того чтобы разработчик имел больше контроля над первыми двумя
аспектами класс Activity предоставляет API для вызова определенных частей логики приложения в моменты переходов между
этапами его жизненного цикла. Это API представлено в двух видах:

- Методы жизненного цикла
- Механизм владельца и смотрителя жизненного цикла

> Второй вид будет рассмотрен в главе [Концепция жизненного цикла](lifecycle-concept.md)
> {style="note"}

## Методы жизненного цикла Activity

![Жизненный цикл Activity](activity_lifecycle.png)

**- `onCreate()`**  

**- `onStart()`**

**- `onResume()`**

**- `onPause()`**

**- `onStop()`**
Before activity gets stopped

**- `onDestroy()`**
Before activity gets destroyed

**- `onRestart()`**


<seealso>
    <category ref="src">
        <a href="https://developer.android.com/reference/android/app/Activity">Android Developers | Activity</a>
        <a href="https://developer.android.com/reference/androidx/lifecycle/package-summary"> Android Developers | Lifecycle Overview</a>
        <a href="https://habr.com/ru/articles/569092/">Habr | Памятка по жизненному циклу Android</a>
    </category>
</seealso>