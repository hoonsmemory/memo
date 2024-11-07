## JPQL
JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공한다.  
JPQL은 객체지향 쿼리 언어다 .따라서 테이블을 대상으로 쿼리 하는 것이 아니라 엔티티 객체를 대상으로 쿼리한다.  
SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는 장점이 있다. JPQL은 엔티티 객체를 대상으로 쿼리, SQL은 데이터베이스 테이블을 대상으로 쿼리  

**JPQL 특징**  
* 엔티티와 속성은 대소문자 구분이 필요하다. (Member, age)  select m from Member as m where m.age > 18
* JPQL 키워드는 대소문자 구분하지 않는다. (SELECT, FROM, where)
* 엔티티 이름 사용, 테이블 이름이 아님(Member)
* 별칭은 필수(m) (as는 생략가능)
* 집합과 정렬 지원
* TypeQuery : 반환 타입이 명확할 때 사용
* Query : 반환 타입이 명확하지 않을 때 사용

**[JPQL과 1차 캐시 질문]**
https://chatgpt.com/share/671aecce-0f10-800e-967a-b6e85f5efae4

<br>

### JPQL 문법
**[프로젝션]**
SELECT 절에 조회할 대상을 지정하는 것  
  
프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입)  
* SELECT m FROM Member m -> 엔티티 프로젝션  
* SELECT m.team FROM Member m -> 엔티티 프로젝션
* SELECT m.address FROM Member m -> 임베디드 타입 프로젝션
* SELECT m.username, m.age FROM Member m -> 스칼라 타입 프로젝션 DISTINCT로 중복 제거

**여러 값 조회 시**
SELECT m.username, m.age FROM Member m  
1. Query 타입으로 조회 
2. new 명령어로 조회(SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m)
<br>

**Fetch Join vs 프로젝션의 차이점**

<img width="377" alt="스크린샷 2024-11-07 오전 10 01 57" src="https://github.com/user-attachments/assets/d790b64d-a927-4eee-838b-00d6e85fed38">

<br>
<br>

### 페이징
JPA는 페이징을 다음 두 API로 추상화  
setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작)  
setMaxResults(int maxResult) : 조회할 데이터 수  
페치 조인 시 페이징하기 어렵다.  

<br>

### 조인
내부 조인 : SELECT m FROM Member m [INNER] JOIN m.team t  
외부 조인 : SELECT m FROM Member m LEFT [OUTER] JOIN m.team t  
세타 조인 : select count(m) from Member m, Team t where m.username = t.name  

조인 - ON 절  
ON절을 활용한 조인(JPA 2.1부터 지원) 
1.조인대상필터링  
2. 연관관계 없는 엔티티 외부 조인(하이버네이트 5.1부터)  

예) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인  
JPQL : SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A'  
SQL : SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and t.name='A'  

<br>

### 서브 쿼리
얘시 : 나이가 평균보다 많은 회원  
select m from Member m where m.age > (select avg(m2.age) from Member m2)  

* [NOT] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참 
    * {ALL | ANY | SOME} (subquery)
* ALL 모두 만족하면 참 
    * ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
* [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참 

팀A 소속인 회원  
select m from Member m where exists (select t from m.team t where t.name = ‘팀A') 

**JPA 서브쿼리 한계**
JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능하다. (하이버네이트 같은 경우 SELECT 절도 가능)  
FROM절의 서브 쿼리는 현재 JPQL에서 불가능하다..  
[하이버네이트에서 from절의 서브 쿼리 적용 지원](https://in.relation.to/2022/06/14/orm-61-final/)

<br>

### 타입 표현과 기타식
* 문자: ‘HELLO’, ‘She’’s’ Boolean: TRUE, FALSE
* 숫자: 10L(Long), 10D(Double), 10F(Float)
* ENUM: jpabook.MemberType.Admin (패키지명 포함)
* 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)
* SQL과 문법이 같은 식
* EXISTS, IN
* AND, OR, NOT
* =, >, >=, <, <=, <> BETWEEN, LIKE, IS NULL

<br>

### 조건식(CASE 등)
기본 CASE 식  
``` java
select  
case when m.age <= 10 then '학생요금' 
          when m.age >= 60 then '경로요금'
           else '일반요금' 
end 
from Member m 
```

<br>

### 경로 표현식
경로 표현식 특징  
* 상태 필드(state field): 경로 탐색의 끝, 탐색X
* 단일 값 연관 경로: 묵시적 내부 조인(inner join) 발생, 탐색O --> select m.team from Member m; team을 조회하기 위해 join 발생
* 컬렉션 값 연관 경로: 묵시적 내부 조인 발생, 탐색X --> FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능 

select o.member.team from Order o --> 성공  
select t.members from Team t --> 성공  
select t.members.username from Team t --> 실패  
select m.username from Team t join t.mebers m --> 성공 

**실무 조언**   
묵시적 내부 조인이 발생하면 성능적 문제가 발생할 수 있으니, 가급적 묵시적 조인 대신에 명시적 조인 사용  
조인은 SQL 튜닝에 중요 포인트 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어려움  







