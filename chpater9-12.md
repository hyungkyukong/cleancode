## 단위 테스트 
* 단위테스트를 사용하는 프로그래머들이 많아졌다. 
* 제대로된 단위테스트를 작성하는 방법을 알아보자 

### TDD 법칙 세 가지 
* 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다. 
* 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위테스트를 작성한다.
* 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다. 

> 테스틐코드가 먼저 생성 되며 그 이후로 바로 실제 코드를 얻을 수 있다.   
> 하지만 방대만 실제 모든 코드를 테스트 케이스로 만든다면 그 방대한 테스트케이스 관리도 힘들어 진다. 

## 깨끗한 테스트 코드 유지하기 
* 테스트 코드를 그저 일외용처럼 사용하면안된다. 
* 네이밍, 결합과 분리 등 깨끗한 코드를 유지해야한다. 
* 만약 실제 개발 코드가 변경이 생기면 테스트 코드도 변경을 해줘야 하는데 깨끗하지 않은 코드라면 
  변경이 쉽지 않게 된다. 그렇게 되면 테스트 케이스는 무용지물이 된다.   
  개발자들은 테스트케이스를 신뢰하지 않게 되고 실제 코드의 수정을 머뭇하게 된다. 

### 테스트는 유연성, 유지보수성, 재사용성을 제공한다.
* 테스트 코드는 깨끗하지 않으면 개발자는 실제 코드 변경을 주저하게 된다. 
* 테스트 코드가 깨끗하지 않으면 더 나아가서는 실제 코드로 깨끗해지지 않는다. 
* 즉, 테스트 코드가 깨끗해야만 변경이 두렵지 않아 변경에 대해 유연하고 유지보수도 좋아진다.

### 깨끗한 테스트 코드 
테스크 코드에서도 가독성이 중요한 역할을 한다. 

* addPage, assertSubString 함수 등 중복적으로 사용하는게 많은 코드이다
```java
public void testGetPageHieratchyAsXml() throws Exception {
	crawler.addPage(root, PathParser.parse("PageOne"));
	crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
	crawler.addPage(root, PathParser.parse("PageTwo"));

	request.setResource("root");
	request.addInput("type", "pages");
	Responder responder = new SerializedPageResponder();
	SimpleResponse response =
		(SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
	String xml = response.getContent();

	assertEquals("text/xml", response.getContentType());
	assertSubString("<name>PageOne</name>", xml);
	assertSubString("<name>PageTwo</name>", xml);
	assertSubString("<name>ChildOne</name>", xml);
}

public void testGetPageHieratchyAsXmlDoesntContainSymbolicLinks() throws Exception {
	WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
	crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
	crawler.addPage(root, PathParser.parse("PageTwo"));

	PageData data = pageOne.getData();
	WikiPageProperties properties = data.getProperties();
	WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
	symLinks.set("SymPage", "PageTwo");
	pageOne.commit(data);

	request.setResource("root");
	request.addInput("type", "pages");
	Responder responder = new SerializedPageResponder();
	SimpleResponse response =
		(SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
	String xml = response.getContent();

	assertEquals("text/xml", response.getContentType());
	assertSubString("<name>PageOne</name>", xml);
	assertSubString("<name>PageTwo</name>", xml);
	assertSubString("<name>ChildOne</name>", xml);
	assertNotSubString("SymPage", xml);
}

public void testGetDataAsHtml() throws Exception {
	crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");

	request.setResource("TestPageOne"); request.addInput("type", "data");
	Responder responder = new SerializedPageResponder();
	SimpleResponse response =
		(SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
	String xml = response.getContent();

	assertEquals("text/xml", response.getContentType());
	assertSubString("test page", xml);
	assertSubString("<Test", xml);
}

```
위의 코드를 더 명확한 함수로 변경하여 최소한의 함수만 부르도록했다.   
그리고 BUILD-OPERATE-CHECK 패턴을 적용했다. 
* 테스트 자료를 만든다.
* 테스트 자료를 조작한다.
* 결과가 맞는지 확인한다. 

```java
public void testGetPageHierarchyAsXml() throws Exception {
	makePages("PageOne", "PageOne.ChildOne", "PageTwo");

	submitRequest("root", "type:pages");

	assertResponseIsXML();
	assertResponseContains(
		"<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}

public void testSymbolicLinksAreNotInXmlPageHierarchy() throws Exception {
	WikiPage page = makePage("PageOne");
	makePages("PageOne.ChildOne", "PageTwo");

	addLinkTo(page, "PageTwo", "SymPage");

	submitRequest("root", "type:pages");

	assertResponseIsXML();
	assertResponseContains(
		"<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
	assertResponseDoesNotContain("SymPage");
}

public void testGetDataAsXml() throws Exception {
	makePageWithContent("TestPageOne", "test page");

	submitRequest("TestPageOne", "type:data");

	assertResponseIsXML();
	assertResponseContains("test page", "<Test");
}

```

