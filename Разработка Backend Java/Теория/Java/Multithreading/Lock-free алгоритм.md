
# Lock-free алгоритм: volatile и immutable

Разбор одного из `lock-free` алгоритмов на примере ключевого слова `volatile` и шаблона проектирования `immutable`.

## Volatile

Думаю, что многие встречали разные определения ключевого слова `volatile`. Приведём некоторые.

Определение ключевого слова "своими словами":

> Ключевое слово `volatile` используется для обозначения  
> переменных, которые могут быть изменены несколькими потоками. Оно  
> гарантирует, что изменения переменной видны другим потокам.

Если копнуть глубже в JMM, найдём такое определение:

> Запись в `volatile` переменную `happens-before` каждое последующее чтение той же самой переменной (A write to a volatile variable `v` happens-before all subsequent reads of `v` by any thread).

Исходя из определения `volatile`, чтение примитива или ссылки всегда будут соответствовать последнему записанному значению без использования каких-либо блокировок, как по монитору объекта и т.п.

## Lock-free

Так причём тут `lock-free` спросите вы? А при том, что если пометить переменную как `volatile`, то не нужно осуществлять никаких блокировок для чтения и записи примитива или ссылки.

Рассмотрим пример кода, использующий `lock-free` и `volatile`:

```java
public class Configuration {
	private volatile boolean tracingEnabled = false;

	public void enableTracing() {
		tracingEnabled = true;
	}

	public void disableTracing() {
		tracingEnabled = false;
	}

	public boolean isTracingEnabled() {
		return tracingEnabled;
	}
}
```

В этом случае каждое обращение кода для запроса необходимости трассировки через метод `isTracingEnabled` не требует никакой синхронизации и блокировки. Всегда будет возвращено последнее установленное значение.

При этом, если из панели администрирования нашего приложения будут вызваны методы `enableTracing` или `disableTracing`, то сразу после установки нового значения переменной `tracingEnabled`, весь работающий код станет писать трассировку, либо прекратит это делать.

А если конфигурация нашего приложения более сложная? Например, мы хотим управлять уровнем логирования "на лету". Это легко сделать, расширив наш класс Configuration:

```java
public class Configuration {
	private volatile boolean tracingEnabled = false;
	private volatile Logger.Level logLevel = Logger.Level.OFF;

	public void enableTracing() {
		tracingEnabled = true;
	}

	public void disableTracing() {
		tracingEnabled = false;
	}

	public boolean isTracingEnabled() {
		return tracingEnabled;
	}

	public void setLogLevel(Logger.Level level) {
		logLevel = level;
	}

	public boolean isLoggable(Logger.Level level) {
		return logLevel.getSeverity() <= level.getSeverity();
	}
}
```

## Immutable

Так причём тут `immutable` спросите вы.

Давайте рассмотрим другой подход к хранению данных нашего класса конфигурации. Сделаем именованные настройки вида `key-value`.

Оставим для совместимости уже реализованные методы по работе с трассировкой и уровнем логирования:

```java
public class Configuration {
	public static final String KEY_TRACING = "tracing";
	public static final String KEY_LOG_LEVEL = "logLevel";

	private volatile Map<String, String> parameters = new HashMap<>();

	public void setValue(String key, String value) {
		parameters.put(key, value);
	}

	public String getValue(String key) {
		return parameters.get(key);
	}

	public void enableTracing() {
		setValue(KEY_TRACING, Boolean.TRUE.toString());
	}

	public void disableTracing() {
		setValue(KEY_TRACING, Boolean.FALSE.toString());
	}

	public boolean isTracingEnabled() {
		String value = getValue(KEY_TRACING);
		return value != null && Boolean.parseBoolean(value);
	}

	public void setLogLevel(Logger.Level level) {
		setValue(KEY_LOG_LEVEL, level.getName());
	}

	public boolean isLoggable(Logger.Level level) {
		String currentLevel = getValue(KEY_LOG_LEVEL);
		return Logger.Level.valueOf(currentLevel).getSeverity() <= level.getSeverity();
	}
}
```

Здесь мы видим два новых метода `setValue` и `getValue`. Но **_данный код небезопасен_**, потому, что ключевым словом `volatile` помечен сложный объект и `happens-before` не распространяется на сам объект, а только на ссылку на него.

Для решения данной проблемы на помощь приходит паттерн `Immutable`. Давайте перепишем код с помощью данного паттерна, чтобы он стал безопасен для использования:

```java
public class Configuration {
	// ...
	private volatile Map<String, String> parameters =
		Collections.unmodifiableMap(new HashMap<>());

	public void setValue(String key, String value) {
		Map<String, String> map = new HashMap<>(parameters);
		map.put(key, value);
		map = Collections.unmodifiableMap(map);
		parameters = map;
	}

	public String getValue(String key) {
		return parameters.get(key);
	}
	// ...
}
```

