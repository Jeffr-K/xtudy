# 생성 패턴 이용하기

객체 관리와 변경에 쉽게 대응하며 유지보수하기 쉬운 코드를 작성하기 위해 이번절에서는 생성형 디자인 패턴을 학습한다.

- 싱글톤 패턴
- 팩토리 메서드 패턴
- 추상 팩토리 패턴
- 빌더 패턴
- 프로토타입 패턴

# 싱글톤 패턴

싱글톤 패턴의 주요 개념은:

- 시스템에 인스턴스가 하나만 존재해야 한다
- 시스템의 모든 부분에서 인스턴스에 접근할 수 있어야 한다.

위 요구사항을 충족하기 위해 다음과 같은 요구사항이 존재한다:

1. 게으른 인스턴스 생성(lazy)
2. 스레드가 안전한 인스턴스 생성
3. 고성능의 인스턴스 생성

##### 게으른 인스턴스 생성

프로그램이 시작되자마자 싱글톤 인스턴스가 만들어지면 안된다. 인스턴스 생성에 많은 비용이 들 수 있기 때문이다.

따라서 인스턴스 생성은 필요한 첫 순간에 이루어져야 한다.

##### 스레드 안전한 인스턴스 생성

두 스레드가 동시에 싱글톤 객체를 생성하려고 할 때 두 스레드가 같은 인스턴스를 획득해야 한다.

##### 고성능의 인스턴스 생성

많은 스레드가 동시에 싱글톤 객체를 생성하려고 할 때 스레드를 너무 오래 기다리게 해서는 안된다.

잘못하면 실행이 중단될 수 있기 때문이다.

### 코틀린에서는 어떻게 싱글톤을 구현할 수 있을까?

자바와 C++ 에서 이를 구현하기 위해서 상당히 많은 코드가 필요했다. 하지만 코틀린에서는 싱글톤 객체 생성을 위해 object 라는 키워드가 도입됐다.

싱글톤은 일반적인 클래스와 동일한 방법으로 선언하되 생성자는 정의하지 않는다는 것이 중점이다.

object 키워드를 이용해 클래스를 작성하고 생성자를 별도로 두지 않는다.

```kotlin
object SingletonObject {
  init {
    println("싱글톤 객체에 처음 접근했다.")
  }
}
```

`init` 은 별도로 구성하지 않아도 된다. 이를 초기화 하는 코드는 다음과 같다.

```kotlin
val OnlyOneSingletonObject = SingletonObject
```

# 팩토리 메서드 패턴

팩토리 메서드는 객체를 생성하는 메서드에 관한 디자인 패턴이다.

하지만 의문을 가질 수 있다. 생성자만으로는 부족할까?

만약 실무에서 `XML` 또는 `JSON` 과 같은 형태나 타입이 다른 파일을 파싱해야 한다고 생각해보자.

팩토리 메서드 패턴이 아니라면 다음과 같이 작성해야 할 것이다.

```kotlin
class XMLParser {}

class JSONParser {}
```

하지만 우리가 팩토리 메서드 패턴을 잘 알고 있다면 다음과 같이 작성할 수 있다:

```kotlin
class Parser {
  
  createParser(files: Any, type: String): IParsedResult {
    when (type) {
      'JSON' -> JSONParsedResult(files, type)
      'XML' -> XMLParsedResult(files, type)
      else -> throw RuntimeException("알 수 없는 포맷: $type")
    }
  }
}
```

중간에 코드가 몇개 빠져있긴 하지만 반환값을 구현하는 여러 구현체를 입맛대로 구현할 수 있다는 점에서 유용한 패턴이다.

# 정적 팩토리 메서드 패턴

팩토리 메서드 패턴과 이름이 비슷해서 자주 햇갈리는 디자인 패턴이다. 이 패턴은 Effective Java 3/E 에서 나온 패턴이다.

이 패턴을 이해하기 위해 자바의 표준 라이브러리에 있는 valueOf() 메서드를 살펴보겠다.

자바에는 문자열로부터 Long 객체(64bit 정수)를 만드는 방법이 두 가지 존재한다.

```java
// 1. 생성자를 이용하는 방법
Long l1 = new Long("1");

// 2. 정적 팩토리 메서드를 이용하는 방법
Long l2 = Long.valueOf("1");
```

왜 생성자를 이용하는 대신 정적 팩토리 메서드를 이용하는 걸까?

정적 팩토리 메서드 패턴을 이용하면 다음과 같은 장점을 가진다:

