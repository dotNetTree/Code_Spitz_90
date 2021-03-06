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
                    builder.append(it.name, ':')
                    val value = it.getter.call(target)
                    builder.append(value, ',')
                }
            builder.append('}')
            return "$builder"   // <--- 1)
        }

        class Json0(val a: Int, val b: String)

        func main(args: Array<String>) {
            print(stringify(Json(3, "abc")))
        }
        ```
        - 실제 출력결과는 우리가 원하는 `{"a": 3,"b":"abc"}` 형태가 아닌 다음과 같이된다. `{a:3,b:abc,}` 이걸 차근차근 고쳐야 한다.
            - *고려사항 1* - 문자열이면 감싸는 따옴표를 붙여야 한다.
            - *고려사항 2* - 문자열 안에 따옴표는 \"로 변경
            - *고려사항 3* - 마지막 요소는 컴마를 제거
        > 1)을 잘보면 string templete을 사용하고 있다. `"$변수명"` 인 경우 변수의 toString()을 자동으로 호출한다. 이는 swift에서 string templete 사용시 변수의 description()을 자동으로 호출하는 것과 같다.
    - 위 고려사항 1, 2를 처리하기 위해 jsonString 작성한다.
        ```kotlin
        fun <T: Any> stringify(target: T): String {
            val builder = StringBuilder()
            builder.append('{')
            target::class.members
                .filterIsInstance<KProperty<*>>()
                .forEach { it ->    // <- it은 KProperty<*>
                    builder.append(jsonString(it.name), ':')
                    val value = it.getter.call(target)
                    builder.append(
                        if (value is String) jsonString(value) else value,  // <--- 2)
                        ','
                    )
                }
            builder.append('}')
            return "$builder"
        }
        private fun jsonString(v: String) = """"${v.replace("\"","\\\"")}""""
        ```
        - 여기까지 하면 `{"a":3,"b":"abc",}`가 출력된다.
        - 2)의 if는 문(statment)이 아닌 식(expression)으로 동작할 수 있다. 단, 조건이 있는데 if도 값을 return 해야하고 else도 값을 return 해야 한다. 같은 방식으로 when도 동일하게 동작할 수 있다. 모든 케이스에 값을 return하고 else도 값을 return 하면 된다.
            - 대부분의 ABC를 계승한 대부분의 개발 언어들은 if를 문(statment)로 인식한다.
            - statement 컴파일을 하면 cpu 등에 내리는 명령어로 바뀌며, expression은 메모리에 할당되는 값으로 바뀐다.
            - 함수형 언어이거나 expression을 강화하려는 언어는 statement expression으로 바꾸려한다. (대표적으로 ruby는 statement가 없다)
            - 사실 runtime에 유연성을 확보한다는 것도 결국 statement를 expression으로 고치는 과정이다. 디자인 패턴이던 함수형이든 수단이 뭐가 되든 결국 statement 제거인 것이다.
        - 2)의 코드의 의미는 builder append 식의 인자에 if else 식이 들어간 것이다.
    - 고려사항 3을 처리하기 위해서 [Iterator\<T\>의 joinTo](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/join-to.html)를 사용한다. 
         ```kotlin
        fun <T: Any> stringify(target: T): String {
            val builder = StringBuilder()
            builder.append('{')
            target::class.members
                .filterIsInstance<KProperty<*>>()
                .joinTo(builder, ",", "{", "}") {
                    val value = it.getter.call(target)
                    "${jsonString(it.name)}:${if (value is String) jsonString(value) else value}"
                }
                .toString()
            builder.append('}')
            return "$builder"
        }
        private fun jsonString(v: String) = """"${v.replace("\"","\\\"")}""""
        ```
        - 여기까지 하면 `{"a":3,"b":"abc"}`가 출력된다. 원하는 결과가 나왔다.
        - 하지만 JSON stringify는 객체 안에 객체, 그 안에 리스트를 순회하면서 만들어내는 것이 백미, 이제 List와 Object도 고려해보자

    31분까지 내용

    

        


    

       