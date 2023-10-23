---
title: "[Waffle] 스프링 프로젝트 구조 설계하기"
date: 2023-10-23 23:02:00 +0900
tags: ["waffle", "spring"]
categories: [Waffle]
toc: true
math: false
img_path: /assets/img/waffle
---

# 현재  상황

구현해야 하는 것
- Member, Waffle, Comment CRUD
- Auth (회원가입, 로그인, 로그아웃)

DB 테이블 설계
- Member, Waffle, Comment + 기타 테이블

# 패키지 구조 정하기

검색해보니 패키지 구조에는 크게 2가지 경우가 있었다.

1. 계층형
2. 도메인형

도메인형 구조는 도메인 별로 패키지를 분리한다. 독립적인 코드를 작성하는 것에 유리하다고 생각해서 도메인형 패키지 구조를 사용하기로 했다.

# 프로젝트 구조 설계하기

스프링 프레임워크를 사용하는 만큼 객체 지향 프로그래밍에 맞게 설계를 하고 싶었다. 특히 객체들이 하나의 책임만을 갖고 (SRP), 구체화보다는 추상화에 의존하면서 (DIP), 확장에는 열려있으나 변경에는 닫혀있는 (OCP) 설계를 원했다. 그래서 계층을 나누고 계층에 해당하는 객체에 하나의 역할만 부여하고자 했다.

![[spring-architecture.png]]

## 계층 나누기

계층(layer)은 크게 다음 4가지로 나눌 수 있다.

1. Presentation Layer - Controller, ControllerAdvice
2. Business Layer - Service
3. Persistence Layer - Repository, DAO
4. Database Layer - DB

각 계층은 순차적으로 연결되며, 다른 계층이 *어떻게* 구현되어 있는지 알지 말아야 한다. 이렇게 의존성을 줄임으로써 각 계층의 역할이 명확해지고 특정 계층에 변화가 생겼을 때 영향을 적게 받는다.  
의존성을 줄이기 위해 추가로 다양한 방법을 적용했다.

- 서비스 로직과 DB 로직 분리하기
- DAO를 테이블에 1:1로 매핑하기
- `Service`에서 호출하는 Persistence Layer의 메서드 이름을 **어떻게**보다 **무엇을** 하는지에 초점 맞추기

## 역할 부여하기

각각의 객체는 하나의 역할만 수행한다.
- `Controller`: 클라이언트 요청 받기
- `ControllerAdvice`: 예외 처리하기
- `Service`: 서비스 로직 수행하기
- `Repository`: DB 로직 수행하기
- `DAO`: 매핑된 테이블에 대한 요청 처리하기

추가로 다음과 같은 객체도 있다.
- `DTO`: 클라이언트 요청 운반하기
- `Validator`: 클라이언트 요청 검증하기
- `Converter`: `DTO` <-> entity 변환하기 (직접 정의한 객체이다. 스프링이 제공하는 `ConversionService`로 대체 예정)

특별히 `DAO`는 테이블에 1대1로 매핑되어 해당 테이블에 대한 원자적인 요청을 처리한다. 다음의 메서드를 `CrudDao` 인터페이스로 만들어둬서 CRUD가 필요한 다른 도메인에서도 구현하도록 강제했다.
- `DAO.save()`
- `DAO.findById()`
- `DAO.delete()`

이 방법의 장점은 도메인이 여러 개일 때 나타난다. 복수의 `Repository`에서 같은 테이블에 접근하는 경우를 생각해보자. 이때 접근 방법이 다양하면 관리도 힘들고 변경이 발생했을 때 관리하기 힘들 것이다.  
**`DAO`를 사용함으로써 원하는 테이블의 `DAO`만 알면 해당 테이블에 대한 요청을 간단하게 보낼 수 있다.**

추가로 DB가 어떻게 설계되어 있는지 `Service`가 알지 못하게 할 수 있다. 만약 `Waffle`을 조회해야하는 경우 `Comment`도 같이 조회를 해야한다면, `WaffleService`는 `waffleRepository.findAll()`, `commentRepository.findAll()`을 호출해야 한다. 이때 `WaffleRepository`가 DB 로직을 분리하여 해당 작업을 책임진다면 `WaffleService`는 `waffleRepository.findAll()`만 호출할 수 있다. 동시에 **DB의 구조를 숨길 수 있게된다.**

## DTO는 어디까지 내려갈까?

검증이 끝난 `DTO`는 어느 계층까지 내려보내야 할까? 검색해보니 일반적으로 `Service`에서 Entity로 변환하는 것 같다. 모든 `DTO`를 `Service`에서 변환하면 좋겠지만 그럴 수 없는 경우도 있다. 불필요하게 미리 변환하게 되면 요청이 변경되었을 때 영향을 받는 계층이 많아지므로 최대한 필요한 순간에 변환하기로 했다. 다만, 그 하한선은 `Repository`로 제한했다.