Здесь мы изменили метод `setValue`. Теперь `happens-before` для корректной установки нового именованного значения в нашу коллекцию настроек обеспечен. Метод `Collections.unmodifiableMap()` не обязателен - если вызов метода удалить, код тоже будет работать, но он даёт чётко понять, что данная коллекция не подлежит изменению.

Это полезно для дальнейшего возможного расширения кода. Или можно оставить комментарий о том, что данная коллекция должна изменяться только путём присвоения в переменную `parameters` **_нового_** объекта, который **_никогда не будет меняться_**.

## Блокировки

И всё-таки они нужны в определённых ситуациях. Если у нас панель администрирования обслуживается одним потоком (только один поток осуществляет изменение в классе `Configuration`), то код класса потокобезопасен и работает как `lock-free` алгоритм.

Но что делать, если пишущих потоков больше? В этом случае в момент пересборки `immutable` объекта мы можем потерять данные от части потоков. Например, к нам в метод `setValue` пришли два потока, оба создали себе копию коллекции `parameters`, добавили каждый свой параметр и по очереди заменили ссылку. При этом данные первого потока, который изменил ссылку на коллекцию `parameters` будут утеряны.

Что делать, чтобы обезопасить изменения с двух и более потоков? Один из вариантов - защитить пересборку `immutable` объекта блокировкой. Давайте изменим метод `setValue`, чтобы изменение данных нашей конфигурации было безопасно из 2 и более потоков:

```java
public class Configuration {
	private volatile Map<String, String> parameters =
		Collections.unmodifiableMap(new HashMap<>());

	public synchronized void setValue(String key, String value) {
		Map<String, String> map = new HashMap<>(parameters);
		map.put(key, value);
		map = Collections.unmodifiableMap(map);
		parameters = map;
	}

	public String getValue(String key) {
		return parameters.get(key);
	}
}
```

В приведённом коде мы добавили синхронизацию по монитору объекта `Configuration` на время пересборки нашего `immutable` объекта и присвоения новой ссылки в нашу `volatile` переменную.

Такой код абсолютно безопасен для использования множеством потоков.

## Atomic

А можно ли написать такой же класс полностью `lock-free`? Можно. Но для этого нам понадобится уже не просто `volatile`, а атомарные операции, построенные на `CAS` (Compare and swap). Они реализованы в пакете `java.util.concurrent.atomic`.

Давайте перепишем метод на использование `AtomicReference` вместо `volatile`:

```java
public class Configuration {
	// ...
	private final AtomicReference<Map<String, String>> parameters =
		new AtomicReference<>(Collections.unmodifiableMap(new HashMap<>()));

	public void setValue(String key, String value) {
		for (;;) {
			Map<String, String> currentMap = parameters.get();
			Map<String, String> newMap = new HashMap<>(currentMap);
			newMap.put(key, value);
			newMap = Collections.unmodifiableMap(newMap);

			if (parameters.compareAndSet(currentMap, newMap)) {
				break;
			}
		}
	}

	public String getValue(String key) {
		return parameters.get().get(key);
	}
	// ...
}
```

В этом коде полностью отсутствую блокировки и он безопасен для использования из любого количества потоков. Однако, стоит иметь в виду, что если пишущих потоков будет много, а наш `immutable` объект большой, то может наблюдаться повышенное CPU и снижение latency пишущего метода из-за того, что они будут одновременно пересобирать объект и мешать друг другу выполнить `compareAndSet`.

## Заключение

`Lock-free` алгоритмы используются в Bercut в нашей ESB-шине и в ряде сервисов, где необходимо добиться минимального времени отклика.

Основные подводные камни, которые надо учесть при работе с `volatile` + `immutable` это:

- Данный подход даёт лучшие показатели при профиле нагрузки от 90% до 99.9(9)% на чтение. Если операций записи больше, то лучше перейти на стандартные блокировки или read/write блокировки.
    
- При пересборке объекта, особенно большого, используется много дополнительной памяти. При этом, необходимо учесть, что высвобождаемый по ссылке `volatile` старый объект скорее всего уже находится в old-памяти.
    
- Если пересборка объекта - длительный процесс, а пишущих потоков много, то возникает риск долгой парковки части потоков при возникновении блокировки. В случае с `AtomicReference` придётся активнее использовать CPU и злоупотреблять потреблением памяти через создание множества новых объектов.
    

Однако, при использовании подхода в правильном месте, можно получить прирост производительности работы вашего многопоточного приложения.