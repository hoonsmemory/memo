### 리플렉션이란?
리플렉션은 클래스의 정보(필드, 메서드, 상속된 클래스와 인터페이스, 적용된 애노테이션 등)를 읽어 객체 생성, 메서드 사용, 필드 값 변경 등 임의대로 조작을 할 수 있도록 자바에서 제공되는 API다.  
대표적으로 리플렉션은 스프링, 하이버네이트, JUnit 등에서 사용된다고 하며 예시를 통해 리플렉션을 사용해보도록 하자.  
<br>

### 리플렉션 사용법
리플렉션은 jdk 1.1버전부터 적용되어 java.lang.reflect 패키지를 임포트 해주면 바로 사용이 가능하다.  
```java
import java.lang.reflect.*
```
<br>
리플렉션을 사용하기 위해 접근 지정자를 적절하게 사용해서 필드와 메서드를 선언해주고 생성자도 만들어준다.  

```java
@MyAnnotation
public class Child extends Parent implements MyInterface{

    public Child(){
    }

    public Child(String a, String d, String e) {
        A = a;
        D = d;
        E = e;

        System.out.println(toString());
    }

    private String A = "A";

    private static String B = "B";

    private static final String C = "C";

    public String D = "D";

    public static String F = "F";

    protected String E = "E";

    public void c(){
        System.out.println("C");
    }

    public int sum(int left, int right) {
        return left + right;
    }

    private void foo(){
        System.out.println("foo");
    }

    public void goo() {
        System.out.println("goo");
    }
```
<br>

### Class<T>에 접근하는 방법
작성한 클래스의 정보를 읽어오기 위해서는 Class<?>를 사용해주어야 하며  Class<?>에 접근하는 방법은 크게 3가지 방법이 있다.  
1. 모든 클래스는 로딩 한 다음 Class<T>의 인스턴스가 생긴다다. → "타입.class"로 접근
2. 모든 인스턴스는 getClass() 메소드를 가지고 있다. → “인스턴스.getClass()”로 접근
3. 클래스를 문자열로 읽어오는 방법 (ClassNotFoundException 주의)
```java
//타입.class로 가져오는 방법
Class<Child> class1 = Child.class;

//인스턴스로 가져오는 방법
Child child = new Child();
Class<? extends Child> class2 = reflection.getClass();

//클래스의 경로를 사용해서 가져오는 방법
Class<?> class3 = Class.forName("me.hoon.reflection.Child");
```

### Class<T>를 통해 할 수 있는 것
1. 생성자 가져오기
2. 메서드 (목록) 가져오기
3. 필드 (목록) 가져오기
4. 인터페이스 (목록) 가져오기
5. 상위 클래스 가져오기
6. 애노테이션 가져오기
...  

이처럼 리플렉션을 사용하면 클래스의 정보를 참조할 수 있고, 객체를 만들거나 메서드를 실행시킬 수도 있다.  
```java
//생성자 가져오기
Arrays.stream(class1.getDeclaredConstructors()).forEach(System.out::println);

//메서드 가져오기
Arrays.stream(class2.getDeclaredMethods()).forEach(System.out::println);

//필드 가져오기
Arrays.stream(class3.getDeclaredFields()).forEach(System.out::println);

//인터페이스 가져오기
Arrays.stream(class1.getInterfaces()).forEach(System.out::println);

//상위 클래스 가져오기
System.out.println(class2.getSuperclass());

//애노테이션 가져오기
//getAnnotations(): 상속받은 (@Inherit) 애노테이션까지 조회
//getDeclaredAnnotations(): 자기 자신에만 붙어있는 애노테이션 조회
Arrays.stream(class3.getDeclaredAnnotations()).forEach(System.out::println);

//객체 만들기
Constructor<?> constructor = class1.getConstructor(String.class,String.class,String.class);
Child child = (Child) constructor.newInstance("A", "B", "C");

//static 필드값 가져오기
//static으로 만들어진 필드의 경우 get 함수에 null을 줘서 값을 가져올 수 있습니다.
Field field1 = class1.getDeclaredField("B");
field1.setAccessible(true);//private 필드에 접근하기 위해서는 true를 하여 가져옵니다.
System.out.println(field1.get(null));

//일반 필드값 가져오기
//set함수와 get함수를 사용할 때 반드시 객체를 넣어줍니다.
Field field2 = class1.getDeclaredField("D");
System.out.println(field2.get(child));
field2.set(child,"D");
System.out.println(field2.get(child));

//메서드 실행하기
Method method = Child.class.getDeclaredMethod("sum", int.class, int.class);
int sum = (int) method.invoke(child,1, 2);
System.out.println(sum);
```
<br>

