---
title: "Spring 기초 - 객체 지향 원리 적용하기"
excerpt: "스프링 핵심 원리 - 기본편 강의를 듣고 객체 지향 원리를 적용하는 부분을 공부하고 정리한 글이다."

categories:
  - Spring
tags:
  - [Spring, OOP, SOLID]

toc: true
toc_sticky: true

date: 2024-05-01
last_modified_at: 2024-05-01
---

# 목표

비즈니스 요구사항에 맞게 설계된 기존 코드를 객체 지향 원리를 적용해 객체 지향적인 코드로 바꾸는 것이 목표이다.

## 기존 비즈니스 요구사항과 설계

- **회원**
  - 회원을 가입하고 조회할 수 있다.
  - 회원은 일반과 VIP 두 가지 등급이 있다.
  - 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)
- **주문과 할인 정책**
  - 회원은 상품을 주문할 수 있다.
  - 회원 등급에 따라 할인 정책을 적용할 수 있다.
  - 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경 될 수 있다.)
  - 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)

<details>
<summary>코드 구조 (tree)</summary>
<div markdown="1">

```bash
main
├── CoreApplication.java
├── MemberApp.java
├── OrderApp.java
├── discount
│   ├── DiscountPolicy.java
│   └── FixDiscountPolicy.java
├── member
│   ├── Grade.java
│   ├── Member.java
│   ├── MemberRepository.java
│   ├── MemberService.java
│   ├── MemberServiceImpl.java
│   └── MemoryMemberRepository.java
└── order
    ├── Order.java
    ├── OrderService.java
    └── OrderServiceImpl.java
```

```bash
test
├── CoreApplicationTests.java
├── discount
│   └── DiscountPolicy.java
├── member
│   └── MemberServiceTest.java
└── order
└── OrderServiceTest.java
```

</div>
</details>

## 새로운 할인 정책 개발

새로운 할인 정책 : 고정 금액 할인이 아니라 좀 더 합리적인 주문 금액당 할인하는 정률% 할인으로 변경하길 원한다.

새로운 할인 정책을 반영해 `RateDiscountPolicy`, `RateDiscountPolicyTest`를 추가한다.

<details>
<summary>코드 구조 (tree)</summary>
<div markdown="1">

```bash
main
├── discount
│   ├── DiscountPolicy.java
│   ├── FixDiscountPolicy.java
│   └── RateDiscountPolicy.java
```

```bash
test
├── CoreApplicationTests.java
├── discount
│   ├── DiscountPolicy.java
│   └── RateDiscountPolicyTest.java
├── member
│   └── MemberServiceTest.java
└── order
    └── OrderServiceTest.java
```

</div>
</details>

## 새로운 할인 정책 적용 후 문제점

- 역할과 구현을 충실하게 분리했는가?
  - OK
- 다형성도 활용하고, 인터페이스와 구현 객체를 분리했는가?
  - OK
- OCP, DIP 같은 객체지향 설계 원칙을 충실히 준수했는가?
  - NO

> **OCP 위반**
>
> - **지금 코드는 기능을 확장해서 변경하면, 클라이언트 코드에 영향을 주기 때문에 OCP를 위반**한다.
>
> **DIP 위반**
>
> - 클래스 의존관계를 분석해보면 주문서비스 클라이언트(`OrderServiceImpl`)는 **추상(인터페이스)** 뿐만 아니라 **구체(구현) 클래스** 에도 의존하고 있다.
>   - **추상(인터페이스)** 의존 : `DiscountPolicy`
>   - **구체(구현) 클래스** 의존 : `FixDiscountPolicy` , `RateDiscountPolicy`

### OCP, DIP 위반 설명

```java
public class OrderServiceImpl implements OrderService{
  private final MemberRepository memberRepository = new MemoryMemberRepository();

  // 기존 할인 정책
  private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

  // 변경된 할인 정책
  private  final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```

기존 할인 정책 코드를 사용하다가 할인 정책이 변경되면 아래 변경된 할인 정책 코드를 사용하게 된다.

- **OCP 위반**
  - 할인 정책이 변경되면서 `FixDiscountPolicy` 를 `RateDiscountPolicy` 로 변경해야 한다.
  - 즉, `OrderServiceImpl` 의 소스 코드가 변경되니 OCP를 위반하게 된다.