### 도메인에  특화된 테스트 언어 

DSL : 특정 도메인을 대상으로 만들어진 특수 프로그램 언어

* 실제 테스트 해야할 비즈니스 로직 API위에 캡슐화처럼 그 로직을 테스트 하기 위한 새로운   
  함수를 만들어 사용하는 특수 API 
* 개발자는 간결하게 작성하고 어떤 테스트를 할건지 목적을 표해야 한다. 
* 구독자는 가독성이 좋아 쉽게 읽히며 이해가 빨라진다. 

### 이중 표준 
테스트 API에서 적용하는 표준은 실제 코드에서 적용하는 표준이 다른 경우도있다. 

최소 테스트 코드다 
```java
@Test
public void turnOnLoTempAlarmAtThreashold() throws Exception {
        hw.setTemp(WAY_TOO_COLD);
        controller.tic();
        assertTrue(hw.heaterState());
        assertTrue(hw.blowerState());
        assertFalse(hw.coolerState());
        assertFalse(hw.hiTempAlarm());
        assertTrue(hw.loTempAlarm());
        }
```

리팩토링을 해서 변경된 코드다   
* 대문자는 '켜짐'이고 소문자는 '꺼짐'을 뜻한다. 문자는 항상 '{heater, blower, cooler, hi-temp-alarm, lo-temp-alarm}' 순서   
* 그릇된 정보를 피하라는 원칙을 위반한것처럼 처음봤을 때 이해하기 어려운 코드를 사용한다. 
```java
@Test
public void turnOnCoolerAndBlowerIfTooHot() throws Exception {
  tooHot();
  assertEquals("hBChl", hw.getState()); 
}
  
@Test
public void turnOnHeaterAndBlowerIfTooCold() throws Exception {
  tooCold();
  assertEquals("HBchl", hw.getState()); 
}

@Test
public void turnOnHiTempAlarmAtThreshold() throws Exception {
  wayTooHot();
  assertEquals("hBCHl", hw.getState()); 
}

@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
  wayTooCold();
  assertEquals("HBchL", hw.getState()); 
}
```

* 효율을 높이려면 stringBuffer를 사용하는것이 더 적합하다. (cpu 및 메모리)
* 하지만 테스트 코드에서는 환경에 구애받지 않는다는 표준을 두어 할 수 있다. 
* 즉 표준을 이중으로 분리되어진다.
```java
public String getState() {
        String state = "";
        state += heater ? "H" : "h";
        state += blower ? "B" : "b";
        state += cooler ? "C" : "c";
        state += hiTempAlarm ? "H" : "h";
        state += loTempAlarm ? "L" : "l";
        return state;
        }

```

### 테스트 당 assert 하나 
* JUnit으로 테스트 코드를 짤 때는 함수마다 assert 문을 단 하나만 사용해야 한다. 
* assert 문이 단하나의 함수는 결론이 하나라서 코드를 이해하기 쉽고 빠르다. 
* given-when-then 규칙을 적용해서 테스트 케이스를 만들면 명확하다. 

```java
public void testGetPageHierarchyAsXml() throws Exception { 
	givenPages("PageOne", "PageOne.ChildOne", "PageTwo");

	whenRequestIsIssued("root", "type:pages");

	thenResponseShouldBeXML(); 
}

public void testGetPageHierarchyHasRightTags() throws Exception { 
	givenPages("PageOne", "PageOne.ChildOne", "PageTwo");

	whenRequestIsIssued("root", "type:pages");

	thenResponseShouldContain(
		"<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
	); 
}
```

### 테스트 당 개념 하나 
* 테스트 코드는 하나의 개념을 테스트 하는 형태여야 한다. 
* 만약 테스트 코드에 여러 개념을 테스트 하면 해당 테스트 코드의 목적을 잃게 된다. 
* 테스트 개념이 한곳에 모여있다면 독자는 해당 테스트 코드가 존재하는 이유를 이해해야 한다.

