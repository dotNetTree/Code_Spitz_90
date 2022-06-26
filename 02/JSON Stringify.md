!주의: BlockQuote는 개인적인 감상을 적은 것입니다.

# 2강 JSON Stringify (Kotlin 함수) - [link](https://www.youtube.com/watch?v=J0U-FFFJztI)
> 2강부터는 코드스피츠 강의의 아이덴티티로 볼 수 있는, 실전 코드와 해당 코드의 설명으로 이뤄진다.

Kotlin으로 객체의 JSON Stringify를 구현하면서 Kotlin의 함수에 대해서 알아본다.
- 이번 강의의 목표
    ```kotlin
    fun <T: Any> stringify(target: T): String {
        /* 이번 강의는 여기를 채우는 것이 목표다 */
        return ""
    }
    ```
- fun stringify는 어떻게 동작해야 하는가
    - target T의 property를 조사하여 해당 property로 부터 name과 json value를 얻어, :(colon)으로 조합한다.
    - json value는 property의 Type에 의해 결정된다. 그러므로 Type에 따른 분기처리가 필요하다.
    - property Type 중에서는 다른 class를 포함할 수 있다. 이 경우 동일한 처리로 drill down해야한다.
    - 결국, 반환값이 String이므로 String 누적연산을 할 수 있는 StringBuilder를 이용한다.
    - 참고: [JSON 개요](https://www.json.org/json-ko.html)
- KClass - Kotlin Class 정의
    - target이 무엇인지 모르기 때문에 reflection을 이용해 타겟 내부를 조사해 property를 얻어내야한다.
    - Kotlin의 메타 데이터(reflection data) 얻기
        ```
        클래스::class
        인스턴스::class
        ```
    - Kotlin의 메타 데이터에는 당여하게도 class의 구성요소들이 들어 있다. property, fun, generic type, parent type 등등의 정보가 들어있다.
    - `Kclass.members`: 속성, 메소드 일체. Collection<KCallable<*>> 형.
    - `KCallable`: 모든 호출 가능 요소.
        - `KProperty`: 속성
        - `KFunction`: 함수, 메소드 
    - class의 property 정보로 stringify를 해야하므로 taget의 KProperty만 바라보면 된다.
        ```kotlin 
        fun <T: Any> stringify(target: T): String {
            target::class.members
                .filterIsInstance<KProperty<*>>()
                .forEach { it ->    // <- it은 KProperty<*>
            
                }
            return ""
        }
        ```
    - stringify는 property 정보를 얻어와 string을 계속 붙여야 하기 때문에 StringBuilder를 이용한다.
        ```kotlin
        fun <T: Any> stringify(target: T): String {
            val builder = StringBuilder()
            builder.append('{')
            target::class.members
                .filterIsInstance<KProperty<*>>()
                .forEach { it ->    // <- it은 KProperty<*>
            
                }
            builder.append('}')
            return "$builder"
        }
        ```