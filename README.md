# cleancode

* 코드는 요구사항을 상세 표현하는 수단이다.
* 그러므로 누구든 읽기 쉬워야하며 이해하기 편해야 한다.


### 나쁜코드
* 나쁜 코드는 계속해서 나쁜 코드를 생성해낸다.
* 개발에 대한 생산성을 늦춘다.


### 깨끗한 코드
* 코드는 단순해야 한다.
* 읽기 쉬워야 한다.
* 설계자의 의도가 숨어있지 말아야한다.

>단 깨끗한 코드란 완벽하게 정의된  것은 없다. 더 좋은 코드로 개발하는 방법은 각기 다르다

### Naming 정의
* 존재 이유 의도가 분명하게 드러나는 명칭을 정의하도록 한다.
* 수행 기능
* 사용 방법


```java
int d; // 경과 시간(단위 : 날짜)
```
이름 d는 아무 의미도 드러나지 않는다.

```java
int elapsedTimeInDays;
int daysSinceCreation;
int daysSinceModification;
int fileAgeInDays;
```
이런식으로 의미 있는 변수를 써야 한다.


### 나름대로 널리 쓰이는 단어를 다른의미로 사용하면 안된다.
accountList인 변수명 있다고 가정 하자
* List데이터 타입으로 사용 가능할것으로 추측
* 하지만 그냥 값의 연속성을 의미하는 것으로 사용
>accountGroup, accounts 같은 다른의미로 사용 가능

### 연속적인 숫자 사용은 권장하지 않는다.
```java
public static void copyChars(char a1[], char a2[]) {
  for(int i = 0 ; i < a1.length; i++) {
    a2[i] = a1[i];
}
```
### 발음하기 쉬운 이름을 사용해라
* 약어로된 이름을 사용하면 처음 입사한 사람에게 다시 부연설명을 해야 하는 상황이 온다.
* 회의를 할때도 그 해당 변수를 말할 때 유용하게 사용된다. 

@예제1
```java
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
    /* ... */
};
```

@예제2
```java
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
    /* ... */
};
```

### 헝가리 표기법은 피하는게 좋다.
>헝가리표기법은 데이터 타입을 약어로 같이 붙여 쓰는 방식이다.
* intMoney변수 -> strMoney로 변경된다면 
* 모든 변수명을 변경해야 한다. 

### 인터페이스 구현 클래스 명칭
* 인터페이스는 인터페이스라는 사실을 알리지 않고, 클래스 처럼 사용하기 위한 의미도 있으므로 
  인터페이스 명칭을 바꾸기 보다는 구현 클래스의 이름을 변경하는것이 좋다. 
ex) MemberService(인터페이스), MemberServiceImpl(구현체)

### 클래스 메서드 명칭 권상사항
* 클래스 : 명사의 형태 권장 
* 메서드 : 동사의 형태 권장
    접근자, 변경자, 조건자는 javabean 표준에 따라 get, set, is를 붙인다. 
* 생성자의 메서드 인수가 있다면 그 인수를 설명하는 메스드 명이 좋다. 
```java
Complex fulcrumPoint = Complex.FromRealNumber(23.0);
```


### 함수 가이드 
* 한개념에 한단어를 사용하는것을 권장한다. 
    >똑같은메서드 및 기능을 fetch, retrieve, get으로 부르면 혼란을 야기한다. 
* 한 단어를 다른 개념으로 사용하면 안된다. 
    >add라는 함수 intA + intB 같이 사칙연산의 함수라고 추측할 수 있다.    
    하지만 xxList에 새로운 아이템을 추가하는 함수도 될수 있다.   
    add라는 명칭은 서로 다른 기능을 대변하는 명칭이 되면 안된다. 
* 기술 개념의 용어를 사용해도 좋다. 
    >전산용어, 알고리즘 이름, 패턴 이름, 수학 용어 등을 말한다. 
