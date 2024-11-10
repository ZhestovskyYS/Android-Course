# Coroutines

<secondary-label ref="todo"/>

## Что такое корутина (сопрограмма)

Сопрограммы были впервые предложены Мелвином Конвеем (Melvin Conway) в 1963 году. В своей работе "Design of a Separable
Transition-diagram Compiler" Конвей описал идею сопрограмм как генерализации подпрограмм, которые могут приостанавливать
и возобновлять свое выполнение в определённых точках. Это отличалось от стандартных функций, которые имеют единственную
точку входа и выхода.

Сопрограмма — это функция, которая может приостанавливать свое выполнение и возвращать управление вызывающему коду,
сохраняя при этом свое состояние для последующего возобновления. Ключевые моменты:

- **Множественные точки входа и выхода**: В отличие от обычных функций, сопрограммы могут входить и выходить из
  выполнения в нескольких местах с помощью специальных инструкций.

- **Сохранение контекста**: При приостановке выполнения сохраняется текущий контекст сопрограммы, включая значения
  локальных переменных и позицию в коде.

- **Явная передача управления**: Управление между сопрограммами передается явно, что позволяет разработчику
  контролировать последовательность выполнения.

### Проблемы, решаемые сопрограммами:

- **Кооперативная многозадачность**: Сопрограммы позволяют различным частям программы выполняться по очереди, передавая
  управление друг другу без необходимости в потоках или прерываниях.

- **Асинхронное программирование**: Они упрощают работу с асинхронными операциями, такими как ввод-вывод, позволяя
  писать
  код в более линейном и понятном стиле.

- **Сохранение состояния между вызовами**: Сопрограммы могут сохранять свое состояние между вызовами, что облегчает
  реализацию генераторов, итераторов и конечных автоматов.

- **Улучшение производительности**: За счёт уменьшения накладных расходов на переключение контекста по сравнению с
  потоками,
  сопрограммы могут быть более эффективными в некоторых сценариях.

### Только ли Kotlin имеет реализацию сопрограмм?

{id="kotlin_only"}

Конечно нет, однако идентичных реализаций нет. Среди языков, которые имеют реализацию сопрограмм в том или ином виде:

- **Python, C#, C++ (20+), JavaScript, Rust, Swift, Dart, Visual Basic .NET, Nim**: поддерживают ключевые функции
  _**async / await**_, некоторые предоставляют расширенный набор методов, например _**yield**_
- **Lua, Ruby (Fiber), Go, Delphi, Erlang & Elixir, Lisp, F#, PHP, Tcl, Haskell, Julia, Perl, Crystal, Racket**: имеют
  собственные идиоматические реализации с соблюдением общей концепции сопрограмм

Реализации, кроме всего прочего, отличаются и уровнем интеграции в язык. Где-то это часть стандартной библиотеки и
синтаксиса языка, но чаще все же это отдельно развиваемые библиотеки, прочно закрепившиеся в экосистеме общего стека
языка.

## Ключевые концепции корутин

Как было сказано выше, есть три основные фичи: **множественные точки входа и выхода**, **сохранение контекста**, **явная
передача управления**.

Чтобы все это работало, в реализации сопрограмм для языка программирования должно быть:

1. Способ пометить точки, в которых выполнение инструкций может быть безопасно приостановлено и вскоре возобновлено

2. Набор примитивов, которые позволят оборачивать контекст выполнения, для сохранения прогресса выполнения инструкций
   между точками приостановки

3. Механизм передачи и сохранения примитивов контекста выполнения

4. Позволяющий вернуть контроль над занятым выполнением инструкций потоку, который инициировал запуск
   сопрограммы, механизм. Он же должен предоставить иной поток для возобновления выполнения инструкций

5. Механизм менеджмента потоков, позволяющий эффективно переключать выполнения сопрограмм

## Реализация сопрограмм в Kotlin

{id="kotlin-realisation"}

Корутины в Kotlin появились во время выпуска версии Kotlin v1.3 в 2018 году в виде библиотеки `kotlinx.corountines`.
Реализация корутин в Kotlin наиболее похожа на таковую в Goroutines языка Go. С помощью этой библиотеки становится
возможным писать асинхронный код в том же стиле и виде, что и обычный. Все дополнительные настройки производятся за счет
кодогенерации.

