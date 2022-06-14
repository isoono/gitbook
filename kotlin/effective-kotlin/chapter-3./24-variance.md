# 아이템 24 : 제네릭 타입과 variance 한정자를 활용하라

다음과 같은 제네릭 클래스가 있다고 하자.

```kotlin
class Cup<T>
```

타입 파라미터 T는 variance 한정자(out 또는 in)가 없으므로, 기본적으로 `**invariant(불공변성)**`이다. invariant라는 것은 **제네릭 타입으로 만들어지는 타입들이 서로 관련성이 없다는 의미**로 **자기 타입만 허용**하는 것이다.

예를 들어 Cup\<Int>와 Cup\<Number>, Cup\<Any>, Cup\<Nothing>은 어떠한 관련성도 갖지 않는다.

```kotlin
class Cup<T>

fun main() {
//    val anys: Cup<Any> = Cup<Int>() Type Mismatch
//    val nothings: Cup<Nothing> = Cup<Int>() Type Mismatch
}
```

만약 어떤 관련성을 원한다면, `**out**` 또는 `**in**`이라는 variance 한정자를 붙인다. `**out**`은 타입 파라미터를 `**covariant(공변성)**`로 만든다. 이는 A가 B의 서브 타입일 때, Cup\<A>가 Cup\<B>의 서브타입이라는 의미이다.

```kotlin
class Cup<out T>
open class Dog
class Puppy: Dog()

fun main() {
    val b: Cup<Dog> = Cup<Puppy>()
//    val a: Cup<Puppy> = Cup<Dog>() Type Mismatch
    
    val anys: Cup<Any> = Cup<Int>()
//    val nothings: Cup<Nothing> = Cup<Int>() Type Mismatch
}
```

`**in**` 한정자는 반대 의미이다. 타입 파라미터를 `**contravariant(반변성)**`으로 만든다. 이는 A가 B의 서브타입일 때, Cup\<A>가 Cup\<B>의 슈퍼타입이라는 것을 의미한다.

```kotlin
class Cup<in T>
open class Dog
class Puppy(): Dog()

fun main() {
//    val b: Cup<Dog> = Cup<Puppy>() Type Mismatch
    val a: Cup<Puppy> = Cup<Dog>()

//    val anys: Cup<Any> = Cup<Int>() Type Mismatch
    val nothings: Cup<Nothing> = Cup<Int>()
}
```

variance 한정자를 그림으로 나타내면 다음과 같다.

![Screen Shot 2022-05-31 at 4.29.07 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/564b30e1-74e9-42fa-b570-5d586a4f8894/Screen\_Shot\_2022-05-31\_at\_4.29.07\_PM.png)

## 함수 타입

함수 타입(아이템35)은 파라미터 유형과 리턴 타입에 따라서 서로 어떤 관계를 갖는다. 예를 들어 Int를 받고, Any를 리턴하는 함수를 파라미터로 받는 함수를 생각해보자.

```kotlin
fun printProcessedNumber(transition: (Int) -> Any) {
    println(transition(42))
}
```

(Int) → Any 타입의 함수는 (Int) → Number, (Number) → Any, (Number) → Number, (Number) → Int 등으로도 작동한다.

```kotlin
fun main() {
    val intToDouble: (Int) -> Number = { it.toDouble() }
    val numberAsText: (Number) -> Any = { it.toShort() }
    val identity: (Number) -> Number = { it }
    val numberToInt: (Number) -> Int = { it.toInt() }
    val numberHash: (Any) -> Number = { it.hashCode() }

    printProcessedNumber(intToDouble)
    printProcessedNumber(numberAsText)
    printProcessedNumber(identity)
    printProcessedNumber(numberToInt)
    printProcessedNumber(numberHash)
}
```

이는 이러한 타입들에 다음과 같은 관계가 있기 때문이다.

![Screen Shot 2022-06-04 at 8.48.23 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/91ae9e8d-f793-45e5-9c4c-d9f48926b637/Screen\_Shot\_2022-06-04\_at\_8.48.23\_AM.png)

