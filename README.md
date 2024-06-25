- [JVM](#jvm)
	- [Cashe](#cashe)
- [Main](#main)
- [String](#string)
	- [formatted](#formatted)
	- [join](#join)
- [BigInteger](#biginteger)
- [BigDecimal](#bigdecimal)
- [Collection](#collection)
	- [List](#list)
	- [Set](#set)
	- [Map](#map)
- [반복문](#loop)
- [Iterator](#iterator)
- [Class](#class)
	- [final](#final)
	- [Singleton](#singleton)
- [Comparable Comparator](#comparable-comparator)
- [Thread](#thread)
	- [Thread Runnable](#thread-runnable)
	- [Thread Callable](#thread-callable)
	- [Thread Future](#thread-future)
	- [Thread Methods](#thread-methods)
		- [isAlive()](#thread-isalive)
		- [interrupt()](#thread-interrupt)
		- [join(\[int?\]) - 끝날때까지 대기](#thread-join)
		- [sleep(\[int\]) - 지연시간](#thread-sleep)
		- [쓰레드 이름](#thread-name)
	- [Thread Group](#thread-group)
	- [Daemon Thread - Thread 내 Thread 연결](#thread-daemon)
	- [Synchronized - Data안전성을 위한 동기화](#thread-synchronized)
	- [volatile](#thread-volatile)
	- [Thread Object Methods - wait(), notifyAll()](#thread-object-methods)
	- [ThreadPool Executors ExecutorService - 동시 쓰레드 개수제한](#threadpool-executors-executorservice)
- [Serialization](#serialization)
- [Reflection](#reflection)
- [Annotation](#annotation)
	- [Meta Annotation](#meta-annotation)
<br/><br/>
- [Java 7+](#java-7)
	- [AutoCloseable](#autocloseable)
- [Java 8+](#java-8)
	- [함수형인터페이스, lambda](#functionalinterface-lambda)
	- [메소드 참조](#method-reference)
	- [Optional](#optional)
	- [Stream](#stream)
		- [직렬 > 병렬 parallel()](#stream-parallel)
		- [병렬 > 직렬 sequential()](#stream-sequential)
		- [스트림 중간 연산](#stream-middle-methods)
			- [boxed](#stream-boxed)
			- [distinct](#stream-distinct)
			- [sorted](#stream-sorted)
			- [skip](#stream-skip)
			- [limit](#stream-limit)
			- [peek](#stream-peek)
			- [map](#stream-map)
			- [filter takeWhile dropWhile](#stream-filter-takewhile-dropwhile)
		- [스트림 최종 연산](#stream-final-methods)
			- [forEach](#stream-foreach)
			- [count min max sum](#stream-count-min-max-sum)
			- [reduce](#stream-reduce)
	- [Thread CompletableFuture](#thread-completablefuture)
		- [supplyAsync(()->{}](#completablefuture-supplyasync)
		- [thenApply(\[data\]->{}](#completablefuture-thenapply)
		- [exceptionally(\[error\]->{}](#completablefuture-exceptionally)
		- [thenCompose(\[CF Data1\]->\[CF객체\].\[thenMethod\](\[CF Data2\]->{})](#completablefuture-thencompose)
		- [thenCombine(\[CF객체\], (\[CF Data1\], \[CF Data2\])->{}](#completablefuture-thencombine)
		- [allOf(\[CF객체\], \[CF객체\], ...)](#completablefuture-allof)
		- [anyOf(\[CF객체\], \[CF객체\], ...)](#completablefuture-anyof)
- [Java 13+](#java-13)
	- [\[String객체\].formatted](#formatted-13)
- [Java 16+](#java-16)
	- [record](#record-16)


## JVM
### Cashe
- 값이 사용되지 않으면 값이 변경되더라도 update하지 않음
```java
Static boolean stop = false;
public static void main(String[] args) {
	new Thread(() -> {
		int i = 0;
		while (!stop) {	//stop 이 true로 변했으나 while문에서 false인 상태 그대로 유지 > 무한반복
			i++;
			System.out.println(i);
		}
		System.out.println("- - - 쓰레드 종료 - - -");
	}).start();
	try {
		Thread.sleep(1000);
	} catch (InterruptedException e) {
	}
	stop = true;
}
```
- 해결방법 1 : volatile
- [참고 - volatile](#thread-volatile)
```java
volatile static boolean stop = false;
public static void main(String[] args) {
	new Thread(() -> {
		int i = 0;
		while (!stop) {	//stop 이 true로 변했으나 while문에서 false인 상태 그대로 유지 > 무한반복
			i++;
			System.out.println(i);
		}
		System.out.println("- - - 쓰레드 종료 - - -");
	}).start();
	try {
		Thread.sleep(1000);
	} catch (InterruptedException e) {
	}
	stop = true;
}
```
- [해결방법 2 : 동기화된 메소드로 stop 접근](#thread-synchronized)
- [참고 - Thread synchronized](#thread-synchronized)
```java
static boolean stop = false;
synchronized public static boolean isStop() {
	return stop;
}
synchronized public static void setStop(boolean b) {
	stop = b;
}
public static void main(String[] args) {
	new Thread(() -> {
		int i = 0;
		while (!isStop()) {
			i++;
		}
		System.out.println("- - - 쓰레드 종료 - - -");
	}).start();
	try {
		Thread.sleep(1000);
	} catch (InterruptedException e) {}
	setStop(true);
}
```

## Main
- public static void main(String[] args){}
- `javac [Java파일명].java` 실행 시 .class 생성
- `java [Java파일명].java [String1] [String2] [String3] [String4] ...` - args 에 String1234 할당됨

## String
### formatted
- [Java 13+ formatted](#formatted-13)
- `formatted([변수1],[변수2]...)` - 문자열 내에 변수를 가독성좋게 할당 가능
	- %b / %d / %f / %c / %s : boolean / 10진수 / 실수 / 문자 / 문자열
	- %n : 개행 (초기 문자열내에서 \n 대신 사용)
```java
String str1 = "%s의 둘레는%n반지름 X %d X %f입니다.";
String circle = "원";
int two = 2;
double PI = 3.14;

String str3 = String.format(str1, circle, two, PI);	// 원의 둘레는\n반지름 X 2 X 3.13입니다.
```

### join
- `String.join([삽입변수], [string배열])` - string배열의 각 string 사이에 삽입변수를 삽입해 문자열 합침
```java
String[] strings = {"123", "456", "789", "012"};
String join1 = String.join(", ", strings);	// 123, 456, 789, 012
String join2 = String.join("-", strings);		// 123-456-789-012
```

## BigInteger
- java.math.BigInteger
- long 범위 이상의 정수를 다룰 수 있음
## BigDecimal
- java.math.BigDecimal
- double의 부동소수점 오차를 해결한 실수 클래스
```java
import java.math.BigInteger;
import java.math.BigDecimal;

BigInteger bigInt1 = new BigInteger("123456789012345678901234567890");
BigInteger bigInt2 = new BigInteger("987654321098765432109876543210");
BigInteger bigInt3 = bigInt1.add(bigInt2);		// 1111111110111111111011111111100
BigInteger bigInt4 = bigInt2.subtract(bigInt1);		// 864197532086419753208641975320
BigInteger bigInt5 = bigInt1.multiply(bigInt2);		// 121932631137021795226185032733622923332237463801111263526900
BigInteger bigInt6 = bigInt2.divide(bigInt1);		// 8

int bigIntCompare1 = bigInt1.compareTo(bigInt2);	// -1
int bigIntCompare2 = bigInt2.compareTo(bigInt1);	// 1


double num1 = 0.2 + 0.3f;				// 0.5000000119209289
double num2 = 0.3f * 0.7f;				// 0.21000000834465027
double num3 = 0.4 - 0.3;				// 0.10000000000000003
double num4 = 0.9f / 0.3;				// 2.9999999205271406
double num5 = 0.9 % 0.6;				// 0.30000000000000004
float num6 = new BigDecimal("0.2")
	.add(new BigDecimal("0.3"))			// 0.5
	.floatValue();
float num7 = new BigDecimal("0.3")
	.multiply(new BigDecimal("0.7"))		// 0.21
	.floatValue();
float num8 = new BigDecimal("0.4")
	.subtract(new BigDecimal("0.3"))		// 0.1
	.floatValue();
double num9 = new BigDecimal("0.9")
	.divide(new BigDecimal("0.3"))		// 3.0
	.doubleValue();
double num10 = new BigDecimal("0.9")
	.remainder(new BigDecimal("0.6"))		// 0.3
	.doubleValue();
```

## Collection
### List
- ArrayList	: 요소들을 들어오는 순서대로 저장
- LinkedList	: Queue를 구현하는 용도로 사용 가능

### Set
- HashSet	: 성능빠름, 순서X
- LinkedHashSet	: 입력순서대로 정렬, 비교적 성능약함
- TreeSet	: 특정기준대로 정렬, 추가/삭제 시간소모

### Map 
- HashMap	: 키를 해시로 저장 (정렬필요 없이 빠른접근 시 사용)
- TreeMap	: 키를 트리형태로 저장 (정렬필요시 사용)

## Loop
- foreach
```java
String[] strings = {"123", "456", "789", "012"};
for (String s : strings) {
	System.out.println(s);	// 123 456 789 012 출력
}
```

## Iterator
- `Iterator<[class명]>` - collection에 저장되어있는 요소들을 순회하는 인터페이스
- 순회상태가 저장될 필요가 있을때 유용
```java
import java.util.Set;
import java.util.HashSet;
import java.util.Map;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Set;

Set<Integer> intHSet = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9));
Iterator<Integer> iterSet = intHSet.iterator();
while (iterSet.hasNext()) {
	if (iterSet.next() % 3 == 0) {	// iter위치 다음으로 이동 후 이동된 위치의 값이 3의배수면
		iterSet.remove();	// 이동된 현재위치 삭제
	}
}
Map<Integer, Double> hashMap = new HashMap<>();

// 아래 Iterator는 원본 hashMap에 영향을 끼치지 않음
// iterKey.remove(), iterValue.remove(), iterMap.remove() 실행해도 hashMap 변동없음
// .keySet(), .values() 를 통해 새로운 객체를 반환했기 때문
Iterator<Integer> iterKey = hashMap.keySet().iterator();		// 키 순회
Iterator<IDouble> iterValue = hashMap.values().iterator();	// 값 순회
Iterator<Map.Entry<Integer, Double>> iterMap = hashMap.entrySet().iterator();	// 키, 값 모두 순회
```

## Class
### final
- field : 값변경불가, 선언과 동시에 초기화
- method : 자식 클래스에서 override 불가
- instance : 다른 객체로 변경 불가, 이미 할당된 객체 내 field는 변경가능
- class : 하위확장 불가(자식클래스 만들수 없음)

### Singleton
- Singleton : 하나의 클래스의 하나의 공유 인스턴스를 만드는 패턴

## Comparable Comparator
- `public class [생성할class명] implements Comparable<[비교기준class명]> {}` - 자신, 다른객체 비교(기본자료형)
- `public class [생성할class명] implements Comparator<[비교기준class명]> {}` - 주어진 두 객체를 비교(Collection)
```java
import java.util.Comparator;

public class ComparableClass implements Comparable<ComparableClass> {
	int a;
	@Override
	public int compareTo(ComparableClass o) {
		return o.a - this.a;
	}
}
public class ComparatorClass implements Comparator<ComparableClass> {
	int a;
	@Override
	public int compare(ComparatorClass o1, ComparatorClass o2) {
		return o2.a - o1.a;
	}
}
```

## Thread
- `extends Thread`
- Thread-safe class 알아보기 (ex) Concurrent, CopyOnWriteArrayList, CopyOnWriteArraySet, Atomic)
### Thread Runnable
- `implements Runnable`
- `[Thread 객체].start()` - 새로운 Thread에서 동작 (run()은 호출한 쓰레드에서 동작함)

### Thread Callable
- `implements Callable<T>` - [Runnable](#thread-runnable) 과 달리 반환값이 있음
- `[Callable 객체].call()` - 새로운 Thread에서 동작 후 값 반환
```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.stream.IntStream;

class CallableTest implements Callable<Integer>{
	int i;
	CallableTest(int i) { this.i = i; }
	@Override
	public Integer call() throws Exception{
		Thread.sleep(100);	// 0.1초 멈춤
		System.out.println(i);
		return i;
	}
}
List<Integer> list = new ArrayList<>();
IntStream.range(0,10).forEach(i -> {
	try {
		list.add(new CallableTest(i).call());
	} catch (Exception e) {
		throw new RuntimeException(e);
	}
});
System.out.println(String.join(",", list.stream().map(String::valueOf).toArray(String[]::new)));
// 0,1,2,3,4,5,6,7,8,9 출력
```
- [참고 - Stream](#stream)
- [참고 - 메소드 참조](#method-reference)
- [참고 - String.join](#join)

### Thread Future
- [참고 - CompletableFuture](#thread-completablefuture)
- `Future<T>`
	- isDone() - future thread 작업이 종료되었으면 true 반환
	- get() - 호출 시 비동기 작업중인 Future Thread 종료 후 이후 코드 동작
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

ExecutorService es = Executors.newSingleThreadExecutor();
Future<String> futureTest = es.submit(() -> {
	System.out.println("future 시작");
	Thread.sleep(1000);	// 1초 멈춤
	return "future 완료";
});
while (!futureTest.isDone()) {
	System.out.println("future 작업중");
	try {
		Thread.sleep(100);
	} catch (InterruptedException e) {}
}
String result = null;
result = futureTest.get();	// futureTest 작업이 끝날때까지 아래코드 실행X
System.out.println(result + " / 2작업 시작");	// future 완료 / 2작업 시작
System.out.println("2작업 끝");
es.shutdown();
```
```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.stream.IntStream;

class CallableTest implements Callable<Integer>{
	int i;
	CallableTest(int i) {
		this.i = i;
	}
	@Override
	public Integer call() throws Exception{
		Thread.sleep(100);	// 0.1초 멈춤
		System.out.println(i);
		return i;
	}
}

ExecutorService es = Executors.newFixedThreadPool(4);
List<Future<Integer>> futureList = new ArrayList<>();
IntStream.range(0, 10).forEach(i -> {
	futureList.add(es.submit(new CallableTest(i)));
	// future는 순차적으로 list에 담기지만 CallableTest의 print는 ThreadPool에 의해 순서가 변할 수 있음
});
es.shutdown();	// 호출 안하면 es 에 작업이 남아있다 판단해서 프로그램 종료X

ArrayList<Integer> intList = new ArrayList<>();
for (Future<Integer> future : futureList) {
	intList.add(future.get());	// 현재 future 작업 끝날때까지 다음 순서 대기
}
System.out.println(String.join(",", intList.stream().map(String::valueOf).toArray(String[]::new)));
// 0,1,2,3,4,5,6,7,8,9 출력
```

### Thread Methods
#### Thread isAlive
- `[Thread 객체].isAlive()` - 해당 Thread가 진행중이면 true, 종료되었으면 false 반환

#### Thread interrupt
- `[Thread 객체].interrupt()` - 쓰레드에서 InterruptedException 호출

#### Thread join
- `[Thread 객체].join([int?])` - 해당 Thread가 끝날때까지 뒷작업을 수행하지 않음
	- int 는 옵션으로 있는경우 쓰레드가 끝나지 않았어도 int ms 시간이 지나면 이후코드 실행
	- InterruptedException 일어날 수 있어서 try catch문 필요

#### Thread sleep
- `Thread.sleep([int])` - 쓰레드를 int ms 만큼 멈춤
	- InterruptedException 일어날 수 있어서 try catch문 필요
	- 멈추고싶은 쓰레드 안해서 호출해야 해당 쓰레드가 멈춤(외부에서 호출시 의도치않은 버그발생)

#### Thread Name
- `[Thread 객체].setName([이름])` - Thread 객체의 이름 설정
- `Thread.currentThread().getName()` - 현재 쓰레드의 이름 반환
```java
public class MakeThread extends Thread{
	@Override
	public void run(){
		for(int i=0;i<2;i++){
			try{
				Thread.sleep(500);		// 0.5초 멈춤
			} catch (InterruptedException e){
				System.out.println("interrupt()");
				return;	// 쓰레드 종료
			}
			System.out.println(Thread.currentThread().getName() + " : " + "1");
		}
	}
}
public class MakeRunnable implements Runnable{
	@Override
	public void run(){
		for(int i=0;i<4;i++){
			try{
				Thread.sleep(500);		// 0.5초 멈춤
			} catch (InterruptedException e){
				throw new RuntimeException(e);
			}
			System.out.println(Thread.currentThread().getName() + " : " + "2");
		}
	}
}
Thread thread1 = new MakeThread();
Thread thread2 = new Thread(new MakeRunnable());
Thread thread3 = new Thread(new Runnable(){
	@Override
	public void run(){
		for(int i=0;i<6;i++){
			try {
				Thread.sleep(200);	// 0.2초 멈춤
			} catch (InterruptedException e) {
				e.printStackTrace();
			}		
			System.out.println(Thread.currentThread().getName() + " : " + "3");
		}
	}
});
thread1.setName("Thread1");
thread2.setName("Thread2");
thread3.setName("Thread3");

// 한 쓰레드 안에서 순차적으로 출력
thread1.run();
thread2.run();
thread3.run();
//main : 1 main : 1 main : 2 main : 2....main : 3 main : 3... 출력

// 각각의 쓰레드에서 실행
thread1.start();
thread2.start();
thread3.start();

System.out.println(thread3.isAlive());	// true
try {
	thread3.join(500);	// thread3이 끝날때까지 혹은 0.5초동안 다른 코드 동작안함(다른 쓰레드도 동작안함)
} catch (InterruptedException e) {
	e.printStackTrace();
}			
System.out.println(thread3.isAlive());	// true (0.5초 동안 thread3 안끝남)
//true Thread3 : 3 Thread3 : 3 true 출력

for(int i=0;i<10;i++){
	System.out.println(i);
}
thread1.interrupt();	// interrupt() 출력 후 thread1 종료
// thread2, thread3 동작중
```

### Thread Group
- `[Thread 객체].getThreadGroup()` - 연관된 쓰레드의 그룹
	- 쓰레드를 일괄적으로 다루거나 보안상 분리하기 위해 사용
	- 객체가 포함된 ThreadGroup 반환
	- mainThrGroup에 기본적으로 할당
	- ThreadGroup 내에 다른 ThreadGroup 포함 가능
	- `[Thread 객체].getThreadGroup()` : Thread 객체가 속한 ThreadGroup 반환
	- `[ThreadGroup 객체].getParent()` : ThreadGroup객체의 부모 ThreadGroup 반환
	- `[ThreadGroup 객체].activeCount()` : ThreadGroup 내 작동중인 쓰레드 개수 반환
	- `[ThreadGroup 객체].activeGroupCount()` : ThreadGroup 내 작동중인 Thread 개수 반환
	- `[ThreadGroup 객체].activeGroupCount()` : ThreadGroup 내 작동중인 ThreadGroup 개수 반환
```java
ThreadGroup threadGroup = thread1.getThreadGroup();	// mainThrGroup 반환
ThreadGroup newThreadGroup = new ThreadGroup("TG1");
Thread testThread = new Thread(newThreadGroup, ()->{});
System.out.println(testThread.getThreadGroup().getName());	// TG1 출력

ThreadGroup newThreadGroup2 = new ThreadGroup(newThreadGroup, "TG2");
newThreadGroup2.getParent().getName();	// TG1 출력

```

### Thread Daemon
- `[Thread 내 Thread 객체].setDaemon(true)` - 다른 Thread의 작업을 보조하는 역할
	- 주 Thread가 종료되면 자동종료
	- Thread 내 Thread 객체의 start()가 호출되기 전에 setDaemon을 선언해야함
```java
Thread mainThread = new Thread(new Runnable(){
	@Override
	public void run(){
		Thread daemonThread = new Thread(new Runnable(){
			@Override
			public void run(){
				for(int i=0;i<6;i++){
					try {
						Thread.sleep(500);	// 0.5초 멈춤
					} catch (InterruptedException e) {
						e.printStackTrace();
					}		
					System.out.println(Thread.currentThread().getName());
				}
			}
		});
		daemonThread.setName("DaemonThread");
		daemonThread.setDaemon(true);	// mainThread가 종료되면 daemonThread도 종료됨
		daemonThread.start();
		
		for(int i=0;i<6;i++){
			try {
				Thread.sleep(200);	// 0.2초 멈춤
			} catch (InterruptedException e) {
				e.printStackTrace();
			}		
			System.out.println(Thread.currentThread().getName());
		}
	}
});
mainThread.setName("MainThread");
mainThread.start();
```

### Thread Synchronized
- `synchronized [function](){}` - 여러 쓰레드가 동시에 접근하는 것을 방지
	- 해당 클래스 내에 다른 메소드에도 synchronized 가 선언되었다면 다른 메소드라도 동시 접근 불가
- `synchronized(this){[동작]}` - 메소드 내 특정 작업만 동기화가 필요할 때
```java
class SynchroTest {
	public static int remain = 100;
	// synchronized 없으면 remain이 음수가 되기도함
	synchronized public void synchroFunction(int minus, String s){	// synchronized 함수
		System.out.println(s + "synchroFunction 시작");
		if(remain-minus<0){
			System.out.println(s + "synchroFunction 불가 /재고 : " + remain);
			return;
		}
		try {
			Thread.sleep(500);	// 0.5초 멈춤
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		remain -= minus;
		System.out.println(s + "synchroFunction 완료 / 재고 : " + remain);
	}
	public void nomalFunction(int minus, String s){
		System.out.println(s + "nomalFunction 시작");
		// synchronized 없으면 remain이 음수가 되기도함
		synchronized(this){				// synchronized 작업
			if(remain-minus<0){
				System.out.println(s + "nomalFunction 불가 /재고 : " + remain);
				return;
			}
		}
		remain -= minus;
		System.out.println(s + "nomalFunction 완료 / 재고 : " + remain);
	}
}

SynchroTest st = new SynchroTest();
Thread thread1 = new Thread(new Runnable(){
	@Override
	public void run(){
		for(int i=0;i<6;i++){
			st.nomalFunction(10, Thread.currentThread().getName() + " / ");
			st.synchroFunction(10, Thread.currentThread().getName() + " / ");
			try {
				Thread.sleep(100);	// 0.1초 멈춤
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
});
Thread thread2 = new Thread(new Runnable(){
	@Override
	public void run(){
		for(int i=0;i<6;i++){
			st.nomalFunction(20, Thread.currentThread().getName() + " / ");
			st.synchroFunction(20, Thread.currentThread().getName() + " / ");
			try {
				Thread.sleep(300);	// 0.3초 멈춤
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
});
Thread thread3 = new Thread(new Runnable(){
	@Override
	public void run(){
		for(int i=0;i<6;i++){
			st.nomalFunction(30, Thread.currentThread().getName() + " / ");
			st.synchroFunction(30, Thread.currentThread().getName() + " / ");
			try {
				Thread.sleep(500);	// 0.5초 멈춤
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
});
thread1.setName("Thread1");
thread2.setName("Thread2");
thread3.setName("Thread3");

thread1.start();
thread2.start();
thread3.start();
```

### Thread volatile
- `volatile [변수]` - 변수의 값이 변경될 때마다 메모리에 update
	- 멀티쓰레드에서 [캐싱에 의한 문제](#cashe) 방지
	- [동기화](#thread-synchronized)와는 다름

### Thread Object Methods
- `wait()` - [동기화](#thread-synchronized) 메소드 사용 중 자기 일을 멈춤 (다른 쓰레드가 사용할 수 있도록 양보)
- `notify()` - 일을 멈춘 상태의 쓰레드에게 자리가 비었음을 통보 (대기열의 쓰레드 중 1개의 쓰레드에만 통보)
- `notifyAll()` - 대기중인 모든 쓰레드에게 자리가 비었음을 통보
```java
import java.util.Arrays;

class ObjectMethodsTest {
	synchronized public void synchroFunction(String s){	// synchronized 함수
		System.out.println(s + "사용중");
		try {
			Thread.sleep(500);	// 0.5초 멈춤
		} catch (InterruptedException e) {
		e.printStackTrace();
		}
		System.out.println(s + "사용완료");

		// 아래 코드가 없으면 1사용중 1사용완료 무한반복
		notifyAll();	// 대기중인 모든 쓰레드에게 자리가 비었음을 통보
		try {
			wait();	// 동작 멈춤
		} catch (InterruptedException e) {
		e.printStackTrace();
		}
	}
}
ObjectMethodsTest omt = new ObjectMethodsTest();
Arrays.stream("1,2,3,4".split(","))
	.forEach(s -> new Thread(new Runnable() {
		@Override
		public void run(){
			while(true) {
				omt.synchroFunction(s);
			}
		}
	})
	.start());
```
- [참고 - String.formatted](#formatted)
- [참고 - Stream foreach](#stream-foreach)

### ThreadPool Executors ExecutorService
- `Executors.newFixedThreadPool([int])` - 동시에 돌아가는 Thread 개수 int 개	ExecutorService 반환
- `Executors.newSingleThreadExecutor()` - 동시에 돌아가는 Thread 개수 1개	ExecutorService 반환
- `[ExecutorService 객체].execute([Runnable 객체])` - [Runnable](#thread-runnable) 인자, return void
- `[ExecutorService 객체].submit([Callable | Runnable 객체])` - [Callable](#thread-callable) [Runnable](#thread-runnable) 인자, return [Future](#thread-future)
```java
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class ThreadPoolTest implements Runnable {
	public static int remain = 100;
	public int n;
	ThreadPoolTest(int n){
		this.n = n;
	}
	@Override
	public void run(){
		System.out.println(Thread.currentThread().getName() + "/" + n + "번째 동작중 / remain : " + ThreadPoolTest.remain);
		try {
			Thread.sleep(1000);	// 0.3초 멈춤
		} catch (InterruptedException e) {
			System.out.println(Thread.currentThread().getName() + "/" + n + "번째 동작중단 / remain : " + ThreadPoolTest.remain);
			return;	// shutdown(), shutdownNow() 시 중지하려면 반드시 return 있어야함
		}
		ThreadPoolTest.remain -= 10;
		System.out.println(Thread.currentThread().getName() + "/" + n + "번째 동작완료 / remain : " + ThreadPoolTest.remain);
	}
}

ExecutorService es = Executors.newFixedThreadPool(5);
int i=1;
while(ThreadPoolTest.remain>0){
	System.out.println(i + "번째 투입");
	es.execute(new ThreadPoolTest(i));
	try {
		Thread.sleep(100);	// 0.1초 멈춤
	} catch (InterruptedException e) {return;}
	i++;
}
//es.shutdown();					// 동작중인 쓰레드는 끝날때까지 동작함
List<Runnable> stopThreads = es.shutdownNow();	// 동작중인 Thread 중단 후 중단된 Thread들 반환
System.out.println(stopThreads);
```

## Serialization
- `[class 명] implements Serializable` - 인스턴스를 ByteStream으로 변환해 다른 곳에 보내거나 파일 등으로 저장
	- `private static final long serialVersionUID` - 해당 class 의 버전 번호로 사용되는 변수
		<br/>직렬화한 인스턴스를 역직렬화할때 버전 정보가 다르면 error 발생
	- `transient [접근제한자] [변수명]` - 직렬화할 때 제외됨
- `ObjectOutputStream` - 직렬화 가능한 인스턴스를 Stream으로 출력
- serialization된 데이터파일은 `.ser` 확장자를 가짐
```java
import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

public class SerializableClass implements Serializable{
	private static final long serialVersionUID = 1L;
	private String s;
	private int i;
	public SerializableClass(String s, int i){
		this.s = s; this.i = i;
	}
	@Override
	public String toString() {
		return super.toString() + String.format(" - %s / int : %d", s,i);
	}
}

SerializableClass sc1 = new SerializableClass("instance1", 1);
SerializableClass sc2 = new SerializableClass("instance2", 2);
List<SerializableClass> scList = new ArrayList<>();
scList.add(sc1); scList.add(sc2);
try (	// try with resource
	FileOutputStream fos = new FileOutputStream("src/testSerialization.ser");
	BufferedOutputStream bos = new BufferedOutputStream(fos);
	ObjectOutputStream oos = new ObjectOutputStream(bos);
) {
	oos.writeObject(sc1);
	oos.writeObject(sc2);
	oos.writeObject(scList);
} catch (Exception e) {
	throw new RuntimeException(e);
}
```
```java
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.ObjectInputStream;
import java.util.ArrayList;
import java.util.List;

SerializableClass newSc1;
SerializableClass newSc2;
List<SerializableClass> newScList = new ArrayList<>();
try (
	FileInputStream fis = new FileInputStream("src/testSerialization.ser");
	BufferedInputStream bis = new BufferedInputStream(fis);
	ObjectInputStream ois = new ObjectInputStream(bis);
) {	//  직렬화할 때와 순서가 동일하지 않으면 에러 발생
	newSc1 = (SerializableClass) ois.readObject();
	newSc2 = (SerializableClass) ois.readObject();
	newScList = (ArrayList) ois.readObject();
} catch (Exception e) {
	throw new RuntimeException(e);
}
System.out.println(newSc1);
System.out.println(newSc2);
System.out.println(newScList);
```

# Reflection
- 런타임에서 Class, Interface, Methods, Fields 등을 분석 및 조작하는 것
- 거의 쓰이지 않고 프레임워크나 라이브러리 만들때 사용
```java
import java.io.Serializable;

public class SerializableClass implements Serializable{
	private static final long serialVersionUID = 1L;
	private String s;
	private int i;
	public SerializableClass(String s, int i){
		this.s = s; this.i = i;
	}
	@Override
	public String toString() {
		return super.toString() + String.format(" - %s / int : %d", s,i);
	}
}
```
```java
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;

public static String objToString (Object o) {
	Class<?> objClass = o.getClass();
	StringBuilder sb = new StringBuilder();
	sb.append(String.format("ClassName : %s\n", objClass.getSimpleName()));	// Class<?>.getSimpleName() : ?class명 반환
	for (Field f : objClass.getDeclaredFields()) {				// Class<?>.getDeclaredFields() : ?class 의 모든 필드 반환
		if (Modifier.isStatic(f.getModifiers())){				// Modifier.isStatic() : 필드가 static 필드이면 true
			continue;
		}
		f.setAccessible(true);					// private 필드인 경우 접근시 에러발생할 수 있어 접근가능하게 설정
		try {							// 필드명, 필드자료형, 필드값
			sb.append(String.format(" - %s (%s) : %s%n", f.getName(), f.getType().getSimpleName(), f.get(o)));
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}
	return sb.toString();
}
SerializableClass sc = new SerializableClass("instance1" 1);
System.out.println(objToString(sc));
//ClassName : SerializableClass
// - s (String) : instance1
// - i (int) : 1
```

# Annotation
- 컴퓨터가 보는 주석
- `@Override` - 오버라이드
- `@Deprecated` - 더이상 사용되지 않는다는것을 알려줌
- `@FunctionalInterface` - 함수형 인터페이스 임을 알려줌
- `@SuppressWarnings` - 컴파일러의 경고를 무시함
- `[접근제한자] @interface [Annotation 명]{ [값] (); [값] (); ...}` - 사용자가 직접 만든 Annotation
	- Annotation 설정 시 값을 넣을 수 있음
	- 값이 하나일 경우 value 로 변수명 설정시 값만 입력가능
	- default 를 통해 기본값 설정 가능
```java
public @interface OneValueTest {
	int value() default 1;
}

public @interface ManyValueTest {
	String s1() default "s1";
	String s2() default "s2";
	OneValueTest annotationField();	// Annotation도 필드로 가질 수 있음
}

public @interface ArrayValueTest {
	String[] strings();
}
```
```java
@OneValueTest			// value = 1 (default)
int a;

@OneValueTest(value = 3)		// value = 3
int b;

@OneValueTest(5)			// value = 5
int c;

@ManyValueTest(annotationField = @OneValueTest(5), s1 = "s1입니다")	// annotationField = @OneValueTest(5), s1 = "s1입니다", s2 = "s2"
String s;

@ArrayValueTest(strings={})		// 빈배열
String arrayString;

@ArrayValueTest(strings={"1", "2", "3"})	// {1,2,3}
String arrayString2;

@ArrayValueTest(strings="1")		// {1} - 하나만 있는 경우 괄호 생략가능
String arrayString3;
```

## Meta Annotation
- 메타어노테이션 - 사용자가 만든 Annotation의 속성을 정의하는 Annotation
- `@Retention(RetentionPolicy.[범위])` - Annotation이 유지되는 범위를 지정
	- SOURCE : 소스파일에만 적용, 컴파일 후 사라짐
	- CLASS :  클래스 파일까지 사용됨, 실행시불가, 컴파일 후 .class 파일 까지 유지
	- RUNTIME : 실행 시 사용가능, [Reflection](#reflection) 사용 시 RUNTIME 으로 설정해야함
```java
@Retention(RetentionPolicy.SOURCE)
public @interface RetentionSource {}

@Retention(RetentionPolicy.CLASS)
public @interface RetentionClass {}

@Retention(RetentionPolicy.RUNTIME)
public @interface RetentionRuntime {}
```
```java
@RetentionSource
int valueSource;	// 소스에서만 적용

@RetentionClass
int valueClass;	// 컴파일 후 .class 에 남음

@RetentionRuntime
int valueRuntime;	// 코드를 실행 할때도 남음
```
- `@Target({ElementType.[타입], ElementType.[타입], ...})` - Annotation이 적용될 수 있는 대상 지정
	- ANNOTATION_TYPE : 다른 어노테이션
	- CONSTRUCTOR : 생성자
	- FIELD : 필드
	- LOCAL_VARIABLE : 지역 변수
	- METHOD : 메소드
	- PACKAGE : 패키지
	- PARAMETER : 매개변수
	- TYPE : 클래스, 인터페이스, enum 등의 타입
```java
@Target(ElementType.ANNOTATION_TYPE)	// Annotation에 사용
public @interface TargetAnnotation {}

@TargetAnnotation			// Annotation에 사용
@Target(ElementType.CONSTRUCTOR)
public @interface TargetConstructor {}	// 생성자에 사용

@Target(ElementType.FIELD)
public @interface TargetField {}		// 필드에 사용

@Target({ElementType.FIELD, ElementType.LOCAL_VARIABLE, ElementType.METHOD})	// 필드, 지역변수, 메소드에 사용
public @interface TargetFLM {}
```
```java
public class testClass{
	@TargetConstructor		// 생성자에 사용
	public testClass();

	@TargetField			// 필드에 사용
	@TargetFLM			// 필드, 지역변수, 메소드에 사용
	int testField;
}
```
- `@Inherited` - Annotation이 클래스에 사용되는 경우 자식 클래스도 해당 Annotation을 물려받음
```java
@Inherited
public @interface InheritedAnnotation {}
```
```java
@InheritedAnnotation
public class testClass{}

public class testClass2 extends testClass {}	// InheritedAnnotation 상속됨
```
- `@Repeatable(Repeats.)` - Annotation을 동일한 요소에 여러번 사용가능하게 함




# Java 7
## AutoCloseable
- `implements AutoCloseable` - try with resource 문에서 사용된 resource 자동으로 close
```java
public class AutoCloseableClass implements AutoCloseable {
	public void makeError() throws Exception{
		throw new Exception();
	}
	@Override
	public void close() throws Exception {
		System.out.println("close");
	}
}

try (AutoCloseableClass acc = new AutoCloseableClass()) {
	acc.makeError();
} catch (Exception e) {
	e.printStackTrace();
	System.out.println("error발생");
} finally {
	System.out.println("finally");
}

// close [printStackTrace()] error발생 finally 출력
```


# Java 8
## FunctionalInterface Lambda
- `@FunctionalInterface` - [Annotation](#annotation)을 Interface 앞에 붙이면 해당 Interface는 메소드를 하나만 가져야함
- `java.util.function` 패키지에서 자주사용하는 인터페이스 제공
	- [Runnable](#thread-runnable) - **java.lang 패키지	methods: run
	- `Supplier<T>`	- methods: get /	반환 : T
	- `Consumer<T>`	- methods: accept /	매개변수 : T
	- `BiConsumer<T, U>`	- methods: accept /	매개변수 : T, U
	- `Function<T, R>`	- methods: apply /	매개변수 : T /	반환 : R
	- `BiFunction<T, U, R>`	- methods: apply	매개변수 : T, U /	반환 : R
	- `Predicate<T>`	: methods: test /	매개변수 : T /	반환 : boolean
	- `BiPredicate<T, U>`	- methods: test /	매개변수 : T, U /	반환 : boolean
	- `UnaryOperator<T>`	- methods: apply /	매개변수 : T /	반환 : T
	- `BinaryOperator<T>`	- methods: apply /	매개변수 : T, T /	반환 : T
```java
import java.lang.Runnable;
import java.util.Function.*;

@FunctionalInterface
public interface FunctionalInterface {
	void method();	// 하나의 메소드만
}

// 기존 구현방식
//FunctionalInterface old = new FunctionalInterface() {
//	@Override
//	public void method() {
//		//동작
//	}
//};
// 함수형 인터페이스
FunctionalInterface functionalInterface = () -> {
	//동작
};

String s = "";
Runnable runnableFunction = () -> System.out.println("매개변수X 반환X");
Supplier<String> supplierFunction = () -> "매개변수X 반환O";
s = supplierFunction.get();		// 매개변수X 반환O

Consumer<String> consuumerFunction = s -> System.out.println(s);
BiConsumer<String, Integer> biConsuumerFunction = (s, i) -> {
	for(int k=0;k<i;k++){ System.out.println(s); }
};
consuumerFunction.accept("매개변수O, 반환X");					// 매개변수O, 반환X
biConsuumerFunction.accept("매개변수多, 반환X", 2);					// 매개변수多, 반환X 매개변수多, 반환X

Function<Integer, String> functionFunction = i -> String.format("매개변수 : %d, 반환 O", i);	// 매개변수O, 반환O
BiFunction<Integer, Integer, String> biFunctionFunction = (i1, i2) ->{			// 매개변수多, 반환O
	String.format("매개변수 : %d %d, 반환 O", i1, i2);
}
s = functionFunction.apply(1);			// 매개변수 : 1, 반환 O
s = biFunctionFunction.apply(1,2);			// 매개변수 : 1 2, 반환 O

Predicate<Integer> predicateFunction = i -> i % 2 == 1;				// 매개변수 : O, 반환 O
BiPredicate<Integer, String> biPredicateFunction = (i, s) -> i == 1 && s.equals(홀수입니다);	// 매개변수 : 多, 반환 O
boolean b1 = predicateFunction.test(1);		// true
boolean b2 = biPredicateFunction.test(1, "짝수입니다");	// false

UnaryOperator<String> unaryOperatorFunction = s -> s;		// 매개변수 : O, 반환 O (매개변수와 반환 타입 동일)
BinaryOperator<String> biNaryOperatorFunction  = (s, s) -> s + s;	// 매개변수 : 多, 반환 O (매개변수들과 반환 타입 동일)
s = unaryOperatorFunction.apply("매개변수와 반환 타입 동일");
s = BinaryOperatorFunction.apply("매개변수 : 多, 반환 O ", "매개변수들과 반환 타입 동일");
```
- [참고 - String.formatted](#formatted)

## Method Reference
- `[class명]::[class.method명]`
- 람다식이 어떤 메소드 하나만 호출할 때 코드를 간편화
- 사용되는 메소드의 매개변수, 반환 타입이 모두 일치해야함
```java
import java.util.Function.*;
//Consumer<String> consuumerFunction = s -> System.out.println(s);
Consumer<String> consuumerFunction = System.out::println;
consuumerFunction.accept("abc");	// abc 출력
```

## Optional
- `Optional<T>` - null일 수도 있는 T 타입의 값
	- .empty() : null 반환
	- of([data]) : null 이 아닌값이 확실할 때 사용 (null 담으면 NullPointException)
	- ofNullable([data]) : null / null 이 아닌값 둘다 가능

## Stream
- `[변수].[메소드1].[메소드2]...` - 일련의 데이터를 연속적으로 가공하는데 유용
	- `Stream<T>` - Wrapped Class stream
	- `[type]Stream` - 원시 자료형 stream
	- `Stream<Entry<String, Character>>` - map stream
	- `Stream.Builder<T>` - StringBuilder 와 비슷
	- `Stream.of([data1], [data2]...)` - 직접 stream 생성
	- `Stream.concat([stream1], [stream2]...)` - stream 합치기
	- `Stream.iterate([동작])` - iterator로 생성
	- `Stream.empty()` - 빈 stream 생성
```java
import java.util.List;
import java.util.ArrayList;
import java.util.Array;

List<Integer> intList = new ArrayList<>(Arrays.asList(5, 2, 0, 8, 4, 1, 7, 9, 3, 6));
Stream<Integer> streamData = int0To9
	.stream()				// Stream<Integer>
	.filter(i -> i % 2 == 1)		// 홀수만
	.sorted(Integer::compare)		// 오름차순정렬
	.map(String::valueOf)		// 문자열로 변환
	.collect(Collectors.joining(", "));	// "0,1,2,3,4,5,6,7,8,9"

// 배열로부터 스트림생성
Integer[] integerArray = {1,2,3,4,5};
Stream<Integer> streamInteger = Arrays.stream(integerArray );
Array ar = streamInteger.toArray();
// streamInteger.toArray(); // 한번 더 실행 시 에러 (stream 데이터는 한번 동작하면 끝남)

// 원시값 배열로부터 스트림생성
int[] intArray = {1,2,3,4,5};
IntStream streamInt = Arrays.stream(intArray);

// 원시Stream > Wrapped Stream
Stream<Integer> convert = streamInt.boxed();

// collection을 통해 스트림 생성
List<Integer> intList = new ArrayList<>(Arrays.asList(5, 2, 0, 8, 4, 1, 7, 9, 3, 6));
Stream<Integer> streamInt3 = intList.stream();

// Map
Map<String, Character> map = new HashMap<>();
Stream<Entry<String, Character>> streamMap = map.entrySet().stream();

// Stream.Builder
Stream.Builder<Integer> builder = Stream.builder();
builder.accept(1); builder.accept(2); builder.accept(3); builder.accept(4); builder.accept(5);
Stream<Integer> streamBuilder = builder.build();

// Stream.of 직접 스트림생성
Stream<Integer> stream1 = Stream.of(11, 22, 33);

// Sream.concat 합치기
Stream<Integer> stream2 = Stream.of(44, 55, 66);
Stream<Integer> concatStream = Stream.concat(stream1, stream2);

// Stream.iterate
// 0 2 4 6 8 10 12 14 16 18
Stream<Integer> withIter1 = Stream.iterate(0, i -> i + 2).limit(10);
// 홀, 홀짝, 홀짝홀, 홀짝홀짝, ... , 홀짝홀짝홀짝홀짝
Stream<String> withIter2 = Stream.iterate("홀", s -> s + (s.endsWith("홀") ? "짝" : "홀")).limit(8);

// Stream.empty
Stream<Double> streamEmpty = Stream.empty();		// 빈 stream 생성

// 기타
IntStream streamRange1 = IntStream.range(10, 20);		// 10~19 미포함
IntStream streamRange2 = IntStream.rangeClosed(10, 20);	// 10~20 포함
IntStream streamRandom= new Random().ints(5, 0, 100);	// 0~100 중 정수 5개 난수생성
IntStream streamChars= "Hello World".chars();			// 문자열의 각 char 의 int값
```

## Stream parallel
- `[List 객체].parallelStream()` - 리스트 병렬스트림 반환
- `[Stream 객체].parallel()` - 직렬스트림 > 병렬스트림 변경
- `[Stream 객체].isParallel()` - 병렬스트림 이면 true

## Stream sequential
- `[List 객체].sequentialStream()` - 리스트 직렬스트림 반환
- `[Stream 객체].sequential()` - 병렬스트림 > 직렬스트림 변경
- `[Stream 객체].isSequential()` - 직렬스트림 이면 true

## Stream Middle Methods
### Stream boxed
- `.boxed()` - 원시Stream 을 Wrapped Stream 으로 변환
```java
import java.util.Stream;
String s = "ABC";
Stream<Integer> boxedStream = s.chars()	// IntStream
				.boxed();	// Stream<Integer>
```
### Stream distinct
- `.distinct()` - 중복요소 제거
### Stream sorted
- `.sorted([정렬함수])` - 각 요소들을 정렬
### Stream skip
- `.skip([int])` - 0번째에서 int 만큼 생략 후 요소들 반환
### Stream limit
- `.limit([int])` - [스트림 최종 연산](#stream-final-methods) 시 순회요소 개수 지정
### Stream peek
- `.peek([동작함수])` - [스트림 최종 연산](#stream-final-methods)과 같이 각 요소를 순회하며 동작 실행
### Stream map
- `.map([반환함수])` - 각 요소를 순회하며 새로운 요소들 반환
### Stream filter takewhile dropwhile
- `.filter([boolean 반환함수])` - 조건문에 해당하는 요소들만 추출
- `.takewhile([boolean 반환함수])` - false 인 순간 뒤의 요소들도 제외 (true 인 동안만 요소들 추출)
- `.dropwhile([boolean 반환함수])` - false 인 동안 요소들 제외 (true 인 순간부터 뒤의 요소들 추출)

## Stream Final Methods
### Stream foreach
- `.forEach([동작함수])` - 요소들 순회하며 동작
### Stream count min max sum
- `.count() / .min() / .max() / .sum()` - 요소의 수 / 최소값 / 최대값 / 총합 반환
### Stream reduce
- `.reduce(([초기값], [이전반환요소], [현재요소])->[반환값])` - 이전반환요소와 현재요소로 작업 후 반환
- 초기값은 옵션으로 있는경우 시작 시 초기값을 이전반환요소로 취급
```java
"qwertyuiopasdfghjklzxcvbnmasdf".chars()
	.distinct()				// 중복제거 qwertyuiopasdfghjklzxcvbnm
	.sorted()				// 정렬 abcdefghijklmnopqrstuvwxyz
	.skip(2)				// 2개 생략 cdefghijklmnopqrstuvwxyz
	.peek(c->System.out.print((char)c))	// 최종 순회 시 같이 순회하며 출력
	.limit(3)				// 최종 순회 시 순회요소 개수 지정
	.map(Character::toUpperCase)	// 대문자로 변환 CDEFGHIJKLMNOPQRSTUVWXYZ
	.filter(c->'J'!=c)			// CDEFGHIKLMNOPQRSTUVWXYZ
	.takewhile(c->'G'==c)		// G에서 false 이므로 이후 요소 제외 > CDEF
	.dropwhile(c->'D'==c || 'F'==c)	// D에서 true 이므로 이후 요소 추출 > DEF
	.forEach(c->System.out.println(c));	// 최종 순회
// cd68\ne69 출력됨
// peek 에서 c부터 순회, forEach에서는 c 없음 > 첫번째 순회 c 출력
// peek 에서 d 출력, forEach 에서 D(68) 출력 > 두번째 순회 d68\n출력
// peek 에서 e 출력, forEach 에서 E(69) 출력 > 세번째 순회 e69\n출력 > limit(3) 완료
	//.count();	// long 2 반환
	//.min();	// OptionalInt[68] 반환
	//.max();	// OptionalInt[69] 반환
	//.sum();	// int 137 반환 (68+69)
	//.reduce((previous, current)->previous+current);	// OptionalInt[137] 반환 (68+69)
	//.reduce(1, (previous, current)->previous+current);	// 138 반환 (1+68+69)
```

## Thread CompletableFuture
- [Future](#thread-future) 보다 편리하고 강력한 기능들 제공
- 연속되는 작업들을 비동기적으로, 함수형으로 작성
- 여러 비동기 작업을 조합하고 병렬적 실행 가능
- 예외 처리 기능 제공
```java
import java.util.Random;

// 임의의 시간이 걸리는 작업
public static void time (boolean error) {	// error발생상황 부여 (true 시)
	try {
		int randomMS = 1000 + new Random().nextInt(500);
		Thread.sleep(randomMS);	// 1.0~1.5 초 멈춤
		System.out.println((randomMS / 1000.0f) + "초 경과");
	} catch (InterruptedException e) { throw new RuntimeException(e); }
	if (error) throw new RuntimeException("-time- 오류 발생");
}
```
### CompletableFuture supplyAsync
- `CompletableFuture.supplyAsync(() -> {[동작]})`
	- supplyAsync 의 리턴타입은 CompletableFuture<T>
	- .get() 을 호출해야만 동작함

### CompletableFuture thenApply
- `[CompletableFuture<T> 객체].thenApply([CompletableFuture<T> 반환 값] -> {[동작]}` - [JS Promise.then](#/Javascript/비동기처리/Promise.md) 역할
	- 이전에 처리된 값을 받아 새로운 값 반환
	- thenApply 의 리턴타입은 CompletableFuture<T>
	- .get() 을 호출해야만 동작함

### CompletableFuture exceptionally
- `[CompletableFuture<T> 객체].exceptionally([error] -> {[동작]}` - [JS Promise.catch](#/Javascript/비동기처리/Promise.md) 역할
	- 이전에 error 발생 시 실행
	- exceptionally 의 리턴타입은 CompletableFuture<T>
	- 이후 thenApply, thenAccept 있을 시 여기서 반환한 값을 기준으로 정상 동작

### CompletableFuture thenAccept
- `[CompletableFuture<T> 객체].thenAccept([CompletableFuture<T> 반환 값] -> {[동작]}`
	- 뒤에 then 체이닝이 안되는 [JS Promise.then](#/Javascript/비동기처리/Promise.md) 역할
	- 이전에 처리된 값을 받아 동작 처리 (값 반환 X)
	- thenAccept 의 리턴타입은 CompletableFuture<Void>
	- .get() 을 호출해야만 동작함
```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public static CompletableFuture<String> completableFutureString(boolean b) throws ExecutionException, InterruptedException {
	return CompletableFuture.supplyAsync(() -> {
		System.out.println("supplyAsync 시작");
		time(false); 	// 임의의 시간이 걸리는 작업
		return "supplyAsync";
	}).thenApply(s ->{
		System.out.println("thenApply_1 시작 : " + s);
		time(false); 	// 임의의 시간이 걸리는 작업
		return s + ", thenApply_1";	// supplyAsync, thenApply_1
	}).thenApply(s ->{
		System.out.println("thenApply_2 시작 : " + s);
		time(b); 	// error 발생
		return s + ", thenApply_2";	// 정상 동작 시 supplyAsync, thenApply_1, thenApply_2 반환
	}).exceptionally(e -> {
		e.printStackTrace();
		return "error 발생";
       	});
}

public static void completableFutureTest() throws ExecutionException, InterruptedException {
	completableFutureString(true).thenAccept(s -> {
		System.out.println("thenAccept 시작 : " + s);	// thenAccept 시작 : error 발생
		time(false); 	// 임의의 시간이 걸리는 작업
		System.out.println(s + ", thenAccept");	// error 발생, thenAccept 출력
	}).get();
}

try {
	completableFutureTest();
} catch (Exception e) {
	e.printStackTrace();
}
```

### CompletableFuture thenCompose
- `[CF<T> 객체1].thenCompose([객체1 data]->[CF<T> 객체2].[then methods](([객체2 data])->{[동작]})`
	- 두 CompleteFuture의 결과를 조합, 두 작업은 동시진행됨

### CompletableFuture thenCombine
- `[CF<T> 객체1].thenCombine([CF<T> 객체2], ([객체1 data], [객체2 data])->{[동작]})`
	- 두 CompleteFuture의 결과를 조합, 두 작업은 동시진행됨
```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
try {
	CompletableFuture<String> cfs1 = completableFutureString(false);
	CompletableFuture<String> cfs2 = completableFutureString(false);
	cfs1.thenCompose(cfs1Data ->
		cfs2.thenApply(cfs2Data->{
			return cfs1Data + " / " + cfs2Data;
		}
	)).thenAccept(System.out::println)
	.get();	// supplyAsync, thenApply_1, thenApply_2 / supplyAsync, thenApply_1, thenApply_2
	
	cfs1.thenCombine(cfs2, (cfs1Data, cfs2Data) -> {
		return cfs1Data + " / " + cfs2Data;
	}).thenAccept(System.out::println)
	.get();	// supplyAsync, thenApply_1, thenApply_2 / supplyAsync, thenApply_1, thenApply_2
} catch (Exception e) {
	e.printStackTrace();
}
```

### CompletableFuture allof
- `allOf([CF객체], [CF객체2], ...)` - 여러 CompletableFuture 작업을 동시에 진행
	- `thenRun(()=>{})` - 결과들을 동기적으로 종합

### CompletableFuture anyof
- `anyOf([CF객체], [CF객체2], ...)` - 가장 먼저 완료된 결과물을 받아옴

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

CompletableFuture<Integer> roll1 = rollDiceFuture();
	CompletableFuture<Integer> cfi1 = CompletableFuture.supplyAsync(() -> { time(false); return 1;});
	CompletableFuture<Integer> cfi2 = CompletableFuture.supplyAsync(() -> { time(false); return 2;});
	CompletableFuture<Integer> cfi3 = CompletableFuture.supplyAsync(() -> { time(false); return 3;});
	CompletableFuture<Integer> cfi4 = CompletableFuture.supplyAsync(() -> { time(false); return 4;});
	
	// List<CompletableFuture<Integer>> cfiList = new ArrayList<>();
	// cfiList.add(cfi1); cfiList.add(cfi2); cfiList.add(cfi3); cfiList.add(cfi4);
	// CompletableFuture.allOf(cfiList.stream().toArray(CompletableFuture[]::new)).thenRun(() -> {
	CompletableFuture.allOf(cfi1, cfi2, cfi3, cfi4).thenRun(() -> {
		System.out.println("결과 모두 나옴");
		Integer int1 = cfi1.join();
		Integer int2 = cfi2.join();
		Integer int3 = cfi3.join();
		Integer int4 = cfi4.join();
		String result = IntStream.of(int1, int2, int3, int4)
			.boxed()
			.map(i ->String.valueOf(i))
			.collect(Collectors.joining(", "));
		System.out.println("allOf 최종 결과 : " + result);
	}).get();	// 1, 2, 3, 4

	CompletableFuture.anyOf(cfi1, cfi2, cfi3, cfi4).thenAccept(result -> {
		System.out.println("anyOf 최종 결과 : " + result);
	}).get();	// 1
```

# Java 13
## formatted 13
- [이전버전 formatted](#formatted)
- formatted([변수1],[변수2]...) - 문자열 내에 변수를 가독성좋게 할당 가능
	- %b / %d / %f / %c / %s : boolean / 10진수 / 실수 / 문자 / 문자열
	- %n : 개행 (초기 문자열내에서 \n 대신 사용)
```java
String str1 = "%s의 둘레는%n반지름 X %d X %f입니다.";
String circle = "원";
int two = 2;
double PI = 3.14;

// 13+
String str2 = str1.formatted(circle, two, PI);	// 원의 둘레는\n반지름 X 2 X 3.13입니다.
```



# Java 16
## record 16
- record : 클래스의 기능을 축소시켜서 데이터의 묶음만 저장하기 위한 단순한 클래스
```java
// record
public record Child(
        String name,
        int birthYear,
        Gender gender
) {}

// 기존 클래스
public class ChildClass {
    private final String name;
    private final int birthYear;
    private final Gender gender;

    public ChildClass(String name, int birthYear, Gender gender) {
        this.name = name;
        this.birthYear = birthYear;
        this.gender = gender;
    }

    public String getName() { return name; }
    public int getBirthYear() { return birthYear; }
    public Gender getGender() { return gender; }
}
```