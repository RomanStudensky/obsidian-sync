**Сделать Kafka синхронной** можно через настройку конфигурации или использование сторонних библиотек, например Spring Kafka. [baeldung.com](https://tr-page.yandex.ru/translate?lang=en-ru&url=https%3A%2F%2Fwww.baeldung.com%2Fspring-kafka-request-reply-synchronous)[client.sbertech.ru](https://client.sbertech.ru/docs/public/SEI/4.9/KFGT/4.9/documents/developer-guide/index.html)[ngdeveloper.com](https://ngdeveloper.com/kafka-asynchronous-synchrous-spring-boot-producer-example/)

### Настройка конфигурации

В Apache Kafka для синхронной работы необходимо указать, что **производитель (Producer) ждёт подтверждения от брокера**, что сообщение доставлено и сохранено в топик. Для этого в конфигурации Producer нужно настроить параметр 

acks

. [selectel.ru](https://selectel.ru/blog/tutorials/go-apache-kafka/)[proglib.io](https://proglib.io/p/vystrelil-i-zabyl-3-osnovnye-strategii-otpravki-soobshcheniy-v-kafka-2024-10-25)[habr.com](https://habr.com/ru/companies/otus/articles/861734/)

Возможные значения:

- acks=0 — продюсер не ждёт подтверждения;
- acks=1 — брокер ждёт подтверждения от одного лидера;
- acks=all  — брокер ждёт подтверждения от всех реплик.

 [habr.com](https://habr.com/ru/companies/otus/articles/861734/)[selectel.ru](https://selectel.ru/blog/tutorials/go-apache-kafka/)

### Использование Spring Kafka

В Spring Kafka для синхронной работы используется компонент **ReplyingKafkaTemplate**. Он управляет взаимодействием между темами запроса и ответа, позволяя реализовать синхронный обмен запросами-ответами. [baeldung.com](https://tr-page.yandex.ru/translate?lang=en-ru&url=https%3A%2F%2Fwww.baeldung.com%2Fspring-kafka-request-reply-synchronous)

Некоторые настройки в конфигурации Spring Kafka для синхронного режима:

- spring: kafka: sync-mode: enabled: true — включает синхронный режим;
- spring: kafka: sync-mode: reply-topic: <reply-topic> — указывает топик ответа;
- spring: kafka: sync-mode: timeout: <время ожидания ответа в секундах>