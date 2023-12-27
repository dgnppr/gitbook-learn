# 프로세스 동기화

<br>

## 경쟁 상태(Race Condition)

여러 프로세스 혹은 쓰레드가 동시에 동일한 리소스에 접근하여 조작했을때, 실행 결과가 접근 순서에 의존하는 상황 


<br>

## 임계 영역(Critical Section)

다른 프로세스 혹은 쓰레드와 공유하는 리소스를 변경하는 작업을 수행하는 코드 조각

<br>

## CSP Solution 필수 조건

1. Mutual Exclusion(상호 배제): 한 프로세스가 임계 영역에서 실행되고 있다면, 다른 프로세스들은 임계 영역에 진입할 수 없어야 한다
2. Progress(진행): 임계 영역에서 실행되는 프로세스가 없고, 임계 영역에 진입하려는 프로세스가 있을 때 진입하려는 프로세스는 진입할 수 있어야 한다. -> **Deadlock 방지**
3. Bounded Waiting(한정된 대기): 진입 허가 요청 대기 시간이 한정적이여야 한다 -> **Starvation 방지**

CSP(Critical Section Problem)을 해결하려면 위 세가지 조건을 모두 만족해야 한다.

<br>

## 상호배제(Mutex)

한 프로세스가 임계 영역에서 실행되고 있다면, 다른 프로세스들은 임계 영역에 진입할 수 없어야 한다

<br>

### 상호배제 조건

1. 두 프로세스는 동시에 CS에 진입할 수 없다
2. 프로세스의 속도나 프로세서 수에 영향을 받지 않는다
3. 공유 자원을 사용하는 프로세스만 다른 프로세스를 차단할 수 있다.
4. 프로세스가 공유 자원을 사용하려고 너무 오래 기다려서는 안된다.

<br>

### 상호배제 방법

| 수준  | 방법                   | 종류                                      |
|-----|----------------------|-----------------------------------------|
| 고수준 | 소프트웨어로 해결            | 데커 알고리즘, 피터슨 알고리즘, 램포트 베이커리 알고리즘        |
|     | 프로그래밍 언어나 OS 수준에서 제공 | 세마포어, 모니터, 뮤텍스                          |
| 저수준 | 하드웨어로 해결             | TAS(Test And Set), CAS(Compare And Set) |

<br>

### 소프트웨어 동기화 - 피터슨 알고리즘

```C
do {
	flag[i] = true;
    turn = j;
    while(flag[j] && turn ==j); // entry section
    
	// critical section
    
    flag[i] = false; // exit section
    
    // remainder section
    
} while(true);
```

- CSP 조건 모두 충족(상호 배제, 진행, 한정된 대기)
- 현대 컴퓨터 구조가 `load`, `store`와 같은 기본적인 기계어를 수행하는 방식 때문에 올바르게 실행된다고 보장하지 않는다.
  - `entry section`에서 인터럽트가 발생하면 `race condition`이 발생할 수 있다.

<br>

### 하드웨어 동기화 - TAS

현대 컴퓨터는 워드 내용을 검사하고 변경하거나, 두 워드의 내용을 원자적으로 교환할 수 있는 원자적 연산을 제공한다.

```C
boolean test_and_set(boolean *target){
	boolean rv = *target;
    *target = true;
    return rv;
}

do{
    while(test_and_set(&lock));  // entry section
        
	// critical section
        
	lock = false; // exit section
    
} while(true);
```

- Mutual Exclusion 조건 만족
- Bounded Waiting 조건 만족하지 않음
  - `busy waiting`을 하기 때문에 다른 프로세스가 무한히 대기할 수 있다.

### 하드웨어 동기화 - CAS

```C
int compare_and_swap(int *value, int expected, int new_value){
	int temp = *value;
    
    if(*value==expected)
    	*value = new_value;
        
	return temp;
}

while(true){
	while(compare_and_swap(&lock,0,1)!=0); // entry section
    
    // critical section
    
    lock = 0;
}
```

- Mutual Exclusion 조건 만족
- Bounded Waiting 조건 만족하지 않음
  - `busy waiting`을 하기 때문에 다른 프로세스가 무한히 대기할 수 있다.

<br>

### 뮤텍스락

```C
acquire(){
	while(!available); // busy wait
    available = false;
}

release(){
	available = true;
}
```

- Mutual Exclusion 조건 만족
- Progress, Bounded Waiting 조건 만족하지 않음
- CPU 코어을 점유하면서 락을 대기하기 때문에 컨텍스트 스위치 오버헤드가 생기지 않는다.
  - Busy waiting(`Spin lock`) 하기 때문에 CPU 싸이클 낭비.

<br>

### 세마포어

```C
wait(S){
	while(S<=0){
    	sleep();
    }
    S--;
}

signal(S){
	S++;
    if(S>0({
    	wakeup();
    }
}
```

- 세마포어 S는 정수이다.  (S = 가용한 자원의 개수)
- wait(),signal() 연산을 사용하여 S 값을 변경한다.
  - wait(): 세마포어 S를 감소
  - signal(): 세마포어 S를 증가
- wait(),signal() 연산은 atomic 하게 수행되어야 한다. 
- **세마포어가 0이하 이면 `Busy waiting`을 하지 않고, 대기 큐로 이동한다.** 
- signal() 연산으로 세마포어가 증가하면 대기 큐에서 대기하는 프로세스는 레디 큐로 이동한다
- `Busy waiting` 해결
- Mutual Exclusion 조건 만족
- Progress, Bounded Waiting 조건 만족하지 않음
- 임계 영역이 짧은 시간에 수행될 경우, 컨텍스트 스위치 오버헤드가 커진다.
- 순서가 정확하지 않으면 timing error가 발생하고 코드가 복잡할수록 에러를 찾기 쉽지 않다.


<br>

### 모니터

![5_17_MonitorConditions](https://github.com/dragonappear/learn/assets/89398909/14d0c0e6-14b6-406a-ac8c-b753d5d86840)

- 모니터는 내부에 `Mutual Exclusion`을 보장하는 연산을 포함하는 `ADT`이다.
- 모니터 내부에서 condition 변수를 사용하여 부가적인 동기화를 작성할 수 있다.
  - condition 변수는 wait(),signal()을 제공한다.
  - signal()은 정확히 하나의 waiting 프로세스를 재개한다.
- 모니터 내부 구현은 뮤텍스락, 세마포어 등 구현하기 나름이다.

<br>