> 아래의 코드를 보면 여러 개념을 하나의 테스트코드에서 테스트를 한다.   
> assert문이 여럿이라는 사실보다는 개념이 여러개가 들어가있다는 것이 문제   
> 즉 개념은 하나이면서 assert문을 최소한으로 줄이는것이 바람직하다.
```java
/**
 * addMonth() 메서드를 테스트하는 장황한 코드
 */
public void testAddMonths() {
	SerialDate d1 = SerialDate.createInstance(31, 5, 2004);
	
	// (6월처럼) 30일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안된다.
	SerialDate d2 = SerialDate.addMonths(1, d1); 
	assertEquals(30, d2.getDayOfMonth()); 
	assertEquals(6, d2.getMonth()); 
	assertEquals(2004, d2.getYYYY());

	// 두 달을 더하면 그리고 두 번째 달이 31일로 끝나면 날짜는 31일이 되어야 한다.
	SerialDate d3 = SerialDate.addMonths(2, d1); 
	assertEquals(31, d3.getDayOfMonth()); 
	assertEquals(7, d3.getMonth()); 
	assertEquals(2004, d3.getYYYY());

	// 31일로 끝나는 한 달을 더하면 날짜는 30일이 되어야지 31일이 되어서는 안된다.
	SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1)); 
	assertEquals(30, d4.getDayOfMonth());
	assertEquals(7, d4.getMonth());
	assertEquals(2004, d4.getYYYY());
}
```

### F.I.R.S.T 
깨끗한 테스트는 다음 5가지 규칙을 따른다. 

* FIRST(빠르게)
  * 테스트 코드느 실행 시간은 빨라야 한다.
  * 느리게 되면 자주 돌리는 것을 주저하게 되고, 초반에 문제를 찾아내지 못해 나중에 고생한다.

* Independent(독립적으로)
  * 각 테스트는 의존하지 않고 독립적으로 실행되어야 한다.
  * 어떤 순서에 실행되도 테스트는 정상 작동해야한다.
  * 의존 적이면 에러 발생 시 원인을 찾기 힘들어 진다. 

* Repeatable(반복가능하게)
  * 어떠한 환경에서도 반복이 가능해야 한다. 
  * 실제 환경, QA 환경 등  환경에 구애받지 않고 실행이 가능해야한다. 
  
* Selft-Validating(자가검증)
  * 테스트는 boolean 값으로 결과를 내야 한다. 
  * 성공 아니면 실패다.
  * 결과를 아이체킹으로 하면 안된다. 
  * 테스트가 스스로 검증 하지 않는다면 판단은 주관적이 된다. 

* Timely(적시에) 
  * 테스트는 적시에 작성해야 한다.
  * 단위 테스트는 테스트 하려는 실제 코드를 구현하기 직전에 구현한다.
  * 실제코드를 만들고 테스트코드를 만들면 실제 코드를 테스트 코드로 만드릭 어려운 경우들이 생긴다. 

> 테스트 코드가 방치되면 코드는 망가지게 되고 실제 코드도 망가지게 된다.   
> 언제나 깨끗한 테스트 코도를 유지하도록 하는게 좋다.

## 클래스 

### 클래스 체계 
* 클래스를 작성할 때 맨처음 변수목록을 작성한다. 
  * 정적 static 
  * 공개 public 
  * 비공개 private
  * 인스턴스 비공개 변수 
  * 공개 변수가 필요한 경우는 거의 없다.
* 그 후에는 공개 함수가 나온다 (생성자 같은..)
* 비공개 함수는 공개함수 직후에 작성한다.
* 이처럼 추상화 단계가 순차적으로 신문처럼 읽히도록 작성한다. 

```java
class A {
	public static final PUBLIC_CONSTANT_VALUE = "public value";
	private static final PRIVATE_CONSTANT_VALUE = "private value";
	private final instanceValue = "instance value";

	public void publicMethod(){
		privateMethod();
	}
	private void privateMethod(){
		...
	}
}
```

### 캡슐화 
* 변수와 유틸리티 함수는 가능한 공개하지 않는 편이 좋다 (상황에 따라서..)

### 클래스는 작아야 한다
* 클래스도 작을 수록 좋다. 
* 클래스가 가진 책임을 척도로 작을 수록 좋다.
* 간결한 클래스이름이 떠오르지 않다면 그만큼 많은 책임을 지고있다는것이다. 
* 모호한 단어인 Processor, Manager, Super 등과 같이 모호한 단어를 쓰면 여러 책임을 떠 안을 확률이 크다.   
* 클래스 설명은 만일(if), 그리고(and), 하며(or), 하지만(but) 을 사용하지 않고 설명이 가능해야 한다.

> 밑의 소스의 SuperDashboard 클래스를 설명하면 마지막으로 포커스를 얻었던 컴포넌트에   
> 접근하는 방법을 제공(하며) 버전(과) 빌드 번호를 추적하는 메커니즘을 제공한다.   
> 이러첨 아래 클래스는 단위가 작은 클래스라는것은 충족하지만 책임이 아직도 많다.
```java
public class SuperDashboard extends JFrame implements MetaDataUser {
    public Component getLastFocusedComponent();
    public void setLastFocused(Component lastFocused);
    public int getMajorVersionNumber();
    public int MinorVersionNumbe();
    public int getBuildNumber();
}
```

