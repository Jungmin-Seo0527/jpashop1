# JPA 기본편

## 4. 연관관계 매핑 기초

### 4-1. 실전 예제 1 - 요구사항 분석과 기본 매핑

#### 요구사항 분석

* 회원은 상품을 주문할 수 있다.
* 주문 시 여러 종류의 상품을 선택할 수 있다.

##### 기능 목록

* 회원 기능
    * 회원 등록
    * 회원 조회
* 상품 기능
    * 상품 등록
    * 상품 수정
    * 상품 조회
* 주문 기능
    * 상품 주문
    * 주문 내역 조회
    * 주문 취소

##### 도메인 모델 분석

* **회원과 주문의 관계**: **회원**은 여러 번 **주문**할 수 있다.
* **주문과 상품의 관계**: **주문**할 때 여러 **상품**을 선택할 수 있다.
    * 반대로 같은 **상품**도 여러 번 **주문**될 수 있다.
    * **주문 상품**이라는 모델을 만들어서 다대다 관계를 일대다, 다대일 관계로 풀어냄

![](https://i.ibb.co/Qn9gdfj/bandicam-2021-07-14-19-50-40-610.jpg)

#### 테이블 설계

![](https://i.ibb.co/31sTmC0/bandicam-2021-07-14-19-51-38-796.jpg)

#### 엔티티 설계와 매핑

![](https://i.ibb.co/d6nV3V3/bandicam-2021-07-14-19-52-04-856.jpg)

#### 데이터 중심 설계의 문제점

* 현재 방식은 객체 설계를 테이블 설계에 맞춘 방식
* 테이블의 외래키를 객체에 그대로 가져옴
* 객체 그래프 탐색이 불가능
* 참조가 없으므로 UML도 잘못됨

### 4-2. 단방향 연관관계

#### 예제

* 시나리오
    * 회원과 팀이 있다.
    * 회원은 하나의 팀에만 소속될 수 있다.
    * 회원과 팀은 다대일 관계다.

![](https://i.ibb.co/3y6L13r/bandicam-2021-07-15-15-18-15-922.jpg)

#### 객체를 테이블에 맞추어 모델링 - 참조 대신에 외래 키를 그대로 사용

##### Member.java

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @Column(name = "TEAM_ID")
    private Long teamId;
}
```

##### Team.java

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;
    private String name;
}
```

##### Main.java

* 외래 키 식별자를 직접 다룸

```java
public class Main {
    public static void main(String[] args) {

        // 팀 저장
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);

        // 회원 저장
        Member member = new Member();
        member.setName("member1");
        member.setTeamId(team.getId());
        em.persist(member);

        // 조회
        Member findMember = em.find(Member.class, member.getId());

        // 연관관계가 없음
        Team findTeam = em.find(Team.class, team.getId());
    }
}
```

##### 문제점

* 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.
* **테이블은 외래 키로 조인**을 사용해서 연관된 테이블을 찾는다.
* **객체는 참조**를 사용해서 연관된 객체를 찾는다.
* 테이블과 객체 사이에는 이런 큰 간격이 있다.

#### 단방향 연관관계

##### 객체 지향 모델링 - 객체 연관관계 사용

![](https://i.ibb.co/ftTfhYg/bandicam-2021-07-15-15-25-08-749.jpg)

##### Member.java

* 객체의 참조와 테이블의 외래 키를 매핑

```java
import javax.persistence.*;

@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    // @Column(name = "TEAM_ID")
    // private Long teamId;

    @ManyToOne
    @JoinColumn(name = "TEAD_ID")
    private Team team;
}
```

##### 객체 지향 모델링 - ORM 매핑

![](https://i.ibb.co/xXmVbLT/bandicam-2021-07-15-15-27-15-982.jpg)

##### Main.java - 연관관계 저장

```java
public class Main {
    public static void main(String[] args) {

        // 팀 저장
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);

        // 회원 저장
        Member member = new Member();
        member.setName("member1");
        member.setTeamId(team); // 단방향 연관관계 설정, 참조 저장
        em.persist(member);

        // 조회
        Member findMember = em.find(Member.class, member.getId());

        // 참조를 사용해서 연관관계 조회
        Team findTeam = findMember.getTeam();
    }
}
```

> MN    
> 단순하게 외래키를 엔티티의 필드로 지정하면 외래키에 해당하는 엔티티를 조회할 때 외래키로 엔티티를 조회했다. 하지만 연관관계를 지정해 주니 연관관계의 엔티티를 조회하지 않아도 Member 엔티티 만으로 Team엔티티를 조회할 수 있다.

## Note