* 의미 있는 맥락을 추가해라.
    >클래스, 함수에 맥락을 부여하면 좋다.      
 state라는 변수만을 사용하는 함수만 있다고 하면은 해당 함수는 state라는 것보다는  addState라는 함수명이 더 좋을 것이다.


### 함수는 최대한 작게 작성해야 좋다. 
* 함수는 몇밸 ~ 몇천 줄에 해당하는 함수들이 존재한다. 
* 그 함수가 무엇을하고자 하는지에 대해서 파악하려면 시간이 많이든다.
* 요약해서 정리 될 수 없다.
* 하지만 함수가 적으면 어느 역할을 하는지 확인이 가능하다.

>첫번째 이미지는 보기만해도 분석하기 어지럽다. 
```java
public static String testableHtml(PageData pageData,boolean includeSuiteSetup) throws Exception {
    WikiPage wikiPage = pageData.getWikiPage(); StringBuffer buffer = new StringBuffer(); 
    if (pageData.hasAttribute("Test")) { 
        if (includeSuiteSetup) {
            WikiPage suiteSetup = PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_SETUP_NAME, wikiPage);
    
        if (suiteSetup != null) {
            WikiPagePath pagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup);
            String pagePathName = PathParser.render(pagePath); 
            buffer.append("! include -setup .")
                  .append(pagePathName) 
                  .append("\n");
        }
    }
        
    WikiPage setup = PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
    if (setup != null) {
        WikiPagePath setupPath = wikiPage.getPageCrawler().getFullPath(setup);
    String setupPathName = PathParser.render(setupPath); buffer.append("! include -setup .")
```

>두번째 이미지를 보면 60자 내로함수를 줄였다.   
정확히 무엇을 하고자 하는 함수인지 알 수가 있다.   
하지만 함수는 더 작으면 작을 수록 좋다.

```java
public static String renderPageWithSetupsAndTeardowns PageData pageData, boolean isSuite) throws Exception{
    boolean isTest Page=pageData.hasAttribute("Test");
    if(isTestPage){
        WikiPage testPage=pageData.getWikiPage();
        StringBuffer newPageContent=new StringBuffer();
        includeSetupPages(testPage,newPageContent,isSuite);
        newPageContent.append(pageData.getContent());
        includeTeardown Pages(testPage,newPageContent,isSuite);
        pageData.setContent(newPageContent.toString());
    }
    return pageData.getHtml();
}
```

> 세번째 이미지를 보자   
10초도 안걸려서 이 함수가 무엇을 하려고 하는지 알 수 있다. 
```java
public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception) {
    if(isTestPage(pageData))
        includeSetupAndTeardownPages(pageData, isSuite);
    return pageData.getHtml();
}
```
* 함수는 한가지 일만 해야한다.
    > 함수가 한가지일만 해야 한다는 것은 추상적인 의미를 파악하고 그 추상적인 의므를 하는 기능만 구현 하는것이 좋다.   
    > 여기서는 TO문단을 사용해 개념을 추상화 할 수 있다. 
 
### Switch문 

switch문은 함수보다 길이가 길어진다.   
그리고 N가지를 처리하기 때문에 한가지의  추상개념을 처리하지 않는다.   
이런 swtich문은 완전히 피할 방법은 없다.   
이럴 경우는 다형성을 사용한다.   

추상 팩토리 인터페이스를 만들고 구현체에서는 각 조건에 맞는 부분을 처리할 각기 다른 파생 클래스를 생성한다.   
그리고 그 클래스내에서 처리하도록 한다.

```java
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}
```
위에 처럼 파생클래스 내에서 추상 클래스를 구현하게되면   
각각의  파생클래스에서는 동일한 함수를 각기 다르게 구현만 하게 된다.

### 함수명은 서술적인 이름을 권장한다.
함수명을 정할 때는 간결하고 의미있는 약어를 사용하기도 하지만   
그럴경우 함수의 이름과 내용을 설명하는 주석을 다는 경우도 생긴다.   
그럴바에는 아예 함수명을 서술적으로 함수명을 길게 표현하는것도 방법이다.   
함수 인수는 최대한 없는것이 좋다.

