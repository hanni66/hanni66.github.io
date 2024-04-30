---
title: "Spring Type Converter"
excerpt: "데이터 타입을 변환하는 기능인 타입 컨버터에 대해 정리한 글이다."

categories:
  - Spring
tags:
  - [Spring, Converter]

toc: true
toc_sticky: true

date: 2024-04-30
last_modified_at: 2024-04-30
---

# 스프링 타입 컨버터란?

스프링 프레임워크에서 타입 컨버터는 입력 데이터와 출력 데이터 간의 타입 변환을 처리하는 핵심 기능이다.

개발 과정에서 다양한 타입의 데이터를 다루게 되는데, 이때 `타입 컨버터`를 사용하여 데이터 타입을 변환할 수 있다.

예를 들어 웹 개발에서 HTTP 요청 파라미터를 처리할 때 문자열 타입의 데이터를 다른 타입(정수, 실수 등)으로 변환해야 하는 경우가 많다.

## 주요 기능

- **기본 타입 변환**: 스프링은 Integer, String, Boolean 등 기본 타입에 대한 내장 컨버터를 제공한다.

- **사용자 정의 타입 변환**: 개발자가 직접 타입 컨버터를 구현하여 사용자 정의 타입에 대한 변환을 처리할 수 있다.

- **Converter 인터페이스**: 스프링은 Converter<S, T> 인터페이스를 제공하여 입력 타입 S와 출력 타입 T 간의 변환을 정의할 수 있다.

- **ConversionService**: 스프링은 ConversionService 인터페이스를 통해 타입 변환 기능을 제공한다. 개발자는 이 인터페이스를 구현하여 사용자 정의 타입 변환을 처리할 수 있다.

## 사용 예시

- **요청 파라미터 타입 변환**: @RequestParam 어노테이션을 사용하여 HTTP 요청 파라미터를 받을 때 스프링이 자동으로 타입 변환을 수행한다.

```
public String helloV2(@RequestParam Integer data) { ... }
```

- **모델 attribute 타입 변환**: @ModelAttribute 어노테이션을 사용하여 모델 데이터를 바인딩할 때 스프링이 자동으로 타입 변환을 수행한다.

```
public String saveProduct(@ModelAttribute Product product) { ... }
```

- **경로 변수 타입 변환**: @PathVariable 어노테이션을 사용하여 URL 경로 변수를 받을 때 스프링이 자동으로 타입 변환을 수행한다.

```
public String getProduct(@PathVariable Long id) { ... }
```

## 사용자 정의 타입 컨버터 구현

스프링은 Converter<S, T> 인터페이스를 제공하여 사용자 정의 타입 변환을 구현할 수 있다.

이를 통해 개발자는 특정 타입에 대한 변환 로직을 직접 작성할 수 있다.

아래 예시에서는 StringToLocalDateConverter 클래스를 구현하여 문자열을 LocalDate 타입으로 변환하는 사용자 정의 컨버터를 만들었다.

```java
public class StringToLocalDateConverter implements Converter<String, LocalDate> {
    @Override
    public LocalDate convert(String source) {
        return LocalDate.parse(source, DateTimeFormatter.ISO_LOCAL_DATE);
    }
}
```

## 스프링 ConversionService

스프링은 ConversionService 인터페이스를 제공하여 타입 변환 기능을 통합적으로 관리할 수 있다. ConversionService는 다음과 같은 기능을 제공한다.

- 내장 타입 컨버터와 사용자 정의 타입 컨버터를 모두 관리한다.
- 타입 변환이 필요한 경우 적절한 컨버터를 찾아 변환을 수행한다.
- 개발자는 ConversionService를 직접 구현하거나 스프링이 제공하는 기본 구현체를 사용할 수 있다.

ConversionService를 사용하면 타입 변환 로직을 중앙에서 관리할 수 있어 코드의 재사용성과 유지보수성이 향상된다.

# 마지막으로

스프링의 타입 컨버터는 다양한 데이터 타입을 처리하는 데 필수적인 기능이다. 스프링은 기본 타입 변환을 지원하며, 개발자는 사용자 정의 타입 컨버터를 구현하여 특정 요구사항을 충족시킬 수 있다. 또한 ConversionService를 통해 타입 변환 로직을 통합적으로 관리할 수 있다. 이러한 스프링의 타입 컨버터 기능은 웹 개발, 데이터 처리 등 다양한 분야에서 활용될 수 있다.