### 단일 책임 원칙(SRP : Single Responsibility Principle) 
* 클래스나 모듈을 변경할 이유는 단 하나뿐이어야 한다는 말이다. 
* SRP는 책임이라는 개념을 정의하고 적절한 클래스 크기를 제시한다.
* 위의 소스에서 보면 SuperDashboard를 변경할 이유가 두 가지 있다. 
  1. SuperDashboard는 소프트웨어 버전 정보를 추적한다. 
    * 버전 정보는 소프트웨어를 출시할 때마다 달라진다.
  2. SuperDashboard는 자바 스윙 컴포넌트를 관리한다.
    * SuperDashboard는 최상위 GUI 윈도의 스윙 표현인 JFrame에서 파생한 클래스다.   
    * 즉 스윙 코드를 변경할 때마다 버전 번호가 달라진다.

> SuperDashboard에서 버전정보를 다루는 메서드 세 개를 따로 빼내 Version이라는   
> 독자적인 클래스를 만든다.   
> Version 클래스는 다른 애플리케이션에서 재사용하기 아주 쉬운 구조다. 

```java
public class Version {
    public int getMajorVersionNumber();
    public int getMinorVersionNumber();
    public int getBuildNumber();
}
```
* SRP는 중요한 개념이고 쉬운 규칙이지만 가장 무시하는 규칙 중 하나이다.
* 소프트웨어를 작동하게 하는것과 깨끗하게 만드는것은 완전 별개이기 때문이다. 
* 개발자들은 단일 책임 클래스가 많아지면 큰 그림을 이해하기 어려워 진다고 생각한다.
* 하지만 작은 크래스가 많은 것이든 큰 클래스가 몇개 뿐이든 돌아가는 부품의 수는 비슷하다.
* 즉 어떻게 관리하는가에 초점을 둬야 한다. 
* 작은 서랍별로 명확한 이름으로 정리하고싶은가 큰 서랍 몇개에 모든걸 던져 놓고 정리하고싶은가.
  * 큰 프로젝트가 어느 정도 수준에 올랐다고 가정하자.
  * 체계적으로 정리했다면 필요한 부분만 알면 되지만 큰 서랍은 알지 않아도 되는 것까지 알아야한다.
  

### 응집도(Cohesion)
* 클래스는 인스턴스 변수가 작아야 한다.
* 일반적으로 메서드가 변수를 더 많이 사용할수록 메서드와 클래스는 응집도가 더 높다.
* 모든 인스턴스 변수를 메서드마다 사용하는 클래스는 응집도가 가장높다.
* 응집도가 높은것은 바람직하지 않다.
* 하지만 개발자는 응집도가 높은 것을 좋아한다. 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는것을 의미하기 때문이다.
* '함수를 작게 매개변수 목록은 짧게' 전략을 따르면   
   때때로 몇몇 메서드만 사용하는 인스턴스 변수가 많아지는 경우가있다.   
   이것은 새로운 클래스로 쪼개야 한다는 신호다.   
   응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두세 개로 쪼개준다. 

### 응집도를 유지하면 작은 클래스 여럿이 나온다. 
* 큰 함수를 작은 함수로 여럿으로 나누면 클래스 수가 많아진다. 
* 책임도 쪼개지기에 유지보수 및 수정에 용이하다.

