# Spring-JPA-정리

위 코드는 김영한님의 강의 [자바 ORM 표준 JPA 프로그래밍 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)을 바탕으로 작성된 것입니다.
<br>

## JPA 영속성 컨텍스트
[영속성 컨텍스트](https://ip99202.github.io/posts/JPA-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8/)
<br><br>

## JPA 데이터베이스 스키마 자동생성
[스키마 자동생성](https://ip99202.github.io/posts/JPA-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%EC%8A%A4%ED%82%A4%EB%A7%88-%EC%9E%90%EB%8F%99%EC%83%9D%EC%84%B1/)
<br><br>

## 다양한 연관관계 매핑
<img width=500px src="https://user-images.githubusercontent.com/52627952/103472912-4885cb00-4dd6-11eb-9d61-3201c196d530.png">  

주의사항 - 외래키가 있는 쪽을 연관관계의 주인으로 설정  

#### MEMBER와 ORDERS는 일대다 양방향 매핑  

Member.java
```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String name;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}
```

Order.java
```java
@Entity
@Table(name = "ORDERS")
public class Order extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
}
```
<br>

#### DELEVERY와 ORDERS는 일대일 양방향 매핑  

Delevery.java
```java
@Entity
public class Delivery {

    @Id @GeneratedValue
    private Long id;

    private DeliveryStatus status;

    @OneToOne(mappedBy = "delivery")
    private Order order;
}
```

Order.java
```java
@Entity
@Table(name = "ORDERS")
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @OneToOne
    @JoinColumn(name = "DELIVERY_ID")
    private Delivery delivery;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();
}
```
<br>

#### ORDER_ITEM과 ITEM은 다대일 단방향 매핑  

OrderItem.java
```java
@Entity
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "ORDER_ID")
    private Order order;

    @ManyToOne
    @JoinColumn(name = "ITEM_ID")
    private Item item;
}
```

Item.java
```java
public class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    private String name;
    private int price;
    private int stockQuantity;

}
```
<br>

#### CATEGORY와 ITEM은 다대다 양방향 매핑  

Category.java
```java
@Entity
public class Category {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();

    @ManyToMany
    @JoinTable(name = "CATEGORY_ITEM",
            joinColumns = @JoinColumn(name = "CATEGORY_ID"),
            inverseJoinColumns = @JoinColumn(name = "ITEM_ID")
    )
    private List<Item> items = new ArrayList<>();
}
```

Item.java
```java
public class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    private String name;
    private int price;
    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();
}
```

**다대다 관계는 실전에서 쓰이지 않는다**  
다대다 -> 일대다와 다대일로 분리해서 사용!
<br><br>

## 상속관계 매핑

<img width=500px src="https://user-images.githubusercontent.com/52627952/103473627-209a6580-4dde-11eb-8d1c-c74d0f6d696a.png">

### 조인전략(JOINED)
<img width=500px src="https://user-images.githubusercontent.com/52627952/103473630-23955600-4dde-11eb-8d86-42caec8c0183.png">  

테이블로 다 나눠서 저장
저장공간 효율적이지만 조회시 조인을 많이 사용하여 성능저하 

### 단일 테이블 전략(SINGLE_TABLE)
<img width=200px src="https://user-images.githubusercontent.com/52627952/103473730-23498a80-4ddf-11eb-86bc-85fcbc144a67.png">  

하나의 테이블에 다 저장
조회성능이 빠르지만 단일 테이블에 모든걸 저장하므로 테이블이 커질 수 있다.

### 구현 클래스마다 테이블 전략(TABLE_PER_CLASS)
<img width=500px src="https://user-images.githubusercontent.com/52627952/103473745-45dba380-4ddf-11eb-9366-63afa8073c6f.png">  

실전에서 쓰면 안됨
데이터를 찾기위해 모든 테이블을 다 뒤져야 한다.
<br><br>

### ITEM 아래 Album, Book, Movie가 존재
<img width=500px src="https://user-images.githubusercontent.com/52627952/103474754-43327b80-4dea-11eb-8805-7df1d27a2631.png">  

단일 테이블 전략을 이용해 매핑

Item.java
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn // 기본이 DTYPE : 어떤 아이템인지 아이템 명을 저장하는 colum
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    private String name;
    private int price;
    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();
}
```
Item.java를 직접 이용하지 않고 그 아래 Album, Book, Movie의 뼈대가 되는 것이기 때문에  
추상 클래스로 만들어준다.


Album.java 나머지도 동일
```java
@Entity
public class Album extends Item {

    private String artist;
    private String etc;

    }
}
```
<br><br>

## MappedSuperclass - 매핑 정보 상속

예를 들어 모든 테이블에 생성시간과 수정시간이 들어가야 한다면  
모든 클래스를 다 수정하는 것이 아니라 공통으로 들어갈 정보의 클래스만 만들고  
해당 클래스를 상속 받게 만들면 정보가 공통으로 들어가게 된다.  

BaseEntitiy.java
```java
@MappedSuperclass
public abstract class BaseEntity {