- **DIP 위반**
  - `OrderServiceImpl`는 `DiscountPolicy` 인터페이스만 의존해야 하는데 구체 클래스(`FixDiscountPolicy` 혹은 `RateDiscountPolicy`)도 함께 의존하고 있다.

## 어떻게 문제를 해결할 수 있을까?

클라이언트 코드인 `OrderServiceImpl` 은 `DiscountPolicy` 의 인터페이스 뿐만 아니라 구체 클래스(`FixDiscountPolicy` 혹은 `RateDiscountPolicy`)도 함께 의존하고 있다.

그래서 구체 클래스를 변경할 때 클라이언트 코드도 함께 변경해야 한다.

또한 DIP를 위반하지 않도록 인터페이스에만 의존하도록 의존관계를 변경하면 된다.

### 인터페이스에만 의존하도록 설계

```java
public class OrderServiceImpl implements OrderService{
  private final MemberRepository memberRepository = new MemoryMemberRepository();
  private DiscountPolicy discountPolicy;
}
```

위 코드는 인터페이스에만 의존하고, 구체 클래스를 의존하지 않도록 변경했다.

## 구현체가 없는데 어떻게 코드를 실행할 수 있을까?

### 해결방안

위의 상황들을 공연과 비유하면 남자 배우는 연기를 하는 책임만 가져야한다.

하지만 남자 배우가 공연도하고, 여자 배우도 직접 섭외하는 등 다양한 책임을 가지고 있다고 생각하면 된다.

그럼 이를 해결하기 위해서는 공연을 구성하고, 담당 배우를 섭외하고, 지정하는 책임을 담당하는 별도의 **공연 기획자**를 만들면, 남자 배우가 가지고 있던 책임을 분리할 수 있게 된다.

즉, 누군가가 클라이언트인 `OrderServiceImpl` 에 `DiscountPolicy` 의 구현 객체를 `대신` 생성하고 주입해주어야 한다.

# AppConfig 로 사용 영역과 구성 영역으로 분리하기

AppConfig는 **구현 객체를 생성**하고, **연결**하는 책임을 가진다.

아래 코드의 AppConfig는 실제 필요한 **구현 객체를 생성** 한다.

또한 생성한 객체 인스턴스의 참조(레퍼런스)를 **생성자를 통해서 주입(연결)** 해준다.

## 구성 영역 - AppConfig

```java
public class AppConfig {
  public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
  }
  public OrderService orderService() {
    return new OrderServiceImpl(
            memberRepository(),
            discountPolicy());
  }
  public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
  }
  public DiscountPolicy discountPolicy() {
    return new FixDiscountPolicy();
  }
}
```

## 사용 영역

### MemberServiceImpl 생성자 주입

- `MemberServiceImpl`은 `MemoryMemberRepository`를 의존하지 않고, `MemberService` 인터페이스만 의존하게 된다.
- `MemberServiceImpl` 입장에서는 생성자를 통해 어떤 구현 객체가 들어오는지(주입되는지) 알 수 없다.
- 어떤 구현 객체를 주입할지는 오직 외부에서만 결정된다. (AppConfig에서 결정)
- `MemberServiceImpl` 은 이제부터 **의존관계에 대한 고민은 외부**에 맡기고 **실행에만 집중**하면 된다.

```java
public class MemberServiceImpl implements MemberService {
  private final MemberRepository memberRepository;

  public MemberServiceImpl(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }
}
```

`MemberServiceImpl` 은 `MemberRepository` 인 추상에만 의존하면 되고, 구체 클래스를 몰라도 된다.

`즉, DIP를 만족하게 된다.`

또한 객체를 생성하고 연결하는 역할과 실행하는 역할이 명확히 분리되었다.