1. 다양한 생성자에 명시적인 이름을 붙일 수 있다. 클래스에 생성자가 많은 경우에 특히 유용하다.
2. 일반적으로 생성자에서는 예외가 발생하지 않으리라는 기대가 있다. 그러나 클래스 인스턴스 생성이 절대 실패하지 않는 것은 아니다. <u>예외가 불가피하면 생성자보다 일반적인 메서드에서 발생하는 편</u>이 훨씬 낫다.
3. 생성자에 기대하는 것이 한 가지 더 있다면 빠르다는 것이다. 그러나 <u>생성하는 데에 시간이 오래 걸릴 수밖에 없는 객체</u> 또한 존재한다. 그런 경우 생성자 대신 정적 팩토리 메서드를 고려하는 것이 낫다.

이외에 정적 팩토리 메서드에는 기술적 장점 또한 존재한다:

1. 캐시
2. 하위 클래스 생성

##### 캐시

정적 팩토리 메서드를 사용하면 캐시를 적용할 수 있다. 실제로 Long 도 캐시를 한다. valueOf() 함수는 모든 값에 대해 항상 새 객체를 반환하는 대신 이미 파싱한 적이 있는 값인지 확인한다.

만약 파싱한 적이 있다면 캐시된 객체를 반환한다. 같은 값으로 정적 팩토리 메서드를 반복 호출하면 생성자를 사용하는 것에 비해 가비지 컬렉션을 해야 하는 객체가 덜 생긴다.

##### 하위 클래스 생성

생성자를 호출하면 항상 그 클래스의 인스턴스를 얻는다. 그러나 정적 팩토리 메서드에는 그런 제한이 없다.

해당 클래스의 인스턴스를 생성할 수도 있지만, 그 하위 클래스의 인스턴스를 만들어 낼 수도 있다.

### 코틀린에서 정적 팩토리 메서드는 어떻게 구현할까?

앞 절에서는 object 키워드를 이용해서 싱글톤 패턴을 구현했다. 이번에는 object 키워드를 이용해 동반 객체를 만드는 방법에 대한 것이다.

우선, 자바에서는 정적 팩토리 메서드는 static 을 이용해 선언한다. 그러나 코틀린에는 그런 키워드가 없기 때문에 인스턴스에 속하지 않는 메서드는 동반 객체 내부에 선언할 수 있다.

```kotlin
class Server(port: Long) {
  init {
    println("$port 포트에서 서버가 시작되었다.")
  }
  
  companion object {
    fun withPort(port: Long) = Server(port)
  }
}
```

companion 키워드를 이용해 동반 객체를 생성하면 싱글톤 패턴과 달리 객체를 패키지 수준이 아닌 클래스 내부 선언을 진행했다.

이 객체는:

1. 게으르게(lazy) 생성된다.
2. 인스턴스의 생성을 숨기고 메서드로 생성의 책임을 위임할 수 있다.

```kotlin
class Server private constructor(port: Long) {
  companion object Starter {
    fun withPort(port: Long) = Server(port)
  }
}
```

사용은 다음과 같이 할 수 있다:

```kotlin
// 컴파일 실패
val server = Server(8080)
// 컴파일 성공
val server = Server.Starter.withPort(8080)
```

# 추상 팩토리 패턴

추상 팩토리 패턴이란, 쉽게 말해서, 팩토리 패턴을 만들어 내는 팩토리 패턴이라고 생각하면 된다. 즉 팩토리 패턴은 다른 클래스를 만들어내는 함수 또는 클래스인데 이 추상 팩토리 패턴은 이러한 여러 팩토리 메서드를 감싸는 클래스이기 때문이다.

예제를 위해 다음과 같은 설정 파일이 있다고 가정한다.

```yaml
server:
  port: 8080
environment: production
```

이 설정 파일을 읽어서 객체를 생성해보자.

```kotlin
interface Property {
  val name: String
  val value: Any
}

interface ServerConfiguration {
  val properties: List<Property>
}

data class PropertyImpl(
  override val name: String,
  override val value: Any
) : Property

data class ServerConfigurationImpl(
  override val properties: List<Property>
) : ServerConfiguration
```

첫번째 팩토리 생성

```kotlin
fun property(prop: String): Property {
  val (name, value) = prop.split(":")
  return when (name) {
    "port" -> PropertyImpl(name, value.trim().toInt())
    "environment" -> PropertyImpl(name, value.trim())
    else -> throw RuntimeException("알 수 없는 속성: $name")
  }
}
```

