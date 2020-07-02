# 목표
- gc 기본구조, jvm 기본구조
- 매이저gc, 마이너gc 이해
- 버전별 gc 차이점

# 두서리
- 가비지 컬렉션이라는 개념은 자바에서 처음 사용된 것이 아니다. 
	- LISP 라는 언어에서 처음 도입된 개념이다. 
	- 하지만, 자바가 가비지 컬렉션이란 개념을 더욱 대중화 시킨데 기여한 부분은 있다.

- Heap 영역의 오브젝트 중 stack 에서 도달 불가능한 (Unreachable) 오브젝트들은 가비지 컬렉션의 대상이 된다.

- Garbage Collection 과정은 Mark and Sweep 이라고도 한다. 
	- JVM의 Garbage Collector 가 스택의 모든 변수를 스캔하면서 각각 어떤 오브젝트를 레퍼런스 하고 있는지 찾는과정이 Mark 다. 
	- Reachable 오브젝트가 레퍼런스하고 있는 오브젝트 또한 marking 한다. 
	- 첫번째 단계인 marking 작업을 위해 모든 스레드는 중단되는데 이를 stop the world 라고 부르기도 한다.
	- 그리고 나서 mark 되어있지 않은 모든 오브젝트들을 힙에서 제거하는 과정이 Sweep 이다.
	- Garbage Collection 이라고 하면 garbage 들을 수집할 것 같지만 
		- 실제로는 garbage 를 수집하여 제거하는 것이 아니라, 
		- garbage 가 아닌 것을 따로 mark 하고 
		- 그 외의 것은 모두 지우는 것이다. 
	- Mark: 변수가 어떤 객체를 레퍼런스 하고 있는지 찾는 과정
	- Sweep: mark 되어 있지 않는 모든 객체를 힙에서 제거하는 과정

- -XX:+PrintCommandLineFlag: java command 라인 명령어 출력
- -verbose:gc : 가비지 컬렉션이 일어날때 자동출력
- -XX:+UseParallelGC: ParallelGC 라는 가비지 컬렉터를 사용

- Java 9 부터는 Graal Visual VM 으로 바뀌었다고 한다.

- 먼저 metaspace 에 대해 알아보자. 
	- 자바 8 에 적용된 변화로 람다, 스트림, 인터페이스의 default 지시자가 대표적이다. 
	- 하지만, 메모리의 관점에서 가장 큰 변화로는 PermGen 이 사라지고 Metaspace 가 이를 대체하게 되었다는 것이다.
	- PermGen 은 자바 7까지 클래스의 메타데이터를 저장하던 영역이었고 Heap 의 일부였다.

```
Permanent Generation은 힙 메모리 영역중에 하나로 
자바 애플리케이션을 실행할때 클래스의 메타데이터를 저장하는 영역이다.(Java 7기준)

아래와 같은 것들이 Java Heap 이나 native heap 영역으로 이동했다.

Symbols -> native heap
Interned String -> Java Heap
Class statics -> Java Heap

OutOfMemoryError: 
	PermGen Space error 는 더이상 볼 수 없고 
	JVM 옵션으로 사용했던 PermSize 와 MaxPermSize는 더이상 사용할 필요가 없다. 
	이 대신에 MetaspaceSize 및 MaxMetaspaceSize 가 새롭게 사용되게 되었다.
```

- 에덴 영역, 서바이버0, 서바이버1, Old Gen, Metaspace
--- Old, Eden, S0, S1

```
JVM:
  heap: 
    Young Generation: 
      Eden: 새로운 객체, 가득차면  Minor GC 발생
      Survivor Space 0: Minor GC 발생 시 살아남은 객체
      Survivor Space 1: S0, S1으로 서로 이동하면서 age 증가
    Old Generation: 특정 age 이상인 객체, 가득차면 Major GC 발생
  Metaspace: 
```