> 아래 예제를 보면 응집도다 현저히 낮은 코드다
```java
package literatePrimes;

public class PrintPrimes {
    public static void main(String[] args) {
        final int M = 1000; 
        final int RR = 50;
        final int CC = 4;
        final int WW = 10;
        final int ORDMAX = 30; 
        int P[] = new int[M + 1]; 
        int PAGENUMBER;
        int PAGEOFFSET; 
        int ROWOFFSET; 
        int C;
        int J;
        int K;
        boolean JPRIME;
        int ORD;
        int SQUARE;
        int N;
        int MULT[] = new int[ORDMAX + 1];

        J = 1;
        K = 1; 
        P[1] = 2; 
        ORD = 2; 
        SQUARE = 9;

        while (K < M) { 
            do {
                J = J + 2;
                if (J == SQUARE) {
                    ORD = ORD + 1;
                    SQUARE = P[ORD] * P[ORD]; 
                    MULT[ORD - 1] = J;
                }
                N = 2;
                JPRIME = true;
                while (N < ORD && JPRIME) {
                    while (MULT[N] < J)
                        MULT[N] = MULT[N] + P[N] + P[N];
                    if (MULT[N] == J) 
                        JPRIME = false;
                    N = N + 1; 
                }
            } while (!JPRIME); 
            K = K + 1;
            P[K] = J;
        } 
        {
            PAGENUMBER = 1; 
            PAGEOFFSET = 1;
            while (PAGEOFFSET <= M) {
                System.out.println("The First " + M + " Prime Numbers --- Page " + PAGENUMBER);
                System.out.println("");
                for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++) {
                    for (C = 0; C < CC;C++)
                        if (ROWOFFSET + C * RR <= M)
                            System.out.format("%10d", P[ROWOFFSET + C * RR]); 
                    System.out.println("");
                }
                System.out.println("\f"); PAGENUMBER = PAGENUMBER + 1; PAGEOFFSET = PAGEOFFSET + RR * CC;
            }
        }
    }
}

```
> 아래 코드는 그런 응집도를 높이고 클래스로 쪼개서 만든 코드들이다. 
```java
package literatePrimes;

public class PrimePrinter {
    public static void main(String[] args) {
        final int NUMBER_OF_PRIMES = 1000;
        int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);

        final int ROWS_PER_PAGE = 50; 
        final int COLUMNS_PER_PAGE = 4; 
        RowColumnPagePrinter tablePrinter = 
            new RowColumnPagePrinter(ROWS_PER_PAGE, 
                        COLUMNS_PER_PAGE, 
                        "The First " + NUMBER_OF_PRIMES + " Prime Numbers");
        tablePrinter.print(primes); 
    }
}
```
```java
package literatePrimes;

import java.io.PrintStream;

public class RowColumnPagePrinter { 
    private int rowsPerPage;
    private int columnsPerPage; 
    private int numbersPerPage; 
    private String pageHeader; 
    private PrintStream printStream;

    public RowColumnPagePrinter(int rowsPerPage, int columnsPerPage, String pageHeader) { 
        this.rowsPerPage = rowsPerPage;
        this.columnsPerPage = columnsPerPage; 
        this.pageHeader = pageHeader;
        numbersPerPage = rowsPerPage * columnsPerPage; 
        printStream = System.out;
    }

    public void print(int data[]) { 
        int pageNumber = 1;
        for (int firstIndexOnPage = 0 ; 
            firstIndexOnPage < data.length ; 
            firstIndexOnPage += numbersPerPage) { 
            int lastIndexOnPage =  Math.min(firstIndexOnPage + numbersPerPage - 1, data.length - 1);
            printPageHeader(pageHeader, pageNumber); 
            printPage(firstIndexOnPage, lastIndexOnPage, data); 
            printStream.println("\f");
            pageNumber++;
        } 
    }

    private void printPage(int firstIndexOnPage, int lastIndexOnPage, int[] data) { 
        int firstIndexOfLastRowOnPage =
        firstIndexOnPage + rowsPerPage - 1;
        for (int firstIndexInRow = firstIndexOnPage ; 
            firstIndexInRow <= firstIndexOfLastRowOnPage ;
            firstIndexInRow++) { 
            printRow(firstIndexInRow, lastIndexOnPage, data); 
            printStream.println("");
        } 
    }

    private void printRow(int firstIndexInRow, int lastIndexOnPage, int[] data) {
        for (int column = 0; column < columnsPerPage; column++) {
            int index = firstIndexInRow + column * rowsPerPage; 
            if (index <= lastIndexOnPage)
                printStream.format("%10d", data[index]); 
        }
    }

    private void printPageHeader(String pageHeader, int pageNumber) {
        printStream.println(pageHeader + " --- Page " + pageNumber);
        printStream.println(""); 
    }

    public void setOutput(PrintStream printStream) { 
        this.printStream = printStream;
    } 
}
```

```java
package literatePrimes;

import java.util.ArrayList;

public class PrimeGenerator {
    private static int[] primes;
    private static ArrayList<Integer> multiplesOfPrimeFactors;

    protected static int[] generate(int n) {
        primes = new int[n];
        multiplesOfPrimeFactors = new ArrayList<Integer>(); 
        set2AsFirstPrime(); 
        checkOddNumbersForSubsequentPrimes();
        return primes; 
    }

    private static void set2AsFirstPrime() { 
        primes[0] = 2; 
        multiplesOfPrimeFactors.add(2);
    }

    private static void checkOddNumbersForSubsequentPrimes() { 
        int primeIndex = 1;
        for (int candidate = 3 ; primeIndex < primes.length ; candidate += 2) { 
            if (isPrime(candidate))
                primes[primeIndex++] = candidate; 
        }
    }

    private static boolean isPrime(int candidate) {
        if (isLeastRelevantMultipleOfNextLargerPrimeFactor(candidate)) {
            multiplesOfPrimeFactors.add(candidate);
            return false; 
        }
        return isNotMultipleOfAnyPreviousPrimeFactor(candidate); 
    }

    private static boolean isLeastRelevantMultipleOfNextLargerPrimeFactor(int candidate) {
        int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
        int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor; 
        return candidate == leastRelevantMultiple;
    }

    private static boolean isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
        for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
            if (isMultipleOfNthPrimeFactor(candidate, n)) 
                return false;
        }
        return true; 
    }

    private static boolean isMultipleOfNthPrimeFactor(int candidate, int n) {
        return candidate == smallestOddNthMultipleNotLessThanCandidate(candidate, n);
    }

    private static int smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
        int multiple = multiplesOfPrimeFactors.get(n); 
        while (multiple < candidate)
            multiple += 2 * primes[n]; 
        multiplesOfPrimeFactors.set(n, multiple); 
        return multiple;
    } 
}

```

