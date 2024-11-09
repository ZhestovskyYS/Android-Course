# Intent

## Intent - это {id="1"}

объект для передачи системных сообщений, с помощью которого можно запрашивать выполнение определенных действий от
компонентов приложения или других приложений.

## Виды
<show-structure for="none"/>

### Явные Intent-ы {id="2.3"}

явно обращаются к компоненту по имени класса

### Неявные Intent-ы {id="2.4"}

обращаются к системе с запросом на выполнение определенного действия, кандидатов на выполнение которого может быть
несколько или не быть вовсе. В первом случае пользователю предоставляется право выбора исполнителя

## Фильтры

определяют типы Intent-ов, на которые могут реагировать компоненты приложения. Когда неявный Intent посылается в
систему, Android просматривает манифесты приложений и ищет те, что объявили соответствующие типы фильтров.

## Основные сценарии использования

это:

- Старт **Activity**
- Старт **Service**
- Доставка **Широковещательного сообщения**

## Intent несет в себе  информацию о: {id="5"}

### Имени компонента

который необходимо запустить. Однако она необходима для обращения к конкретному компоненту приложения. Без неё Intent -
неявный

### Действии

которое нужно совершить. Это строковая константа, обозначающая тип действия. Предопределенна в системе, или
дополнительно объявлена в приложении (тогда следует указать название, включая пакет приложения)

### Данных

которые нужно передать компоненту. Например, URI объекта, над которым нужно совершить действие, и/или тип передаваемых
данных по стандарту [MIME](https://ru.w3docs.com/uchebnik-html/mime-tipy.html).


> Указание URI в свойстве **content** Intent-а говорит системе о том, что данные расположены на устройстве и
> контролируются [ContentProvider-ом](https://developer.android.com/reference/android/content/ContentProvider)
> В таком случае [MIME](https://ru.w3docs.com/uchebnik-html/mime-tipy.html) тип данных виден системе без указания
> { style="note"}

### Категории

- это строка, содержащая дополнительную информацию о подходящем для обработки Intent-а компонента,
например: [CATEGORY_BROWSABLE](https://developer.android.com/reference/android/content/Intent#CATEGORY_BROWSABLE), [CATEGORY_LAUNCHER](https://developer.android.com/reference/android/content/Intent#CATEGORY_LAUNCHER).

### Дополнительные данные (Extra)

различного вида, есть как предопределенные в системе константы так и почти не ограниченная поддержка любых объектов,
помещенных в Bundle в формате ключ-значение.


> Ограничения
> - Не используйте `Parcelable` или `Serializable`, когда отправляете интент, который направлен на компонент стороннего
>   приложения. Если компонент попытается считать из `Bundle` парсел или серилизованный класс, который не знает, то
>    возникнет [RuntimeException](https://developer.android.com/reference/java/lang/RuntimeException)
> - Максимальный размер транзакции с участием `Bundle` - 1мб. Однако Текущий размер транзакции бывает не очевиден, из-за
>    чего рекомендуется не превышать 50кб.
> 
> Избегайте передачи больших объемов данных через Intent. Для больших данных используйте другие механизмы, такие как:
> - **ContentProvider**: Для структурированных данных.
> - **FileProvider**: Для файлов.
> - **SharedPreferences**: Для небольших объемов данных.
>
> Либо сжимайте данные, передавайте ссылки, по которым компонент может самостоятельно получить к ним доступ
{style="warning"}

### Флаги

служат метаданными для
Intent-а. [Дополнительная информация](https://developer.android.com/guide/components/activities/tasks-and-back-stack#:~:text=the%20app%20launcher%20and%20are%20used%20to%20start%20a%20task.)

<seealso>
    <category ref="related">
        <a href="ipc.md">Inter-Process Communication</a>
    </category>
    <category ref="src">
        <a href="https://developer.android.com/guide/components/intents-filters?hl=en">Intents and intent filters</a>
        <a href="https://developer.android.com/guide/components/activities/tasks-and-back-stack#:~:text=the%20app%20launcher%20and%20are%20used%20to%20start%20a%20task.">Tasks and Back stack</a>
    </category>
</seealso>