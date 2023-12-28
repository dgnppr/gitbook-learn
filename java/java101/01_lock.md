# Java Lock

<br>

자바는 모니터락 기반 동기화를 제공한다

## 모니터락(Monitor Lock)

![5_17_MonitorConditions](https://github.com/dragonappear/learn/assets/89398909/14d0c0e6-14b6-406a-ac8c-b753d5d86840)

- 모니터는 내부에 `Mutual Exclusion`을 보장하는 연산(ex: 뮤텍스락, 세마포어)을 포함하는 ADT 이다.
- 모니터 내부에서 `condition` 변수를 사용하여 부가적인 동기화를 작성할 수 있다.
- `condition` 변수는 `wait()`과 `notify()` 메서드를 통해 사용한다.
  - `wait()` 메서드는 `condition` 변수를 통해 특정 조건을 만족할 때까지 대기한다. 
    - 임계 영역 내에서 특정 조건이 만족될 때까지 현재 쓰레드를 대기 상태로 만든다.
    - `wait()` 메서드를 호출하면 현재 쓰레드는 대기 상태가 되고, 모니터락을 반환한다.
    - `wait()` 메서드는 `synchronized` 블록 내에서만 사용할 수 있다.
  - `notify()`는 대기하고 있는 쓰레드 중 하나를 깨우고, `notifyAll()`은 대기하고 있는 모든 쓰레드를 깨운다.
    - `notify()`와 `notifyAll()`은 `signal()`과 다르게 `synchronized` 블록 내에서만 사용할 수 있다.

자바에서는 쓰레드 간 동기화를 하는 방법으로 `synchronized` 키워드를 사용하는 것과 `java.util.concurrent` 패키지에 있는 클래스를 사용하는 방법이 있다.

<br>

## synchronized

- 임계 영역에 해당하는 코드 블록에 선언하는 키워드
- 해당 코드 블록에는 모니터락 객체 인스턴스를 획득해야 진입 가능 
  - 모니터락 객체 인스턴스 지정 가능
- 메서드에 선언하면 메서드 코드 블록 전체가 임계 영역으로 지정됨
  - 이 경우 모니터락을 가진 객체 인스턴스는 this 객체 인스턴스

```Java
class SynchronizedTest {

    static final int MAX = 10000;

    public static void main(String[] args) throws InterruptedException {
        increment1(); // 메서드 전체에 synchronized(클래스 자체가 모니터락 소유)-> 50000
        increment2(); // 메서드 내부에서 synchronized -> 50000
        increment3(); // 쓰레드마다 다른 모니터락 객체 인스턴스 설정 -> 49998
        increment4(); // 쓰레드마다 같은 모니터락 객체 인스턴스 설정 -> 50000
    }

    static void increment1() throws InterruptedException {
        Monitor.init();

        Thread[] threads = createThreads();

        start(threads, () -> {
            for (int j = 0; j < MAX; j++) {
                Monitor.increment1();
            }
        });

        join(threads);

        System.out.println("[Ex1] count = " + Monitor.count);
    }

    static void increment2() throws InterruptedException {
        Monitor.init();

        Thread[] threads = createThreads();

        start(threads, () -> {
            for (int j = 0; j < MAX; j++) {
                Monitor.increment2();
            }
        });

        join(threads);

        System.out.println("[Ex2] count = " + Monitor.count);

    }


    private static void increment3() throws InterruptedException {
        Monitor.init();

        Thread[] threads = createThreads();

        for (int i = 0; i < threads.length; i++) {
            Monitor monitor = new Monitor();
            threads[i] = new Thread(() -> {
                for (int j = 0; j < MAX; j++) {
                    monitor.increment3();
                }
            });

            threads[i].start();
        }

        join(threads);

        System.out.println("[Ex3] count = " + Monitor.count);
    }

    static void increment4() throws InterruptedException {
        Monitor.init();

        Monitor monitor = new Monitor();
        Thread[] threads = createThreads();

        start(threads, () -> {
            for (int j = 0; j < MAX; j++) {
                monitor.increment3();
            }
        });

        join(threads);

        System.out.println("[Ex4] count = " + Monitor.count);
    }

    private static Thread[] createThreads() {
        return new Thread[5];
    }

    private static void start(Thread[] threads, Runnable task) {
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(task);
            threads[i].start();
        }
    }

    private static void join(Thread[] threads) throws InterruptedException {
        for (int i = 0; i < threads.length; i++) {
            threads[i].join();
        }
    }
}

class Monitor {
    static int count = 0;
    static Object object = new Object();

    static void init() {
        count = 0;
    }

    synchronized public static void increment1() {
        count++;
    }

    static void increment2() {
        synchronized (object) {
            count++;
        }
    }

    public void increment3() {
        synchronized (this) {
            count++;
        }
    }
}

```

<br>

## java.util.concurrent

### ReentrantLock

- `synchronized` 키워드와 같은 기능을 하는 클래스
- `synchronized` 키워드와 달리 `lock()`과 `unlock()` 메서드를 사용하여 명시적으로 임계 영역을 지정한다.
- **재진입이 가능한 락이다. 이미 획득한 락을 반복해서 획득할 수 있다.**
  - 재귀 호출이나 반복 호출에서 락을 여러번 획득할 때 사용
- 타임 아웃 설정으로 **Lock polling** 지원
- 대기 중인 쓰레드를 **선별적으로** 깨울 수 있음
- 락 획득을 위해 레디 큐에 있는 쓰레드에게 **인터럽트를** 걸 수 있음

```Java
class ReentrantLockTest {

    @Test
    @DisplayName("생성자")
    void test1() throws Exception {
        new ReentrantLock();

        boolean fair = true; // true로 주면 가장 오래 기다린 쓰레드가  lock을 얻을 수 있도록 처리
        new ReentrantLock(fair); // 어떤 쓰레드가 가장 오래 기다렸는지 확인하는 과정이 필요하므로 성능이 떨어진다
    }

    @Test
    @DisplayName("API")
    void test2() throws Exception {
        ReentrantLock lock = new ReentrantLock();

        lock.lock(); // 락 잠금
        lock.unlock(); // 락 해제
        lock.isLocked(); // 락 잠금 확인
        lock.tryLock(); // long polling 방식
        lock.tryLock(1, TimeUnit.MINUTES); // 타임아웃 설정
    }

    @Test
    @DisplayName("Condition 으로 선별적 통지")
    void test3() throws Exception {
        ReentrantLock lock = new ReentrantLock();

        Condition task1 = lock.newCondition();
        Condition task2 = lock.newCondition();
        
        task1.await();
        task1.signal();
    }
}
```

<br>

### Atomic

![Screenshot 2023-12-27 at 15 06 59](https://github.com/dragonappear/learn/assets/89398909/a19ebba0-40cf-452a-a074-805e608cdfba)

- CAS(Compare And Swap) 연산을 통해 원자적 연산 제공, ThreadSafe
- 메모리 가시성 보장. 즉, 한 스레드에서 수정한 값은 다른 스레드에도 즉시 반영된다
- 논 블락 알고리즘(비차단 알고리즘) -> 락 기반 알고리즘이 아님
  - CPU 코어를 점유하면서 반복적으로 조건 평가

```Java
class AtomicTest {

    static final int MAX = 100_000;

    @Test
    @DisplayName("AtomicBoolean")
    void test1() throws Exception {
        AtomicBoolean flag = new AtomicBoolean(false);

        while (!flag.compareAndSet(false, true)) {
            // 임계 영역
        }
    }

    @Test
    @DisplayName("AtomicInteger")
    void test2() throws Exception {
        AtomicInteger count = new AtomicInteger();

        Thread[] threads = createThreads();

        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < MAX; j++) {
                    count.incrementAndGet();
                }
            });
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].join();
        }

        System.out.println("atomicInteger.get(): " + count.get());
    }

    private Thread[] createThreads() {
        Thread[] threads = new Thread[10];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread();
        }
        return threads;
    }
}
```