위의 코드를 정리해서 말하면 아래와 같다. 
* PrimePrinter 
  * main 함수 하나를 포함하여 실행 환경을 책임진다. 
  * 호출 방식을 바꾸러면 PrimePrpinter 클래스에서 변경하면된다.
* RowColumnPagePrinter 
  * 숫자 목록을 주어진 행과 열에 맞춰 페이지에 출력을 책임진다. 
  * 출력하는 모양새를 변경하려면 RowColumnPagePrinter에서 변경하면된다.
* PrinterGenerator 
  * 소수 목록을 생성 
  * 소수를 계산하는 알고리즘을 변경하려면 PrinterGenerator를 변경하면된다.

### 변경하기 쉬운  클래스 
* 대다수의 시스템은 지속적으로 변경이 이루어진다. 
* 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 대한 위험도를 낮춘다.

```java
public class Sql {
    public Sql(String table, Column[] columns)
    public String create()
    public String insert(Object[] fields)
    public String selectAll()
    public String findByKey(String keyColumn, String keyValue)
    public String select(Column column, String pattern)
    public String select(Criteria criteria)
    public String preparedInsert()
    private String columnList(Column[] columns)
    private String valuesList(Object[] fields, final Column[] columns) 
	private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```
* 새로운 SQL 문 update같은 것을 지원하려면 반드시 Sql클래스를 손대야 한다. 
* 기존 SQL문을 수정하려해도 Sql 클래스를 손대야 한다.
* 예를 들어 select문에 내장된 select문을 수정하려면 Sql 클래스를 손대야한다.
* 이렇든 변갱해야 할 이유가 두 가지 이므로 Sql은 SRP를 위반하게 된다.

```java
abstract public class Sql {
	public Sql(String table, Column[] columns) 
	abstract public String generate();
}
public class CreateSql extends Sql {
	public CreateSql(String table, Column[] columns) 
	@Override public String generate()
}

public class SelectSql extends Sql {
	public SelectSql(String table, Column[] columns) 
	@Override public String generate()
}

public class InsertSql extends Sql {
	public InsertSql(String table, Column[] columns, Object[] fields) 
	@Override public String generate()
	private String valuesList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extends Sql { 
	public SelectWithCriteriaSql(
	String table, Column[] columns, Criteria criteria) 
	@Override public String generate()
}

public class SelectWithMatchSql extends Sql { 
	public SelectWithMatchSql(String table, Column[] columns, Column column, String pattern) 
	@Override public String generate()
}

public class FindByKeySql extends Sql public FindByKeySql(
	String table, Column[] columns, String keyColumn, String keyValue) 
	@Override public String generate()
}

public class PreparedInsertSql extends Sql {
	public PreparedInsertSql(String table, Column[] columns) 
	@Override public String generate() {
	private String placeholderList(Column[] columns)
}

public class Where {
	public Where(String criteria) public String generate()
	public String generate() {
}

public class ColumnList {
	public ColumnList(Column[] columns) public String generate()
	public String generate() {
}
```
* Sql 클래스를 상속 하여 파생 클래스로 만들었다.
* 클래스가 단순해 졌다.
* 수정이 이루어졌을 때 다른 클래스나 함수가 망가질 염려가 없다.
* update문을 추가할 때 기존 클래스를 변경할 필요가 전혀 없다.
* OCP(open-closed Principle)도 따른다.

### 변경으로부터 격리
* 구현하는 클래스는 구현이 바뀌면 위험에 빠진다.
* 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리해야 한다.
* 상세한 구현에 의존하는 코드는 테스트가 어렵다.
* 시스템의 결합도를 낮춰야 한다. 
* 결합도가 낮다는 소리는 변경으로부터 잘 격리 되어있다는 소리이다.
* 이렇게 결합도를 낮추면 DIP(Dependency Inversion Principle)를 따르는 클래스가 나온다. 
* 본질적으로 DIP는 상세한 구현이 아니라 추상화에 의존해야한다는 원칙이다.

## 시스템 

### 시스템 제작과 시스템 사용을 분리하라.
* 소프트웨어 시스템은 객체 제작 및 의존성 연결 등 준비과정과 런타임 로직을 분리해야 한다.
* 관심사 분리는 우리 분야에서 가장 오래되고 가장 중요한 설계 기법 중 하나이다.
* 대부분 시작 단계라는 관심사를 분리하지 않는다.   
  준비 과정 코드를 주먹구구식으로 구현할 뿐만 아니라 런타임 로직과 마구 뒤섞는다.   
  다음이 전형적인 예이다.
