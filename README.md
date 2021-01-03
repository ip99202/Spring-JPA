# Spring-JPA-실습

위 코드는 김영한님의 강의 [자바 ORM 표준 JPA 프로그래밍 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)을 바탕으로 작성된 것입니다.

## JPA 영속성 컨텍스트
[영속성 컨텍스트](https://ip99202.github.io/posts/JPA-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8/)

## JPA 데이터베이스 스키마 자동생성
[스키마 자동생성](https://ip99202.github.io/posts/JPA-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%EC%8A%A4%ED%82%A4%EB%A7%88-%EC%9E%90%EB%8F%99%EC%83%9D%EC%84%B1/)

## 다양한 연관관계 매핑
<img width=500px src="https://user-images.githubusercontent.com/52627952/103472912-4885cb00-4dd6-11eb-9d61-3201c196d530.png">  

### DELEVERY와 ORDERS는 일대일 매핑  

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
