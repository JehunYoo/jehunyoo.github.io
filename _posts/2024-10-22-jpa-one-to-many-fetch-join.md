---
title: "JPA - OneToMany fetch join 탐구하기"
date: 2024-01-22 23:30:00 +0900
tags: ["jpa", "fetch join"]
categories: [JPA]
toc: true
math: false
img_path: /assets/img/
---

# 연관 관계 설정하기

다음과 같은 연관 관계를 갖는 entity를 정의했다.

```
Alpha (N : 1) Beta (1 : N) Gamma (N : 1) Delta
```

이때 `Alpha`에서 부터 `Delta`까지 탐색이 가능하게 양방향 연관 관계를 설정해두었다. \
또한 모든 entity는 `String name`이라는 속성을 갖게 정의했다.

``` java
Delta delta1 = new Delta("delta1");
Delta delta2 = new Delta("delta2");
Beta beta1 = new Beta("beta1");
Beta beta2 = new Beta("beta2");

Gamma gamma11 = new Gamma("gamma11", beta1, delta1);
Gamma gamma12 = new Gamma("gamma12", beta1, delta2);
Gamma gamma21 = new Gamma("gamma21", beta2, delta1);
Gamma gamma22 = new Gamma("gamma22", beta2, delta2);

Alpha alpha11 = new Alpha("alpha11", beta1);
Alpha alpha12 = new Alpha("alpha12", beta1);
Alpha alpha21 = new Alpha("alpha21", beta2);
Alpha alpha22 = new Alpha("alpha22", beta2);
```

| `Alpha` | `Beta` | `Gamma` | `Delta` |
|---|---|---|---|
| alpha11 | beta1 | gamma11 | delta1 |
| alpha11 | beta1 | gamma12 | delta2 |
| alpha12 | beta1 | gamma11 | delta1 |
| alpha12 | beta1 | gamma12 | delta2 |
| alpha21 | beta2 | gamma21 | delta1 |
| alpha21 | beta2 | gamma22 | delta2 |
| alpha22 | beta2 | gamma21 | delta1 |
| alpha22 | beta2 | gamma22 | delta2 |

# 처음부터 끝까지 fetch join하기

``` sql
select distinct a from Alpha a join fetch a.beta b join fetch b.gammas g join fetch g.delta d
```

모두 join했기 때문에 쿼리는 한 번만 나간다.
결과는 다음과 같다.

```
alpha11 beta1 gamma11 delta1
alpha11 beta1 gamma12 delta2
alpha11 beta1 gamma11 delta1
alpha11 beta1 gamma12 delta2

alpha12 beta1 gamma11 delta1
alpha12 beta1 gamma12 delta2
alpha12 beta1 gamma11 delta1
alpha12 beta1 gamma12 delta2

alpha21 beta2 gamma21 delta1
alpha21 beta2 gamma22 delta2
alpha21 beta2 gamma21 delta1
alpha21 beta2 gamma22 delta2

alpha22 beta2 gamma21 delta1
alpha22 beta2 gamma22 delta2
alpha22 beta2 gamma21 delta1
alpha22 beta2 gamma22 delta2
```

결과를 보면 데이터가 중복되었다. \
이는 `Beta`와 `Gamma`가 일대다 관계에 있기 때문이다. \
일대다 조인의 경우 결과가 증가할 수 있다. \
따라서 주의해야 한다.

이 방식을 사용하면 pagination을 사용하지 않는 것이 좋다. \
JPA에서는 지금과 같은 쿼리에 pagination을 할 경우
1. 쿼리의 결과를 모두 메모리에 올려서
2. 메모리에서 pagination을 한다.

여기서 쿼리의 결과는 중복된 데이터도 포함하기 때문에 OutOfMemory가 발생할 수 있다.


# fetch join과 default_batch_fetch_size 사용하기

```
hibernate.default_batch_fetch_size=2
```

lazy loading을 할 때 한 번에 몇 개의 entity를 가져올 것인지에 대한 설정이다. \
위의 예시에서는 관련된 entity가 최대 2개였으므로 2로 설정했다.

``` sql
select a from Alpha a join fetch a.beta
```

이때 위의 쿼리를 실행하면 어떻게 될까?

``` java
List<Alpha> alphas = em.createQuery("select a from Alpha a join fetch a.beta b", Alpha.class).getResultList();

for (Alpha alpha : alphas) {  
    Beta beta = alpha.getBeta();  
    System.out.println("gammas size " + beta.getGammas().size());  
    for (Gamma gamma : beta.getGammas()) {  
        Delta delta = gamma.getDelta();  
        System.out.println(alpha.getName() + " " + beta.getName() + " " + gamma.getName() + " " + delta.getName());  
    }  
}
```

우선 `Beta`까지는 fetch join으로 한 번에 가져왔으므로 `Alpha`와 `Beta`는 Persistence Context에 존재한다. \

## `default_batch_fetch_size` 설정이 없다면?

이후에 반복문을 돌면서 매번 `gamma.getName()`, `delta.getName()`을 만날 때 lazy loading이 발생했을 것이다. \
이렇게 되면 쿼리를 계속 DB로 전송해야 하므로 네트워크 사용으로 인한 지연이 발생한다.

## `default_batch_fetch_size`을 설정해주면 어떻게 될까?

```
alpha11 beta1 gamma11 delta1
alpha11 beta1 gamma12 delta2

alpha12 beta1 gamma11 delta1
alpha12 beta1 gamma12 delta2

alpha21 beta2 gamma21 delta1
alpha21 beta2 gamma22 delta2

alpha22 beta2 gamma21 delta1
alpha22 beta2 gamma22 delta2
```

처음 `gamma.getName()`, `delta.getName()`을 만날 때 lazy loading이 발생한다. (심지어 `Delta`도 가져온다!!!) \
이때 설정 값만큼 `Gamma`, `Delta`를 한번에 fetch한다. \
따라서 네트워크 사용으로 인한 지연이 상대적으로 적어지는 장점이 있다.

(정확히는 아직 초기화 되지 않은 프록시 `Gamma`, `Delta`를 만나서 다음과 같은 `gamma.getName()`, `delta.getName()`메서드가 호출될 때마다 lazy loading이 발생한다. 그렇기 때문에 적절한 `default_batch_fetch_size` 설정이 중요하다.)

또 중요한 차이점으로, 앞서 모든 것에 대해 fetch join을 했을 때와 다르게 중복된 데이터가 없다. \
우리가 예상하는 정확한 데이터만을 가져올 수 있다. \
따라서 pagination을 사용할 수 있다. (메모리에서 하는 pagination이 아니다.)