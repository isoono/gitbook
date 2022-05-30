# 아이템 8 : 적절하게 null을 처리하라

**null**은 ‘**값이 부족하다**(lack of value)’는 것을 나타내며, 프로퍼티가 null이라는 것은 값이 설정되지 않았거나 제거되었다는 것을 뜻한다.

함수가 null을 리턴한다는 것은 함수에 따라서 여러 의미를 가질 수 있다. 예를 들어,

* String.toIntOrNull( )은 String을 Int로 적절하게 변환할 수 없을 경우 null을 리턴
* Iterable\<T>.firstOrNull( ( ) → Boolean )은 주어진 조건에 맞는 요소가 없는 경우 null을 리턴

API를 사용하는 개발자가 nullable 값을 처리해야 하기 때문에 null은 최대한 명확한 의미를 갖는 것이 좋다.

```kotlin
val printer: Printer? = getPrinter()
printer.print() //컴파일 오류

printer?.print() //안전 호출
if(printer != null) printer.print() //스마트 캐스팅
printer!!.print() //not-null assertion
```

기본적으로 nullable 타입은 세 가지 방법으로 처리한다.

* `?. (안전 호출)`, `스마트 캐스팅`, `Elvis 연산자` 등을 활용해서 안전하게 처리한다.
* 오류를 throw 한다.
* 함수 또는 프로퍼티를 리팩터링해서 nullable 타입이 나오지 않게 바꾼다.

각 내용을 알아보자.

## null을 안전하게 처리하기

null을 안전하게 처리하는 방법 중 널리 사용되는 방법으로는 **안전 호출**(safe call)과 **스마트 캐스팅**(smart casting)이 있다.

```kotlin
printer?.print() //안전 호출

if (printer != null) printer.print() //스마트 캐스팅
```

두 가지 모두 printer가 null이 아닐 때 print 함수를 호출한다. 애플리케이션 사용자의 관점에서 가장 안전한 방법이며, 개발자에게도 편리한 방법이다.

코틀린은 nullable 변수와 관련된 처리를 광범위하게 지원한다. 인기 있는 다른 방법은 **Elvis 연산자**를 사용하는 것이다.

```kotlin
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?:
		throw Error("Printer must be named")
```

Elvis 연산자는 오른쪽에 return 또는 throw를 포함한 모든 표현식이 허용된다.

많은 객체가 nullable과 관련된 처리를 지원한다. 예를 들어 컬렉션 처리를 할 때 값이 없다는 것을 나타내야 한다면 **null이 아닌 빈 컬렉션을 사용하는 것이 일반적**이다. 예로 `Collection<T>?.orEmpty()` 확장 함수를 사용하면 nullable이 아닌 List\<T>를 리턴받는다.

스마트 캐스팅은 코틀린의 규약 기능(contracts feature)을 지원한다. 이 기능을 사용하면 다음 코드처럼 스마트 캐스팅할 수 있다.

```kotlin
fun main() {
    println("What is your name?")
    val name = readLine()
    if (!name.isNullOrBlank()) {
        println("Hello ${name.toUpperCase()}") //smart cast
    }

    val news: List<News>? = getNews()
    if (!news.isNullOrEmpty()) {
        news.forEach { notifyUser(it) } //smart cast
    }
}

class News

fun getNews(): List<News>? {
    val random = Random()
    val result = random.nextInt()
    val news = News()
    if (result >= 0 ) {
        return listOf(news)
    } else {
        return null
    }

}

fun notifyUser(news: News) {
    println("notified : $news")
}
```

지금까지 살펴본 null을 적절하게 처리하기 위한 방법들은 반드시 알아 두어야 한다.

{% hint style="info" %}
❓ **방어적 프로그래밍과 공격적 프로그래밍** 모든 가능성을 올바른 방식으로 처리하는 것(예를 들어 null일 때는 출력하지 않기 등)을 **방어적 프로그래밍**(defensive programming)이라고 부른다. 이는 코드가 프로덕션 환경으로 들어갔을 때 발생할 수 있는 수많은 것들로부터 프로그램을 방어해서 안정성을 높이는 방법을 나타내는 포괄적인 용어이다. 상황을 처리할 수 있는 올바른 방법이 있을 때는 굉장히 좋다.&#x20;

\
하지만 모든 상황을 안전하게 처리하는 것은 불가능하다. \


이러한 경우에는 **공격적 프로그래밍**(offensive programming)이라는 방법을 사용한다. 공격적 프로그래밍이란 예상하지 못한 상황이 발생했을 때, 이러한 문제를 개발자에게 알려서 수정하게 만드는 것이다. ‘아이템 5’에서 나온 require, check, assert가 바로 공격적 프로그래밍을 위한 도구이다.

