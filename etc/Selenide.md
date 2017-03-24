# Selenide

Selenide(셀레나이드)는 테스트 코드 실행으로 브라우저에서의 액션을 테스트 할 수 있게 해주는 테스팅 도구다.

이름에서 "어? 어디서 본 듯한?"하는 말이 떠오를 수 있는데, 그 짐작이 맞다. 
Selenide가 WebDriver라는 기술 기반으로 브라우저 테스팅을 위한 저수준 API를 제공한다면, **Selenide는 Selenium을 더 쓰기 쉽도록 고수준 Java API를 제공하는 Wrapper**라고 할 수 있다. 

이 글 역시 예전에 작성했던 [Selenium 꿀팁](http://hanmomhanda.github.io/2015/09/23/Selenium-%EA%BF%80%ED%8C%81/)의 속편이라고 볼 수 있겠다. 위 글에 댓글 달아주신 Disqus 아이디 @gaemi82님 덕에 Selenide를 알게 되었다. 감사드리면서.. 시작~

# Selenide를 이용한 실제 테스트 영상(40초)

수동으로 실행하면 **5분 정도 소요**되지만,
자동화를 통해 **40초**만에 완료(화면 빨리 돌린거 아님..)

{% youtube bSixvofQ2Lk %}

# Selenide를 사용하기 위한 pom.xml

```xml
<dependency>
    <groupId>com.codeborne</groupId>
    <artifactId>selenide</artifactId>
    <version>3.1.3</version>
    <scope>test</scope>
</dependency>
```

# Selenium보다 나은점

## 브라우저 테스트 용 코드가 무지 깔끔해졌다.

기본적으로 jQuery와 유사한 생김새의 API를 제공해주고 있어서 코드 작성 및 읽기가 아주 쉽고 직관적이다. 

이 부분은 구구절절 여기다 썰 푸는 것보다 [Selenide vs Selenium](https://github.com/codeborne/selenide/wiki/Selenide-vs-Selenium)를 보는 것이 낫겠다.

## Select Box의 선택이 무리없이 잘 된다.

예전에 'Selenium 꿀팁'을 작성할 때는 Select Box의 선택이 뜻하는대로 되지 않는 경우가 있어 이를 극복하는 꼼수(?)를 얘기했었는데, 이번에 Selenide를 써보니 별다른 꼼수없이 그냥 제공해주는 API로 쓰면 예상했던 대로 잘 작동한다.

물론 최신 버전의 Selenium도 그 부분이 개선되었을지 모르지만, Selenide의 소스를 보니 Select Box를 wrapping하면서 뭔가 조치를 취한 흔적이 보인다.

## waitUntil

어떤 요소가 어떤 텍스트/값/속성을 가지게 되거나 어떤 조건을 만족할 때까지 기다려 야 할 때 쉽고 요긴하게 쓸 수 있는 waitUntil 메서드 제공

```java
// SelenideElement.waitUntil(텍스트/값/속성/상태, 기다릴 시간 in millisecond)의 형식으로 사용

$("#welcome").waitUntil(hasText("Hello, Johny!"), 2000);
$("#username").waitUntil(hasAttribute("name", "user.name"), 2000);
$("#username").waitUntil(hasClass("green-button"), 2000);
$("#username").waitUntil(hasValue("123"), 2000);
$("#username").waitUntil(matchesText("Johny"), 2000);
$("#username").waitUntil(not(matchesText("Noname")), 2000);
$("#username").waitUntil(matchText("^Johny$"), 2000);

$("#username").waitUntil(present, 5000);
$("#username").waitUntil(enabled, 5000);
$("#username").waitUntil(disabled, 5000);
$("#username").waitUntil(visible, 5000);
$("#username").waitUntil(appears, 5000);
$("#username").waitUntil(disappears, 5000);
```


## Selenium에서 할 수 있는 건 다 할 수 있다.

Wrapper이면서도 Selenium의 기능을 다 쓸 수 있도록 열어두었다.

```java
// Selenium의 WebDriver를 가져올 수 있다.
WebDriver driver = WebDriverRunner.getWebDriver();

// JavaScript를 사용할 수 있는 JavaScriptExecutor를 가져올 수 있다.
JavascriptExecutor js = (JavascriptExecutor)driver;

// SelenideElement 인터페이스의 getWrappedElement()로
// Wrapping 되어 있는 원래의 WebElement를 가져올 수 있다.
WebElement we = $("#css-selector").getWrappedElement();
    ```

# 아쉬운 점

- Selenium IDE 처럼 브라우저 상에서의 액션을 녹화해서 소스 코드를 자동으로 만들어주는 기능은 없다.
    - 하지만, Selenium IDE에서 자동 생성한 코드도 scroll등 어느 정도 손을 봐줘야 한다는 점을 감안하면, API의 뛰어난 직관성이 이를 충분히 커버해준다.

- Selenium과 마찬가지로 화면(viewport)에 렌더링되지 않은 요소에는 값 세팅이나 이벤트 유발이 불가능하다.
    - 결국 해당 요소가 렌더링 되도록 적절히 화면을 적절히 scroll 해주는 소스를 직접 작성해야 하는데, 그나마 `$("#css-selector").scroll();` 처럼 쉽고 직관적인 API를 제공해준다.

# 꿀팁

아쉬운 점이 없었다면 꿀팁도 필요 없었겠지만~

## JavaScript의 사용

아무리 쉬운 API라 한 들, 브라우저 테스트라면 결국 DOM을 다뤄야 하는데, JavaScript를 쓸 수 없다면 한계가 많을 것이다. 위에 잠깐 언급했지만 다음과 같이 JavaScript를 사용할 수 있다.

```java
WebDriver driver = WebDriverRunner.getWebDriver();
JavascriptExecutor js = (JavascriptExecutor)driver;

js.executeScript("document.getElementById('name').value = 'Selenide'");
```

타이핑이 귀찮으니 다음과 같이 얄팍한 수를 쓰자.

```java
private static void J(String javaScriptSource) {
    Object obj = js.executeScript(javaScriptSource);
//    System.out.println(obj);  // null이 나오기도 값이 나오기도 한다. 궁금하면 찾아보기~
}
```
그럼 아래와 같이 더 쉽게 쓸 수 있다.

```java
WebDriver driver = WebDriverRunner.getWebDriver();
JavascriptExecutor js = (JavascriptExecutor)driver;

J("document.getElementById('name').value = 'Selenide'");
```

## Alert 창의 처리

요즘 같은 호화로운 UI 시대에 웬 Alert 창?

그래도 알아는 보자. 사실 아래의 `closeAlertAndGetItsText` 메서드는 Selenium IDE가 녹화해 줄 때 자동으로 생성되는 소스 코드다.

```java
WebDriver driver = WebDriverRunner.getWebDriver();

~~ 중략 ~~

$("#saveAcademy").click();
assertEquals("학력 정보를 성공적으로 저장했습니다.", closeAlertAndGetItsText());

~~ 중략 ~~

private String closeAlertAndGetItsText() {
    try {
        Alert alert = driver.switchTo().alert();
        String alertText = alert.getText();
        if (acceptNextAlert) {
            alert.accept();
        } else {
            alert.dismiss();
        }
        return alertText;
    } finally {
        acceptNextAlert = true;
    }
}
``` 

##  readonly 의 처리

readonly의 값은 Selenium으로도 Selenide로도 모두 설정할 수 없다. 어떻게 보면 당연하기도 한데, 실무에서는 readonly의 값을 설정해야할 때도 있다. 

예를 들면, 달력이나 또는 우편번호/주소 같은 경우 보통 팝업 창 등을 이용해서 검색을 통해 값이 설정되므로 readonly로 설정하지만, 테스트에서는 팝업 창 할 필요 없이 그냥 직접 값을 설정해주는 것이 편하기 때문이다.

이럴 때는 DOM API를 직접 쓰거나, 테스트 할 페이지가 jQuery를 사용할 수 있게 되어 있다면 jQuery로도 값을 설정할 수 있다.

```java
// id가 'read-only'인 요소가 readonly 일 때

// Selenide API 사용 시 에러 발생
// org.openqa.selenium.InvalidElementStateException: Element is read-only and so may not be used for actions
$("#readonly-text").setValue("ReadOnly는 Selenide API로는 안 된다고 전해라");

// DOM API 사용 가능
J("document.getElementById('readonly-text').value = 'DOM API는 된다'");

// jQuery 사용 가능(단, 테스트 대상 웹 페이지 자체가 jQuery를 쓸 수 있게 되어 있어야 함)
J("$('#readonly-text').val('jQuery도 된다')");
```   

## hidden의 처리

화면에 렌더링 되지 않는 hidden 요소의 값도 Selenium이나 Selenide로는 변경하지 못한다.

```java
// id가 'hidden-text'인 요소가 hidden 일 때

// Selenide API 사용 시 에러 발생
// Element not found {#hidden-text}
$("#hidden-text").setValue("hidden은 Selenide API로는 안 된다고 전해라");

// DOM API 사용 가능
J("document.getElementById('hidden-text').value = 'DOM API는 된다'");

// jQuery 사용 가능(단, 테스트 대상 웹 페이지 자체가 jQuery를 쓸 수 있게 되어 있어야 함)
J("$('#hidden-text').val('jQuery도 된다')");
```

## .가 포함된 아이디의 CSS Selector

이건 팁이라기보다는 일반적인 사용법에 가까운데, 아이디에 . 가 포함되면 CSS Selector를 작성할 때 적절히 escape 처리를 해줘야 한다.

```java
// Selenide API 사용 시 . 앞에 \를 2회 써준다.
$("#obj\\.prop").setValue("역슬래쉬 2회");

// getElementById()는 CSS Selector를 사용하지 않으므로 escape 불필요
J("document.getElementById('obj.prop').value = 'Escape 처리 불필요'");

// CSS Selector를 사용하는 
// document.querySelector(), document.querySelectorAll() 과 jQuery의 $ 메서드에는 역슬래쉬 4회 써준다.
J("$('#obj\\\\.prop').val('역슬래쉬 4회')");

```

# 정리

>- 브라우저로 사용자 테스트 할 때 Selenide 쓰자. Selenium에 비해 정말 많이 편해졌다.
>
>- 화면에 보이지 않는 요소에는 값 설정, 클릭 등의 액션을 처리할 수 없으므로, 적절히 화면 scroll 해주는 소스를 직접 추가해줘야 한다.
>
>- waitUntil()로 다양한 상황에서의 대기 처리를 쉽게 할 수 있다.
>
>- Selenide API로 안되는 여러 상황(hidden, readonly, alert 창 등)에서는 Selenium에서 했던 방법대로 JavaScript를 사용하자. 

# 참고 자료

- https://github.com/codeborne/selenide/wiki/Selenide-vs-Selenium
- https://gist.github.com/mkpythonanywhereblog/947633ba1bf0bc239639
- http://selenide-recipes.blogspot.kr/2015/08/6-waits.html