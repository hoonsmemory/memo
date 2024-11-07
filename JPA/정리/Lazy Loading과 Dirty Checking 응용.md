## Lazy Loading과 Dirty Checking 응용
Lazy Loading(지연 로딩)은 연관관계가 매핑된 엔티티를 조회했을 때 조회한 해당 엔티티의 데이터만 가져오는 걸 뜻한다.  
조회하지 않은 엔티티는 get 메서드 호출 시 영속성 컨텍스트를 비교 후 값이 없다고 판단하면 그때서야 데이터를 가져온다.  
Dirty Checking(변경 감지)은 영속성 컨텍스트에서 관리하는 엔티티의 값이 변경되었을 경우 1차 캐시 안의 스냅샷과 비교 후 변경을 감지하고, Update 쿼리를 만들어 트랜잭션 커밋 시점에 update 처리를 해준다.  

<br>

### 엔티티 구현
<img width="875" alt="스크린샷 2024-11-07 오후 12 47 45" src="https://github.com/user-attachments/assets/77bb1730-c275-4174-a87e-d4b014edd2ab">

각 엔티티 간의 간략한 관계도를 그려봤다.  
Order를 중심으로 Delivery와 1:1 양방향 관계, OrderItem과는 1:N 양방향 관계가 맺어있고 OrderItem은 Item과 N:1 단방향 관계를 맺고 있다.  

```java
@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;
}


@Entity
@Getter @Setter
public class Delivery {

    @Id @GeneratedValue
    @Column(name = "delivery_id")
    private Long id;

    @OneToOne(mappedBy = "delivery", fetch = LAZY)
    private Order order;
}


@Entity
@Table(name = "order_item")
@Getter @Setter
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;
}


@Entity
@Getter @Setter
public class Item {

    @Id
    @GeneratedValue
    @Column(name = "")
    private Long id;
}
```
그림을 토대로 연관관계를 맺은 엔티티를 구현하였고, 모든 @xToOne 연관관계에도 지연 로딩할 수 있도록 LazyFetch 옵션을 주었다.  

<br>

### 예제 코드
```java
//주문 취소 메서드
@Transactional
public void cancelOrder(Long orderId) {
    //1. order 조회(영속성 컨텍스트에 보관)
    Order order = orderRepository.findOne(orderId);

    //2. deilvery 조회(Lazy Loading)
    if(order.getDelivery().getStatus() == DeliveryStatus.COMP) {
        throw new IllegalStateException("이미 배송완료된 상품은 취소가 불가능합니다.");
    }
    
    //3. order 변경(Dirdy Checking)
    order.setStatus(OrderStatus.CANCEL);

    //4. orderItem(Lazy Loading)
    for (OrderItem orderItem: order.getOrderItems()) {
        //5. item 조회(Lazy Loading)
        Item item = orderItem.getItem();
        
        //6. item 변경(Dirdy Checking)
        item.setStockQuantity(item.getStockQuantity() + orderItem.getCount());
    }
}
```
위의 메서드는 주문을 취소하는 메서드다.  
코드만 보면 분명 더 좋은 코드로 구현할 수 있겠지만, JPA를 처음 배우는 사람에게 JPA의 장점을 설명하기 위한 김영한 님의 노력에 감탄할 수밖에 없었다.  

메서드 순서는 이렇다.  
1. 주문을 취소하고자 하는 Order의 데이터를 조회한다. (조회한 데이터는 영속성 컨텍스트에 보관된다.)
2. Order와 1:1 매핑된 Delivery의 상태를 가져온다. (지연 로딩으로 get 메서드 호출 시점에 Delivery의 데이터를 조회한다.)
3. Delivery의 상태가 배송이 완료되지 않았다면 Order의 주문 상태를 취소로 변경한다. (변경 감지로 상태 데이터를 업데이트한다.)
4. 주문한 수량을 가져오기 위해 OrderItem을 조회한다. (지연 로딩으로 get 메서드 호출 시점에 OrderItem의 데이터를 조회한다.)
5. 주문 취소 전의 총수량을 가져오기 위해 Item을 조회한다.(지연 로딩으로 get 메서드 호출 시점에 Item의 데이터를 조회한다.)
6. 취소한 수량을 더하기 위해 기존 수량과 취소한 수량을 더하여 총수량을 변경한다. (변경 감지로 수량 데이터를 업데이트한다.)
 
직접 사용한 쿼리는 한 번이지만 지연 로딩과 변경 감지를 통해  JPA가 자동으로 업데이트, 조회 쿼리를 만들어 DB에 반영해주었다.  
사실 한 번 사용한 쿼리마저 JpaRepository 를 위임받아 사용했다면 쿼리 작성 없이 메서드를 구현할 수 있었을 것이다.  

간단한 CRUD 쿼리라도 직접 하나하나 구현하였고, 테이블 설계를 변경하는 날에는 쿼리도 일일이 수정해야 해서 정말 정말 번거로웠는데 JPA를 쓴다면 이런 문제를 해결할 수 있다.  














