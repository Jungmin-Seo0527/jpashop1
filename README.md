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

## Note