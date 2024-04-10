---
title: 테스트 코드의 중요성
except: 테스트 코드의 중요성을 경험하다.
categories:
  - WebClass
tags: 
permalink: /project/webClass/테스트 코드의 중요성
toc: true
toc_sticky: true
date: 2024-04-10
last_modified_at: 2024-04-10
---
아래 내용은 사이드 프로젝트(WebClass)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

## 문제 발생

유튜브를 보고 결제 관련해서 기능을 개발하던 도중에 \'이 기능은 같이 개발하는 분이랑 협의해서 정해야겠는데?' 라는 생각이 들어서 일부 java파일을 삭제하던 도중에 현재 기능에 필요한 파일까지 삭제해서 기존 코드가 돌아가지 않는 상황

### Payment Class
``` java 
@Entity  
public class Payment {  
  
    @Id  
    @GeneratedValue    @Column(name = "payment_id")  
    private Long id;  
  
    @OneToOne(mappedBy = "payment")  
    private order order;  
  
    @Enumerated(EnumType.STRING)  
    PaymentMethod paymentMethod;  
  
    LocalDateTime orderDate;  
  
    public static Payment createPayment(PaymentMethod paymentMethod) {  
        Payment payment = new Payment();  
        payment.paymentMethod = paymentMethod;  
        payment.orderDate = LocalDateTime.now();  
  
        return payment;  
    }  
  
    public void updateOrder(order order) {  
        this.order = order;  
    }  
}
```

결제 관련해서 세부적인 결제 방식(현금, 카드 등)을 구현하려고 몇 가지 클래스를 만들었다가 삭제하는 과정에서 ``Payment``클래스도 같이 삭제가 되어버렸다. ``Payment``는 ``order``클래스와 엮여있기 떄문에 없어서는 안될 클래스였다.  

### Payment 구현
``Payment``를 복구할 수 있는 방법이 인텔리제이에서 지원을 하는 것 같지만(스냅샷? 이란 기능으로 상태를 임시 저장하는 기능이 있는 것 같다.), 그 방법을 지금은 사용하지 못했기 때문에 다시 구현을 했다.  

근데 클래스를 처음 구현할 때 여기저기 찾아보면서 구현을 했기 떄문에 다시 구현한 클래스가 제대로 돌아가나? 뭐 빠뜨린게 있나? 싶은 생각이 들었다.  

이 때 생각난 것이 Test코드이다.  Test코드가 제대로 동작한다면,  \'적어도 내가 하고자 했던 기능은 구현이 된거겠지?' 라는 생각을 가지고 Test코드를 돌려봤다.  

### Test 코드

``` java
@Test  
@DisplayName("Order를 통해 Book을 생성하면 널이 나오지 않는다.")  
public void createOrderAndNotNullTest() {  
    //given  
    Member member = new Member();  
    Delivery delivery = Delivery.createDelivery("zipcode", "defaultAddress", "detailAddress", "deliveryMemo");  
    Payment payment = Payment.createPayment(PaymentMethod.CREDIT_CARD);  
    Book book = Book.createBook("name", 10000, 10, "12345");  
  
    OrderItem orderItem = OrderItem.createOrderItem(book, 10000, 10);  
  
    order my_order = order.createOrder(member, "010", delivery, payment, orderItem);  
  
    //when  
  
    assertThat(my_order).isNotNull();  
    //than  
}
```

테스트 코드 중 일부를 가져온 것인데, 처음 실행했을 때 에러가 나서 확인해보니, ``Payment.createPayment`` 메소드를 구현하지 않아서 생긴 문제였다. 관련 메소드를 구현하고 다시 Test를 실행했더니 작성했던 모든 테스트가 통과하였다.  

``Payment``가 간단하게 구현되었기 때문에 Test코드 없이도 금방 복구를 했겠지만 Test코드가 있으니 무엇이 빠졌는지 쉽게 확인할 수 있었고, 실제로 확인할 근거가 있으니 좀 더 안심이 되고 믿음이 갔다.  

Test코드가 중요한 건 알았지만, 이런 사소한 실수로 그 중요성을 다시 한번 깨닫게 되다니.. Test코드는 참 좋은 것 같다.