### 함수인수 가이드
* 아예 없는것이 제일 좋은 함수이고, 1개정도는 괜찮고 2개까지도 봐줄만 하다 하지만 3개 이상 부터는 그닥 좋지않다.
* 함수의 기능도 파악해야 하지만 인수가 많으면 그  인수들의 의미도 파악해야 하므로 코드를 읽기 힘들어진다.
* 각 인수들에 대한 테스트 케이스가 늘어나고 복잡해지므로 적을 수록 좋다. 

#### 플래그 인수는 좋지않다. 
boolean으로 인수를 넘긴다면 함수는 true / flase 일 때 각각 다른 기능을 수행한다는 의미가 되는데
함수는 한가지의 일만 수행해야 하므로 권장하지 않는다.

#### 이항함수
* 인수가 2개인 함수는 1개인 함수보다 더 읽기 힘들다.
* wirteField(name) / writeEidel(outputStream, name)  2개 중  인수가 하나인게 읽기 더 편하고 의미도 더 명확하다.    name이라는 인수고  무언가를 쓰다고 추측이 가능하다.
* 하지만 2개의 함수를 꼭 사용해야 하는 경우도 있다.
    > Point p = new Point(x, y) 처럼  하나의 값이 좌표라는 2개의 정보가 필요한 경우다   
      그리고 그 정보는 순서상 의미도 갖는다,   순서상 첫번째가 x  그리고 y가 나중에 오는것이 일반적이다.

    > 하지만 잘못된  예가 있다.   
      assertEqual(expected, actual) 이라는 함수다.      
      인수 2개가 서로 연관이 있는것도 아니고 어떤 순서가있는것도 아니다.   
      두번째 actual 값을 실수로 첫번째 excepted라는 곳에 잘못 넣는 경우도 생긴다.   
      이런 함수는 그냥 외워서 순서를 입력해야 하는 상황이 생긴다.

#### 인수객체
함수의 인수가 많아 지면  의미있는것을 묶어 하나의 클래스로 만드는것도 좋은 방법이다.

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```
> 위미 이미이처럼 x,y는 하나의 의미로 객체를 만들 수있다.   
  하나로 묶었다는건 추상적인 개념이 있기 때문에 묶을 수 있는것이다.

### 부수 효과를  일으키면 안된다. 
함수내에서 사용 하는 전역변수의 값이 변경되거나   한가지 일이 아닌 다른 일을 하게 된다면 혼란이 된다.

```java
public class UserValidator {
    private Cryptorapher cryptorapher; 
    
