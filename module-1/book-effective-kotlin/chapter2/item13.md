# Item 13: Unit? 을 리턴하지 말라

만약 함수에서 `Unit?` 을 리턴한다면 그 이유는 무엇일까? 마치 `Boolean` 이 `true` 또는 `false` 를 갖는 것처럼, `Unit?` 은 `Unit` 또는 `null` 이라는 값을 가질 수 있다.

따라서 `Boolean` 과 `Unit?` 은 서로 바꿔서 사용할 수 있다. 일반적으로 `Unit?` 을 사용한다는 것은 이런 경우가 있다.

```kotlin
fun keyIsCollect(key: String): Boolean = { ... }

if (!keyIsCorrect(key)) return
```

다음처럼 사용할 수도 있다.

```kotlin
fun verifyKey(key: String): Boolean = { ... }

verifyKey(key) ?: return
```

저자가 말하고자 하는 바는 <u>`Unit?` 으로 `bool` 타입을 표현하는 것은 오해의 소지가 있을 수 있고 예측하기 어려운 코드를 만들 수 있다.</u>

`getData()?.let { view.showData(it) } ?: view.showError()`

위 코드는 `showData` 가 `null` 을 리턴하고, `getData` 가 `null` 이 아닌 값을 리턴할 때 `showData` 와 `showError` 가 모두 호출된다.

이런 코드보다는 `if-else` 를 사용하는 것이 훨씬 이해하기 쉽고 깔끔할 것이다.


<u>1번 사례</u>

```kotlin
if (person != null && person.isAdult) {
  view.showPerson(person)
} else {
  view.showError()
}
```

<u>2번 사례</u>

```kotlin
if(!keyIsCorrect(key)) return

verifyKey(key) ?: return
```

위 사례들을 겉에서만 봐도 1번 사례가 더 가독성이 있다는 것을 알 수 있다.


