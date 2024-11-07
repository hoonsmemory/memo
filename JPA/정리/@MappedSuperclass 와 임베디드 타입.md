## @MappedSuperclass 와 임베디드 타입
@MappedSuperclass 와 임베디드 타입은 여러 엔티티에서 공통으로 사용되는 필드를 별도의 클래스로 분리하여 부모 엔티티에 적용하는 방식이다. 하지만 @MappedSuperclass 의 경우 상속을 받아 적용하고, 임베디드 타입의 경우 위임을 받아 적용하므로 서로 다른 상황에서 쓰일 수 있다.  

### @MappedSuperclass 사용 예시
```java
DataInfo.java

@MappedSuperclass
public abstract class DataInfo {
    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}
 

Member.java

@Entity
public class Member extends DataInfo
```
@MappedSuperclass 의 경우 엔티티가 상속하면 적용된다.  
하지만 @Inheritance(strategy=InheritanceType.XXX) 를 적용한 것처럼 상속관계 매핑이 아니며, 상속받았다고 해서 DataInfo 테이블이 따로 생성되지 않고 Member 엔티티의 필드로 사용이 된다.  

@MappedSuperclass 는 단순 반복되는 필드를 적용할 때 사용하는 게 좋다.  
예를 들면 등록자, 등록일, 수정자, 수정일과 같은 필드는 어느 엔티티에나 필요로 하지만 비즈니스적인 필드가 아니기에 사용하기 적합하다.  

@MappedSuperclass 를 적용한 클래스는 직접 생성해서 사용할 일이 없으므로 추상 클래스로 만드는 관례가 있으며, 엔티티 클래스는 다른 @Entity가 지정된 클래스와 @MappedSuperclass로 지정한 클래스만 상속 가능한 특징이 있다.  

<br>

### 임베디드 타입 사용 예시
```java
Address.java

@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;
}
 

member.java

@Entity
public class Member extends DataInfo {
    @Embedded
    private Address homeAddress;
    
    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name = "city", column = @Column(name="WOKR_CITY")),
            @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
            @AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE"))
    })
    private Address workAddress;
}
```
만약 멤버 엔티티는 집주소와 회사주소 즉 두 개의 공통된 필드인 주소를 가져야 할 경우 임베디드 타입으로 만들어서 사용할 수 있다.  
물론 멤버 엔티티가 주소를 하나만 가지더라도 다른 엔티티에서 사용할 경우 재사용성을 위해 임베디드 타입으로 구현할 수도 있다.  
임베디드 타입의 경우 상속이 아닌 위임을 받으므로 위의 멤버 엔티티처럼 여러 번 적용할 수 있는 게 가장 큰 장점이다.  

<br>

**하지만, 임베디드 타입이 꼭 좋은 것만은 아니다.**  
```java
class DataInfo {
    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}
```
위 클래스를 임베디드 타입으로 만들고 JPQL 쿼리를 할 경우 아래와 같이 필드 앞에 항상 dataInfo를 붙여야 한다.  
select m from Member m where m.dataInfo.createDate > ?  

하지만 상속을 할 경우 아래와 같이 편리한 장점이 있다.  
select m from Member m where m.createDate > ?  

그리고 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 사이드 이펙트가 생길 수 있다.  
```java
Address address = new Address("city", "street", "zip");

Member member1 = new Member();
member1.setName("member1");
member1.setHomeAddress(address);
em.persist(member1);

Member member2 = new Member();
member2.setName("member2");
member2.setHomeAddress(address);
em.persist(member2);

member1.getHomeAddress().setCity("new city");
```
위처럼 집주소에 동일한 인스턴스를 넣고 member1을 변경했다면 의도치 않게 member2의 주소도 변경되는 문제가 발생하므로 임베디드 타입의 경우 설계에도 신경 써야 하는 단점이 있다.  

<br>

### 결론
내 기준에서는 비즈니스적으로 관계가 없으면서 많은 엔티티에 적용해야 할 경우 @MappedSuperclass 를 선택하고, 조금 복잡한 게 있지만, 비즈니스적으로 관계가 있으면 임베디드 타입으로 분리해서 사용하는 것이 좋다고 생각한다. 