    public boolean checkPassword(String userName, String password) {
        User user = UserGateway.findByName(userName);
        if(user != User.NULL) {
            String codedPhrase = user.getPhraseEncodeByPassword();
            String pharse = cryptorapher.decrypt(codeedPharse, password);
            if("Valid Passowrd".equals(phrase)) {
                Session.initalize();
                return true;
            }
        }
    return false;
    }
}
```
> 위의 함수를 보면 checkPassword는  비밀번호가 맞는지 확인하는 함수이다.   
  하지만 Session.initalize() 함수는 이 함수 추상적인 개념에서 없는 내용이다.   
  이렇게 부수효과가 생기게 하면 안된다.

### 출력인수

appendFooter(s)를 한다고 하면은  일반적으로는 s인수를 받아서 무언가를 처리하기를 기대한다.   
선언부를 보면 아래와 같이 되어있다.

```java
public void appendFooter(StringBuffer report)
```

report를 footer에 붙인다는 얘기 인지  report의 footer를 보여준다는 의미인지 추측이 어렵다.   
이런 인수를 받아서 처리하는 방법은 옳지 않다.   
함수에서 상태를 변경해야 한다면 함수가 속한 객체  상태를 변경하는 함수로 해야한다.   
report.appendFooter() 이런식으로 말이다.   


### 명령과 조회를 분리해야 한다. 
함수는  두가지 경우가 존재한다. 
- 객체 상태를 변경하거나.
- 객체 정보를 반환하거나.
- 둘 다 하면 혼란을 초래한다. 
```java
if (set("username", "unclebob"))...
```
> 위의 이미지에서 set은  어떤 것을 지칭하는지 모른다   
  unclebob이라는것을 username에 넣으라는것인지   
  username속성의 값이 unclebob이면 true를 내뱉는다는것인지..

> 실제는 username에 unclebob을 넣고 잘 저장되었다는거에 대한 boolean값을 반환하도록되어있다.
```java 
if(attiributeExists("username")) {
    setAttribute("username", "unclebob");
    ...
}
```

> 위와 같이 명령과 조회를 분리해야 한다.

### 오류코드 보다는 예외를 사용해야 한다.
함수내에서 오류가 있을 경우 오류 코드를 반환 한다면 오류 코드를 처리하는 로직이 들어 가야 한다.

```java
if (deletePage(page) == E_OK) {
    if (registry.deleteReference (page.name) == E_OK) { 
        if (configkeys.deleteKey(page.name.makeKey()){
            logger.log("page deleted");
        } else {
            logger.log("configkey not deleted");
        }
    } else {
        logger.log("deleteRefernece from registry failed");
    }
} else {
    logger.log("delete failed");
    return E_ERROR;
}
```

> 하지만 만약 예외처리로 뺀다면 실제 함수 내에서 처리할 코드를 분리해서 보기 좋게 표현이 가능하다.

```java
try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
} 
catch (Exception e) {
    logger.log(e.getMessage());    
}
```

>만약 오류 코드를 반환하는 방식을 사용한다면  클래스든 enum 이든  무언가 코드의 사전정의가 되어있을것이다.   
 그리고 그것은 import하여 여러 클래스에서 사용할것이다.      
 만약 정의 대해서 수정이 생기게 된다면 모든 소스에도 수정이 이루어져야 하는 불상사가 생긴다.

### 주석
* 주석은 가능하면 안쓰는 것이 좋다.
* 주석이 있다는건 코드의 표현력이 부족하다는 말과 같다.
* 코드는 언제 변화하지만 주석을 변경하고 관리하는 일은 드물다.
* 최신 코드가 부합하지 않는 주석이 존재하게 되고 그 주석은 오보하는 경우가 생기기 때문에 주석이 없는것을 권장한다.

#### 주석이 필요한경우도 있다.

- 법적인 라이센스 주의를 주는 이런 부분에서는 주석을 사용하기도 한다.
- 의미를 명확하게 하는 주석이 있다. 예를 들면  표준 라이브러리같이 변경하지 못하는 코드가 있을 경우는   
  의미를 명료하게 하는 주석이 필요 하다
  ```java 
  assertTrue(a.compareTo(a) == 0);  // a == a  이런식으로 말이다.
  ```
- 경고성 주석이 있을 수 있따 테스트케이스에서 테스트를 돌릴 때  ‘시간이 오래 걸리니 주의’ 라는 형식의 경고를 주는 주석도있다.
- 중요성을 강조 하는 주석이 있다. 대수롭지 않고 여겨질 뭔가의 중요성을 강조하고 싶을 때
- 공개 API에서 Javadocs를 사용하는 경우

#### 나쁜주석은 다음 경우에 해당한다.

- 아무 의미 없이 그냥 개발자 본인만 알려고 씌여져있는 주석
- 코드로서 알 수 있는 내용을 적은 주석 ( 코드보다 더 많은 정보를 제공해주지 않는다)
- 의무적으로 Javadocs를 다는 경우
- 소스 수정에 대한 이력을 다는 경우 (요즘에는 형상관리가있어서 무의미하다)
- 너무나 당연한걸 주석으로 달아서 있으나 마나한 주석들
- 저자를 추가 하는 주석
- 레거시 코드를 주석 ( 형상관리가 있으므로 삭제해도 된다)
- 근처 코드에있는 내용만 주석을 달아야하는데 아예 전반적인 주석을 다는 경우
- 명확하지 않고 애매모호하게 정보를 전달하는 경우
