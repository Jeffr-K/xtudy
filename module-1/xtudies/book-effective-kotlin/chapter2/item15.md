# Item 15: 리시버를 명시적으로 참조하라

우리는 무언가를 더 자세하게 설명하기 위해서 명시적으로 긴 코드를 사용할 때가 있다.

대표적으로 함수와 프로퍼티를 지역 또는 톱레벨 변수가 아닌 다른 리비서로부터 가져온다는 것을 나타낼 때가 있다. 예로 클래스의 메서드라는 것을 나타내기 위한 `this` 가 있다.

```kotlin
class User: Person() {
  private var beersDrunk: Int = 0

  fun drinkBeers(num: Int) {
    this.beersDrunk += num
  }
}
```

비슷하게 확장 리비서(확장 메서드에서의 this)를 명시적으로 참조하게 할 수도 있다. 비교를 위해 일단 리시버를 명시적으로 표현하지 않은 퀵소트(quick sort) 구현을 살펴보자.

```kotlin
fun <T: Comparable<T>> List<T>.quickSort(): List<T> {
  if (size < 2) return this
  val pivot = first()
  val (smaller, bigger) = drop(1).partition { it < pivot }
  
  return smaller.quickSort() + pivot + bigger.quickSort()
}
```

위 코드를 명시적으로 표시해보면:

```kotlin
fiun <T: Comparable<T>> List<T>.quickSort(): List<T> {
  if (this.size < 2) return this
  if (size < 2) return this
  val pivot = first()
  val (smaller, bigger) = drop(1).partition { it < pivot }
  
  return smaller.quickSort() + pivot + bigger.quickSort()
}
```

위 두 코드의 사용 방법은 차이가 없다.

```kotlin
listOf(3,2,5,1,6).quickSort()
listOf("C","D","A","B").quickSort()
```

## 여러 개의 리시버

스코프 내부에 둘 이상의 리시버가 있는 경우 리시버를 명시적으로 나타내면 좋다. `apply` 와 `with` 및 `run` 함수를 사용할 때가 대표적이다.

```kotlin
class Node(val name: String) {
  fun makeChild(childName: String) =
    create("$name.$childName")
      .apply { print("Created ${name}" )}

  fun create(name: String): Node? = Node(name)
}

fun main() {
  val node = Node("parent")
  node.makeChild("child")
}
```

위 코드는 명시적으로 <u>'Created parent.child'</u>가 출력된다고 예상하지만 실제로는 `'Created parent'` 가 출력된다. 그 이유는 무엇일까?

이 상황을 이해하기 위해 명시적으로 리시버를 붙여보자.

```kotlin
class Node(val name: String) {
  fun makeChild(childName: String) =
    create("$name.$childName")
        .apply { print("Created ${this.name}" )}

  fun create(name: String): Node? = Node(name)
}
```

그 이유는 `this` 의 타입이 `Node?` 라서 그렇다. 이를 직접 사용할 수 없기에 언팩(Unpack)을 하고 호출해야 한다.

```kotlin
class Node(val name: String) {
  fun makeChild(childName: String) =
    create("$name.$childName")
        .apply { print("Created ${this?.name}" )} // Unpacked

  fun create(name: String): Node? = Node(name)
}
```

따라서 위의 에제는 `apply` 함수의 잘못된 사용 예를 보여주고 있다. 만약 `also` 함수와 `파라미터 name` 을 사용했다라면 이런 문제 자체가 일어나지 않는다.

`also` 를 사용하면 이전과 마찬가지로 명시적으로 리시버를 지정하게 된다. 일반적으로 `also` 또는 `let` 을 사용하는 것이 `nullable` 값을 처리할 때 훨씬 좋은 선택지라고 할 수 있다.

```kotlin
class Node(val name: String) {
  fun makeChild(childName: String) =
    create("$name.$childName")
        .also { print("Created ${it?.name}" )}

  fun create(name: String): Node? = Node(name)
}
```

만약 리시버가 명확하지 않다면 명시적으로 리시버를 적어서 이를 명확하게 해주어야 한다. 레이블 없이 리시버를 사용하면 <u>가장 가까운 리시버를 의미</u>하게 된다. 외부에 있는 리시버를 사용하려면 레이블을 사용해야 한다. 둘 모두를 사용하는 예를 살펴보자.

```kotlin
class Node(val name: String) {
  fun makeChild(childName: String) =
    create("$name.$childName").apply {
       print("Created ${this?.name} in " + ${this@Node.name}" )
    }

  fun create(name: String): Node? = Node(name)
}

fun main() {
  val node = Node("parent")
  node.makeChild("child") // Created parent.child in parent
```

어떤 리시버를 활용하는지 의미가 훨씬 명확해졌다. 이렇게 명확하게 작성하면 코드를 안전하게 사용할 수 있을 뿐 아니라 가독성도 향상된다.

## DSL 마커

코틀린 DSL 을 사용할 때는 여러 리시버를 가진 요소들이 중첩되더라도 리시버를 명시적으로 붙이지 않는다. DSL 은 원래 그렇게 사용하도록 설계되었기 때문이다. 그런데 DSL 에서는 외부의 함수를 사용하는 것이 위험한 경우가 있다. 예로 간단하게 HTML table 요소를 만드는 HTML DSL 을 생각해보자.

```kotlin
table {
  tr {
    td { +"Column 1" }
    td { +"Column 2" }
  }
  tr {
    td { +"Value 1" }
    td { +"Value 2" }
  }
}
```

기본적으로 모든 스코프에서 외부 스코프에 있는 리시버의 메서드를 사용할 수 있다. 하지만 이렇게 하면 코드에 문제가 발생한다.

```kotlin
table {
  tr {
    td { +"Column 1" }
    td { +"Column 2" }
    tr {
      td { +"Value 1" }
      td { +"Value 2" }
    }
  }
  
}
```

이러한 잘못된 사용을 막으려면 암묵적으로 외부 리시버를 사용하는 것을 막는 DslMarker 라는 메타 어노테이션을 사용한다. 다음과 같은 형태로 사용한다.

```kotlin
@DslMarker
annotation class HtmlDsl

fun table(f: TableDsl.() -> Unit) { ... }

@HtmlDsl
class TableDsl { ... }
```

이렇게 하면 암묵적으로 외부 리시버를 사용하는 것이 금지된다.

```kotlin
table {
  tr {
    td { +"Column 1" }
    td { +"Column 2" }
    tr { // +컴파일오류
      td { +"Value 1" }
      td { +"Value 2" }
    }
  }
  
}
```

따라서 만약 외부 리시버의 함수를 사용하려면 다음과 같이 명시적으로 해야한다.

```kotlin
table {
  tr {
    td { +"Column 1" }
    td { +"Column 2" }
    this@table.tr {
      td { +"Value 1" }
      td { +"Value 2" }
    }
  }
  
}
```

DSL 마커는 가장 가까운 리시버만을 사용하게 하거나, 명시적으로 외부 리시버를 사용하지 못하게 할 때 활용할 수 있는 굉장히 중요한 메커니즘이다. DSL 설계에 따라서 사용 여부를 결정하는 것이 좋으므로 설계에 따라서 사용해야 한다.

<u>짧은 코드가 항상 가독성이 좋은 코드는 아니다. 여러개의 리시버가 있는 상황 등에는 리비서를 명시적으로 적어줌으로 코드 가독성을 오히려 높일 수가 있다.</u>