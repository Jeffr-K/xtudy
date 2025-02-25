# Item 1. 가변성을 제한하라

코틀린은 모듈로 프로그램을 설계한다. 모듈은 클래스, 객체, 함수, 타입 별칭, 탑 레벨(top-level) 프로퍼티 등 다양한 요소로 구성된다.

이러한 요소 중 일부는 상태(state)를 가질 수 있다. 예를 들어 읽고 쓸 수 있는 프로퍼티(read-write property) var 를 사용하거나 mutable 객체를 사용하면 상태를 가질 수 있다.

```kotlin
var a = 10
var list: MutableList<Int> = mutableListOf()
```

이처럼 요소가 상태를 갖는 경우 해당 요소의 동작은 사용 방법 뿐 아니라 그 이력(history)에도 의존하게 된다.

### 문제가 되는 코드 분석

아래의 코드를 분석해보자.

```kotlin
class BankAccount {
  var balance = 0.0
    private set
  
  fun deposit(depositAmount: Double) {
    balance += depositAmount;
  }

  @Throws(InsufficientFunds::class)
  fun withdraw(withdrawAmount: Double) {
    if (balance < withdrawAmount) {
      throw InsufficientFunds()
    }
    
    balance -= withdrawAmount
  }
}

class InsufficientFunds : Exception()

val account = BankAccount()

println(account.balance)

account.deposit(100.0)
println(account.balance)

account.withdraw(50.0)
println(account.balance)
```

위 BankAccount 에는 계좌에 돈이 얼마 있는지 나타내는 상태가 있다. 상태를 갖게 하는 것은 양날의 검이다.

시간의 변화에 따라서 변하는 요소를 표현할 수 있다는 것은 유용하지만 상태를 적절하게 관리하는 것이 꽤나 어렵다.

1. 프로그램을 이해하고 디버그하기 힘들어진다.
2. 가변성이 존재하면 코드의 실행을 추론하기 어려워진다.
3. 멀티스레드 프로그램일 때는 적절한 동기화가 필요하다.
4. 테스트 하기 어렵다.
5. 상태 변경이 일어날 때 이러한 변경을 다른 부분에 알려야하는 경우가 다수 존재한다.

# 문제가 되는 코드 II

```kotlin
var num = 0

for (i in 1..1000) {
  thread {
    Thread.sleep(10)
    num += 1
  }
}

Thread.sleep(5000)

// num 은 1000 을 예상하지만 아닐 확률이 매우 높다.
// 실행할 때마다 다른 숫자가 나온다.
print(num)
```

# 문제가 되는 코드 III

코틀린의 코루틴을 활용하면 더 적은 스레드가 관여되므로 충돌과 관련된 문제가 줄어든다. 하지만 문제 자체가 사라지는 것은 아니다.

```kotlin
suspend fun main() {
  var num = 0

  coroutineScope {
    for (i in 1..1000) {
      launce {
        delay(10)
        num += 1
      }
    }
  }
  // num 은 1000 을 예상하지만 아닐 확률이 매우 높다.
  // 실행할 때마다 다른 숫자가 나온다.
  print(num)
}
```
실제 프로젝트에서 사용하면 안되는 코드. 일부 연산이 충돌되어 사라지므로 적절하게 추가로 동기화를 구현해야 한다.

예를 들어 다음 코드를 살펴보자. 동기화를 잘 구현하는 것은 괴앚ㅇ히 어려운 일이며 변할 수 있는 지점이 많아지면 많아질수록 훨씬 더 어려워지게 된다.

따라서 변할수 있는 지점을 제한하는 것이 좋다.

### 문제가 되는 코드 III 을 Lock 을 이용해 동기화하기

```kotlin
val lock = Any()
var num = 0
for (i in 1..1000) {
  thread {
    Thread.sleep(10)
    synchronized(lock) {
      num += 1
    }
  }
}

Thread.sleep(1000)

print(num) // 1000
```

가변성을 완벽하게 제한하라는 말은 아니다. 가변성은 시스템의 상태를 나타내기 위한 중요한 방법이다. 하지만 변경이 일어나는 부분을 신중하고 확실하게 결정하고 사용하자.

# 코틀린에서 가변성 제한하기

코틀린은 가변성을 제한 할 수 있게 설계되었다. 그래서 immutable(불변) 객체를 만들거나 프로퍼티를 변경할 수 없게 막는 것이 굉장히 쉽다.

- 읽기 전용 프로퍼티(val)
- 가변 컬렉션과 읽기 전용 컬렉션 구분
- 데이터 클래스의 copy

