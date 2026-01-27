
| Характеристика       | `stream()`                            | `parallelStream()`                                                            |
| -------------------- | ------------------------------------- | ----------------------------------------------------------------------------- |
| Исполнение           | Последовательное, в вызывающем потоке | Параллельное в `ForkJoinPool.commonPool`                                      |
| Распределение работы | Обходит элементы по одному            | `Spliterator` делит данные на подзадачи; каждая — `ForkJoinTask`              |
| Число потоков        | 1                                     | По умолчанию `CPU‑ядер – 1`; меняется системным свойством или отдельным пулом |

## Способы задать число потоков для `parallelStream()`

| Подход                                                 | Когда применять                                                                                                                              | Как указать «сколько потоков»                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Переопределить  `ForkJoinPool.commonPool`**       | ‑ весь процесс работает с одним общим пулом; менять «на лету» нельзя                                                                         | JVM‑флаг **`‑Djava.util.concurrent.ForkJoinPool.common.parallelism=<N>`**  <br>или  <br>`System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "8")` _**до**_ первого обращения к `commonPool` [docs.oracle.com](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html?utm_source=chatgpt.com)[Stack Overflow](https://stackoverflow.com/questions/49009662/set-java-util-concurrent-forkjoinpool-common-parallelism-java-property?utm_source=chatgpt.com) |
| **2. Запустить конвейер в собственном `ForkJoinPool`** | ‑ нужно разное число потоков в разных местах  <br>‑ нет желания трогать «общий» пул                                                          | Создаём `new ForkJoinPool(N)` и `submit()` в нём лямбду со стримом                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| **3. Использовать любой другой `Executor`**            | ‑ код уже работает через `CompletableFuture`/`ExecutorService`  <br>‑ хочется ограничить не только size, но и очерёдность, приоритеты и т.д. | `CompletableFuture.supplyAsync(() -> list.parallelStream() … , customExecutor)`                                                                                                                                                                                                                                                                                                                                                                                                                            |

Ниже — короткие, готовые к копипасту фрагменты.

---

### 1. Изменяем **общий** пул при старте JVM

```bash
 #Linux / macOS 
 java -Djava.util.concurrent.ForkJoinPool.common.parallelism=8 -jar app.jar  
 # Windows PowerShell 
 java "-Djava.util.concurrent.ForkJoinPool.common.parallelism=8" -jar app.jar`
```

> ⚠️ Свойство читается **один раз** при первом создании `ForkJoinPool.commonPool()`; поменять его после старта невозможно. [Stack Overflow](https://stackoverflow.com/questions/49009662/set-java-util-concurrent-forkjoinpool-common-parallelism-java-property?utm_source=chatgpt.com)

Проверка:
```java
System.out.println(ForkJoinPool.commonPool().getParallelism()); // 8
```
---

### 2. Собственный `ForkJoinPool`

``` java
import java.util.Arrays;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ExecutionException;

public class CustomPoolExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        var numbers = Arrays.asList(1, 2, 3, 4, 5);

        ForkJoinPool pool = new ForkJoinPool(4);   // ← хотим ровно 4 потока
        int sum = pool.submit(() ->
            numbers.parallelStream()
                   .mapToInt(i -> i * 2)
                   .sum()
        ).get();   // join() тоже подойдёт

        System.out.println(sum);   // 30
        pool.shutdown();
    }
}
```
- Конвейер внутри лямбды **остаётся параллельным**, но теперь задачи разбирают воркеры `ForkJoinPool-1-worker‑X`, а не `ForkJoinPool.commonPool‑worker‑X`.
- Можно держать несколько разных пулов (например, CPU‑bound и I/O‑bound). [Baeldung](https://www.baeldung.com/java-8-parallel-streams-custom-threadpool?utm_source=chatgpt.com)
---

### 3. Любой другой `Executor`

Если проект уже использует `CompletableFuture`:

``` java
ExecutorService ioPool = Executors.newFixedThreadPool(6);