다른 언어들과 마찬가지로 trim() 함수는 문자열에서 공백을 제거하는 함수다. 이제 이 서비스의 포트와 환경을 나타내는 두 속성을 만들어 보자.

```kotlin
val portProperty = property("port: 8080")
val environment = property("environment: production")
```

이 코드에는 작은 문제가 있는데 문제를 파악하기 위해 port 속성의 값을 다른 변수에 저장해보자.

```kotlin
val port: Int = portProperty.value
```

팩토리 메서드에서 port 가 Int 타입으로 파싱된 것을 확인했다. 그러나 이 정보는 사라져 버렸다.

value 가 Any 타입으로 선언돼 있기 때문이다. 그래서 value 는 String 일 수도 있고 Int 일수도 있고 어떤 타입일 수도 있다.

##### 캐스팅

타입 언어에서 캐스팅은 컴파일러가 추론한 타입 대신 프로그래머가 지정한 타입을 사용하도록 강제하는 것이다.

값의 타입이 무엇인지 확실히 알 경우:

```kotlin
val port: Int = portProperty.value as Int
```

안전하지 않은 캐스팅을 이용할 수 있다.

반면 안전한 캐스팅은:

```kotlin
val port: Int? = portProperty.value as? Int
```

이 경우 프로그램이 예기치 못한 상황에서 다운되는 경우는 피할 수 있지만 명시적으로 null 처리를 해주어야 한다.

##### 하위 클래스 생성

이번에는 캐스팅 말고 다른 접근을 시도해본다. Any 타입의 값을 갖는 단일 구현체를 사용하는 대신 2개의 구분된 구현체를 사용할 것이다.

```kotlin
data class IntProperty(
  override val name: String,
  override val value: Int
) : Property

data class StringProperty(
  override val name: String,
  override val value: String
) : Property
```

이 두 클래스 중 하나를 리턴하려면 앞서 구현한 팩토리 메서드를 약간 수정해야 한다.

```kotlin
fun property(prop: String): Property {
  val (name, value) = prop.split(":")
  return when (name) {
    "port" -> IntProperty(name, value.trim().toInt())
    "environment" -> StringProperty(name, value.trim())
    else -> throw RuntimeException("알 수 없는 속성: $name")
  }
}
```

이렇게 구성하고 컴파일을 시도하면 에러가 난다. 여전히 컴파일러는 파싱된 속성을 Any 타입으로만 알고 있을 뿐이다.

```shell
Type mismatch: inferred type is Any but Int was expected
```

##### 스마트 캐스팅

위의 문제를 해결하기 위해 스마트 캐스팅을 사용해 볼 수 있다. `is` 키워드를 사용하면 객체의 타입을 검사할 수 있는데:

```kotlin
println(portProperty is IntProperty) // true
```

이 `is` 키워드와 `if` 조건절을 사용하면 이 캐스팅을 자동으로 해준다. 더 이상 컴파일 오류는 발생하지 않고, null 또한 체크할 필요가 없다.

만약 스마트 캐스팅이 실패하면 `null` 을 반환한다.

```kotlin
val port: Int? = portProperty.value as? Int

if (port != null) {
  val port: Int = port
}
```

만약 `port` 가 `null` 인지 검사해서 `null` 이 아니라는 것을 확인했다면 `port` 는 자동으로 `null` 불가 타입으로 캐스팅 된다.

##### 변수 가리기

만약 변수 가리기가 없었다면 어떤 코드가 됐을지 상상해본다:

```kotlin
val portOrNull: Int? = portProperty.value as? Int

if (portOrNull != null) {
  val port: Int = port
}
```

위 코드처럼 2개를 선언해야 한다. 다음과 같은 단점을 상쇄한다:

1. 변수 이름이 상당히 길어진다.
2. `if` 조건절 이후 변수는 사용할 일이 없어진다.

##### 팩토리 메서드의 모음

이번엔 배운 내용을 토대로 두번째 팩토리 메서드를 구현해보자.

```kotlin
fun server(propertyStrings: List<String>): ServerConfiguration {
  val parsedProperties = mutableListOf<Property>()
  for (p in propertyStrings) {
    parsedProperties += property(p)
  }
  return ServerConfigurationImpl(parsedProperties)
}
```

이 메서드는 앞서 구현한 property() 팩토리 메서드를 활용해 입력으로 받은 설정 파일의 각 줄을 Property 객체로 변환한다.