> #### 참고 - DI(Dependency Injection)
>
> `AppConfig` 객체는 `memoryMemberRepository` 객체를 생성하고, 그 참조값을 `memberServiceImpl` 를 생성하며 생성자로 전달하게 된다.
>
> <details>
> <summary>코드 참조</summary>
> <div markdown="1">
>
> ```java
> public class AppConfig {
>   public MemberService memberService() {
>     return new MemberServiceImpl(memberRepository());
>   }
> }
> ```
>
> </div>
> </details>
>
> 클라이언트인 `memberServiceImpl` 입장에서 보면 의존 관계를 외부에서 주입해주는 것 같다고 해서 DI(의존관계 주입, 의존성 주입) 라고 한다.

### OrderServiceImpl 생성자 주입

- `OrderServiceImpl` 은 `FixDiscountPolicy` 를 의존하지 않고, `DiscountPolicy` 인터페이스만 의존한다.
- `OrderServiceImpl` 입장에서 생성자를 통해 어떤 구현 객체가 들어올지(주입될지)는 알 수 없다.
- `OrderServiceImpl` 의 생성자를 통해서 어떤 구현 객체를 주입할지는 오직 외부에서 결정한다. (AppConfig에서 결정)
- `OrderServiceImpl` 은 이제부터 **실행**에만 집중하면 된다.
- `OrderServiceImpl` 에는 `MemoryMemberRepository`와 `FixDiscountPolicy` 객체의 의존관계가 주입된다.

```java
public class OrderServiceImpl implements OrderService {
  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;

  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

`OrderServiceImpl` 은 `MemberRepository` 와 `DiscountPolicy`인 추상에만 의존하면 되고, 구체 클래스를 몰라도 된다.

그렇게 되면 `DIP를 만족`하게 된다.

### AppConfig 실행

#### 사용 클래스 - MemberApp & OrderApp

```java
 public class MemberApp {
  public static void main(String[] args) {
    AppConfig appConfig = new AppConfig();
    MemberService memberService = appConfig.memberService();
  }
}

public class OrderApp {
  public static void main(String[] args) {
    AppConfig appConfig = new AppConfig();
    MemberService memberService = appConfig.memberService();
    OrderService orderService = appConfig.orderService();
  }
}
```

#### 테스트코드 오류 수정

```java
class MemberServiceTest {
  MemberService memberService;

  @BeforeEach
  public void beforeEach() {
    AppConfig appConfig = new AppConfig();
    memberService = appConfig.memberService();
  }
}

class OrderServiceTest {
  MemberService memberService;
  OrderService orderService;

  @BeforeEach
  public void beforeEach() {
    AppConfig appConfig = new AppConfig();
    memberService = appConfig.memberService();
    orderService = appConfig.orderService();
  }
}
```

> #### @BeforeEach
>
> 테스트 코드에서 `@BeforeEach` 는 각 테스트를 실행하기 전에 호출된다.
>
> Test가 실행되기전 `beforeEach`를 실행시켜 AppConfig에서 `memberService`나`orderService`를 할당 해준다.
>
> 그 이후에 `@Test` 코드를 실행하게 된다.

# 새로운 할인 정책 적용

맨 윗 부분으로 돌아가서 정액 할인 정책을 정률% 할인 정책으로 변경하는 것을 코드에 적용해보면 AppConfig 부분만 고치면 된다.

```java
public class AppConfig {
  public DiscountPolicy discountPolicy() {
    // return new FixDiscountPolicy();
    return new RateDiscountPolicy();
  }
}
```

`AppConfig` 에서 할인 정책 역할을 담당하는 구현을 변경하면 되는 것이다. (`FixDiscountPolicy()` -> `RateDiscountPolicy()`)

## 결론

`AppConfig`의 등장으로 애플리케이션이 크게 `사용 영역`과, 객체를 생성하고 `구성(Configuration)하는 영역`으로 분 리되었다.

이제 할인 정책을 변경해도, 애플리케이션의 구성 역할을 담당하는 AppConfig만 변경하면 된다.

### 사용 영역

클라이언트 코드인 `MemberServiceImpl`이나 `OrderServiceImpl` 를 포함해 사용 영역의 어떤 코드도 변경할 필요가 없다.

### 구성 영역

구성 역할을 담당하는 AppConfig를 애플리케이션이라는 공연의 기획자로 생각해보면 공연 기획자는 공연 참여자인 구현 객체들을 모두 알아야 한다.
따라서 구성 영역은 당연히 변경된다.
