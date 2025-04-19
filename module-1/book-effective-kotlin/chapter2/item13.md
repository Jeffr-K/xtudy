# Item 13: Unit? 을 리턴하지 말라

만약 함수에서 `Unit?` 을 리턴한다면 그 이유는 무엇일까? 마치 Boolean 이 true 또는 false 를 갖는 것처럼, Unit? 은 Unit 또는 null 이라는 값을 가질 수 있다.

따라서 Boolean 과 Unit? 은 서로 바꿔서 사용할 수 있다. 일반적으로 Unit? 을 사용한다는 것은 이런 경우가 있다.

```kotlin
fun keyIsCollect(key: String): Boolean = { ... }

if (!keyIsCorrect(key)) return
```

다음처럼 사용할 수도 있다.

```kotlin
fun verifyKey(key: String): Boolean = { ... }

verifyKey(key) ?: return
```