위 그림에서 계층 구조의 아래로 가면, 타이핑 시스템 계층에서 파라미터 타입이 더 높은 타입으로 이동하고, 리턴 타입은 계층 구조의 더 낮은 타입으로 이동한다. (왼쪽에 있는 파라미터 타입은 Int → Number → Any, 오른쪽에 있는 리턴 타입은 Any → Number → Int로 점점 슈퍼타입이 된다.)

![Screen Shot 2022-06-04 at 8.50.01 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5f96d4f0-0b5f-4372-9bcf-e672c720ad3f/Screen\_Shot\_2022-06-04\_at\_8.50.01\_AM.png)

코틀린의 타입 계층

코틀린 함수 타입의 모든 파라미터 타입은 contravariant(반변성)이며, 모든 리턴 타입은 covariant(공변성)이다.

![Screen Shot 2022-06-04 at 8.51.05 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e305e961-dc0f-4c5c-99c2-cace20275edc/Screen\_Shot\_2022-06-04\_at\_8.51.05\_AM.png)

함수 타입을 사용할 때는 이처럼 자동으로 variance 한정자가 사용된다.

코틀린에서 자주 사용되는 것으로는 covariant(out 한정자)를 가진 List가 있다. 이는 variance 한정자가 붙지 않은 MutableList와 다르다. 왜 MutableList보다 List를 더 많이 사용하는지, 그리고 어떠한 부분이 다른 것인지는 variance 한정자의 안정성과 관련된 내용을 이해하면 알 수 있다.

## variance 한정자의 안정성

자바의 배열은 \*\*covariant(공변성)\*\*이다. 이유는 다양하지만, 배열을 기반으로 제네릭 연산자는 정렬 함수 등을 만들기 위해서라고 이야기 한다.

하지만 자바의 배열이 covariant라는 속성을 갖기 때문에 문제가 발생한다.

간단한 예로, 다음 코드는 컴파일 중에 문제가 없지만 런타임 오류가 발생한다.

```java
// java
Integer[] numbers = { 1, 4, 2, 1 };
Object[] objects = numbers;
objects[2] = "B"; // 런타임 오류 : ArrayStoreException
```

numbers를 Object\[ ]로 캐스팅해도 내부에서 사용되고 있는 실질적인 타입이 바뀌는 것은 아니며, 여전히 Integer이다. 따라서 이러한 배열에 String 타입의 값을 할당하면 오류가 발생한다.

이는 자바의 명백한 결함이다. 코틀린은 이러한 결함을 해결하기 위해서 Array(IntArray, CharArray 등)를 invariant(반변성)로 만들었다(Array\<Int>를 Array\<Any> 등으로 바꿀 수 없다).

파라미터 타입을 예측할 수 있다면, 어떤 서브타입이라도 전달할 수 있다. 따라서 아규먼트를 전달할 때, 암묵적으로 업캐스팅할 수 있다.

```kotlin
open class Dog
class Puppy: Dog()
class Hound: Dog()

fun takeDog(dog: Dog) {}

fun main() {
    takeDog(Dog())
    takeDog(Puppy())
    takeDog(Hound())
}
```

이는 covariant하지 않다. covariant 타입 파라미터(out 한정자)가 in 한정자 위치(예를 들어 타입 파라미터)에 있다면, covariant와 업캐스팅을 연결해서, 우리가 원하는 타입을 아무것이나 전달할 수 있다.

즉, value가 매우 구체적인 타입이라 안전하지 않으므로, value를 Dog 타입으로 지정할 경우, String 타입을 넣을 수 없다.

```kotlin
class Box<out T> {
    private var value: T? = null

// 코틀린에서는 사용할 수 없는 코드    
    fun set(value: T) {
        this.value = value
    }
    
    fun get(): T = value ?: error("Value not set")
}

fun main() {
    val puppyBox = Box<Puppy>()
    val dogBox: Box<Dog> = puppyBox
    dogBox.set(Hound())
    
    val dogHouse = Box<Dog>()
    val box: Box<Any> = dogHouse
    box.set("Some String")
    box.set(42)
}
```

이런 상황은 안전하지 않다. 캐스팅 후에 실질적인 객체가 그대로 유지되고, 타이핑 시스템에서만 다르게 처리되기 때문이다.

