# Item 17: 이름 있는 아규먼트를 사용하라

만약 `joinToString` 함수에 대해서 알고 있다면 `|` 표시는 구분자(seperator)라는 것을 알 수 있을 것이다.

하지만 모른다면 접두사(prefix)로 생각할 수도 있을 것이다. 따라서 명확하지 않게 보일 수 있겠다.

```kotlin
val text = (1..10).joinToString("|")
```

이런 경우 이름 있는 아규먼트(named argument)를 사용하면 된다.

```kotlin
val text = (1..10).joinToString(seperator = "|")
```

naemd argument 를 사용하지 않으면, 함수 호출 시 인자를 잘못 넣을 수도 있다.

```kotlin
val sperator = "|"
val text = (1..10).joinToString(seperator = seperator)
```

# 이름 있는 아규먼트(Named argument)는 언제 사용해야 할까?

named argument 를 사용하면 코드가 길어지지만 다음과 같은 두 가지 장점이 생긴다.

- 이름을 기반으로 값이 무엇을 나타내는지 알 수 있다.
- 파라미터 입력 순서와 상관 없으므로 안전하다.

argument 이름은 함수를 사용하는 개발자 뿐 아니라 코드를 읽는 다른 사람들에게도 굉장히 중요한 정보를 줄 수 있기 때문이다.

### <u>Magic Number 로 인한 코드의 설명 불충분할 때</u>

```kotlin
// bad
sleep(100)

// good
sleep(timeMills = 100)

// good(custom function)
sleep(Mills(100))

//good(property extension)
sleep(100.ms)
```

만약 성능에 영향을 줄 것 같다고 생각한다면 `Item 46: 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라` 에서 다루는 인라인 클래스 사용하기.

### <u>default argument 의 경우</u>

만약 프로퍼티가 default argument 를 가질 경우 항상 이름을 붙여서 사용하는 것이 좋다. 일반저긍로 함수 이름은 필수 파라미터들과 관련되어 있기 때문에 default 값을 갖는 옵션 파라미터(Optional Parameter)의 설명이 명확하지 않다.

### <u>같은 타입의 파라미터가 많은 경우</u>

만약 함수 파라미터가 모두 다른 타입이라면 위치를 잘못 입력하면 오류가 발생한다. 하지만 파라미터가 모두 같은 타입이라면?

```kotlin
fun sendMail(to: String, message: String) { /* ...code */}
```

이런 경우 named argument 를 사용하는 것이 바람직하다.

```kotlin
sendMail(
  to = "oscar",
  message = "hello, world!"
)
```

### <u>함수 타입 파라미터</u>

함수 타입 파라미터는 조금 특별하게 다루어질 필요가 있다. 일반적으로 함수 타입 파라미터는 마지막 위치에 배치하는 것이 좋다. 예를 들어 `repeat` 을 생각해보자.

`repeat` 뒤에 오는 람다는 반복될 블록을 나ㅏ낸다. `thread` 도 그 이후의 블록이 스레드 본문이라는 것을 알 수 있다. 이러한 이름들은 일반적으로 마지막에 위치하는 함수 파라미터에 대해서만 설명한다.

여러 함수 타입의 옵션 파라미터가 있는 경우에는 더 햇갈린다.

```kotlin
fun call(before: () -> Unit = {}, after: () -> Unit = {}) {
  before()
  print("Middle")
  after()
}

call({ print("CALL") })
call({ print("CALL") })
```

위 예제를 named argument 를 이용해 가독성을 증대시킨다.

```kotlin
call(before = { print("CALL") })
call(before = { print("CALL") })
```

<u>리액티브 라이브러리를 이용할 때:</u>

- 자바의 경우

```java
observable.getUsers()
  .subscribe((List<User> users) -> {

  }, (Throwable throwable) -> {
    // ...code
  }, () -> {
    // ...code
  })
```

- 코틀린인 경우

```kotlin
observable.getUsers()
  .subscribeBy(
    onNext = { users: List<User> -> 
      // ...code
    },
    onError = { throwable: Throwable ->
      // ...code
    },
    onCompleted = {
      // ...code
    }
  )
```

# Think

가독성과 설명을 보충하기 위해 반드시 사용해야 하는 기능인 것 같다. 조금 코드를 더하더라도

주변 동료를 위해 사용하는 쪽을 선택해야겠다.