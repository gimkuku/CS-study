# 6. Process Synchronization

## 데이터의 접근

**Execution - Box** 

- CPU
- 컴퓨터 내부
- 프로세스

**Storage - Box**

- Memory
- 디스크
- 그 프로세스의 주소공간

## Race condition

- 여러 프로세스들이 동시에 공유 데이터를 접근할 때
- 공용 데이터에 대한 접근이 어떤 순서에 따라 이루어졌는지에 따라 그 실행 결과가 달라지는 상황
    
    → data inconsistency 발생 
    
    = 실행 순서를 정해주는 매커니즘이 필요하다! 

- S-box를 공유하는 E-box가 여럿 있는 경우, Race Condition의 가능성이 있음
    
    → Multiprocdessor system
    
    → 커널 내부 데이터를 접근하는 루틴들 간, 공유 메모리를 사용하는 프로세스들
    
    ( 예: 커널 모드 수행 중 인터럽트로 커널모드 다른 루틴 수행 시) 
    

## Race condition이 발생하는 원인

1. kernel 수행 중 인터럽트 발생 시
    1. 커널 모드 running 중 interrupt가 발생하여 인터럽트 처리루틴이 수행 
    2. 양 쪽 모두 커널 코드이무로 kernel addreeess space를 공유함
2. Process 가 system call을 하여 kernel mode로 수행중인데, context switch가 일어나는 경우
    
    ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/fae85d40-5bbd-4f8e-b4d2-79c37a8acc50/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220406%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220406T152030Z&X-Amz-Expires=86400&X-Amz-Signature=9381e6ccf634f850d9c71c89dbfb3931185e5f0deeb073c432d5d93b3a80cd3f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)
    
    1. 두 프로세스의 address space간에는 data sharing이 없음
    2. 하지만, system call을 하는 동안에는 kernel address space의 data를 access하게 됨
    3. 작업 중간에 cpu를 선점해가면 race condition 발생
    
    → 해결 : 커널 모드에서 수행중일 때는 CPU를 선점 X
    
3. Multiprocessor(cpu가 n개)에서 shared memory 내의 kernel data
    1. 어떤 CPU가 마지막으로 count를 store했느냐에 따라 결과가 달라짐 
    2. 이 경우, interrupt를 막는 것만으로 해결되지 않음 
        
        → 해결1: 한번에 하나의 CPU만이 커널에 들어갈 수 있게 하는 방법
        
        → 해결2: 커널 내 공유 데이터에 접근할때마다 그 데이터에 대한 lock/ unlock을 거는 법 
        
    

## Critical-Section Problem

**Critical - Section이란?**

- n 개의 프로세스가 공유 데이터를 동시에 사용하기를 원하는 경우
- 각 프로세스의 code segment에는 공유 데이터를 접근하는 코드
    
    → 이때 한번에 하나의 critical section만 작업을 해야 함! 
    

## Critical - Section Problem의 해결법

1. Mutal Exclusion (상호 배제)
    - 한번에 하나의 critical section만 진행해야 함
2. Progress (진행)
    - 아무도 진행하고 있지 않으면,  critical section에 들어가겠다는 프로세스를 들어가게 해야함
3. Bounded Waiting (한정 대기)
    - critical section에 들어가기 위해 기다리는 시간이 무한대면 안됨

→ 여기서 가정!

- 모든 프로세스의 수행속도 > 0
- 프로세스간의 상대적 수행속도는 가정하지 않는다.

### 1. Swap-turn 방식

- 한번씩 교대로 들어가는 방식 → mutual exclusion 만족

```c
int turn;
initially turn = 0;

Process P0
do{
	while(turn != 0);             // 내 차례가 아니라면 무한정 대기
	critical section              
	turn = 1;                     // 상대방 차례로 내가 바꿔줌
	remainder section
} while(1);
```

**한계** 

- 계속 remainder에 남아있으면, 상대방이 turn을 바꿔줘도 시작되지 않음
    
    → progress 조건 불만족
    
- 상대방이 turn을 자신으로 바꿔줘야만, 진입 가능
    
    → 하나의 프로세스가 더 빈번하게 critical section에 들어가야 한다면?
    

### 2. flag 방식

- 내 차례, 남의 차례를 모두 flag 에 담아두는 방식

```c
boolean flag[2];
initially flag[모두] = false;    // 아무도 CS에 없다

Process P0
do{
	flag[i] = true                // 나 들어가고 싶다고 표기
	while(flag[j] = true);        // 상대방이 들어가고 싶다하면 무한정 대기
	critical section              
	flag[i] = false               // 나 끝났다고 표기
	remainder section
} while(1);
```

