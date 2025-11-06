### Зачем вообще Spring AOT и «хинты» (runtime hints)

- **AOT (Ahead-of-Time)-обработка** запускает ваш `ApplicationContext` на этапе _сборки_, а не в рантайме. Контейнер создает только **BeanDefinition’ы**, генерирует Java-код, заранее собирает прокси-классы и формирует JSON-файлы с метаданными, которые нужны GraalVM, — благодаря этому нативное приложение стартует за миллисекунды и занимает меньше памяти. [Home](https://docs.spring.io/spring-boot/docs/3.2.3/reference/html/native-image.html)
    
- GraalVM работает по «закрытому миру»: все динамическое (рефлексия, загрузка ресурсов, JDK-прокси, сериализация, JNI, время инициализации классов) должно быть описано **явно** в **hint-файлах**. Именно их Spring генерирует или ждет от вас. [Home](https://docs.spring.io/spring-framework/reference/core/aot.html)
    

---

## Что именно генерирует Spring AOT

|Артефакт|Где лежит (Maven)|Кому нужен|
|---|---|---|
|**Java-код** (`*_BeanDefinitions.java`, `ApplicationContextInitializer`)|`target/spring-aot/main/sources`|Ускоряет старт на JVM и в native-image|
|**Прокси-классы**|`target/spring-aot/main/classes`|Замена CGLIB во время билда|
|**Hint-JSON**  <br>`reflect-config.json`, `resource-config.json`, `proxy-config.json`, `serialization-config.json`, `jni-config.json`|`META-INF/native-image`|Подхватываются GraalVM при компиляции [Home](https://docs.spring.io/spring-boot/docs/3.2.3/reference/html/native-image.html)|

Всё это добавляется автоматически, когда вы вызываете `mvn -Pnative spring-boot:build-image` или `./gradlew bootBuildImage` (при наличии зависимости **GraalVM Native Support**).

## Как запустить JVM-приложение в AOT-режиме


```bash
./mvnw -DskipTests spring-boot:process-aot   # генерируем артефакты 
java -Dspring.aot.enabled=true -jar target/app.jar
```

System-property `spring.aot.enabled=true` говорит Boot читать сгенерированный `ApplicationContextInitializer` и JSON-хинты вместо обычного динамического старта. [Home](https://docs.spring.io/spring-boot/reference/packaging/aot.html?utm_source=chatgpt.com)

---

## Частые ошибки и «грабли»

|Проблема|Как лечить|
|---|---|
|**«Class must be initialized at run time»**|Добавьте `@InitializationHint(types = MyTroublesome.class, initTime = InitializationTime.BUILD)` или настройте `--initialize-at-build-time=` через hints.|
|**Отсутствие прокси-метаданных**|Зарегистрируйте интерфейсы в `hints.proxies()`.|
|**Wildcard-рефлексия** (`*.**`) раздувает образ|Уточняйте категории (`MemberCategory`) и список классов.|
|**Профили Spring или `@Conditional…` меняют состав бинов**|В AOT-режиме решения принимаются на билд-тайме; переключать профили после сборки нельзя. [Home](https://docs.spring.io/spring-boot/docs/3.2.3/reference/html/native-image.html)|
|**Динамическая регистрация бинов после `ApplicationReadyEvent`**|Невозможна в native-image; перенесите регистрацию в `BeanFactoryPostProcessor` до AOT.|