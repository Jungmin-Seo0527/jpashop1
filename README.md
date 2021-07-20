# JPA 기본편

## 9. 객체지향 쿼리 언어(JPQL)

### 9-1. 객체지향 쿼리 언어 소개

* **JPQL**
* JPA Criteria
* **QueryDSL**
* 네이티브 SQL
* JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

#### JPQL

* 가장 단순한 조회 방법
    * `EntityManager.find()`
    * 객체 그래프 탐색(`a.getB().getC())`
* **나이가 18살 이상인 회원을 모두 검색하고 싶다면?**
* JPA를 사용하면 엔티티 객체를 중심으로 개발
* 문제는 검색 쿼리
* 검색을 할 때도 **테이블이 아닌 엔티티 객체를 대상으로 검색**
* 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
* 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요
* JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
* SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
* JPQL은 엔티티 객체를 대상으로 쿼리
* SQL은 데이터베이스 테이블을 대상으로 쿼리

```
String jpql = "select m From Member m where m.name like '%hello%'";
List<Member> result = em.createQuery(jpql, Member.class)
      .getResultList();
```

* 테이블이 아닌 객체를 대상으로 검색하는 개체 지향 쿼리
* SQL을 추상화해서 특정 데이터베이스 SQL에 의존 X
* JPQL을 한마디로 정의하면 객체 지향 SQL

```
String jpql = "select m From Member m where m.name like '%hello%'";
List<Member> result = em.createQuery(jpql, Member.class)
      .getResultList();
```

```roomsql
select
    m.id as id, 
    m.age as age,
    m.USERNAME as USERNAME,
    m.TEAM_ID as TEAM_ID
from
    Member m
where
    m.age > 18
```

#### Criteria

```java
public class Main {
    public static void main(String[] args) {

        // Criteria 사용 준비
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Member> query = cb.createQuery(Member.class);

        // 루트 클래스 (조회를 시작할 클래스)
        Root<Member> m = query.from(Member.class);

        // 쿼리 생성
        CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
        List<Member> resultList = em.createQuery(cq).getResultList();
    }
}
```

* 문자가 아닌 자바코드로 JPQL을 작성할 수 있음
* JPQL 빌더 역활
* JPA 공식 기능
* **단점: 너무 복잡하고 실용성이 없다.**
* Criteria 대신에 **QueryDSL 사용 권장**

#### QueryDSL

```java
public class Main {
    public static void main(String[] args) {
        JPAFactoryQuery query = new JPAQueryFactory(em);
        QMember m = QMember.member;

        List<Member> list =
                query.selectFrom(m)
                        .where(m.age.gt(18))
                        .orderBy(m.name.desc())
                        .fetch();
    }
}
```

* 문자가 아닌 자바코드로 JPQL을 작성할 수 있음
* JPQL 빌더 역할
* 컴파일 시점에 문법 오류를 찾을 수 있음
* 동적 쿼리 작성 편리함
* **단순하고 쉬움**
* **실무 사용 권장**

#### 네이티브 SQL

* JPA가 제공하는 SQL을 직접 사용하는 기능
* JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
* 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

```java
public class Main {
    public static void main(String[] args) {
        String sql =
                "SELECT ID, AGDE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'";
        list<Member> resultList =
                em.createNativeQuery(sql, Member.class).getResultList();
    }
}

```

#### JDBC 직접 사용, SpringJdbcTemplate 등

* JPA를 사요하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스등을 함께 사용 가능
* 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시 필요
* 예) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트 수동 플러시

### 9-2. JPQL - 기본 문법과 기능

#### JPQL 소개

* JPQL은 객체지향 쿼리 언어다. 따라서 테이블을 대상으로 쿼리하는 것이 아니라 **엔티티 객체를 대상으로 쿼리**한다.
* JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
* JPQL은 결국 SQL로 변환된다.

#### JPQL 문법

```
select_문 :: =
  select_절
  from_절
  [where_절]
  [having_절]
  [orderby_절]
  
update_문 :: = update_절 [where_절]
delete_문 :: = delete_절 [where_절]
```

* select m from **Member** as m where m.age > 18
* 엔티티와 속성은 대소문자 구문O (Member, age)
* JPQL 키워드는 대소문자 구문X (SELECT, FROM, where)
* 엔티티 이름 사용, 테이블 이름이 아님(Member)
* **별칭은 필수(m)** (as는 생략 가능)

#### 집합과 정렬

```
select
  COUNT(m),
  SUM(m.age),
  AVG(m.age),
  MAX(m.age),
  MIN(m.age)
from Member m
```

* GROUP BY, HAVING
* ORDER BY

#### TypeQuery, Query

* TypeQuery: 반환 타입이 명확할 때 사용
* Query: 반환 타입이 명확하지 않을 때 사용

```
TypeQuery<Member> query = 
      em.createQuery("SELECT m FROM Member m", Member.class);
```

```
Query query = 
      em.createQuery("SELECT m.username, m.age, from Member m");
```

#### 결과 조회 API

* `query.getResultList()`: **결과가 하나 이상일 때** 리스트로 반환
    * 결과가 없으면 빈 리스트 반환
* `query.getSingleResult()`: **결과가 정확히 하나** 단일 객체 반환
    * 결과가 없으면: `javax.persistence.NoResultException`
    * 결과가 둘 이상이면: `javax.persistence.NonUniqueResultException`

#### 파라미터 바인딩 - 이름 기준, 위치 기준

```
SELECT m FROM Member m where m.username =: username
query.setParameter("username", usernameParam);
```

```
SELECT m FROM Member m where m.username=?1
query.setParameter(1, usernameParam);
```

* 위치 기준 보다는 이름을 기준으로 바인딩을 하는것이 좋다.

### 9-3. 프로젝션

* SELECT 절에 조회할 대상을 지정하는 것
* 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입)
* SELECT m FROM Member m -> 엔티티 프로젝션
* SELECT m.team FROM Member m -> 엔티티 프로젝션
* SELECT m.address FROM Member m -> 임베디드 타입 프로젝션
* SELECT m.username, m.age FROM Member m -> 스칼라 타입 프로젝션
* DISTINCT로 중복 제거

#### 프로젝션 - 여러 값 조회

* SELECT m.username, m.age FROM Member m
* Query 타입으로 조회
* Object[] 타입으로 조회
* new 명령어로 조회
    * 단순 값을 DTO로 바로 조회
        * SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m
        * 패키지 명을 포함한 전체 클래스 명 입력
        * 순서와 타입이 일치하는 생성자 필요

### 9-4. 페이징 API

* JPA는 페이징을 다음 두 API로 추상화
* `setFirstResult(int startPosition)`: 조회 시작 위치 (0부터 시작)
* `setMaxResults(int maxResult)`: 조회할 데이터 수

```
String jpql = "select m from Member m order by m.name desc";
List<Member> resultList = em.createQuery(jpql., Member.class);
        .setFirstResult(10)
        .setMaxResult(20)
        .getResultList();
```

## Note