# JVM GC 과정
1. 신규 객체는 에덴영역에 할당. S0, S1은 비워진 상태로 시작
2. 에덴이 가득차면, 마이너GC 발생
3. 마이너 발생 시, 에덴영역에서 Unreachable 객체는 삭제, Reachable 객체는 S0 이동
4. 다음 마이너 발생 시, 에덴영역에서 3번 과정 반복
  이 때 기존 S0에 살아남은 Reachable 객체는 S1 으로 이동하며 age 증가
  살아남은 모든 객체가 S1으로 이동
  그리고 에덴과 S0가 클리어
5. 다음 마이너 발생 시, 4번 과정이 반복
  이 때 S1이 가득차 있었으므로 S1에서 살아남은 객체는 S0 로 이동
  그리고 에덴과 S1이 클리어
6. Young Gen 에서 계속 살아남으며 age 값이 증가하는 객체들은 
  age 값이 특정값 이상이 되면 Old Gen 으로 옮겨짐, 이 단계를 Promotion
7. 마이너가 계속 반복되면, Promotion 작업도 반벅됨
8. Promotion 작업이 반복되면서 Old Gen 이 가득차면 Major GC 발생
9. 요약
  - 에덴이 가득차면 Minor GC 발생, S0, S1 으로 이동함
  - 살아남으면서 S0, S1 에서 서로 할때 age 증가
  - Promiton: 특정 age 이상이면 Old Gen 으로 이동
  - Old Gen 이 가득차면 Major GC 발생
  - Full GC: Heap 전체를 클리어, Young/Old 모두

# JVM GC 종류 간략히
1. Serial GC
	- -XX:+UseSerialGC 옵션을 줘서 사용할 수 있는 Serial GC 는 Java SE 5, 6 에서 사용되는 디폴트 가비지 컬렉터이다.
	- MinorGC, MajorGC 모두 순차적으로 시행된다.
	- Mark-Compact collection method 를 사용한다.
	- Mark-Compact collection method 란, 새로운 메모리 할당을 빠르게 하기 위해서 기존의 메모리에 있던 오브젝트들을 힙의 시작위치로 옮겨 놓는 방법이다. 창고에서 필요없는 물건들을 버린 후, 창고에 물건을 차곡차곡 쌓기위해 창고안을 정리하는 것이라 생각할 수 있다.
	
2. Parallel GC
	- -XX:+UseParallelGC 옵션으로 사용 가능한 Parallel 가비지 컬렉터는 young generation 에 대한 가비지 컬렉션 수행시 멀티스레드를 사용한다. 멀티스레딩을 할 수 있는 ParallelGC 를 사용하도록 옵션을 주었더라도, 호스트 머신이 싱글 CPU 라면 디폴트 가비지 컬렉터(Serial GC)가 사용된다. 하지만, 호스트의 CPU 가 두개 이상이라면 young generation 의 가비지 컬렉션 시간을 줄일 수 있다.
	- 또한, -XX:+UseParallelOldGC 옵션을 사용한다면, old generation 의 가비지 컬렉션에서도 멀티스레딩을 활용할 수 있다.

3. Concurrent Mark Sweep (CMS) Collector
	- Concurrent Low Pause Collector 라고도 불리는 CMS 컬렉터는 -XX:+UseConcMarkSweepGC 옵션으로 사용할 수 있다. 대부분의 가비지 컬렉션 작업을 애플리케이션 스레드와 동시에 수행함으로써 가비지 컬렉션으로 인한 stop-the-world 시간을 최소화하는 GC이다.
	- CMS 컬렉터는 young generation 에 대한 가비지 컬렉션시 Parallel GC 와 같은 알고리즘을 사용하는데, -XX:ParallelCMSThreads=<N> 옵션으로 스레드 개수를 설정할 수 있다.

4. G1 Garbage Collector
	- Garbage First 라는 의미의 G1 가비지 컬렉터는 Java 7 부터 사용가능

# 또다시 두서리
- stop-the-world란, GC을 실행하기 위해 JVM이 애플리케이션 실행을 멈추는 것이다. stop-the-world가 발생하면 GC를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춘다.

- 















