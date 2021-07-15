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

### 4-3. 양방향 연관관계와 연관관계의 주인1 - 기본

#### 양방향 매핑

![](https://i.ibb.co/yn8FS3r/bandicam-2021-07-15-15-33-15-447.jpg)

##### Member.java

* 코드는 단방향과 동일하다.
* 문제는 `Team`엔티티에서 연관관계에 있는 `Member`엔티티 정보의 유무이다.

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

##### Team.java - 컬렉션 추가

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToMany;

@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    List<Member> members = new ArrayList<>();
}
```

##### Main.java - 양방향 조회

```java
public class Main {
    public static void main(String[] args) {

        // 조회
        Team findTeam = em.find(Team.class, team.getId());
        int memberSize = findTeam.getMembers().size()
    }
}
```

#### 연관관계의 주인과 mappedBy

* `mappedBy`: JPA 멘탈붕괴 난이도
* `mappedBy`는 처음에는 이해하기 어렵다.
* 객체와 테이블간에 연관관계를 맺는 차이를 이해해야 한다.

##### 객체와 테이블이 관계를 맺는 차이

* **객체 연관관계 = 2개**
    * 회원 -> 팀 연관관계 1개(단방향)
    * 팀 -> 회원 연관관계 1개(단방향)

* **테이블 연관관계 = 1개**
    * 회원 < - > 팀의 연관과계 1개(양방향)

* 객체의 **양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개이다.**
* 객체를 양방향으로 참조하려면 **단방향 연관관계를 2개**만들어야 한다.
* 테이블은 **외래키 하나**로 두 테이블의 연관관계를 관리한다.

#### 연관관계의 주인(Owner)

* 양방향 매핑 규칙
    * 객체의 두 관계중 하나를 연관관계의 주인으로 지정
    * **연관관계의 주인만이 외래 키를 관리(등록, 수정)**
    * **주인이 아닌쪽은 읽기만 가능**
    * 주인은 mappedBy 속성 사용X
    * 주인이 아니면 mappedBy 속성으로 주인 지정

* 누구를 주인으로 하는가?
    * 외래 키가 있는 곳을 주인으로 정해라
    * 여기서는 `Member.team`이 연관관계의 주인

> MN    
> 간단하게 생각하면 된다. 결국 서로 같은 개념을 매핑하는 것이다!!   
> 일대다 관계에서 관계형 DB는 `일`에 해당하는 테이블에서 `다`의 기본키를 외래키로 가진다. 따라서 `일`의 외래키와 `다`의 기본키가 같은 컬럼을 조인해서 서로 일치하는 컬럼들을 조회할 수 있다.
>
> 이전에는 `일`의 외래키에 해당하는 필드를 `teamId`로 지정했다. 이러다 보니 조회를 할 때 `teamId`로 `Team`엔티티를 다시 조회해야 했다. `Member`엔티티 자체에서 `Team`객체가 있으면 좋겠다는 생각이 들어서 외래키로 `teamId`대신에 `Team`객체 통체로 주입했다.
>
> 여기까지 오면 `Member`엔티티 입장에서는 `Team`엔티티가 주입되어 있으므로 바로 조회가 가능하다. 하지만 양방향 관계인 경우 `Team`엔티티 입장에서는 `Member`에 대한 정보가 전혀 없다. 따라서 컬렉션으로 `Member` 엔티티를 저장한다. 여기서 컬렉션으로 여러 `Member`엔티티를 저장하는 이유는 `Team`엔티티가 `다`이기 때문이다.(선수는 하나의 팀에만 속할 수 있고, 팀은 여러 선수로 이루어져 있다.)     
> 여기서 끝나지 않고 `Team`입장에서는 `Member`객체의 어떤 필드로 자신과의 연관관계를 정의할 수 있는지 알고 있어야 한다. 즉 자신과 관계를 가지는 `Member`엔티티에서 외래키가 무엇인지 알고 있어야 한다. 그래서 `mappedBy`를 이용해서 `Member`엔티티의 `team`필드가 외래키임을 명시해 준다.
>
> 그리고 연관관계 주인은 외래키를 가지고 있는 엔티티를 주인으로 설정한다. 결국 관계의 핵심은 외래키이다. 그리고 그 핵심을 가지고 있는 엔티티를 연관관계의 주인으로 알고 있으면 된다. (아마 연관관계 주인 메소드에서 연관관계 메소드를 생성하겠지...)

### 4-4. 양방향 연관관계와 연관관계의 주인2 - 주의점, 정리

#### 양방향 매핑시 가장 많이 하는 실수

* 연관관계의 주인에 값을 입력하지 않음

```java
public class Main {
    public static void main(String[] args) {
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);

        Member member = new Member();
        member.setName("member1");

        // 역방향(주인이 아닌 방향)만 연관관계 설정
        team.getMembers().add(member);

        em.persist(member);
    }
}
```

* member1 엔티티를 조회해 보면 TEAM_ID 값이 null 이다.

* 양방향 매핑시 연관관계의 주인에 값을 입력해야 한다.
    * 순수한 객체 관계를 고려하면 항상 양쪽다 값을 입력해야 한다.

```java
public class Main {
    public static void main(String[] args) {
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);

        Member member = new Member();
        member.setName("member1");

        team.getMembers().add(member);

        // 연관관계의 주인에 값 설정
        member.setTeam(team);

        em.persist(member);
    }
}
```

#### 양방향 연관관계 주의

* **순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자.**
* 연관관계 편의 메소드를 생성하자.
* 양방향 매핑시에 무한 루프를 조심하자.
    * 예) toString(), lombok, JSON 생성 라이브러리

#### 양방향 매핑 정리

* **단방향 매핑만으로도 이미 연관관계 매핑은 완료**
* 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐
* JPQL에서 역방향으로 탐색할 일이 많음
* 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 됨(테이블에 영향을 주지 않음)
* 비즈니스 로직을 기준으로 연관관계의 주인을 선택하면 안됨
* **연관관계의 주인은 외래 키의 위치를 기주으로 정해야 함**

## Note