### 읽기 전용 프로퍼티(val)

코틀린은 val 을 사용해 읽기 전용 프로퍼티를 만들 수 있다. 이렇게 선언된 프로퍼티는 마치 값(value)처럼 동작하며 일반적인 방법으로는 값이 변하지 않는다.

- `val` - 읽기 전용(immutable)
- `var` - 읽고 쓰기 전용(mutable)

하지만 주의해야 할 점이 있다. 읽기 전용 프로퍼티가 <u>완전히 변경이 불가능하다라고 판단하는 것</u>은 상당히 위험한 일이다.

만약 읽기 전용 프로퍼티가 가변성(mutable)의 객체를 담고 있다면 내부적으로는 *변할 가능성*이 있다.

```kotlin
val list = mutableListOf(1,2,3)
list.add(4)

print(4) // [1,2,3,4]
```

*읽기 전용 프로퍼티는 사용자 정의 게터*로도 정의할 수 있다:

```kotlin
var name: String = "Marcin"
var surname: String = "Moskala"

val fullname
  get() = "$name $surname"

fun main() {
  println(fullName) // Marcin Moskala
  name = "Maja"
  println(fullName) // Maja Moskala
}
```

코틀린의 프로퍼티는 기본적으로 캡슐화되어 있고 추가적으로 사용자 정의 접근자(Getter, Setter)를 가질 수 있다. 이러한 특성으로 코틀린은 API 를 변경하거나 정의할 때 굉장히 유연하다.

`var` 는 게터와 세터를 모두 제공하지만 `val` 은 변경이 불가능하므로 게터만 제공한다. 따라서 포함 관계는 `val < var` 이므로 읽기 전용 프로퍼티를 읽고 쓰기 전용 프로퍼티로 오버라이드 할 수 있다.

```kotlin
interface Element {
  val active: Boolean
}

class ActualElement: Element {
  override var active: Boolean = false
}
```

일반적으로 읽기 전용 프로퍼티(val)의 값이 변경될 수 있긴 하지만 프로퍼티의 레퍼런스 자체를 변경할 수 없으므로 동기화 문제 등을 줄일 수 있다. 일반적으로 var 보다 val 을 많이 사용한다.

### 완전히 변경할 필요가 없다면 final 키워드를 사용하자

final 키워드를 사용하면 완전한 불변성을 가질 수 있다.

```kotlin
val name: String? = "Marton"
val surname: String = "Braun"

val fullName: String?
  get() = name?.let { "$it $surname" }

val fullName2: String? = name?.let { "$it $surname" }

fun main() {
  if (fullName != null) {
    println(fullName.length) // 오류
  }

  if (fullName2 != null) {
    println(fullName2.length) // Marton Braun
  }
}
```

`fullName` 은 게터로 정의했으므로 스마트 캐스트를 사용할 수 없다. 게터를 활용하므로 값을 사용하는 시점의 `name` 에 따라 다른 결과가 나올 수 있기 때문이다.

`fullName2` 처럼 지역 변수가 아닌 프로퍼티가 `final` 이고 사용자 정의 게터를 갖지 않을 경우 스마트 캐스트할 수 있다.

### 가변 컬렉션과 읽기 전용 컬렉션 구분하기

위에서 살펴본 것 처럼 코틀린은 읽고 쓸 수 있는 프로퍼티(var) 와 읽기 전용 프로퍼티로 구분된다. 컬렉션도 이와 같다.

- 읽기 전용 컬렉션
- 읽고 쓰기 전용 컬렉션

이와 나뉘어진 이유는 컬렉션 계층이 설계된 방식 때문이다.

<<그림>>

왼쪽은 읽기 전용이고 오른쪽은 읽고 쓰기 전용이다. 읽고 쓰기 전용 컬렉션이 읽기 전용 컬렉션 인터페이스를 확장하기 때문에 읽고 쓸 수 있는 것이다.

하지만 읽기 전용 컬렉션이 위에서 말한 것처럼 내부의 값을 변경할 수 없다는 것은 아니다. 대부분의 경우에는 변경할 수 있으나 읽기 전용 인터페이스가 이를 지원하지 않으므로 변경할 수 없다.

예를 들어 `Iterable<T>.map` 과 `Iterable<T>.filter` 함수는 `ArrayList` 를 리턴한다. `ArrayList` 는 변경할 수 있는 리스트이다.

다음 코드는 Iterable<T>.map 의 구현을 단순하게 나타낸 것이다.