이름 때문에 둘이 충돌되는 것처럼 보이지만, 코드의 안전을 위해 모두 필요하다. 둘을 모두 이해하고 적절하게 사용할 수 있어야 한다.
{% endhint %}

## 오류 throw 하기

다른 개발자가 어떤 코드를 보고 선입견처럼 ‘당연히 그럴것이다’라고 생각하게 되는 부분이 있고, 그 부분에서 문제가 발생할 경우에는 개발자에게 오류를 강제로 발생시켜 주는 것이 좋다.

오류를 강제로 발생시킬 때는 `throw`, `!!`, `requireNotNull`, `checkNotNull` 등을 활용한다.

```kotlin
fun process(user: User) {
    requireNotNull(user.name)
    val context = checkNotNull(context)
    val networkService = 
        getNetworkService(context) ?:
        throw NoInternetConnection()
    networkService.getData { data, userData -> 
        show(data!!, userData!!)
    }
}
```

* requireNotNull : 매개변수의 값이 null이면 IllegalArgumentException 반환
* checkNotNull : 매개변수의 값이 null이면 IllegalStateException 반환

## not-null assertion(!!)과 관련된 문제

nullable을 처리하는 가장 간단한 방법은 not-null assertion(!!)을 사용하는 것이다. 하지만 `!!`를 사용하면 자바에서 nullable을 처리할 때 발생할 수 있는 문제가 똑같이 발생한다.

!!은 사용하기 쉽지만 좋은 해결방법은 아니다. 예외가 발생할 때, 어떤 설명도 없는 제네릭 예외가 발생하며, 코드가 짧고 사용하기 쉬워서 남용하게 되는 문제도 있다.

!!은 null이 나오지 않는다는 것이 거의 확실한 상황에서 많이 사용되지만, 현재 확실하다고 미래가 확실한 것은 아니다. 문제는 미래의 어느 순간에 일어난다.

```kotlin
fun largestOf (a: Int, b: Int, c: Int, d: Int): Int = 
    listOf(a, b, c, d).maxOrNull()!!
```

모든 파라미터를 리스트에 넣은 뒤에 maxOrNull 함수를 사용해서 가장 큰 값을 찾는 코드이다. 문제는 컬렉션 내부에 아무것도 없을 경우 null을 리턴하는 것이다. 그래서 !! 연산자를 사용하였다.

이런 간단한 함수에서도 !!는 NPE로 이어질 수 있다. 미래의 누군가가 함수를 리팩터링하면서 컬렉션이 null일 수 있다는 것을 놓칠 수 있기 때문이다.

```kotlin
fun largestOf (vararg nums: Int): Int =
    nums.maxOrNull()!!

fun main() {
    largestOf() //NPE
}
```

nullability(널일 수 있는지)와 관련된 정보는 숨겨져 있으므로, 놓치기 쉽다.

변수와 비슷하다. 변수를 일단 선언하고, 이후에 사용하기 전에 값을 할당해서 사용하기로 하고, 다음과 같은 코드를 작성했다고 해보자.

```kotlin
class UserControllerTest {

    private var dao: UserDao? = null
    private var controller: UserController? = null

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }

    @Test
    fun test() {
        controller!!.doSomething()
    }
}

class UserDao
class UserController(
    val userDao: UserDao
) {
    fun doSomething() {
        println("This is UserController !")
    }
}
```

이처럼 변수를 null로 설정하고, 이후에 !! 연산자를 사용하는 방법은 좋지 않다. 이후에 프로퍼티를 계속해서 언팩(unpack)해야 하므로 사용하기 귀찮다. 또한 해당 프로퍼티가 실제로 이후에 의미 있는 null 값을 가질 가능성 자체를 차단한다. 이런 코드를 작성하는 올바른 방법은 `**lateinit**` 또는 `**Delegates.notNull**`을 사용하는 것이다.

예외는 예상하지 못한 잘못된 부분을 알려 주기 위해서 발생하는 것이다(아이템 7: 결과 부족이 발생할 경우 null과 Failure를 사용하라). 하지만 **명시적 오류**는 제네릭 NPE보다는 훨씬 더 많은 정보를 제공해 줄 수 있으므로 !! 연산자를 사용하는 것보다는 훨씬 좋다.

!! 연산자가 의미 있는 경우는 굉장히 드물다. 일반적으로 nullability가 제대로 표현되지 않는 라이브러리를 사용할 때 정도에만 사용해야 한다.