Int를 설정하려고 하는데, 해당 위치는 Dog만을 위한 자리이다. 만약 이것이 가능하다면, 오류가 발생한다. 그래서 코틀린은 public in 한정자 위치에 covariant 타입 파라미터(out)가 오는 것을 금지하여 이러한 상황을 막는다.

```kotlin
class Box<out T> {
    var value: T? = null // 오류
    
    fun set(value: T) { // 오류
        this.value = value
    }
    
    fun get(): T = value ?: error("Value not Set")
}
```

가시성을 private으로 제한하면 오류가 발생하지 않는다. 객체 내부에서는 업캐스트 객체에 covariant(out)를 사용할 수 없기 때문이다.

```kotlin
class Box<out T> { 
    private var value: T? = null

    private fun set(value: T) { 
        this.value = value
    }

    fun get(): T = value ?: error("Value not Set")
}
```

covariant(out)는 public out 한정자 위치에서도 안전하므로 따로 제한되지 않는다. 이러한 안정성의 이유로 생성되거나 노출되는 타입에만 covariant(out)를 사용하는 것이다.

좋은 예로 T는 covariant인 List\<T>가 있다. 지금까지 설명한 이유로 함수의 파라미터가 List\<Any?>로 예측된다면, 별도의 변환 없이 모든 종류를 파라미터로 전달할 수 있다.

다만 MutableList\<T>에서 T는 in 한정자 위치에서 사용되며, 안전하지 않으므로 invariant(불변성)이다.

```kotlin
fun append(list: MutableList<Any>) {
    list.add(42)
}

fun main() {
    val strs = mutableListOf<String>("A", "B", "C")
    append(strs) // 코틀린에서 사용 불가
    val str: String = strs[3]
    print(str)
}
```

또 다른 예로는 Response가 있다. Response를 사용하면 다양한 이득을 얻을 수 있다. 다음 코드 스니펫에서 어떻게 사용하는지 확인해보자.

variance 한정자 덕분에 다음 내용은 모두 참이다.

* Response\<T>라면 T의 모든 서브타입이 허용된다. 예를 들어 Response\<Any>가 예상된다면, Response\<Int>와 Response\<String>이 허용된다.
* Response\<T1, T2>라면 T1과 T2의 모든 서브타입이 허용된다.
* Failure\<T>라면, T의 모든 서브타입이 Failure가 허용된다. 예를 들어 Failure\<Number>라면, Failure\<Int>와 Failure\<Double>이 모두 허용된다.
* 다음 코드를 보자. covariant와 Nothing 타입으로 인해서 Failure는 오류 타입을 지정하지 않아도 되고, Success는 잠재적인 값을 지정하지 않아도 된다.

```kotlin
sealed class Response<out R, out E>
class Failure<out E>(val error: E): Response<Nothing, E>()
class Success<out R>(val value: R): Response<R, Nothing>()
```

covariant와 public in 위치와 같은 문제는 contravariant 타입 파라미터(in 한정자)와 public out 위치(함수 리턴 타입 또는 프로퍼티 타입)에서도 발생한다.

out 위치는 암묵적인 업캐스팅을 허용한다.

```java
open class Car
interface Boat
class Amphibious: Car(), Boat

fun getAmphibious(): Amphibious = Amphibious()

val car: Car = getAmphibious()
val boat: Boat = getAmphibious()
```

사실 이는 contravariant(in 한정자)에 맞는 동작이 아니다.

다음 코드를 보면, 어떤 상자(Box)에 어떤 타입이 들어 있는지 확실하게 알 수 없다.

```java
class Box<in T>(
    // 코틀린에서 사용할 수 없는 코드
    val value: T
)

val garage: Box<Car> = Box(Car())
val amphibiousSpot: Box<Amphibious> = garage
val boat: Boat = garage.value //Car 인스턴스가 들어가야 함

val noSpot: Box<Nothing> = Box<Car>(Car())
val boat: Nothing = noSpot.value
// 아무것도 만들 수 없다.
```

이러한 상황을 막기 위해, 코틀린은 contravariant 타입 파라미터(in 한정자)를 public out 한정자 위치에 사용하는 것을 금지한다.

```java
class Box<in T> {
    var value: T? = null //오류
    
    fun set(value: T) {
        this.value = value
    }
    
    fun get(): T = value //오류
        ?: error("Value not set")
}
```

