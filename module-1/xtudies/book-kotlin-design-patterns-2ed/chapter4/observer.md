# Design Pattern: Observer Pattern

아래의 예시는 `Cat` 이라는 주체가 관찰자를 등록하고, 관찰자를 해제하며, 관찰자에게 이벤트를 발송하는 예제를 담고 있다.

간단한 예제이지만, Map 에 키로 함수를 등록하고, 값 또한 함수로 등록하여 메세지 발송 시 루프를 돌아 함수를 실행시키는 구조를 가지고 있다.

주체(Subject): `Cat`

```kotlin
class Cat {
    private val participants = mutableMapOf<() -> Unit, () -> Unit>()

    /*
      * 옵저버 등록
    */
    fun joinChoir(whatToCall: () -> Unit) {
        participants[whatToCall] = whatToCall
    }

    /*
      * 옵저버 해제
    */
    fun leaveChoir(whatNotToCall: () -> Unit) {
        participants.remove(whatNotToCall)
    }

    /*
      * 이벤트 발생 시 관찰자에게 알림
    */
    fun conduct() {
        for (p in participants.values) {
            p()
        }
    }
}
```

관찰자(Observer): `Bat`, `Dog`, `Turkey`

```kotlin
class Bat {
    fun screech(message: Message) {
        when (message) {
          is HighMessage -> {
            for (i in 1..message.repeat) {
              println("${message.pitch} 이----")
            }
          }
          else -> println("낼 수 없는 소리에요 :(")
        }
    }
}

class Turkey {
    fun gobble() {
        println("Gob-gob")
    }
}

class Dog {
    fun bark() {
        println("Woof")
    }

    fun howl() {
        println("Auuuu")
    }
}
```

메세지(Message): SoundPitch

```kotlin
typealias Times = Int

enum class SoundPitch { HIGH, LOW }

interface Message {
    val repeat: Times
    val pitch: SoundPitch
}

data class LowMessage(override val repeat: Times) : Message {
    override val pitch = SoundPitch.LOW
}

data class HighMessage(override val repeat: Times) : Message {
    override val pitch = SoundPitch.HIGH
}
```

사용: Main.kt

```kotlin
fun main() {
    val catTheConductor = Cat()
    val bat = Bat()
    val dog = Dog()
    val turkey = Turkey()

    catTheConductor.joinChoir(bat::screech)   // 박쥐 등록
    catTheConductor.joinChoir(dog::howl)      // 개 하울링 등록
    catTheConductor.joinChoir(dog::bark)      // 개 짖기 등록
    catTheConductor.joinChoir(turkey::gobble) // 칠면조 등록
    catTheConductor.conduct()                 // * 이벤트 publish
    catTheConductor.conduct()                 // * 이벤트 publish
}
```