Вот пример как будет выглядеть код:

```kotlin
override suspend fun loadChooseCardScreenData(): ChooseLuckyCardScreenEntity =
        withContext(dispatchers.io) {
            Log.d("Calling cloud function", CLOUD_FUNC_REQUEST_LUCKY_CARD_DECK)
            val response = ParseCloud.callFunction<Map<String, Any>?>(
                CLOUD_FUNC_REQUEST_LUCKY_CARD_DECK, HashMap<String, Any>()
            )?.toModel<ChooseLuckyCardScreenEntity>()
                ?: ChooseLuckyCardScreenEntity()
            return@withContext response
        }
```

{ collapsible="true" collapsed-title="Функция `loadChooseCardScreenData`"}

после декомпиляции:

```kotlin
   @Nullable
   public Object loadChooseCardScreenData(@NotNull Continuation $completion) {
      return BuildersKt.withContext((CoroutineContext)this.dispatchers.getIo(), (Function2)(new Function2((Continuation)null) {
         int label;

         public final Object invokeSuspend(Object var1) {
            Object var9 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
            switch (this.label) {
               case 0:
                  ChooseLuckyCardScreenEntity var10000;
                  label17: {
                     ResultKt.throwOnFailure(var1);
                     Log.d("Calling cloud function", "requestLuckyCardDeck");
                     Map var3 = (Map)ParseCloud.callFunction("requestLuckyCardDeck", (Map)(new HashMap()));
                     if (var3 != null) {
                        Map $this$toModel$iv = var3;
                        int $i$f$toModel = false;
                        String var7 = (new JSONObject($this$toModel$iv)).toString();
                        Intrinsics.checkNotNullExpressionValue(var7, "toString(...)");
                        String jsonString$iv = var7;
                        ChooseLuckyCardScreenEntity var4 = (ChooseLuckyCardScreenEntity)(new Gson()).fromJson(jsonString$iv, ChooseLuckyCardScreenEntity.class);
                        if (var4 != null) {
                           var10000 = var4;
                           break label17;
                        }
                     }

                     var10000 = new ChooseLuckyCardScreenEntity((TarotCard)null, (List)null, (String)null, (Integer)null, (Integer)null, (Integer)null, 63, (DefaultConstructorMarker)null);
                  }

                  ChooseLuckyCardScreenEntity response = var10000;
                  return response;
               default:
                  throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
            }
         }

         public final Continuation create(Object value, Continuation $completion) {
            return (Continuation)(new <anonymous constructor>($completion));
         }

         public final Object invoke(CoroutineScope p1, Continuation p2) {
            return ((<undefinedtype>)this.create(p1, p2)).invokeSuspend(Unit.INSTANCE);
         }

         // $FF: synthetic method
         // $FF: bridge method
         public Object invoke(Object p1, Object p2) {
            return this.invoke((CoroutineScope)p1, (Continuation)p2);
         }
      }), $completion);
   }
```

{ collapsible="true" collapsed-title="Декомпилированная функция `loadChooseCardScreenData`"}



Рассмотрим концепции, перечисленные в предыдущем разделе:

- **Точками приостановки и возобновления** считается место вызова **_suspend_** функции в запущенной корутине

- **Примитивы для сохранения контекста выполнения** - это _**Continuation**_, он отвечает
  за сохранение прогресса выполнения и возобновление/остановку корутины, тем самым отвечая и на пункт 3 (**механизм
  передачи и сохранения примитивов контекста выполнения**)

- _**Continuation**_ в купе с _**CoroutineContext**_ и _**Dispatcher**_ участвуют в механизме **менеджмента потоков**,
  доступных корутинам и переключения выполнения инструкций между ними.

### Как запустить корутину

Для запуска корутины нужно использовать один из билдеров корутин:

- **_runBlocking_**:
    - редко используемый билдер, так как лишает корутину ее главного преимущества (не блокирования
      потоков выполнения), но позволяет дождаться окончания асинхронной работы и получить ее результат на месте. Стоит
      использовать только если вы 100% уверены в том что делаете.

- **_launch_**

- **_async_**

- **_coroutineScope_**

- **_supervisorScope_**