    private String createBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;

}
```

Item.java
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn // 기본이 DTYPE : 어떤 아이템인지 아이템 명을 저장하는 colum
public abstract class Item extends BaseEntity { // BaseEntity를 상속하여 생성시간 수정시간 만듦

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    private String name;
    private int price;
    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();
}
```
<br><br>

## 즉시 로딩과 지연 로딩
JPA에서 테이블 간 연관 관계는 객체의 참조에 의해 이루어진다.  
서비스가 커지고 복잡해질수록 참조하는 객체가 많아지는데  
이렇게 객체가 커지면 DB에서 연관된 객체까지 한꺼번에 가져오는 것은 엄청난 부담이된다.  
그렇기 때문에 JPA는 참조하는 객체들의 데이터를 가져오는 시점을 정할 수 있다.  
이것이 Fetch Type이다.  

Fetch Type에는 즉시 로딩인 EAGER와 지연로딩인 LAZY가 있다.  

<img width=500px src="https://user-images.githubusercontent.com/52627952/103505290-391d8500-4e9d-11eb-863f-f6ecb3844990.png">  

위의 예시에서 지연 로딩을 사용하면 member를 조회할 때 team에는 실제 값이 아닌 프록시가 들어가고  
team을 조회할 때 다시 쿼리를 날려 team을 가져오게 된다.  
<br>

<img width=500px src="https://user-images.githubusercontent.com/52627952/103505230-1be8b680-4e9d-11eb-8542-023b61bd18a2.png">  

즉시 로딩을 사용하게 되면 member를 조회할 때에도 join을 사용하여 team을 같이 가져오게 된다.  

**실전에서는 가급적 지연로딩만 사용한다.**  
**즉시 로딩은 JPQL에서 N+1문제가 발생한다.**  
@ManyToOne과 @OneToOne은 기본이 즉시 로딩이다.  
@OneToMany와 @ManyToMany는 기본이 지연 로딩이다.  


Category.java
```java
@Entity
public class Category extends BaseEntity {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY) //***
    @JoinColumn(name = "PARENT_ID")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();

    @ManyToMany
    @JoinTable(name = "CATEGORY_ITEM",
            joinColumns = @JoinColumn(name = "CATEGORY_ID"),
            inverseJoinColumns = @JoinColumn(name = "ITEM_ID")
    )
    private List<Item> items = new ArrayList<>();
}

```
<br><br>


## 영속성 전이 : CASCADE
특정 Entity를 영속 상태로 만들 때 연관된 Entity도 함께 영속 상태로 만들고 싶을 때 사용  
부모 Entity를 저장할 때 자식 Entity도 같이 저장한다.  

영속성 전이는 연관관계를 매핑하는 것과 관련이 없다.  
단지 연관된 Entity를 함께 저장하는 편리함을 제공  

Order.java
```java
@Entity
@Table(name = "ORDERS")
public class Order extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL) //***
    @JoinColumn(name = "DELIVERY_ID")
    private Delivery delivery;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL) //***
    private List<OrderItem> orderItems = new ArrayList<>();

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;
}
```