**한계** 

- 계속 remainder에 남아있으면, 상대방이 turn을 바꿔줘도 시작되지 않음
    
    → progress 조건 불만족
    
- 둘다 2행 ( flag =true)인 상태로 있으면 서로 끊임없이 양보하는 상황 발생
    
    → bounded wating 불만족
    

### Peterson’s Algorithm

- flag도 하고 turn 도 하는 방식 ( 1, 2 방식 합친 버전)

```c
do{
	flag[i] = true;                // 나 들어가고 싶다고 표기
	turn = j;                      // 일단 남한테 양보
	while(flag[j] && turn ==j);    // 상대방이 들어갔고, 순서도 상대방이면 무한정 대기
	critical section              
	flag[i] = false                // 나 끝났다고 표기
	remainder section
} while(1);
```

- 3가지 조건 모두 충족!!!

**한계** 

- 계속 상대방의 상태를 while문으로 체크해줘야 함
    
    → busy waiting ( 계속 cpu와 memory를 소모) 
    

## Synchronization Hardware

- 하드웨어 적으로 Test & modify (상대방  체크하고 turn 바꾸기) 를 한번에 할 수 있다면, 
이런 문제는 해결할 수 있음!!

```c
do{
	while(Test_and_Set(lock));
	critical section              
	lock = false                
	// 상대방은 Test_and_Set(false)로 기다리고 있다가 lock=true로 바꾸고 탈출하게 됨! 
	remainder section
} while(1);
```

- 이것도 busy waiting !

## Semaphores

- atomic한 방식을 추상화 시킴
- atomic한 방식이란? 이 사이에서 context switch가 불가능 한 것!!

```c
do{
	P(mutex);
	critical section              
	V(mutex);	
	remainder section
} while(1);
```

- 이때 P와 V를 busy waiting 방식으로 구현하면 비효율적임
    
    → P를 **Block & Wakeup 방식** 구현
    

### Block / Wakeup 방식

1. Semaphore를 다음과 같이 정의
    
    ```c
    typedef struct{
    	int value;
    	struct process *L;       // 이게 wait queue
    }semaphore;
    ```
    
2. block과 wakeup을 다음과 같이 가정
    - block : 커널은 block을 호출한 프로세스를 suspend 시킴
        
        → 프로세스의 PCB를 wait queue에 넣음 
        
    - wakeup(P) : block된 프로세스를 wakeup 시킴
        
        → 프로세스의 PCB를 ready queue로 옮김 
        
    - Semaphore 의 구조
        
        value L → PCB → PCB → PCB → ,,,
        
3. semaphore 연산이 다음과 같이 정의됨
    
    ```c
    P(S) : 대기 큐에 넣는 애
    S.value --;
    if(S.value <0){
    	add this process to L;
    	block();
    }
    
    V(s) : 대기 큐에서 깨우는 애
    if(S.value <= 0){
    	remove a process P from S.L;
    	wakeup(P);
    }
    ```
    
    - 대기 하면서 CPU를 소모하지 않음 ⇒ Busy Waiting 문제 해결
    - 이때 Semaphore이 1이라면 (= binary semaphore) **mutex**라고 부름

### Block/ wakeup 을 사용하면 좋을 때?

- critical section의 길이가 길때!
- critical section의 길이가 매우 짧다면 block/ wakeup 오버헤드가 더 커질 수 있음
- 하지만 일반적으로는 Block/ Wakeup방식이 더 좋다!

## Monitor

### Semaphore의 문제점

- 코딩하기 힘들다
- 정확성 입증이 어렵다
- 자발적 협력이 필요하다
- 한번의 실수가 모든 시스템에 치명적 영향을 끼친다.

### Monitor 란?

- 동시 수행중인 프로세스 사이에서 abstract data type의 안전한 공유를 보장하기 위한, 
고수준의 synchronization construct
- 모니터로 프로세스들을 묶어놓고 한번에 하나의 객체(process)만 실행
    - 프로그래머가 semaphore를 명시적으로 코딩할 필요가 없음

```c
/*ex) Bounded-Buffer Problem*/

monitor bounded_buffer{
	int buffer[N];
	condition full,empty;

	void produce(int x){
		if there is no empty buffer
			empty.wait();                 // 다른 프로세스가 signal로 깨워주기 전에  suspend
		add x to an empty buffer
		full.signal();                  // suspend된 프로세스를 resume
	}
	
	void consume(int* x){
		if there is no full buffer
			full.wait();
		remove an item from buffer and store. it to *x
		empty.signal(); 
	}
}
```