```kotlin
println(server(listOf("port: 8080", "environment: production")))
```

```kotlin
> ServerConfigurationImpl(properties=[IntProperty(name=port, value=8080), StringProperty(name=environment, value=production)])
```

이 두 메서드는 서로 연관되어 있기 때문에 같은 클래스에 두는 것이 좋다. 이 클래스를 Parser 라고 부르자.

여기서는 파일을 실제로 파싱하지는 않고 이미 내용이 줄 단위로 주어진다고 가정한다.

그러나 파일의 내용을 실제 읽는 작업도 그리 어렵지는 않다.

```kotlin
class Parser {
  companion object {
    fun property(prop: String): Property {
      // ...
    }

    fun server(propertyStrings: List<String>): ... {
      // ...
    }
  }
}
```

이 패턴을 사용하면 연관된 객체를 가족처럼 하나로 묶을 수 있다. 여기서는 ServerConfig 가 여러 속성의 부모 역할을 한다.

위 코드는 추상 팩토리를 구현하는 한 가지 방법에 지나지 않는다. 대신 인터페이스를 구현하는 방법도 있다.

```kotlin
interface Parser {
  fun property(prop: String): Property
  fun server(propertyStrings: List<String>): ServerConfiguration
}

class YAMLParser : Parser {
  // ... implementation
}

class JSONParser : Parser {
  // ... implementation
}
```

팩토리 메서드가 커져서 코드양이 많아질 때 유용하다.

### 실제 코드에서 추상 팩토리가 어디에서 쓰일까?

한 가지 예는 `java.util.Collection` 클래스다. 이 클래스에는 `emptyMap` 과 `emptyList` 그리고 `emptySet` 과 같은 메서드들이 존재하는데 모두 다른 클래스를 생성하지만 동시에 모두 집합 자료구조라는 공통점을 가지고 있다.

# 빌더 패턴

매우 단순한 객체는 생성자 하나로 충분할 때가 있다. 하지만 객체 생성이 굉장히 복잡하고 많은 매개변수를 사용할 때가 있는데, 이미 그러한 경우 더 나은 생성자를 사용할 수 있도록 하는 패턴인 `정적 팩토리 메서드 패턴` 을 알아보았다.

이번에는 복잡한 객체를 보다 쉽게 만들 수 있는 빌더 패턴이다.

우선 코틀린은 빌더 패턴을 <u>굳이 사용할 필요가 없다</u>라는 것이다.

`data` 키워드와 기본 매개변수 및 명명 매개변수를 잘 활용하면 객체의 생성을 <u>쉽게 생성할 수 있다.</u>

우리가 빌더 패턴을 사용하는 이유는 생성자에 필수값과 아닌 값을 넣어 생성의 다양함을 주고, 코드가 일관적이고 가시성있게 작성되면서 얻는 가독성을 주안으로 두고 있다.

또한 생성 시 순서대로 데이터를 넣어야하는 불편함도 개선하기 위해서다. 

##### `data` 키워드와 기본 매개변수 및 명명 매개변수를 활용한 객체 생성

```kotlin
data class Mail(
  val to: List<String>,
  val cc: List<String> = listOf(),
  val title: String = "",
  val message: String = "",
  val important: Boolean = false
)
```

이렇게 구성해놓고 사용할때에는:

```kotlin
val mail = Mail(
  to = listOf("oscar@gmail.com"),
  cc = emptyList(),
  title = "안녕하세요",
  message = "안녕하세요. 긴급한 요청이 있어서 메일 드렸습니다.",
  important = true
)
```

이런식으로 사용할 수 있다.

### Builder 패턴을 활용한 고전적인 객체 생성 방식

Spring boot 에서의 Lombok 기능을 사용하면 @Builder 가 존재하기에 조금 더 가시성 있는 코드가 만들어지지만 고전적인 방법으로 빌더를 만들어보자.

```kotlin
class MailBuilder internal construtor(
  val to: List<String>,
  val cc: List<String> = listOf(),
  val title: String = "",
  val message: String = "",
  val important: Boolean = false
) {
  fun message(message: String): MailBuilder {
    this.message = message
    return this
  }

  fun build(): Mail {
    if (to.isEmpty()) {
      throw RuntimeException("To 속성이 비어있다")
    }
    return Mail(to, cc, title, message, important)
  }
}
```

사용할 때에는:

```kotlin
val mail = MailBuilder()
  .to(listOf(""))
  .title("")
  .build()
```

처럼 사용할 수 있다.

### Fluent Setter 를 활용한 객체 생성 방식

```kotlin
data class Mail(
  val to: List<String>,
  private val _cc: List<String>? = null,
  private val _title: String = null,
  private val _message: String = null,
  private val _important: Boolean = null
) {
  fun message(message: String) = apply {
    _message = message
  }
}
```

여기서 apply 함수는 모든 코틀린 객체를 대상으로 호출할 수 있는 `시야 지정 함수(scoping function)` 중 하나다. 이 함수는 다음 코드와 같은 동작을 수행한다.

```kotlin
fun message(message: String): MailBuilder {
  this.message = message
  return this
}
```

따라서 mail 객체는 아래와 같이 사용된다.

```kotlin
val mail = Mail(listOf("oscar@gmail.com")).message("안녕")
```

또는 apply 함수를 이용하여:

```kotlin
val mail = Mail("oscar@gmail.com").apply {
  message = "Example"
  title = "Apply"
}
```

하지만 이 방법에도 몇 가지 단점이 존재한다.

- 선택적 인수를 모두 가변 필드로 선언해야 한다. 스레드 안전성을 위해 값을 추적하기 용이한 불변 필드를 사용하는 것이 더 낫다.
- 모든 선택적 인수가 null 을 가질 수 있다. 코틀린의 null 안정성을 위해 접근할때마다 체크하는 로직이 들어가야 한다.
- 문법이 너무 장황하다. 각 필드에 대해 똑같은 패턴을 계속 반복해야 한다.

# 프로토타입 패턴

프로토타입 디자인 패턴은 유사하면서도 조금 다른 객체를 그때 그때 목적에 맞게 생성하기 위해 사용한다.

예를 들어 사용자와 권한을 관리하는 시스템을 만든다고 상상해보자.

```kotlin
data class User(
  val name: String,
  val role: Role,
  val permissions: Set<String>,
) {
  fun hasPermission(permission: String) = permission in permissions
}

enum class Role {
  ADMIN,
  SUPER_ADMIN,
  REGULAR_USER
}
```

요구사항:

- 각 사용자는 하나의 역할을 가져야 한다.
- 각 역할은 여러 권한을 가질 수 있다.

이때 새로운 사용자를 만들면 동일한 역할을 갖는 다른 사용자와 비슷한 권한을 부여한다고 해보자.

```kotlin
val allUsers = this.userRepository.findAll()

fun createUser(name: String, role: Role) {
  for (user in allUsers) {
    if (u.role == role) {
      allUsers += User(name, role, u.permissions)
      return
    }
  }

  // 다른 권한을 갖는 다른 사용자가 존재하지 않는 경우 처리
}
```

만약 여기서 User 클래스에 tasks 라는 새로운 필드를 추가해야 한다면?

```kotlin
data class User(
  val name: String,
  val role: Role,
  val permissions: Set<String>,
  val tasks: List<String>
) {
  fun hasPermission(permission: String) = permission in permissions
}
```

이때 기존 코드에 똑같이 tasks 를 추가해주어야 한다.

```kotlin
allUsers += User(name, role, u.permissions, u.tasks)
```

<u>이런식으로 요구사항이 변경될 때마다 사용자 측의 코드를 매번 바꿔주어야 하는 번거로움이 존재한다.</u>

### 프로토타입 패턴으로 코드 변경에서 자유로워지자

프로토타입 패턴의 핵심 아이디어는 객체를 쉽게 복사할 수 있도록 하는 것이다. 적어도 다음의 두 가지 경우에 프로토타입 패턴이 필요하다.

- 객체 생성에 많은 비용이 드는 경우
- 비슷하지만 조금씩 다른 객체를 생성하느라 비슷한 코드를 매번 반복하고 싶지 않은 경우


```kotlin
fun createUser(_name: String, role: Role) {
  for (user in allUsers) {
    if (u.role == role) {
      allUsers += u.copy(name = _name)
      return
    }
  }

  // 다른 권한을 갖는 다른 사용자가 존재하지 않는 경우 처리
}
```

이처럼 빌더 패턴에서 봤던 것과 비슷하게 명명 인수를 사용해서 순서에 관계없이 속성을 설정할 수 있다. 또한 변경하고 싶은 속성만 지정해 주면 된다. 다른 데이터는 모두 그대로 복사된다.