CompletableFuture<Integer> result =
    CompletableFuture.supplyAsync(() ->
        list.parallelStream()
            .mapToInt(MyService::heavyCalculation)
            .sum(),
        ioPool    // ← свой пул
    );

System.out.println(result.join());
ioPool.shutdown();

```

Здесь:

- `parallelStream()` внутри таска всё ещё дробит работу на _подзадачи_, но они бегут **по тем же** шести потокам `ioPool`, поскольку `ForkJoinTask` пересоздаваться не будет — весь конвейер уже выполняется внутри задания, отправленного в ваш executor.

---

### Как решить, **сколько** потоков ставить?

1. **CPU‑bound** расчёты → `N = Runtime.getRuntime().availableProcessors()` или чуть меньше (чтобы оставить место GC/IO).
2. Блокирующие вызовы (I/O, БД) → ставьте столько, сколько реально может «зависнуть» одновременных задач, но следите, чтобы пул не рос бесконтрольно.
3. Помните про Amdahl’s Law: если конвейер обрабатывает маленькие коллекции или «лёгкие» элементы, увеличивать N бессмысленно — накладные расходы всё съедят.
---

#### Мини‑чек‑лист

-  Тяжёлые операции?
-  Нет синхронизаций/блокировок внутри `map`/`filter`?
-  Коллекция >= 10 000 элементов?
-  Вы не запускаете ещё одно «тяжёлое» CPU‑bound‑приложение на той же машине?

Если ответ «да» хотя бы на первые два пункта – параллельность почти точно окупится.

Теперь у вас есть три практических способа выбрать нужный уровень параллелизма и подстроить его под конкретные задачи.

### Когда `parallelStream()` помогает

|Сценарий|Мини‑пример|Почему параллельность выгодна|Источник|
|---|---|---|---|
|**Массив/диапазон с тяжёлым CPU‑расчётом** (поиск простых чисел)|`java\nlong primes = IntStream.range(2_000_000, 3_000_000)\n .parallel()\n .filter(MyMath::isPrime)\n .count();\n`|Каждый элемент проверяется независимо и дорого по времени; диапазон легко делится на под‑задачи, загрузив все ядра.|[NUS CS2030S](https://nus-cs2030s.github.io/2425-s2/37-parallel.html)|
|**Большая `ArrayList` (≈1 000 000 + записей) с тяжёлым фильтром**|`java\nlong rich = customers.parallelStream()\n .filter(c -> c.getReceivables() > 25_000)\n .count();\n`|`ArrayList` раскалывается `Spliterator`‑ом без копирования; при таком объёме выигрывают даже слияние и планирование задач.|[InfoQ](https://www.infoq.com/articles/java-collections-streams/)|

---

### Когда `parallelStream()` хуже, чем `stream()`

| Сценарий                                                         | Анти‑пример                                                         | Почему медленнее/опасно                                                                                                               | Источник                                                                                                           |
| ---------------------------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Малая коллекция / лёгкие операции, плюс `System.out.println`** | `names.parallelStream()<br>.forEach(System.out::println);`          | Координация потоков дороже самого вывода; кроме того, `println` синхронизирован и превращается в «бутылочное горлышко».               | [Stack Overflow](https://stackoverflow.com/questions/20375176/should-i-always-use-a-parallel-stream-when-possible) |
| **Каждый элемент делает блокирующий I/O (REST‑запрос, БД)**      | `results = ids.parallelStream() .map(api::call) // сеть .toList();` | Потоки общего `ForkJoinPool` залипают на I/O, не отдавая ядра CPU‑задачам; накладные расходы растут, а пропускная способность падает. | [LinkedIn](https://www.linkedin.com/pulse/optimizing-parallel-streams-java-best-practices-amit-jindal-ul7of)       |

---

**Правило большого пальца**  
Если N × Q (N — число элементов, Q — «тяжесть» операции) не велико, параллельность может дать _минус_ производительность; для лёгких операций часто нужен минимум ≈ 10 000 элементов, чтобы «отбить» накладные расходы.