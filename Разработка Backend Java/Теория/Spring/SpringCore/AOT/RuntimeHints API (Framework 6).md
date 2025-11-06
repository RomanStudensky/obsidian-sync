
- Ядро — класс `org.springframework.aot.hint.RuntimeHints`.  
    _Собирает требования_ по категориям: `reflection()`, `resources()`, `serialization()`, `proxies()`, `jni()`, `classInitialization()` и т.д. [Home](https://docs.spring.io/spring-framework/reference/core/aot.html)
    
- Реализуйте **`RuntimeHintsRegistrar`** и зарегистрируйте его через `@ImportRuntimeHints` или файл `META-INF/spring/aot.factories` — тогда ваши подсказки попадут в общий `RuntimeHints` контейнера. [Home](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/aot/hint/RuntimeHintsRegistrar.html)
    

```java
@Component
@ImportRuntimeHints(MyHints.class)
class GreetingService { … }

public class MyHints implements RuntimeHintsRegistrar {
  @Override
  public void registerHints(RuntimeHints hints, ClassLoader cl) {
    hints.reflection().registerType(
	    Greeting.class,
	    MemberCategory.INVOKE_PUBLIC_CONSTRUCTORS,
	    MemberCategory.INVOKE_PUBLIC_METHODS
    );
    
    hints.resources().registerPattern("i18n/messages_*.properties");
    hints.proxies().registerJdkProxy(ApiClient.class);
  }
}
```

## Аннотации-шорткаты

Spring Boot предоставляет ряд meta-аннотаций поверх `RuntimeHints` (доступны из пакета `org.springframework.aot.hint`):

|Аннотация|Для чего|
|---|---|
|**`@ReflectionHint` / `@RegisterReflectionForBinding`**|быстрый способ описать классы, которые должны быть доступны через рефлексию (часто для JSON-binding).|
|**`@ResourceHint`**|добавить шаблоны ресурсов (например, `schema/*.sql`).|
|**`@JdkProxyHint`**|описать JDK-прокси-интерфейсы.|
|**`@InitializationHint`**|указать, какие классы или пакеты нужно инициализировать _на этапе билда_ (`BUILD`) или оставить на рантайм (`RUNTIME`). [Home](https://docs.spring.io/spring-native/docs/current/api/org/springframework/nativex/hint/InitializationHint.html)|

---

## Когда хинты нужны вручную

1. **Рефлексия в сторонних библиотеках**, которые ещё не публикуют reachability-metadata.
2. **Динамически регистрируемые бины** (но помните: сами бины должны появиться _до_ AOT-фазы).
3. **Загрузка файлов по шаблону** (`/dicts/*.txt`, Liquibase changelog’и).
4. **JPA/Hibernate**: сущности, не обнаруженные сканером (см. `PersistenceManagedTypes`). [Home](https://docs.spring.io/spring-framework/reference/core/aot.html)
5. **Кастомные JDK-прокси** (RPC-клиенты, security-advices).

Совет: сначала запустите приложение с агентом

```java 
-agentlib:native-image-agent=config-output-dir=src/main/resources/META-INF/native-image/ \      -Dspring.aot.enabled=true -jar app.jar
```

и «прокликайте» критические маршруты; агент сгенерирует базовые JSON-конфиги, которые потом можно «перевести» в `RuntimeHints`. [Home](https://docs.spring.io/spring-boot/docs/3.2.3/reference/html/native-image.html)