이번에도 요소가 private일 때는 아무런 문제가 없다.

```java
class Box<in T> { 
    private var value: T? = null //오류

    fun set(value: T) {
        this.value = value
    }

    private fun get(): T = value //오류
        ?: error("Value not set")
}
```

이런 형태로 타입 파라미터에 contravariant(in 한정자)를 사용한다. 추가적으로 많이 사용되는 예로는 kotlin.coroutines.Continuation이 있다.

```java
public interface Continuation<in T> {
    public val context: CoroutineContext
    public fun resumeWith(result: Result<T>)
}
```

## variance 한정자의 위치

variance 한정자는 크게 두 위치에 사용할 수 있다. 첫 번째는 **선언 부분**이며, 일반적으로 이 위치에 사용한다. 이 위치에서 사용하면 클래스와 인터페이스 선언에 한정자가 적용된다. 따라서 클래스와 인터페이스가 사용되는 모든 곳에 영향을 준다.

```java
// 선언 쪽의 variance 한정자
class Box<out T>(val value: T)
val boxStr: Box<String> = Box("Str")
val boxAny: Box<Any> = boxStr
```

두 번째는 **클래스와 인터페이스를 활용하는 위치**이다. 이 위치에 variance 한정자를 사용하면 특정한 변수에만 variance 한정자가 적용된다.

```java
class Box<T>(val value: T)
val boxStr: Box<String> = Box("Str")
//사용하는 쪽의 variance 한정자
val boxAny: Box<out Any> = boxStr
```

모든 인스턴스에 variance 한정자를 적용하면 안되고, 특정 인스턴스에만 적용해야 할 때 이렇게 사용한다.

예를 들어 MutableList는 in 한정자를 포함하면, 요소를 리턴할 수 없으므로 in 한정자를 붙이지 않는다(자세한 내용은 이후에 설명).

하지만 단일 파라미터 타입에 in 한정자를 붙여서 contravariant를 가지게 하는 것은 가능하다. 이렇게 하면 여러 가지 타입을 받아들이게 할 수 있다.

```java
interface Dog
interface Cutie
data class Puppy(val name: String): Dog, Cutie
data class Hound(val name: String): Dog
data class Cat(val name: String): Cutie

fun fillWithPuppies(list: MutableList<in Puppy>) {
    list.add(Puppy("Jim"))
    list.add(Puppy("Bean"))
}

fun main() {
    val dogs = mutableListOf<Dog>(Hound("Pluto"))
    fillWithPuppies(dogs)
    println(dogs)
//[Hound(name=Pluto), Puppy(name=Jim), Puppy(name=Bean)]

    val animals = mutableListOf<Cutie>(Cat("Felix"))
    fillWithPuppies(animals)
    println(animals)
//[Cat(name=Felix), Puppy(name=Jim), Puppy(name=Bean)]
}
```

참고로 variance 한정자를 사용하면, 위치가 제한될 수 있다.

예를 들어 MutableListOf\<out T>가 있다면, get으로 요소를 추출했을 때 T 타입이 나올 것이다. 하지만 set은 Nothing 타입의 아규먼트가 전달 될 거라 예상되므로 사용할 수 없다.

이는 모든 타입의 서브타입을 가진 리스트(Nothing 리스트)가 존재할 가능성이 있기 때문이다.

## 정리

코틀린은 타입 아규먼트의 관계에 제약을 걸 수 있는 굉장히 강력한 제네릭 기능을 제공한다. 이러한 기능으로 제네릭 객체를 연산할 때 굉장히 다양한 지원을 받을 수 있다.

코틀린에는 다음과 같은 타입 한정자가 있다.

* 타입 파라미터의 기본적인 variance의 동작은 invariant 이다.
* out 한정자는 타입 파라미터를 covariant하게 만든다.
* in 한정자는 타입 파라미터를 contravariant하게 만든다.

코틀린에서는

* List와 Set의 타입 파라미터는 covariant(out 한정자)이다.
* 함수 타입의 파라미터 타입은 contravariant(in 한정자)이다.
* 리턴만 되는 타입에는 covariant(out 한정자)를 사용한다.
* 허용만 되는 타입에는 contravariant(in 한정자)를 사용한다.