```kotlin
inline fun <T, R> Iterable<T>.map(transformation: (T) -> R): List<R> {
  val list = ArrayList<R>()
  
  for (elem in this) {
    list.add(transformation(elem))
  }

  return list
}
```

이러한 컬렉션은 완벽한 불변성(Immutable)로 설계하지 않고 읽기 전용 컬렉션으로 설게했다. 이러한 점으로 이 컬렉션은 더 많은 유연성을 얻을 수 있었다.

내부적으로 인터페이스를 사용하고 있으므로 실제 컬렉션을 리턴할 수 있다. 따라서 플랫폼 고유의 컬렉션을 사용할 수 있게 되었다.

이는 코틀린이 내부적으로 `Immutable` 하지 않은 컬렉션을 외부적으로 `Immutable` 처럼 보이게 만들어서 얻어지는 안전성이다.

하지만 개발자가 시스템 해킹을 시도해서 다운 캐스팅(down cast)을 할 때 문제가 된다. 실제로 코틀린 프로젝트를 진행하면 이를 허용해서는 절대 안되는 큰 문제다.

리스트를 읽기 전용으로 리턴하면 이를 읽기 전용으로만 사용해야 한다. 이를 단순한 계약의 문제라고 할 수 있다.

컬렉션 다운 캐스팅은 이러한 계약을 위반하고 추상화를 무시하는 행위이다. 이런 코드는 안전하지 않고 예측하지 못한 결과를 초래한다.

```kotlin
val list = listOf(1,2,3)

if (list is MutableList) {
  list.add(4)
}
```

이 코드의 실행 결과는 플랫폼에 따라 다르다. JVM 에서 listOf 는 자바의 List 인터페이스를 구현한 Array.ArrayList 인스턴스를 리턴한다. 자바의 List 인터페이스는 add, set 과 같은 메서드를 제공한다 따라서 코틀린의 MutableList 로 변경할 수 있다. 하지만 Arrays.ArrayList 는 이러한 연산을 구현하고 있지 않다. 따라서 다음과 같은 에러가 발생한다.

```shell
Exception in thread "main"
java.lang.UnsupportedOperationException
at java.util.AbstractList.add(AbstractList.java:148)
at java.util.AbstractList.add(AbstractList.java:108)
```

따라서 코틀린에서 읽기 전용 컬렉션을 mutable 컬렉션으로 다운 캐스팅하면 안된다. 읽기 전용에서 mutable 로 변경해야 한다면 복제를 통해서 새로운 mutable 컬렉션을 만드는 list.toMutableList 를 활용해야 한다.

```shell
val list = listOf(1,2,3)

val mutableList = list.toMutableList()

mutableList.add(4)
```

이렇게 코드를 작성하면 어떠한 규약도 어기지 않을 수 있으며 기존의 객체는 여전히 immutable 이라 수정할 수 없으므로 안전하다고 할 수 있다.

### 데이터 클래스의 copy

String 이나 Int 처럼 내부적인 상태를 변경하지 않는 `Immutable` 객체를 많이 사용하는데에는 이유가 있다.

`Immutable` 을 사용하면:

- 한번 정의된 상태가 유지되므로 코드를 이해하기 쉽다.
- `Immutable` 객체는 공유했을 때도 충돌이 따로 이루어지지 않으므로 병렬 처리를 안전하게 진행할 수 있다.
- `Immutable` 객체에 대한 참조는 변경되지 않으므로 쉽게 캐시할 수 있다.
- `Immutable` 객체는 방어적 복사본(depensive copy)을 만들 필요가 없다. 또한 객체를 복사할 때 깊은 복사를 따로 하지 않아도 된다.
- `Immutable` 객체는 다른 객체(mutable 또는 immutable)를 만들 때 활용하기 좋다. `Immutable` 는 또한 실행을 더 쉽게 예측할 수 있다.
- `Immutable` 객체는 set() 또는 map() 의 키로 사용할 수 있다. 참고로 `mutable` 객체는 이러한 것으로 사용할 수 없다. 이는 셋과 맵이 내부적으로 해시 테이블을 사용하고 해시 테이블은 처음 요소를 넣을 때 요소의 값을 기반으로 버킷을 결정하기 때문이다. 따라서 요소에 수정이 일어나면 해시 테이블 내부에서 요소를 찾을 수 없게 되어버린다.

```kotlin
val names: SortedSet<FullName> = TreeSet()
val person = FullName("AAA", "AAA")

names.add(person)
names.add(FullName("Jordan", "Hansen"))
names.add(FullName("David", "Blanc"))

print(names) // [AAA AAA, David Blanc, Jordan Hansen]
print(person in names) // true

person.name = "ZZZ"
print(names) // [ZZZ AAA, David Blanc, Jordan Hansen]
print(person in names) // false
```