```java
public Service getService() {
    if(service == null) {
        service = new MyServiceImp(...); //모든 상황에 적합한 기본값일까?
        return service;
    }
}
```
장점
* 초기화 지연 혹은 계산 지연 이라는 기법이다.
* 필요할 때까지 객체를 생성하지 않아 불필요한 부하를 막을 수 있다.
  * 그만큼 애플리케이션 시작하는 시간이 단축된다.
* 어떤 경우에도 null 포인터를 반환하지 않는다.
단점
* 하지만 getService 메서드는 MyServiceImp과 생성자 인수에 명시적으로 의존한다.
* 런타임에서 MyServiceImpl 객체를 전혀 사용 하지 않더라도 의존성을 해결하지 않으면 컴파일이 안된다.
* getService에 대한 테스트도 문제다. MyServiceImpl이 무거운 객체라면 단위 테스트에서는 적절한 테스트 전용 객체로 교체해야 한다.
* null인지 아닌지 분기에 대한 테스트 코드도 양이 늘어난다. 
* MyServiceImpl이 모든 상황에 적합한 객체인지 모른다.

```java
//생성로직 
if (service == null) {
        service = new MyServiceImpl(...);
}
```

```java
//사용로직 
return service;
```

### Main 분리 
* 시스템 생성과 시스템 사용을 분리하는 한 가지 방법 
* 생성 관련 코드는 main이나 main이 호출하는 모듈로 옮긴다.

<img src="https://quaint-drill-0fe.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F8c577b52-40ea-4764-91b6-ce2653c4968e%2FUntitled.png?table=block&id=0d522afb-cc00-48e8-a617-d4f2327139bd&spaceId=ede5285e-0482-4b4e-a93f-68588263cf6e&width=2000&userId=&cache=v2" width="90%"></img>
> main , builder, configured Object, application
* main 함수에서 시스템에 필요한 객체를 생성한 후 이를 애플리케이션에 넘긴다.
* 애플리케이션은 그저 객체를 이용할 뿐이다.
* 애플리케이션은 main 이나 객체 생성과정을 전혀 모른다.

### 팩토리 
* 객체가 생성되는 시점을 애플리케이션이 결정할 필요도 있다.
  * 예를 들어 주문처리 시스템에서 애플리케이션은 LineItem 인스턴스를 생성해 Order에 추가한다.
* 여기서 ABSTRACT FACTORY 패턴을 사용한다.
  * LineItem 생성하는 시점은 애플리케이션이 결정하지만   
    LineItem을 생성하는 코드는 애플리케이션이 모른다.
<img src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fcd3a8022-30d7-4c9d-8416-2316260d5fb5%2FUntitled.png?table=block&id=faed0628-87cb-4b39-83f8-b2606f35495d&spaceId=ede5285e-0482-4b4e-a93f-68588263cf6e&width=2000&userId=f4a0ae3f-1de7-44f4-8f5d-2c76d8126d03&cache=v2" width="90%"></img>


### 의존성 주입
* 생성과 사용을 분리하는 강력한 메커니즘 하나가 의존성 주입(Dependency Injection)이다.
* 의존성 주입은 제어 역전 IoC(Inversion of Control)기법을 의존성 관리에 적용한 메커니즘이다. 
* 제어 역전은 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다.
* 새로운 객체는 넘겨받은 책임만 맡으면 되므로 SRP(단일 책임 원칙)을 지키게 된다.


### 확장 
* 단순한 아키텍처를 복잡한 아키텍처로 조금씩 키울 수 없다는 현실이 있다. 
* 관심사를 적절히 분리한다면 아키텍처는 점진적으로 발전이 가능하다. 
* 만약 비즈니스 로직이 클래스를 생성하는 컨테이너에서 적용되어잇다면 비즈니스로직은 컨테이너와 밀접하게 결합되어있게 된다.   
  * 독자적인 단위테스트가 어렵다.
  * 테스트할 때도 서버 context의 영향을 받게되므로 다른 환경에서 테스트를 할 수가 없다. 
  * 재사용하기 어려운 코드가 된다.

### 횡단(cross-cutting) 관심사 
* EJB2 아키텍처는 관심사를 거의 완벽하게 분리했다.
* 완벽하게 분리된 관심사에서는 동일한 방법으로 중복적인 부분이 생길 수 있다. 
  * 영속성 관심사는 인스턴스를 생성 할 때 동일한 방법으로 생성할 하는 것처럼...
* 이렇게 완벽하게 분리된 관심사 속에 동일한 기능이 여기저기 흩어져 있는 관심사를 횡단 관심사라고 한다.
* 이러한 개념은 미래에 관점 지향 프로그래밍 AOP(Aspect-Oriented Programing)을 예견했다. 


