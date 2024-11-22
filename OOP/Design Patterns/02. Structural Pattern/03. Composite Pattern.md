## 컴포지트 패턴 (Composite Pattern)
<img width="780" alt="스크린샷 2024-11-22 오후 4 44 44" src="https://github.com/user-attachments/assets/c9abedec-5d4d-47a9-9646-0331283f61a3">

컴포지트 패턴은 객체들을 트리 구조로 구성하여 **부분-전체 계층 구조**를 구현하고, 클라이언트가 단일 객체와 복합 객체를 **동일하게 다룰 수 있도록** 하는 구조 패턴이다. 개별 객체와 복합 객체를 하나의 인터페이스로 처리하여 **재귀적인 합성**을 가능하게 한다.  
<br>

### 사용 목적
1. **부분-전체 계층 표현 :** 객체들을 트리 구조로 구성하여 부분과 전체를 나타낼 수 있다.
2. **일관된 인터페이스 제공 :** 단일 객체와 복합 객체를 동일한 인터페이스로 처리하여 코드의 일관성을 유지한다.
3. **확장성 향상 :** 새로운 종류의 구성 요소를 추가하더라도 클라이언트 코드를 수정할 필요가 없다.
<br>

### 구성 요소
**Component (구성 요소)**  
- 객체들의 인터페이스를 정의한다.
- 공통된 연산이나 기본 동작을 선언한다.

**Leaf (잎 노드)**  
- 자식이 없는 최종 구성 요소이다.
- 실제 작업을 수행하며, 더 이상 하위 요소가 없다.

**Composite (복합 노드)**  
- 자식 구성 요소를 가질 수 있는 노드이다.
- 자식 노드의 추가, 제거, 접근 등의 연산을 구현한다.
- `Component` 인터페이스를 구현하여 클라이언트가 복합 노드와 잎 노드를 동일하게 다룰 수 있게 한다.

**Client (클라이언트)**  
- `Component` 인터페이스를 사용하여 구성 요소들과 상호 작용한다.
- 개별 객체와 복합 객체를 구분하지 않고 동일하게 처리한다.
<br>

### 예제
파일 시스템의 디렉토리와 파일 구조를 예제로 살펴보자.

**Component 인터페이스**  
```java
interface FileSystemComponent {
    void showDetails();
}
```

**Leaf 클래스**  
```java
class File implements FileSystemComponent {
    private String name;

    public File(String name) {
        this.name = name;
    }

    public void showDetails() {
        System.out.println("파일: " + name);
    }
}
```

**Composite 클래스**  
```java
import java.util.ArrayList;
import java.util.List;

class Directory implements FileSystemComponent {
    private String name;
    private List<FileSystemComponent> components = new ArrayList<>();

    public Directory(String name) {
        this.name = name;
    }

    public void addComponent(FileSystemComponent component) {
        components.add(component);
    }

    public void removeComponent(FileSystemComponent component) {
        components.remove(component);
    }

    public void showDetails() {
        System.out.println("디렉토리: " + name);
        for (FileSystemComponent component : components) {
            component.showDetails();
        }
    }
}
```

**클라이언트 코드**  
```java
public class Client {
    public static void main(String[] args) {
        FileSystemComponent file1 = new File("파일1.txt");
        FileSystemComponent file2 = new File("파일2.jpg");

        Directory dir1 = new Directory("문서");
        dir1.addComponent(file1);

        Directory dir2 = new Directory("사진");
        dir2.addComponent(file2);

        Directory rootDir = new Directory("루트");
        rootDir.addComponent(dir1);
        rootDir.addComponent(dir2);

        rootDir.showDetails();
    }
}
```

**실행 결과**  
```
디렉토리: 루트
디렉토리: 문서
파일: 파일1.txt
디렉토리: 사진
파일: 파일2.jpg
```
<br>

### 주의사항
1. **재귀적 합성 구조 설계:** 컴포지트 패턴은 재귀적인 트리 구조를 사용하므로, 순환 참조나 무한 루프에 주의해야 한다.
2. **공통 인터페이스의 설계:** `Component` 인터페이스에 모든 필요한 메서드를 선언해야 하며, 일부 메서드는 잎 노드에서 의미가 없을 수 있다.
3. **성능 고려:** 트리 구조를 순회하거나 연산할 때 성능 이슈가 발생할 수 있으므로 최적화가 필요할 수 있다.

### 장점
1. **단순성 제공:** 클라이언트는 개별 객체와 복합 객체를 구분하지 않고 동일한 방식으로 다룰 수 있다.  
2. **확장성 향상:** 새로운 구성 요소를 추가하더라도 기존 코드를 수정할 필요가 없어 시스템의 확장성이 높아진다.  
3. **객체 구성의 유연성:** 객체들을 자유롭게 조합하여 복잡한 트리 구조를 만들 수 있다.  

### 단점
1. **설계의 복잡성 증가:** 트리 구조와 재귀적인 합성으로 인해 코드가 복잡해질 수 있다.  
2. **타입 안전성 감소:** 모든 구성 요소를 동일한 인터페이스로 처리하므로, 특정 타입의 구성 요소에 대한 제약을 두기 어렵다.  
3. **오버헤드 발생:** 복합 객체의 연산이 잦을 경우 성능 저하가 발생할 수 있다.