마지막 출력을 보면 셋 내부에 해당 객체가 있음에도 false 를 리턴하는 것을 확인할 수 있다. 객체를 변경했기 때문에 찾을 수 없는 것이다.

지금까지 살펴본 것처럼 mutable 객체는 예측하기 어려우며 위험하다는 단점이 있다. 반면 immutable 객체는 변경할 수 없다는 단점이 있다. 따라서 immutable 객체는 자신의 일부를 수정한 새로운 객체를 만들어 내는 메서드를 가져야 한다. 예를 들어 Int 는 immutable 이다. 그래도 Int 는 내부적으로 plus 와 minus 메서드로 자신을 수정한 새로운 Int 를 리턴할 수 있다. Iterable 도 읽기 전용이다.

그래도 map 과 filter 메서드로 자신을 수정한 새로운 Iterable 객체를 만들어서 리턴한다. 우리가 직접 만드는 immutable 객체도 비슷한 형태로 작동해야 한다.

예를 들어 User 라는 immutable 객체가 있고 성을 변경해야 한다면 withSurname 과 같은 메서드를 제공해서 자신을 수정한 새로운 객체를 만들어 낼 수 있게 해야한다.

```kotlin
class User(val name: String, val surname: String) {
  fun withSurname(surname: String) = User(name, surname)
}

fun main() {
  var user = user("MaJa", "Oscar")
  user = user.withSurname("Moskba")
  print(user) // User(name=MaJa, surname=Oscar)
}
```

다만 이러한 모든 프로퍼티를 대상으로 이런 함수를 하나하나 만드는 것은 굉장히 귀찮은 일이다. 그럴 때는 data 한정자를 사용하면 된다. data 한정자는 copy 라는 이름의 메서드를 만들어 준다. copy 메서드를 활용하면 모든 기본 생성자 프로퍼티가 같은 새로운 객체를 만들어 낼 수 있다. 이외에 data 한정자로 만들어지는 메서드 등은 아이템 37: 데이터 집합 표현에 data 한정자를 사용하라 에서 살펴보자

```kotlin
data class User(val name: String, val surname: String)

fun main() {
  var user = user("MaJa", "Oscar")
  user = user.withSurname("Moskba")
  print(user) // User(name=MaJa, surname=Oscar)
}
```

코틀린에서는 이와 같은 형태로 immutable 특성을 가지는 데이터 모델 클래스를 만든다.

변경을 할 수 있다는 측면만 보면 mutable 객체가 더 좋아 보이지만 이렇게 데이터 모델 클래스를 만들어 immutable 객체로 만드는 것이 더 많은 장점을 가지므로 기본적으로는 이렇게 만드는 것이 좋다.

# 다른 종류의 변경 가능 지점

변경할 수 있는 리스트를 만들어야 한다고 해보자,. 다음과 같은 두 가지 선택지가 있다.

- mutable collection
- var 를 활용한 읽고 쓸 수 있느 프로퍼티

```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf()
```

두 가지 모두 변경할 수 있다. 다만 방식이 다르다.

```kotlin
list1.add(1)
list2.list2 + 1
```

물론 두 가지 코드 모두 다음과 같이 += 연산자를 활용해서 변경할 수 있지만 실질적으로 이루어지는 처리는 다르다.

```kotlin
list1 += 1 // list1.plusAssign(1)로 변경된다.
list2 += 1 // list2 = list2.plus(1)로 변경된다.
```

두 가지 모두 정상적으로 동작하지만 장단점이 존재한다. 두 가지 모두 변경 가능 지점이 있지만 그 위치가 다르다. 첫번째 코드는 구체적인 리스트 구현 내부에 변경 가능 지점이 있다.

멀티 스레드 처리가 이루어질 경우 내부적으로 적절한 동기화가 되어 있는지 확실하게 알 수 없으므로 위험하다. 두 번째 코드는 프로퍼티 자체가 변경 가능 지점이다. 따라서 멀티 스레드 안정성이 더 좋다고 할 수 있다(물론 잘못 만들면 일부 요소가 손실될 수도 있다)

```kotlin
var list = listOf<Int>()

for (i in 1..1000) {
  thread {
    list = list + i
  }
}

Thread.sleep(1000)

print(list.size) // 1000 이 되지 않는다.
```

