## Redis를 활용한 분산락 적용 사례
기존 서비스에서는 주문할 때 재고 수량의 검증을 엄격하게 다루지 않았다.  
부족한 재고는 각 입점사에게 연락하여 재고가 있을 경우 **배송 준비 중**으로 주문 상태를 변경하였고, 재고가 없을 경우 **주문 취소**를 하였다.  
이러한 프로세스는 고객으로부터 많은 컴플레인을 받아왔고, MD 파트, CS 파트의 업무 불만도 높은 상태였다.  
위 문제를 해결하기 위해 분산락을 활용하여 주문서 생성 시 재고 검증을 하게 되었고, 재고 관련한 이슈를 해결할 수 있었다.   
구현 코드 : [redis-distributed-lock-test](https://github.com/hoonsmemory/redis-distributed-lock-test)  
<br>

### 해결 방법
**SELECT ... FOR UPDATE 활용**  
```sql
SELECT s.*
  FROM stocks s
 WHERE s.product_id = ?
   FOR UPDATE;
```
SELECT ... FOR UPDATE 은 데이터베이스에서 특정 행을 읽으면서 동시에 배타적 락을 설정하는 SQL 구문이다.  
이 구문은 테이블 전체가 아닌, WHERE 조건에 따라 반환된 **특정 행만 락의 대상**이 된다.  
오라클 기준 기본 옵션은 락을 획득하지 못 한 커넥션은 락을 점유중인 커넥션이 트랜잭션을 종료할 때까지 대기한다.  
WAIT 혹은 NOWAIT 옵션을 적용해 락을 일정시간동만 기다리게 처리할 수도 있다.  
<br>

**Redis 활용**  
```sql
SET key value NX EX seconds
```
Redis의 SET 명령어는 NX와 EX 옵션을 함께 사용하여 분산락을 구현할 수 있는 강력한 방법을 제공한다.  
NX 옵션은 키가 존재하지 않을 때만 값을 설정하도록 보장하며, EX 옵션은 TTL값을 설정하여 락의 유효 기간을 제한한다.  

그 외에 다양한 방법들이 있지만 장단점을 따져보고 최종적으로 Redis를 사용하기로 결정했다.  
<br>

### Redisson 사용  
```java
<T> RFuture<T> tryLockInnerAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    return evalWriteSyncedNoRetryAsync(getRawName(), LongCodec.INSTANCE, command,
            "if ((redis.call('exists', KEYS[1]) == 0) " +                             // 키가 존재하지 않을 경우 (락이 없는 상태)
                        "or (redis.call('hexists', KEYS[1], ARGV[2]) == 1)) then " +  // 락 키가 존재하지만 동일 스레드가 소유한 경우 (재진입 가능 상태)
                    "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +                  // 스레드별 락 카운터를 증가시킴 (첫 락이거나 재진입한 경우)
                    "redis.call('pexpire', KEYS[1], ARGV[1]); " +                     // 락 키의 TTL(만료 시간)을 설정
                    "return nil; " +                                                  // 락 획득 성공 시 'nil' 반환
                "end; " +                                                             // 위 조건에 해당하지 않으면 락 획득 실패
                "return redis.call('pttl', KEYS[1]);",                                // 실패 시 TTL 반환
            Collections.singletonList(getRawName()), unit.toMillis(leaseTime), getLockName(threadId));
    }
```
Redisson은 스프링 환경에서 분산락을 사용할 수 있도록 구현된 클라이언트다.  
코드를 직접 구현해서 적용해도 되지만, 꾸준히 릴리즈되고 있는 버전과 성능을 최적화하고 있는 Redisson을 사용하기로 결정했다.  

대표 메서드는 tryLock()이 있으며 락을 대기할 시간과 점유할 시간을 입력하면 락을 획득할 수 있다.  
내부 코드에서는 Lua Script가 작성되어 있으며 락 설정 가능 여부 확인, 락 설정 또는 재진입 처리, 락 획득 실패 시 TTL 반환의 로직을 담고 있다.  
<br>

### 주의사항
**트랜잭션 범위**  
```java
@Transactional
public OrderResponse createOrder(OrderCreateServiceRequest request, LocalDateTime registeredDateTime) {
    List<ProductPurchase> purchases = request.getProductPurchases();

    // 1. 구매 가능한 상품 조회
    List<Product> products = productService.findAllByIdInAndStatus(purchases, ProductStatus.AVAILABLE);

    // 2. 재고 차감
    purchases.stream()
             .forEach(this::deductStockQuantitiesWithLock);

    // 3. 주문
    Order order = Order.create(products, purchases, registeredDateTime);
    Order savedOrder = orderRepository.save(order);

    return OrderResponse.of(savedOrder);
}

private void deductStockQuantitiesWithLock(ProductPurchase purchase) {
    String stockCheckLockId = "stock_check_lock_" + purchase.getProductId();
    RLock stockCheckLock = redissonClient.getLock(stockCheckLockId);
    try {
        if (stockCheckLock.tryLock(20, 10, TimeUnit.SECONDS)) {
            stockService.deductStockQuantities(purchase.getProductId(), purchase.getQuantity());
        } else {
            log.info("{} 락을 획득하지 못 했습니다.", stockCheckLockId);
        }
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    } finally {
        stockCheckLock.unlock();
    }
}
```
Redisson을 활용해 손 쉽게 분산락을 구현할 수 있었지만, 코드 레벨에서의 트랜잭션 범위에 따라 Data Race가 발생할 수 있다.  
그 이유는 @Transactional이 적용된 메서드가 종료 시점에 커밋을 하게 되는데, 그 전에 다른 스레드에서 커밋 전의 데이터를 조회하기 때문이다.  
<br>

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void deductStockQuantities(long productId, int quantity) {
    // 1. 재고 데이터 조회
    Stock stock = stockRepository.findByProductId(productId);

    // 2. 재고 차감
    stock.deductQuantity(quantity);
}
```
따라서, 내가 선택한 방법은 재고 조회와 재고 차감 부분만 새로운 트랜잭션으로 관리하였고, 락을 반환하기 전 트랜잭션을 커밋할 수 있도록 하였다.  














