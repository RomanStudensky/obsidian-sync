
Все классы наследуют Object

# Методы

## ***wait(), notify(), notifyAll()***
методы для установки потока в статус ожидания и пробуждения потоков.

## ***toString()***
метод для формирования строки по объекту. 
**По умолчанию** метод возвращает строку, содержащую имя класса, символ «@» и шестнадцатеричное представление хэш-кода объекта без знака.

Контракт между `equals()` и `hashCode()` в Java — это фундаментальное правило для корректной работы коллекций (`HashMap`, `HashSet`, `ConcurrentHashMap` и т.д.).

## ***getClass(), clone()***

По умолчанию метод ***clone()*** выполняет **поверхностное копирование** — копируется только «поверхность» объекта, а не вся сложная структура.

## ***equals() & hashCode()***

### Правила (контракт):

1. **Если `a.equals(b) == true`, то `a.hashCode() == b.hashCode()` обязательно.**  
    → одинаковые объекты должны попадать в одну "корзину" в хэш-структурах.
    
2. **Если `a.equals(b) == false`, то `a.hashCode()` _может_ быть равным, но не обязан.**  
    → коллизии допустимы, но замедляют работу.
    
3. **`hashCode()` должен быть стабильным во времени, пока объект не изменился.**  
    → если объект лежит в `HashSet`, менять поля, участвующие в `equals/hashCode`, опасно.
    
4. **`equals()` должен быть:**
    - **рефлексивен**: `a.equals(a)` всегда `true`
    - **симметричен**: если `a.equals(b)`, то и `b.equals(a)`
    - **транзитивен**: если `a.equals(b)` и `b.equals(c)`, то `a.equals(c)`
    - **согласован**: многократные вызовы дают один результат, если екты не изменились
    - **не равен `null`**: `a.equals(null)` всегда `false`
---

### Пример правильной реализации

``` java
import java.util.Objects;

public class Person {

	private final String name;
	private final int age;
	
	public Person(String name, int age) {
		this.name = name;
		this.age = age;
	}
	
	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (!(o instanceof Person)) return false;
		Person person = (Person) o;
		return age == person.age && Objects.equals(name, person.name);
	}  
	
	@Override     
	public int hashCode() {
		return Objects.hash(name, age);
		}
	}
}
```

---

### Если контракт нарушить:

- **equals true, hashCode разные** → два одинаковых объекта могут оказаться в разных бакетах `HashMap` и фактически потеряться.
    
- **equals false, hashCode одинаковые** → допустимо, но будут коллизии → падение производительности.