mutable 리스트 대신 mutable 프로퍼티를 사용하는 형태는 사용자 정의 세터(또는 이를 사용하는 델리게이트)를 활용해서 변경을 추적할 수 있다. 예를 들어 Delegates.observable 을 사용하면 리스트에 변경이 있을 때 로그를 출력할 수 있다.

```kotlin
var names by Delegates.obserable(listOf<String>()) { _, old, new -> println("names changed from $old to $new") }

fun main() {
  names += "Fabio" // names 가 [] 에서 [Fabio] 로 변경됨.
  names += "Bill" // names 가 [Fabio] 에서 [Fabio, Bill] 로 변경됨.
}

mutable 컬렉션도 이처럼 관찰(observe) 할 수 있게 만드려면 추가적인 구현이 필요하다 따라서 mutable 프로퍼티에 읽기 전용 컬렉션을 넣어 사용하는 것이 더 쉽다.

이렇게 하면 여러 객체를 변경하는 여러 메서드 대신 세터를 사용하면 되고, 이를 private 으로 만들 수도 있기 때문이다.

```kotlin
var announcements = listOf<Announcement>()
  private set
```

mutable 컬렉션을 사용하는 것이 처음에는 더 간단하게 느껴지겠지만, mutable 프로퍼티를 사용하면 객체 변경을 제어하기가 더 쉽다. 참고로 최악의 방식은 프로퍼티와 컬렉션을 모두 변경 가능한 지점으로 만드는 것이다.

```kotlin
// DO NOT WRITE THIS DOWN AGAIN
var list3 = mutableListOf<Int>()
```

이렇게 코드를작성하면 변경될 수 있는 두 지점 모두에 대ㅏㄴ 동기화를 구현해야 한다. 또한 모호성이 발생해서 += 를 사용할 수 없게 된다.

상태를 변경할 수 있는 불필요한 방법은 만들지 말자. 상태를 변경하는 모든 방법은 코드를 이해하고 유지해야 하므로 비용이 발생한다. 따라서 가변성을 제한하는 것이 좋다.

# 변경 가능 지점 노출하지 말기

상태를 나타내는 mutable 객체를 외부에 노출하는 것은 굉장히 위험하다. 간단한 예를 살펴보자.

```kotlin
data class User(val name: String)

class UserRepository {
  private val storedUsers: MutableMap<Int, String> =
    mutableMapOf()

  fun loadAll(): MutableMap<Int, String> {
    return storedUsers
  }
}
```

loadAll 을 사용해서 private 상태인 UserRepository  를 수정할 수 있다.

```kotlin
val userRpeository = UserRepository()

val storedUsers = userRepository.laodAll()

storedUsers[4] = "Kirill"
```

이러한 코드는 돌발적인 수정이 일어날 때 위험할 수 있다. 이를 처리하는 방법은 두 가지다. 첫번째는 리턴되는 mutable 객체를 복제하는 것이다.

이를 방어적 복제(defensive copying)이라고 부른다. data 한정자로 만들어지는 copy 메서드를 활용하면 좋다.

```kotlin
class UserHolder {
  private val user: MutableUser()

  fun get(): MutableUser {
    return user.copy()
  }
}
```

지금까지 언급했던 것처럼 가능하다면 무조건 가변성을 제한하는 것이 좋다. 컬렉션은 객체를 읽기 전용 슈퍼 타입으로 업캐스트하여 가변성을 제한할 수도 있다.

```kotlin
data class User(val name: String)

class UserRepository {
  private val storedusers: MutableMap<Int, String> = mutableMapOf()

  fun laodAll(): Map<Int, String> {
    return storedusers
  }
}
```

# 정리

- var 보다는 val 을 사용하는 것이 좋다.
- mutable 프로퍼티보다는 immutable 프로퍼티를 사용하는 것이 좋다.
- mutable 객체와 클래스보다는 immutable 객체와 클래스를 사용하는 것이 좋다.
- 변경이 필요한 대상을 만들어야 한다면, immutable 데이터 클래스로 만들고 copy 를 활용하는 것이 좋다.
- 컬렉션에 상태를 저장해야 한다면 mutable 컬렉션보다는 읽기 전용 컬렉션을 사용하는 것이 좋다.
- 변이 지점을 적절하게 설계하고 불필요한 변이 지점은 만들지 않는 것이 좋다.
- mutable 객체를 외부에 노출하지 않는 것이 좋다.

다만 몇 가지 예외가 있다. 가끔 효율성 때문에 immutable 객체보다 mutable 객체를 사용하는 것이 좋을 때가 있다.

이러한 최적화는 코드에서 성능이 중요한 부분에서만 사용하는 것이 좋다.
