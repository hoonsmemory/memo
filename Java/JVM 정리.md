### JDK, JRE, JVM이란?
![image](https://github.com/user-attachments/assets/b3915101-889d-414f-b12d-a373ca196faf)
### JVM
JVM(Java Virtual Machine)은 .class로 컴파일된 파일(자바 바이트 코드)을 클래스 로더(Class Loader)를 통해 읽고 분석하여 OS에 특화된 코드(기계어)로 변환 및 실행될 수 있는 런타임 환경을 제공하는 사양이다.  
한번쯤 자바 언어는 **"한 번의 코드 작성으로 어느 환경에서도 실행 가능하다.(WORA)"** 란 말을 들어봤을 것이다.  
JVM은 WORA을 실현시켜줄 수 있도록 하는 역할을 맡고 있다.  

### JRE
JRE(Java runtime environment)는 .class로 컴파일된 파일을 Java 애플리케이션을 실행할 수 있도록 구성된 배포판이다.  
따라서 JRE에는 JVM을 비롯해 Java 애플리케이션 실행에 필요한 핵심 라이브러리 및 자바 런타임 환경에서 사용하는 프로퍼티 세팅이나 리소스 파일을 가지고 있다.  
컴파일된 .class 파일만 있다면 자바 애플리케이션을 동작시킬 수 있다.  

### JDK
JDK(Java Development Kit)는 Java 애플리케이션을 개발, 디버깅, 컴파일 및 모니터링하기 위한 개발 도구와 함께 JRE에 있는 모든 것이 포함되어 있다.  
Java 애플리케이션을 개발해야 한다면 반드시 JDK가 필요하다.  

<details>
<summary>JDK와 함께 제공되는 몇 가지 중요한 구성 요소는 다음과 같다.</summary>
appletviewer - 이 도구는 웹 브라우저 없이 Java 애플릿을 실행하고 디버그 하는데 사용할 수 있다.
<br>
apt – 주석 처리 도구  
<br>
extcheck – JAR 파일 충돌을 감지하는 유틸리티  
<br>
javac - java 소스 파일을 class 파일로 변환  
<br>
javadoc – 소스 코드 주석에서 문서를 자동으로 생성하는 문서 생성기  
<br>
jar – 관련 클래스 라이브러리를 단일 JAR 파일로 패키징 하는 아카이버. 이 도구는 또한 JAR 파일을 관리하는 데 도움이 된다.  
<br>
jarsigner – jar 서명 및 확인 도구  
<br>
javap – 클래스 파일 디스 어셈블러
<br>
javaws – JNLP 애플리케이션용 Java Web Start 실행기
<br>
JConsole – 자바 모니터링 및 관리 콘솔  
<br>
jhat – 자바 힙 분석 도구  
<br>
jrunscript – Java 명령줄 스크립트 셸  
<br>
jstack – Java 스레드의 Java 스택 추적을 인쇄하는 유틸리티  
<br>
keytool – 키 저장소를 조작하기 위한 도구  
<br>
policytool – 정책 생성 및 관리 도구  
<br>
xjc – JAXB(XML 바인딩용 Java API) API의 일부입니다. XML 스키마를 수락하고 Java 클래스를 생성한다.  
</details>
<br>
<br>

### JVM 구조
![image](https://github.com/user-attachments/assets/c31c0632-757e-4dd1-8319-4dd40861f953)
JVM은 크게 바이트 코드를 JVM 메모리 영역에 로딩하는 Class Loader, 메모리 영역인 Runtime Data Area, 로딩된 바이트 코드를 해석하는 Execution Engine과 힙 메모리를 관리하는 Garbage Collector로 나뉜다.
<br>
<br>

### 자바 어플리케이션 실행 과정
1. 어플리케이션이 실행되면 JVM이 OS로부터 메모리를 할당 받는다. (JVM은 할당 받은 메모리를 용도에 따라 영역을 구분하여 관리)  
2. 자바 컴파일러(javac.exe)가 자바 소스코드(.java)를 읽어 바이트 코드(.class)로 변환한다.  
3. Class Loader를 통해 바이트 코드를 JVM으로 로딩한다.  
4. 로딩된 바이트 코드는 Execution Engine을 통해 해석된다.  
5. 해석된 바이트 코드는 Runtime Data Area에 배치되어 실행된다.(실행되는 과정에서 GC 같은 작업이 수행됨)  
<br>

### Class Loader
![image](https://github.com/user-attachments/assets/a26b4d86-55ec-44c4-8dcd-11b11dafdc77)
Class Loader는 Java Compiler에 의해 컴파일 후 변환된 바이트 코드를 로드하고, 링크를 통해 코드를 검증하는 작업을 수행하는 모듈이다.  
로드한 코드들을 엮어서 JVM 메모리 영역인 Runtime Data Area에 배치한다.  
이 때 모든 코드를 한 번에 올리지 않고 어플리케이션에서 필요한 경우 동적으로 메모리에 적재한다.  

#### ClassLoder 세부
**Bootstrap classloader**  
JVM 시작 시 가장 최초로 실행되는 클래스 로더이다.  
부트스트랩 클래스 로더는 자바 클래스를 로드하는 것이 아닌, 자바 클래스를 로드할 수 있는 자바 자체의 클래스 로더와 최소한의 자바 클래스(java.lang.Object, Class, ClassLoader)만을 로드한다.  
```JAVA
//Java 8
jre/lib/rt.jar 및 기타 핵심 라이브러리와 같은 JDK의 내부 클래스를 로드한다.

//Java 9 이후
더 이상 /re.jar이 존재하지 않으며, /lib 내에 모듈화되어 포함됐다. 이제는 정확하게 ClassLoader 내 최상위 클래스들만 로드한다.
```

**Extension classloader**  
확장 클래스 로더는 부트스트랩 클래스 로더를 부모로 갖는 클래스 로더로서, 확장 자바 클래스들을 로드한다.  
java.ext.dirs 환경 변수에 설정된 디렉토리의 클래스 파일을 로드하고, 이 값이 설정되어 있지 않은 경우 ${JAVA_HOME}/jre/lib/ext 에 있는 클래스 파일을 로드한다.
```JAVA
//Java 8  
URLClassLoader를 상속하며, jre/lib/ext 내 모든 클래스를 로드한다.

//Java 9 이후
Platform Loader로 변경되었으며, URLClassLoader가 아닌 BuiltinClassLoader를 상속한다. Inner Static 클래스로 구현되어 있다.
```

**Application classloader**  
자바 프로그램 실행 시 지정한 Classpath에 있는 클래스 파일 혹은 jar에 속한 클래스들을 로드한다.  
쉽게 말하자면, 개발자가 만든 .class 확장자 파일을 로드한다.  
<br>

### Linking
Linking 은 로드된 클래스 파일들을 검증하고, 사용할 수 있게 준비하는 과정을 의미한다.  
Linking 또한 Verification , Preparation , 그리고 Resolution 이라는 세 가지 단계로 이루어져 있다.

**Verification**  
클래스 파일이 유효한지를 확인하는 과정이다. 클래스 파일이 JVM 의 구동 조건 대로 구현되지 않았을 경우에는 VerifyError 를 던진다.  

**Preparation**  
클래스 및 인터페이스에 필요한 static field 메모리를 할당하고, 이를 기본값으로 초기화한다.  
기본값으로 초기화된 static field 값들은 뒤의 Initialization 과정에서 코드에 작성한 초기값으로 변경된다. 이 때문에 JVM 에 탑재된 클래스 파일의 코드를 작동시키지는 않는다.  

**Resolution**  
Symbolic Reference 값을 JVM 의 메모리 구성 요소인 Method Area 의 런타임 환경 풀을 통하여 Direct Reference 라는 메모리 주소 값으로 바꾸어준다.  
해당 단계의 영향을 받는 JVM Instruction 요소는 new 및 instanceof 가 있다.  

### Initialization
Linking 과정을 거치면 Initialization 단계에서는 클래스 파일의 코드를 읽게 된다. Java 코드에서의 class 와 interface 의 값들을 지정한 값들로 초기화 및 초기화 메서드를 실행시켜준다.  
이때, JVM 은 멀티 쓰레딩으로 작동을 하며, 같은 시간에 한 번에 초기화를 하는 경우가 있기 때문에 초기화 단계에서도 동시성을 고려해주어야 한다.  
Class Loader 를 통한 클래스 탑재 과정이 끝나면 본격적으로 JVM 에서 클래스 파일을 구동시킬 준비가 끝나게 된다.  
<br>
<br>

### Runtime Data Area
![image](https://github.com/user-attachments/assets/706a7f91-ebf5-456a-9885-05c62c27aa44)
Runtime Data Area는 크게 5가지로 구성이 되어있으며, 어플리케이션이 동작하기 위해 OS에서 할당받은 메모리 공간을 의미한다.  

**Method Area**  
Method Area는 모든 스레드가 접근할 수 있는 영역으로 static으로 선언된 변수들을 포함하여 Class 레벨의 모든 데이터가 이곳에 저장된다.  
JVM마다 단 하나의 Method Area가 존재한다.  

**저장되는 정보의 종류**  
* Field Info : 멤버 변수의 이름, 데이터 타입, 접근 제어자의 정보
* Method Info : 메서드 이름, Return 타입, 매개변수, 접근 제어자의 정보
* Type Info : Class인지 Interface인지 확인 가능한 정보, Type의 속성, 이름, Super class의 이름
* Runtime Constant Pool 영역(상수 자료형을 저장하여 참조하는 역할)
<br>

**Heap Area(Java8)**  
![image](https://github.com/user-attachments/assets/e7acac36-f2ea-4ad3-ad64-8ab4969882a8)
Heap Area 또한 모든 스레드가 접근할 수 있는 영역으로 new 연산자로 생성된 모든 Object와 Instance 변수 그리고 배열을 저장한다.  
Heap 영역은 물리적으로 Young Generation 과 Old Generation으로 구부할 수 있다.  
Young Generation : 생명 주기가 짧은 객체를 GC 대상으로 하는 영역이며, 초기에 Eden 영역에 할당 후 Survivor 0과 1을 거친다.  
Old Generation : Survivor 0, 1 영역을 여러번 거치다보면 GC에 의해 생명 주기가 긴 Old Generation 영역으로 옮겨진다.  
또한, Heap 영역에 있는 데이터는 Minor GC, Major GC를 통해 메모리가 정리된다.  
<br>

**Stack Area**  
![image](https://github.com/user-attachments/assets/2d923351-3966-4da3-8c65-a57a67c29d16)  
Stack Area는 각 스레드를 위해 분리된 Runtime Stack 영역이다.  
메서드를 호출할 때 마다 Stack Frame으로 불리는 Entry가 Stack Area에 생성되며, 스레드가 종료되면 바로 소멸되는 특성의 데이터를 저장한다.  
<br>

**PC Register**  
![image](https://github.com/user-attachments/assets/634cb78d-84bc-4c45-8f05-17e886df3593)  
PC Register는 각 스레드가 시작될 때 생성되며, 현재 실행중인 상태 정보(어떤 명령을 실행해야 할 지에 대한 정보)를 저장하는 영역이다.  
스레드가 생성될 때마다 1:1로 존재하게 되며, 로직을 처리하면서 지속적으로 수행중인 부분의 주소를 가지게 된다.  
<br>

**Native Method Stack**
![image](https://github.com/user-attachments/assets/d626b8fd-b41d-41d6-8843-bcdd3c0d03c6)  
Java로 구현된 소스가 아닌 C, C++ 등으로 구현된 소스를 실행 시키는 영역이다.  
Java Native Interface를 통해 바이트 코드로 전환하여 저장되며, 각 스레드별로 생성이 된다.  
<br>

**Execution Engine**  
![image](https://github.com/user-attachments/assets/1eefd0f9-ea7a-4c80-9a46-d6cc1dfae790)  
Runtime Data Area에 할당된 바이트 코드를 실행시키는 역할을 하며 코드를 실행하는 방식은 크게 2가지가 있다.  
Interpreter : 바이트 코드를 해석하여 실행하는 역할을 하며 동일한 메서드라도 반복 호출될 경우 매번 새로 수행한다.  
JIT(Just In Time) Compiler : 반복되는 코드를 발견하여 전체 바이트 코드를 컴파일하고 Native Code로 변경하여 사용한다.  
<br>

**Garbage Collector**  
Heap 영역이나 Method 영역인 동적으로 할당했던 메모리 영역 중 사용되지 않는 객체의 메모리를 Garbage라고 부르며, 정해진 스케줄에 의해 정리해주는 것을 Garbage Collection(GC)이라고 한다.  
메모리를 정리할 때 JVM이 멈추게 되는데 이를 Stop The World 라고도 부른다.  
GC가 작동하는 동안 GC관련 스레드를 제외하고 모든 스레드가 멈추게 되므로 성능 저하를 유발한다.  
<br>

**일반적인 GC 과정**  
1. 맨 처음 객체가 생성되면 Eden 영역에 생성된다.  
2.  일정한 시간이 지나면 Minor GC가 발생하여 사용되지 않는 객체는 제거가 되며, 사용중인 객체는 Survivor1, 2 영역으로 이동한다.
(단, 객체의 크기가 Survivor 영역보다 클 경우 바로 Old Generation으로 이동한다.)
3. Survivor1과 2 영역을 서로 오가며 다른 한 곳은 비어지게 되는데 이러한 과정을 반복하면 score가 누적이 되고 기준치 이상이 되면 Old Generation으로 이동한다.(Promotion)
4. Old Generation 영역에서 살아남았던 객체들이 일정 수준으로 쌓이게 되면, 미사용으로 판단된 객체들을 제거해주는 Major GC가 발생되며 효율적으로 제거하기 위해 JVM은 GC 관련 스레드를 제외하고 모든 스레드를 잠시 멈추게 한다.
<br>

### GC 종류
**Serial GC**  
하나의 CPU로 Young Generation과 Old Generation을 연속적으로 처리하는 방식이다.  
가장 오래된 GC이며 Mark and Comapct 알고리즘을 사용한다.  
<br>

**Parallel GC**  
자바 7~8 버전에서 Default로 설정되어 있는 GC  
Parallel GC의 목표는 다른 CPU가 GC의 진행시간 동안 대기 상태로 남아 있는 것을 최소화 하는 것이다.  
GC 작업을 병렬로 처리하여 Stop The World 시간이 비교적 짧다.  
<br>

**Parallel Compacting GC**  
Parallel GC에서 Old Generation의 처리 알고리즘을 변경하였다.  
<br>

**Concurrent Mark Sweep(CMS) GC**  
기존에는 GC가 실행되면 어플리케이션에서 사용되는 스레드가 정지되었지만 CMS GC의 경우 GC 관련 스레드와 동시에 실행되어 Stop The World 를 최소화하였다.  
Parallel GC와의 가장 큰 차이점은 Compaction 작업 유무로 구분된다.  
Compaction : 메모리 공간에서 사용하지 않는 빈 공간이 없도록 옮겨서 메모리 분산을 제거하는 작업  
<br>

**Garbage First(G1) Gc**  
큰 메모리에서 사용하기 적합한 GC(대규모 Heap 사이즈에서 짧은 GC 시간을 보장하는데 목적을 둠)  
전체 Heap 영역을 Region이라는 영역으로 분할하여 상황에 따라 역할이 동적으로 부여된다.  
<br>

**Z GC**
ZPage라는 영역을 사용하며, G1 GC의 Region은 크기가 고정인데 비해 Zpage는 2mb 배수로 동적으로 운영된다.  
정지 시간이 최대 10ms를 초과하지 않은 것을 목적으로 운영되며 Heap 크기가 증가하더라도 정지 시간이 증가하지 않는 것이 특징이다.  
<br>
<br>

**참고자료**  
[자바(Java) 메모리 구조 소개](https://youtu.be/zta7kVTVkuk)  






