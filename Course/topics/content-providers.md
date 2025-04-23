# Content Providers

**ContentProvider** — это компонент Android, который используется для обмена данными между приложениями. Он
предоставляет стандартизированный интерфейс для доступа к данным приложения, таким как контакты, медиафайлы или данные
базы данных. С помощью ContentProvider можно делиться данными с другими приложениями или организовать внутренний доступ
к данным в рамках одного приложения.

## Что такое ContentProvider?

{id="what-is-contentprovider"}

`ContentProvider` предоставляет механизмы для:

- Организации обмена данными между приложениями.
- Управления доступом к данным с использованием URI (Uniform Resource Identifier).
- Обеспечения безопасности данных через контроль разрешений.

Пример встроенного `ContentProvider`:

- Контакты: content://contacts/people
- Медиафайлы: content://media/external/images

## Когда использовать ContentProvider?

{id="when-to-use-contentprovider"}

`ContentProvider` используется в следующих случаях:

- Необходимо предоставить доступ к данным другим приложениям (например, контактам или изображениям).
- Требуется организовать централизованный доступ к данным внутри приложения с использованием стандартного API.

> Если данные используются только внутри одного приложения, то
> `ContentProvider` необязателен, и можно использовать другие
> механизмы хранения данных (например, базы данных или файлы).

## Создание ContentProvider

Шаги для создания ContentProvider:

1. Наследование класса ContentProvider: Создайте класс, наследующийся от ContentProvider, и реализуйте его методы.
2. Регистрация ContentProvider в AndroidManifest.xml: Укажите имя класса и его атрибуты, такие как authority.
3. Реализация основных методов ContentProvider
    - Реализуйте методы, такие как query(), insert(), update(), delete(), которые
      обрабатывают запросы к данным.

Реализация ContentProvider:
Пример:

```kotlin
class MyContentProvider : ContentProvider() {
    private lateinit var database: SQLiteDatabase

    override fun onCreate(): Boolean {
        val dbHelper = MyDatabaseHelper(context)
        database = dbHelper.writableDatabase
        return true
    }

    override fun query(
        uri: Uri,
        projection: Array<String>?,
        selection: String?,
        selectionArgs: Array<String>?,
        sortOrder: String?
    ): Cursor? {
        return database.query("my_table", projection, selection, selectionArgs, null, null, sortOrder)
    }

    override fun insert(uri: Uri, values: ContentValues?): Uri? {
        val id = database.insert("my_table", null, values)
        return Uri.withAppendedPath(uri, id.toString())
    }

    override fun update(uri: Uri, values: ContentValues?, selection: String?, selectionArgs: Array<String>?): Int {
        return database.update("my_table", values, selection, selectionArgs)
    }

    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?): Int {
        return database.delete("my_table", selection, selectionArgs)
    }

    override fun getType(uri: Uri): String? {
        return "vnd.android.cursor.dir/vnd.com.example.my_table"
    }

}
```

Регистрация в `AndroidManifest.xml`:
`ContentProvider` необходимо зарегистрировать в манифесте, указав authority — уникальный идентификатор провайдера.
Пример:

```xml

<provider
        android:name=".MyContentProvider"
        android:authorities="com.example.myprovider"
        android:exported="true"
        android:grantUriPermissions="true"/>
```

## Работа с ContentResolver

`ContentResolver` — это класс, который используется для взаимодействия с ContentProvider. Он позволяет отправлять
запросы (например, query(), insert(), update(), delete()) к провайдерам данных.

Пример работы с ContentResolver:

- Чтение данных:

```kotlin
val uri = Uri.parse("content://com.example.myprovider/my_table")
val cursor = contentResolver.query(uri, null, null, null, null)
cursor?.use {
    while (it.moveToNext()) {
        val columnValue = it.getString(it.getColumnIndex("column_name"))
        println(columnValue)
    }
}
```

- Добавление данных:

```kotlin
val uri = Uri.parse("content://com.example.myprovider/my_table")
val values = ContentValues().apply {
    put("column_name", "value")
}
val resultUri = contentResolver.insert(uri, values)
println("Inserted at: $resultUri")
```

- Обновление данных:

```kotlin
val uri = Uri.parse("content://com.example.myprovider/my_table")
val values = ContentValues().apply {
    put("column_name", "new_value")
}
val rowsUpdated = contentResolver.update(uri, values, "id=?", arrayOf("1"))
println("Rows updated: $rowsUpdated")
```

- Удаление данных:

```kotlin
val uri = Uri.parse("content://com.example.myprovider/my_table")
val rowsDeleted = contentResolver.delete(uri, "id=?", arrayOf("1"))
println("Rows deleted: $rowsDeleted")
```

## Особенности ContentProvider

{id="contentprovider-advantages"}

- Безопасность:
    - Для защиты данных можно настроить разрешения в манифесте, чтобы ограничить доступ к ContentProvider. Пример:
  ```xml
  <provider
    android:name=".MyContentProvider"
    android:authorities="com.example.myprovider"
    android:exported="true"
    android:permission="com.example.permission.MY_PERMISSION" />
  ```  

- Работа с URI:
    - URI — это уникальный идентификатор ресурса, с которым работает ContentProvider
    - Пример URI: `content://com.example.myprovider/my_table`.

- Права доступа (URI Permissions):
    - Для временного предоставления доступа к данным можно использовать `grantUriPermissions`:
  ```XML
  <provider
    android:name=".MyContentProvider"
    android:authorities="com.example.myprovider"
    android:exported="true"
    android:grantUriPermissions="true" />
  ```
- Оптимизация:
    - Для работы с большими объемами данных рекомендуется использовать курсоры (Cursor) для минимизации нагрузки на
      память.

## Преимущества и недостатки ContentProvider

{id="contentprovider-ups-n-downs"}

Преимущества:

- Удобный механизм обмена данными между приложениями.
- Централизованное управление доступом к данным.
- Поддержка стандартизированных операций с использованием ContentResolver.

Недостатки:

- Сложность реализации.
- Перегрузка производительности для небольших объемов данных.

## Пример встроенного ContentProvider

{id="contentprovider-example"}

Для работы с контактами в Android используется встроенный `ContentProvider`.
Пример получения списка контактов:

```kotlin
val uri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI
val projection = arrayOf(
    ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME,
    ContactsContract.CommonDataKinds.Phone.NUMBER
)

val cursor = contentResolver.query(uri, projection, null, null, null)
cursor?.use {
    while (it.moveToNext()) {
        val name = it.getString(
            it.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME)
        )
        val phoneNumber = it.getString(
            it.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER)
        )
        println("Name: $name, Phone: $phoneNumber")
    }
}
```

<seealso>
  <category ref="src">
    <a href="https://apptractor.ru/info/techhype/voprosy-s-sobesedovaniy-chto-takoe-kontent-provayder-content-provider-v-android.html">Вопросы с собеседований: Что такое контент-провайдер (Content Provider) в Android</a>
    <a href="https://medium.com/@zekromvishwa56789/understanding-contentprovider-and-contentresolver-in-android-with-kotlin-f31952062649">Content Provider in Android with  Kotlin.</a>
  </category>
</seealso>