# Java 17을 사용해야 하는 이유와 Java 17 변경점

{% hint style="success" %}
이 글은 팀원 [Jinny](https://github.com/jinny-l)가 작성했습니다.
{% endhint %}

## 💬 들어가며

처음 Java를 공부하면서 지금까지 `Java 11`을 썼는데 \
이번 팀프로젝트를 시작하며 팀원인 준이 `Java 17`을 사용해보자고 제안하여 `Java 17`을 도입하게 되었습니다.

준이 17의 특징과 장점에 대해 설명을 해주었지만, 저는 처음 접하는 버전이라 이번에 기회에 `Java 17` 공부를 해보았습니다. \
(나머지 팀원은 이전 프로젝트를 진행하며 17에 대해 공부를 이미 한 상태였습니다.)



## ☕️ 왜 Java 17을 사용해야 할까?

{% hint style="info" %}
💡 참고:

* `Java 17`은 2021년 9월에 공개된 LTS 버전으로 Oracle JDK 기준 2029년 9월까지 지원됩니다.
* [공식 문서](https://openjdk.org/projects/jdk/17)
{% endhint %}



우선 Java 17을 사용해야 하는 이유에 대해 알아보겠습니다.



### **Java LTS 버전의 기술 지원 기간**

`Java 17` 이전의 LST 버전은 `Java 8`과 `Java 11`이 대표적입니다. \
그런데 현재 서비스하고 있는 많은 레거시 프로젝트이 대부분 `Java 8`을 사용하고 있습니다.

`Jetbrains`에서 제공하는 통계만 봐도 `Java 8`의 점유율이 앞도적입니다.&#x20;



⏷ [2021년 Java 버전별 점유율 - Jetbrains](https://www.jetbrains.com/ko-kr/lp/devecosystem-2021/java/#Java\_which-versions-of-java-do-you-regularly-use)

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

이에 따라 `Java 8`은 11보다 더 긴 지원 시간을 갖게 되었습니다.

⏷ Java 버전별 기술 지원 기간:

| Java 버전 | 지원 종료 일자  |
| ------- | --------- |
| Java 8  | \~2030.12 |
| Java 11 | \~2026.09 |
| Java 17 | \~2029.09 |



### Java 17을 사용해야 하는 이유

이번 프로젝트에서 `Java 17`을 사용하기로 한 이유는 개인적으로 학습이 주된 것이었는데요. \
현업자인 제이든이 작성하신 [블로그](https://techblog.gccompany.co.kr/%EC%9A%B0%EB%A6%AC%ED%8C%80%EC%9D%B4-jdk-17%EC%9D%84-%EB%8F%84%EC%9E%85%ED%95%9C-%EC%9D%B4%EC%9C%A0-ced2b754cd7) 글을 보면 현업에서는 다음과 같은 이유로 사용한다고 합니다.

> 1. 신규 버전을 위한 대비
>    * 회사들은 `Java 8`의 지원 종료일이 다가옴에 따라 새로운 버전으로 마이그레이션을 준비해야 하는 상황
>    * 8 → 17 이후 버전으로 바로 마이그레이션하기에는 리스크가 있으니, 17버전의 기술 적응을 완료된 상태로 마이그레션을 하면 영향이 최소화 될 것
> 2. 다음 세대 플랫폼 호환을 위한 준비
>    * `SpringBoot 3.0`부터 `Java 17` 이상을 지원하여 다음 세대 플랫폼 호환 준비도 주된 이유



실제로 2022년 Java 버전별 점유율을 보면 `Java 8`은 감소하고 `Java 17`은 크게 증가한 것을 확인할 수 있습니다.

⏷ [2022년 Java 버전별 점유율 - Jetbrains](https://www.jetbrains.com/ko-kr/lp/devecosystem-2022/java/#Java\_which-versions-of-java-do-you-regularly-use)

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>



### Java 17의 변경점

그럼 `Java 11`과 비교했을 때 `Java 17`의 변경점에 대해 알아보겠습니다.\
해당 글에서는 모든 변경점을 다루지 않고 아래 몇가지 추가된 기능만 추려서 다룰 예정입니다.

* Text Block
* Record
* Sealed Classes
* Switch Expression
* Stream.toList()

#### Text Block

* Java 11

`Java 11` 버전에서는 `Json` 형식의 문자열을 다음과 같이 표현해야 했습니다.\
한눈에 봐도 가독성이 매우 안좋은 것을 확인할 수 있습니다.

```java
String jsonString = "{\n" +
    "  \"name\": \"Jinny\",\n" +
    "  \"age\": 20\n" +
    "}";
```

* Java 17

`Java 17`에서는 텍스트 블록을 제공해서 3개의 큰 따옴표로 랩핑해서 표현할 수 있습니다.

```java
String jsonString = """
        {
          "name": "Jinny",
          "age": 20
        }
        """;
```



#### Record

{% hint style="info" %}
**Record:**

* 멤버변수는 private final로 선언된다.
* 필드별 getter가 자동으로 생성된다.
* equals, hashcode, toString이 자동으로 생성된다.
* 기본생성자는 제공하지 않으므로 필요한 경우 직접 생성해야 한다.
* final 클래스이므로 다른 클래스를 상속하거나/상속시킬 수 없다.
* private final fields 이외의 인스턴스 필드를 선언할 수 없다.
{% endhint %}

* Java 11

롬복 어노테이션이 편리성을 제공해주긴 하지만...\
`Dto` 클래스를 만들 때 보일러 플레이트 코드를 추가해야 했습니다.

```java
public class Dto {
    
    private final int data;

    public Dto(int data) {
        this.data = data;
    }

    public int getData() {
        return data;
    }
}
```

* Java 17

Java 17에서는 `Record`를 활용하면 불필요한 코드를 제거할 수 있고, 클래스 자체로 명확한 의도를 표현할 수 있습니다.

```java
public record Dto(
    int data
) {
}
```



#### Sealed Class

Java 17에 `Sealed Class/Interface`가 추가되었습니다.

`Sealed` 클래스는 상속하거나(extends), 구현(implements) 할 클래스를 지정해두고, 해당 클래스들만 상속/구현이 가능하도록 제한하는 기능입니다.

이에 따라 개발자는 `Sealed` 클래스 코드만 봐도 어떤 클래스가 구현/상속했는지 쉽게 파악할 수 있습니다.\
또한, 의도치 않은 클래스가 상속받았을 경우, 컴파일 시점에 에러를 체크할 수 있어 의도치 않은 실수를 방지할 수 있습니다.

{% hint style="info" %}
**Sealed Class:**

* super-class 에 `sealed` 키워드를 사용한다.&#x20;
* `permits` 키워드 뒤에 해당 클래스를 상속받을 `sub-class`를 선언다.&#x20;
* `sealed` 된 클래스를 활용하기 위해서는 같은 모듈 혹은 같은 패키지 안에 존재 해야한다.
{% endhint %}

```java
// Parent.java
public sealed class Parent permits Son, Daughter {
    ...
}


// Son.java
// permits 로 선언된 class 만 Parent class 를 상속할 수 있다.
public sealed class Son extends Parent {}
```

#### Switch Expression

{% hint style="info" %}
**변경된 Switch문 문법:**

* \-> 화살표를 사용해 실행문을 나타낸다.
* 조건들을 `,`를 이용해 나열할 수 있다.
* `break`를 사용하지 않는다.
{% endhint %}

* Java 11

기존의 `switch`문은 다수의 `case`와 `break`가 존재하며 불필요한 중복 코드가 발생하고 있습니다.

```java
public static void printDayOfWeek(String dayOfWeek) {
    switch (dayOfWeek) {
        case "Monday":
        case "Tuesday":
        case "Wednesday":
        case "Thursday":
        case "Friday":
            System.out.println("평일입니다.");
            break;
        case "Saturday":
        case "Sunday":
            System.out.println("주말입니다.");
            break;
    }
```

* Java 17

```java
public static void printDayOfWeek(String dayOfWeek) {
    switch (dayOfWeek) {
        case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday" 
            -> System.out.println("평일입니다.");
        case "Saturday", "Sunday"
            -> System.out.println("주말입니다.");
    }
```

또한 출력문 안에서 사용이 가능해졌습니다.

```java
public static void printDayOfWeek(String dayOfWeek) {
    System.out.println(
            switch (dayOfWeek) {
                case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday" -> System.out.println("평일입니다.");
                case "Saturday", "Sunday" -> System.out.println("주말입니다.");
            }
        );
    }
    
```



#### Stream.toList()

* Java 11

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

List<Integer> result = numbers.stream()
        .filter(number -> number > 1)
        .collect(Collectors.toList());
```

* Java 17

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

List<Integer> result = numbers.stream()
        .filter(number -> number > 1)
        .toList(); // 변경
System.out.println(result);
```

{% hint style="danger" %}
**그렇다면 두 메서드는 완전 동일한 것으로 볼 수 있을까?**

* &#x20;JavaDocs를 살펴보면 두 메서드의 return type이 다릅니다.
{% endhint %}

`Collectors.toList()`

* `ArrayList`를 리턴하고 있으며, 불변 타입이 아닙니다.

<figure><img src="../.gitbook/assets/스크린샷 2023-10-23 오후 2.44.43 (2).png" alt=""><figcaption></figcaption></figure>

`toList()`

* 불변 `List`를 리턴하고 있습니다.

<figure><img src="../.gitbook/assets/스크린샷 2023-10-23 오후 2.44.55 (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
**그렇다면 `toList()`는 Collectors.toUnmodifiableList() 과 동일할까요?**

* Collectors.toUnmodifiableList()는 내부에서 null 체크를 하고 있지만, `toList()`는 null 체크를 하지 않아 null 값이 들어갈 수 있습니다.
{% endhint %}

`Collectors.toUnmodifiableList()`

* Java

> _The returned Collector disallows null values and will throw NullPointerException if it is presented with a null value._
>
> \
> 번역_:_ 반환된 Collector는 null 값을 허용하지 않으며, null 값을 전달받으면 NullPointerException을 throw합니다.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

***

## 🔗 참고 자료:

* [https://velog.io/@ililil9482/Java17%EC%9D%84-%EA%B3%A0%EB%A0%A4%ED%95%B4%EC%95%BC%ED%95%A0%EA%B9%8C](https://velog.io/@ililil9482/Java17%EC%9D%84-%EA%B3%A0%EB%A0%A4%ED%95%B4%EC%95%BC%ED%95%A0%EA%B9%8C)
* [https://techblog.gccompany.co.kr/%EC%9A%B0%EB%A6%AC%ED%8C%80%EC%9D%B4-jdk-17%EC%9D%84-%EB%8F%84%EC%9E%85%ED%95%9C-%EC%9D%B4%EC%9C%A0-ced2b754cd7](https://techblog.gccompany.co.kr/%EC%9A%B0%EB%A6%AC%ED%8C%80%EC%9D%B4-jdk-17%EC%9D%84-%EB%8F%84%EC%9E%85%ED%95%9C-%EC%9D%B4%EC%9C%A0-ced2b754cd7)
* [https://binux.tistory.com/146](https://binux.tistory.com/146)