<img src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F7151c8a3-86e6-41f1-ba03-4df50503889e%2FUntitled.png?table=block&id=f1e68c71-47ae-4431-9857-c3510de34f93&spaceId=ede5285e-0482-4b4e-a93f-68588263cf6e&width=2000&userId=f4a0ae3f-1de7-44f4-8f5d-2c76d8126d03&cache=v2" width="90%"></img>


### 자바 프록시 
* 개별 객체나 클래스에서 메서드 호출을 감싸는 경우에 사용이 적합하다. 
* 코드의 양과 크기는 프록시의 두가지 단점이다.

### 순수 자바 AOP 프레임워크
* 횡단 관심사를 프록시나 바이트코드 라이브러리를 사용해 관점을 분리하는 프로그래밍 기법을 말한다.
* 트랜잭션, 캐싱, 보안 등 많은 부분에서 사용 가능하다. 
* 스프링관련 코드와 애플리케이션코드과 독립적으로 되어 있어 유지보수가 쉬우며 
* XML 및 어노테이션으로 기존 코드 변경 없이 수정이 용이하다.

### 테스트 주도 시스템 아키텍처 구축
* 괌점으로 관심사를 분리하면 그 위력은 막강하다. 
* 테스트 주도 아키텍처 구축이 가능해진다. 
* 확장이 용이하다.

### 명백한 가치가 있을 때 표준을 현명하게 사용하라
* 표준을 사용하면 재사용성이 좋고, 경험을 가진 사람을 구하기 좋으며 캡슐화도 좋다. 
* 하지만 표준에 어울리지 않은 프로그램을 개발할 때는 무용지물일뿐이다. 
  * 가볍게 간단한 설계로 충분한 프로젝트를 표준에 맞추게 되면 시간과 비용을 많이 소모하게 된다. 

### 시스템은 도메인 특화 언어가 필요하다.
* DSL(Domain-Specific Language)은 간단한 스크립트 언어나 표준 언어로 구현한 API를 가리킨다.
* 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 '의사소통' 간극을 줄여준다. (stream, collector...)


## 창발성 

### 창발적 설계로 깔끔한 코드를 구현하자
단순한 설계 규칙 4가지 (켄트 백)
* 모든 테스트를 실행한다.
* 중복을 없앤다.
* 프로그래머 의도를 표현한다.
* 클래스와 메서드 수를 최소로 줄인다.

### 단순한 설계 규칙 1 : 모든 테스트를 실행하라.
* 설계를 의도한 대로 돌아가는 시스템을 개발해야한다.
* 테스트가 가능한 시스템을 만들어야 한다 (즉 검증도 가능함)
* SRP를 준수하는 클래스는 테스트가 더 쉽다.
* 결합도가 높으면 테스트가 어려워 진다. DIP같은 원칙을 적용하고 DI, 인터페이스 등을 사용해서 결합도를 낮춰야 한다. 

### 단순한 설계 규칙 2~4: 리팩터링 
* 코드를 점진적으로 리팩터링 해나간다.
* 새로 추가하는 코드는 리팩터링한 후 테스트 케이스를 돌려 기존 기능에 문제가 없는 것을 확인한다.
* 소프트웨어의 설계 품질을 높여라. 

### 중복을 없애라
* 우수한 설계에서 중복은 커다란 적이다.
* 중복은 추가 작업, 추가 위험, 불필요한 복잡도를 뜻하기 때문이다.
* 소규모 재사용성은 시스템 복잡도를 줄여 준다.

```java
int size() {}
boolean isEmpty() {}
```
=>
```java
boolean isEmpty() {
    return 0 == size();    
}
```

### 표현하라
* 클래스나 함수의 이름은 좋은 이름으로 선택해라.
* 함수와 클래스의 크기는 가능한 줄인다.
* 표준 명칭을 사용하라(알고리즘이나 패턴 이름등..)
* 단위 테스트 케이스를 꼼꼼히 작성한다.

### 캘르새으 메서드 수를 최소로 줄여라
중복을 제거하고 의도를 표현하고 SRP를 준수한다는 기본적인 개념도 극단으로 치달으면 득보다는 실이 많다.      
(클래스와 메서드 크기를 최소로 하다보니 수가 많아진다..)   

그래서 함수와 클래스 수즐 가능한 줄이라고 제안한다.    

무의미하고 독단적인 정책 탓에 클래스 및 메서드 수가 늘어나기도 한다. (클래스 마다 인터페이스를 구현하라는 정책..)   

즉 함수와 클래스 크기를 작게 유지하면서 동시에 시스템 크기도 작게 유지하는데 있다.   
하지만 이건 위의 규칙중에 우선순위가 가장 낮다. 
