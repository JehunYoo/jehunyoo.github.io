---
title: "[Waffle] 댓글 조회 REST API 설계하기"
date: 2023-10-16 23:10:00 +0900
tags: ["waffle", "rest api"]
categories: [Waffle]
toc: true
math: false
img_path: /assets/img/waffle
---

# 뭔가 이상한데?

만들었던 API 명세에는 단일 댓글 조회를 이렇게 해놨다.

`GET /waffles/{waffleId}/comments/{commentId}`

여기서 `waffleId`, `commentId`가 정확히 어떤 의미인지 혼동이 왔다.
저 URI가 의미가 있으려면 계층 구조를 살려서 다음과 같이 사용되야 한다고 생각했다.

`/waffles/10/comments/2` (10번째 waffle의 2번째 댓글)

path variable 자리에 몇 번째인지 의미를 부여했다.
이렇게 하면 각 waffle에 대한 n번째 댓글을 요청할 수 있으므로 요청을 보낼 때도 의미가 자연스러워 보였다.

# 구현하기

`/waffles/10/comments/2`라는 요청을 받았다고 가정하자.
어떻게 구현할 것인가?

현재 DB의 comment 스키마는 다음과 같다.

![](comment-schema.png)

몇 번째 waffle인지 알고 해당 waffle의 몇 번째 댓글인 것을 알고 있기 때문에 다음과 같이 조회할 수 있다.

``` sql
select * from comment where waffle_id = 10 order by created_at limit 1, 1;
```

# 사용자 피드에 적용하기

그럼 이제 어떤 사용자가 `/waffles/10/comments/2`라는 요청을 보낸 상황을 생각해보자.
이 요청은 어떤 사용자 A의 10번째 waffle의 2번째 댓글에 대한 요청이다.

그런데 이것이 REST API 설계에 맞는 것인지 의문이 들었다.
사용자 A에 대한 정보가 URI에 포함되어 있지 않기 때문이다.
(수정한다면 이런 방식으로 `/members/{memberId}/waffles/{waffleId}/comments/{commentId}`)

또한 각 사용자마다 몇 번째 waffle인지에 대한 계산을 해야하는 부담도 있다.
하나의 waffle에 대해 사용자마다 피드에서 보이는 순서는 모두 다를 수 있기 때문이다.

여기에 더해서 매번 댓글을 조회할 때 정렬에 대한 비용이 발생할 수 있다.

# 계층 구조 제거하기

URI에서 계층 구조를 사용하지 않는다면 문제를 해결할 수 있다.
예를 들어, `comments/{commentId}`처럼 직접 댓글을 요청하는 것이다.
(그러나 이렇게 되면 몇 번째 댓글인지에 대한 정보는 더 이상 의미가 없게 된다.)

요청을 처리할 때 구현은 간단해졌다.

``` sql
select * from comment where id = comment_id;
```

이제 id에 순서에 대한 의미를 부여하지 말고 pk를 id로 사용하자.

문제가 없는 것 같지만 아직 한 개가 남아있다.

# 복수 댓글 조회 API와 비교

복수 댓글 조회 API 명세는 다음과 같다.

`GET /waffles/{waffleId}/comments`

만약 waffle_id가 10인 (더 이상 10번째 waffle이 아니다.) waffle에 대한 모든 댓글을 조회하고 싶다면 다음과 같이 요청하는 것이 자연스럽다.
`/waffles/10/comments`

심지어 이 요청에는 계층 정보가 포함되어 있다.

- 복수 조회: `/waffles/{waffleId}/comments`
- 단일 조회: `comments/{commentId}`

비슷한 API임에도 전혀 다른 URI를 가지고 있다.
통일감도 없고 REST API 설계에도 맞지 않아보인다.

# 해결 방법

생각했던 해결 방법은 간단하다.

- 복수 조회: `/waffles/{waffleId}/comments`
- 단일 조회: `/waffles/{waffleId}/comments/{commentId}`

다만 한가지 어색한 점은 단일 조회에서 `comment_id`를 알기 때문에 굳이 `waffle_id`가 필요하지 않다는 점이다.
그럼에도 굳이 추가를 한 이유는 요청의 유효성을 검증하기 위해서이다.

만약 `waffle_id = 10`인 waffle에 대한 댓글을 조회한다고 가정하자.
이때 해당 waffle과 전혀 관련없는 `comment_id = 200`인 댓글을 잘못 요청한 경우 서버에서 잘못된 요청임을 파악해야 한다. (404 에러를 반환할 수 있다.)

이런 생각의 흐름으로 위와 같은 결론을 얻었다.