코틀린을 대상으로 설계된 API를 활용한다면, !! 연산자를 사용하는 것을 이상하게 생각해야 한다.

일반적으로 !! 연산자 사용을 피해야 한다. 이러한 제안은 코틀린 커뮤니티 전체에서 널리 승인되고 있는 제안이다.

## 의미 없는 nullability 피하기

nullability는 어떻게든 적절하게 처리해야 하므로, 추가 비용이 발생한다. 따라서 필요한 경우가 아니라면, nullability 자체를 피하는 것이 좋다.

null은 중요한 메시지를 전달하는 데 사용될 수 있다. 따라서 다른 개발자가 보기에 의미가 없을 때는 null을 사용하지 않는 것이 좋다. 만약 이유 없이 null을 사용했다면, 다른 개발자들이 코드를 작성할 때, 위험한 !! 연산자를 사용하게 되고, 의미 없이 코드를 더럽히는 예외 처리를 해야 할 것이다.

nullability를 피할 때 사용할 수 있는 몇 가지 방법이다.

* 클래스에서 nullability를 따라 여러 함수를 만들어서 제공할 수도 있다. 대표적인 예로 List\<T>의 get과 getOrNull 함수가 있다. (관련된 내용은 아이템7에서 설명)
* 어떤 값이 클래스 생성 이후에 확실하게 설정된다는 보장이 있다면, **lateinit** 프로퍼티와 **notNull 델리게이트**를 사용하라.
* 빈 컬렉션 대신 null을 리턴하지 말자. List\<Int>?와 Set\<String?>과 같은 컬렉션을 빈 컬렉션으로 둘 때와 null로 둘 때는 의미가 완전히 다르다. null은 컬렉션 자체가 없다는 것을 나타낸다. **요소가 부족하다는 것을 나타내려면, 빈 컬렉션을 사용**하라.
* nullable enum과 None enum 값은 완전히 다른 의미이다. null enum은 별도로 처리해야 하지만, None enum 정의에 없으므로 필요한 경우에 사용하는 쪽에서 추가해서 활용할 수 있다는 의미이다.

## lateinit 프로퍼티와 notNull 델리게이트

클래스가 클래스 생성 중에 초기화할 수 없는 프로퍼티를 가지는 것은 드문 일은 아니지만 분명 존재하는 일이다. 이런 프로퍼티는 사용 전에 반드시 초기화해서 사용해야 한다.

예로 JUnit의 @BeforeEach 처럼 다른 함수들보다도 먼저 호출되는 함수에서 프로퍼티가 설정되는 경우가 있다.

```kotlin
class UserControllerTest {

    private var dao: UserDao? = null
    private var controller: UserController? = null

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }

    @Test
    fun test() {
        controller!!.doSomething()
    }
}
```

물론 lateinit를 사용할 경우에도 비용이 발생한다. 만약 초기화 전에 값을 사용하려고 하면 예외가 발생한다.

처음 사용하기 전에 반드시 초기화가 되어 있을 경우에만 lateinit을 붙이는 것이다. 만약 그런 값이 사용되어 예외가 발생한다면, 그 사실을 알아야 하므로 예외가 발생하는 것은 오히려 좋은 일이다. lateinit은 nullable과 비교해서 다음과 같은 차이가 있다.

* !! 연산자로 언팩(upack)하지 않아도 된다.
* 이후에 어떤 의미를 나타내기 위해서 null을 사용하고 싶을 때, nullable로 만들 수도 있다.
* 프로퍼티가 초기화된 이후에는 초기화되지 않은 상태로 돌아갈 수 없다.

**lateinit**은 프로퍼티를 처음 사용하기 전에 **반드시 초기화될 거라고 예상되는 상황에 활용**한다. 이러한 상황으로는 라이프 사이클을 갖는 클래스처럼 메서드 호출에 명확한 순서가 있을 경우가 있다. 안드로이드 Activity의 onCreate, 리액트 React.Component의 componentDidMount 등이 대표적인 예다.

반대로 **lateinit**을 사용할 수 없는 경우도 있다. JVM에서 Int, Long, Double, Boolean과 같은 기본 타입과 연결된 타입으로 프로퍼티를 초기화해야 하는 경우이다. 이런 경우에는 lateinit보다는 약간 느리지만, Delegates.notNull을 사용한다.

프로퍼티 위임(property delegation)을 사용하는 패턴은 ‘아이템 21: 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라'에서 자세하게 다룬다.

프로퍼티 위임을 사용하면, nullability로 발생하는 여러 가지 문제를 안전하게 처리할 수 있다.