### 실습
이제 리플렉션을 사용하는 방법을 배웠으니 실습을 해보자.  
실습할 내용은 스프링에서 @Autowired 애노테이션 사용하면 빈을 주입받는 것처럼 새로운 애노테이션을 만들어 필드에 그 애노테이션을 사용하면 객체가 만들어질 수 있도록 구현해보겠다.  
```java
@Retention(RetentionPolicy.RUNTIME) //런타임에서 참조할 수 있도록 설정
@Target(ElementType.FIELD) // 필드에서만 사용할 수 있도록 설정
public @interface Inject {
}
```
런타임 때 참조할 수 있도록 하며 필드 외 사용할 수 없는 애노테이션을 만들었다.  

```java
public class ContainerService {

    public static <T> T getObject(Class<T> classType) {
    	// 인스턴스 생성 
        T instance = createInstance(classType); 
        //인스턴스의 모든 필드 확인
        Arrays.stream(classType.getDeclaredFields()).forEach(f->{
           //필드에 Inject 애노테이션이 적용되었는지 검사
            if(f.getAnnotation(Inject.class) != null){
                //Inject 애노테이션이 있다면 그 필드에 대한 인스턴스 생성
                Object filedInstance = createInstance(f.getType());
                f.setAccessible(true);
                try {
                    //필드에 생성한 인스턴스로 초기화
                    f.set(instance, filedInstance);
                } catch (IllegalAccessException e) {
                    throw new RuntimeException(e);
                }
            }
        });

        return instance;
    }

    //리플렉션을 이용하여 인스턴스 생성
    public static <T> T createInstance(Class<T> classType) {
        try {
            return classType.getConstructor(null).newInstance();
        } catch (InstantiationException | IllegalAccessException |InvocationTargetException | NoSuchMethodException e) {
            throw new RuntimeException(e);
        }
    }
}
```
실제 스프링에서 빈 주입하는 과정보다는 많이 부족하지만 getObject() 함수를 사용해서 @Inject를 사용한 필드에도 인스턴스를 생성하여 빈을 주입한 것처럼 사용이 가능해졌다.  
그러면 함수를 사용하기 위해 간단한 클래스를 만들고 그 클래스에 @Inject 애노테이션을 사용해서 동작이 되는지 테스트를 해보겠다.  
먼저 AccountRepository 클래스에서는 save()라는 함수를 만들고, AccountService 클래스에서는 save() 함수를 사용하기 위해 @Inject 애노테이션을 이용해 AccountRepository 클래스의 인스턴스를 생성할 수 있도록 해주었다.  

```java
public class AccountRepository {

    public void save(){
        System.out.println("Save!!");
    }
}
public class AccountService {

    @Inject
    AccountRepository accountRepository;

    public void join(){
        System.out.println("Join!!");
        accountRepository.save();
    }
}
public class App {

    public static void main(String[] args) {
        AccountService service = ContainerService.getObject(AccountService.class);
        service.join();
    }
}
```

**실행 결과**  
```java
Join!!
Save!!
```
getObject() 함수를 통해 인스턴스를 생성 후 join() 함수를 실행시켜 정상 작동되는 걸 확인해보았다.  
이번 시간에는 강의의 예제를 통해 실습하였지만, 다음번에는 회사 프로그램에 적용된 사례를 통해 실습을 해볼 수 있도록 많은 연습을 해봐야겠다.  
<br>

### 리플렉션 사용 시 주의할 점
1. 지나친 사용은 성능 이슈를 야기할 수 있으니 반드시 필요한 경우에만 사용할 것을 권장한다.  
예) 리플렉션 사용으로 호출한 메서드는 일반적으로 호출한 메서드보다 느리다.  
2. 컴파일 타임에 확인되지 않고 런타임 시에만 발생하는 문제를 만들 가능성이 있다.  
3. 리플렉션을 사용하게 되면 소스 코드가 복잡해지므로 관리하기 어려울 수있다.  
4. 접근 지정자를 무시할 수 있으니 사용에 유의해야 한다.  






















