Более гибкий аналог [[Synchronized]] 
работает так
```java
ReentrantLock reentrantLock = new ReentrantLock();
int i = 0;

public void increment() {
	reentrantLock.lock();
	i++;
	reentrantLock().unlock();
}
```

Добавляются **conditions**
Удобно использвать для реализации последовательности выполнения методов в многопоточке [[Happens-before]]

```java
public class FooSafe {
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition firstMethodCalled = lock.newCondition();
    private final Condition secondMethodCalled = lock.newCondition();

    public void first() {
        lock.lock();
        try {
            System.out.println("first");
            firstMethodCalled.signal();
        } finally {
            lock.unlock();
        }
    }

    public void second() {
        lock.lock();
        try {
            firstMethodCalled.await();
            System.out.println("second");
            secondMethodCalled.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void third() {
        lock.lock();
        try {
            secondMethodCalled.await();
            System.out.println("third");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```
