# cleancode - chapter 4-8

## 형식 맞추기 

형식에 맞춰서 잘 짜야 그 코드가 신뢰가 생기고 우아한코드가 된다.

### 형식을 맞추는 목적 
* 코드 형식은 의사 소통의 일환이다. 
    > 처음 짠 코드라 나중에 새로운 버전으로 수정이 발생해도 처음 잡아놓은 구현 스타일   
      , 가독성 수준은 계속 영향을 미친다.

### 적절한 행 길이를 유지하라. 
세로 길이는 작을 수록 좋다. 함수와 같은 개념이다.
junit, tomcat같은 큰 프로젝트도 대다수 파일이 500줄이 넘어가는 파일이 없으며,   
대다수 200줄 미만이라고 한다.   
즉, 파일 라인 수가 작게 만들어도 거대한 시스템을 개발할 수 있다는 말이 된다. 

### 신문 기사처럼 작성해라.
* 최상단은 요약하라 *구독자는 처음을 읽고 더 읽을지 말지를 결정한다.* 
* 밑으로 내려갈수록 구체적으로 작성하라.

### 개념은 빈 행으로 분리하라. 
* 각 행은 수식이나 절을 나타낸다.
* 일련의 행 묶음은 완결된 생각 하나를 표현한다.
* 생각 사이에는 빈 행을 넣어 분리해야 마땅하다. 
* 빈 행은 새로운 개념을 시작한다는 시각적 단서이다. 

```java
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?'''", Pattern.MULTILINE + Pattern.DOTALL);
    
    public BoldWidget(ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1));
    }
    
    public String render() throws Exception {
        StringBuffer html = new StringBuffer("<b>");
        html.append(childHtml()).append("</b>");
        return html.toString();
    }
}
```

> 패키지 선언부, import, 각 함수 사이에 빈 행이 들어간다.   
  기본적인 규칙이지만 가독성 및 의미부여에 중요한 역할을 한다. 

```java
package fitnesse.wikitext.widgets; 
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?'''", Pattern.MULTILINE + Pattern.DOTALL);
    public BoldWidget (ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher (text);
        match.find();
        addChildWidgets (match.group(1));}
    public String render() throws Exception { 
        StringBuffer html = new StringBuffer("<b>"); 
        html.append(childHtml()).append("</b>");
    return html.toString();
}
}
```
> 위의 코드는 빈 행을 뺀 코드이다.   
  보기 불편하고 가독성이 떨어진다.



### 세로 밀집도 
* 줄바꿈이 개념을 분리한다면 세로 밀집도는 연광성을 의미한다.
* 서로 밀접한 코드 행은 세로로 가까이 놓아야 한다.

 
```java
public class ReporterConfig {
    /**
    *리포터 리스너의 클래스 이름
    */
    private String m_className;

    /**
    * 리포터 리스너의 속성
    */
    private List<Property> m_properties = new ArrayList<Property>(); 
    public void addProperty (Property property) {
        m_properties.add(property);
    }
```
> 주석으로 세로를 띄워 놓아 더 혼란스럽기만 할 뿐이다 아래 코드를 보자 


```java
public class ReporterConfig {
    private String m_className;
    private List<Property> m_properties = new ArrayList<Property>();
    public void addProperty (Property property) {
        m_properties.add(property);
    }
```
> 더 깔끔하고 한눈에 잘보인다. 

### 수직거리 
* 서로 밀접한 개념은 세로로 가까이 두어야 한다. 
* 위 아래 중구난방이면 해당 소스를 찾기 위해 시간과 노력이 든다. 

### 변수선언
* 변수는 사용하는 위치에 최대한 가까이에 선언한다.

```java
private static void readPreferences() {
	InputStream is = null;
	try {
		is = new FileInputStream(getPreferencesFile());
		setPreferences(new Properties(getPreferences()));
		getPreferences().load(is);
	} catch (IOException e) {
		try {
			if (is != null)
				is.close();
		} catch (IOException e1) {
	}
}

```

### 인스턴스 변수 
* 인스턴스변수는 클래스 맨 처음에 선언한다.
* 변수간에 세로로 거리를 두지 않는다. 

```java
public class TestSuite implements Test {
	static public Test createTest(Class<? extends TestCase> theClass, String name) {
		...
	}
	
	public static Constructor<? extends TestCase>
	getTestConstructor(Class<? extends TestCase> theClass)
	throw NoSuchMethodException {
		...
	}
	
	public static Test warning(final String message){
		...
	}
	
	private static String exceptionToString(Throwable t) {
		...
	}
	
	private String fName;
	private Vector<Test> fTests = new Vector<Test>(10);
	
	public TestSuite() {
	}
	
	public TestSuite(final Class<? extends TestCase> theClass) {
		...
	}
}

```

> 인스턴스가 위와 같이 중간에 끼면 찾기 불편하다. 

### 종속함수 
* 함수가 다른 함수를 호출한다면 세로로 가까이 배치한다.
* 호출 되는 순서에 따라 배치를 위에서 아래로 한다.
* 다음에 정의하는 함수가 무엇인지 예상할 수 있어서 코드가 읽기 편해진다.

### 개념적 유사성 
* 개념적으로 유사한 코드는 서로 같은 위치에 놓는것이 좋다. 

```java
public class Assert {
    static public void assertTrue(String message, boolean condition) {
        if(!confition) {
            fail(message);
        }
    }

    static public void assertTrue(boolean condition) {
        assertTrue(null, condition);
    }

    static public void assertFalse(String message, boolean condition) {
        assertTrue(message, condition);
    }
}
```

> 위의 함수들은 개념적인 친화도가 매우 놓다.   
  명명법이 똑같고 기능이 유사하다.


### 세로 순서
* 호출되는 함수는 호출하는 함수보다 나중에 배치한다.
* 신문기사 처럼 중요한 내용을 먼저 표출 하고, 자세한 내용을 나중에 표출하는것과 일맥상통하다.
* 첫 함수 몇개만 읽어도 개념을 파악하기 쉬워진다.

### 가로 형식 맞추기 
* 세로 줄과 같이 가로줄도 대형 프로젝트를 비교해보면  20 ~ 60자 사이가 많다 평군 45자이다. 
* 이 결과는 개발자들은 짧은 행을 선호한다는 것이다.

### 가로 공백과 밀집도 
* 가로로는 공백을 사용해 밀집한 개념과 느슨한 개념을 표현한다. 다음함수를 살펴보자 

```java
private void measureLine(String line) {
    lineCount++;
    int lineSize = line.length();
    totalChars = lineSize;
    lineWidthHistogram.addLine(lineSize, lineCount);
    recordWidestLine(lineSize);
}
```

* 할당 연산자를 강조 하기 위해 앞뒤로 공백을 넣었다. 
* 함수안의 인수는 공백으로 분리했다.

### 가로 정렬

```java
public class FitNesseExpediter implements ResponseSender {
    private     Socket              socket;
    private     InputStream         input; 
    private     OutputStream        outpu;
    private     Request             request; 
    private     Response            response;
    private     FitNesseContext     context;
    protected   long                requestParsingTimeLimit;
    private     long                requestProgress;
    private     long                requestParsngDeadline;
    
    public FitnesseExpediter(Socket FitnesseContext context) throws Exception {
        this.context = context;
        socket = s; 
        input  = s.getInputStream();
        outpu  = s.getOutputStream();
    }
} 
```

* 위와 같이 정렬을 하면 오른쪽에 있는 변수만 보게 된다. 왼쪽에있는 데이터 타입을 보지 않게 된다. 
* 변수에만 집중하게 되어 더욱 혼란을 가지고 올 수 있다. 

> 하지만 위의 포맷도 나름 괜찮은 포맷이라고 생각한다.   
  빠르게 변수를 찾을 수 있고 데이터 타입은 요새 IDE가 잘되어있어서 바로 찾을 수 있기 때문이다. 

### 들여쓰기
* 들여쓰기는 소스전체의 윤곽도를 잡아준다. 
* 클래스, 함수, 블록 등 각각의 범위로 들여쓰기를 해서 가독성을 높여준다. 
* 전체 파일을 이루는 클래스는 들여쓰기를 하지 않는다. 
* 메서드 및 블록 등은 들여쓰기를 한다. 
* 가독성이 좋아 구독자가 원치 않은 부분은 스킵이 가능하다. 

```java
public class FitnesseServer implements SocketServer { private FitNeesseContext context; public FitNesseServer(FitNesseContext
context) { this.context = context; public void serve(Socket s) { serve(s, 1000);} public void serve(Socket 
s, long requestTimeout) {try {FitNesseExpediter sender = new FitNesseExpediter(s, context); 
sender.setRequestParsingTimeLimit(requestTimeout); sender.start();} catch(Exception e) { 
e.printStackTrace();}}}
```
> 위과 같이 들여쓰기를 하지 않았다면 가독성이 현저히 떨어진다.   
  만약 들여쓰기를 잘했더라면 변수, 생성자, 함수 등이 한눈에 들어 올것이다.

### 들여쓰기 무시하기 
* if 문, while문, 짧은 함수 등 들여쓰기를 무시하고 한줄로 표현하고싶을 때가 생긴다. 
* 개발할 때는 편하고 개발 당시에는 가독성이 좋은것처럼 느껴지지만 실제로는 그렇지 않다. 

```java
public CommentWidget(ParentWidget parent, String text){super(parent, text);}
```
  
> 위의 코드는 한줄로 표현되어서 깔끔해 보일 수도 있다.   
> 하지만 실제로 분석을 하려고 하면 중간의 중, 소 괄고 때문에 눈이 아플 것이다. 
> 아래와 같이 작성을 하도록 하자 

```java
public CommentWidget (ParentWidget parent, String text) {
    super(parent, text);
}
```

### 팀 규칙 
* 클린코드란 궁극적인 목적은 읽기 쉬워야 하며 개발이 쉬어야 한다는것이다. 
* 많은 규칙을 정의 하고있지만 제일 중요한 규칙은 팀 규칙이다. 
* 팀내에서 어떤 코드 포맷으로 할지 정해지면 그 코드로 모든 팀원이 개발을 한다. 
* 어떤 파일을 보더라고 같은 포맷으로 구성되어있다면 개발 및 분석이 더 쉬워진다. 
* 이런 중요함 때문에 여러 IDE에서는 포맷을 설정하고 그 설정을 공유할 수 있도록 하는 시스템도 존재한다. 

## 객체와 자료 구조 

### 자료 추상화 
* 클래스의 구현체를 사용하는것보다는 인터페이스를 이용해 사용하는것이 좋다. 
* 클래스 구현체는 구현체를 수정하고싶은 욕구가 생긴다. 
* 인터페이스는 조회 및 함수 등을 강제 하여 수정하지 못하게 한다. 
* 구현하는 논리만 강제 하여 실제 구현체는 몰라도 된다.

### 자료 객체 비대칭 
* java에는 객체 지향프로그램으로 객체가 무조건 좋다라는 의미가 강하다. 
* 하지만 자료구조와 객체를 상황에 맞게 쓰는것을 권장한다. 

| 자료구조                | 객체                | 
|---------------------|-------------------|
| 자료 데이터만 존재, 함수 존재 x | 자료 데이터 x, 함수 존재 o | 
| 함수추가 쉬움             | 함수 추가 어려움         | 
| 새로운 객체 추가 어려움       | 새로운 객체 추가 쉬움      |


### 디미터 법칙 
* 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다. 

```java 
@Getter
public class Employee {

    private final String name;
    private final Enterprise enterprise;
    
    public int getEnterprisePostalCode() {
        return this.enterprise.getAddressPostalCode();
    }

}

@Getter
public class Enterprise {

    private final int employeeNumber;
    private final String domain;
    private final Address address;

    public int getAddressPostalCode() {
        return this.address.getPostalCode();
    }
    
}

@Getter
public class Address {

    private final String street;
    private final int postalCode;
    private final String city;

}
```

```java
public class App {

    public static void main(String[] args) {
        final Address address = new Address("종로구 청와대로 1", 03054, "서울특별시");
        final Enterprise enterprise = new Enterprise(100, "청와대", address);
        final Employee employee = new Employee("안주형", enterprise);

        // 1번 - 디미터 법칙 위반 코드!!
        System.out.println(employee.getEnterprise().getAddress().getPostalCode());

        // 2번 - 디미터 법칙 준수 코드!!
        System.out.println(employee.getEnterprisePostalCode()); 
    }
}
```

> 어떤 객체가  어떠한 자료를 가지고있는지 지나치게 잘 아는것이 문제가 된다.    
> 만약 체이닝으로된 소스가 소스에 많아지면 모든 소스를 수정하는 상황이 온다 하지만 함수는 함수 내부만 수정하면된다.    
> 즉 객체의 결합도를 줄이기 위한 개념이다. 


### 자료 전달 객체 
* 자료구조는 때로는 자료 전달 객체(Data Transfer Object) DTO라고도 불리운다.
* bean으로 등록이 되면언 get, set함수가 붙어진다 
* 자료구조지만 bean에서는 의무적으로 get, set 함수를 정의해야한다. 



## 오류 처리 

### 오류 코드보다 예외를 사용해라 
* 오류코르를 리턴 하는 방법은 오류코드를 분석해야 해서 가독성도 떨어지고 시간도 많이 걸린다. 
```java
if( handle != DeviceHandle.INVALID){
        ...
        if(recod.getStatus() != DEVICE_SUSPEND){
            ...
        } else {
            ...
        }
}
```

* 예외를 사용하면 가독성이 보기 좋고 컴파일 에러도 줄일 수 있다. 
```java
try {
        ...    
} catch {
    throw new ErrorHandler();    
}
``` 

### Try-Catch-Finally 문부터 작성하라. 
* try 블록에는 실행할 로직이, catch 블록에는 에러가 발생하면 처리할 로직을 작성한다.
* try-catch 문을 작성하면 구분이 명확해서 읽기 쉬워진다


### 미왁인unchecked 예외를 사용 하라

checked 예외는 컴파일 단계에서 확인되며 반드시 처리해야 하는 예외
* IOException
* SQLException

Unchecked 예외는 실행 단계에서 확인되며 명시적인 처리를 강제하지는 않는 예외
* NullPointerException
* IllegalArgumentException

확인 예외 단점
* 하위 메소드에서 예외를 명시하면 상위 레벨 메소드에서도 명시된 예외를 명시해야 한다.
* 각 메소드마다 의존적이으로 OCP에 위배된다.

>단순한 코드
```java
public void printA(bool flag) {
     if(flag) {
      System.out.println("called");
    }
}

public void func(bool flag) {
     printA(flag);
 }
```

>명시적 예외를 작성한 코드

```java
public void printA(bool flag) throws NotPrintException {
     if(flag)
         System.out.println("called");
     else
         throw new NotPrintException();
 }

 public void func(bool flag) throws NotPrintException {
     printA(flag);
 }
``` 

### 예외에 의미를 제공하라
예외를 던질 때 예외 메시지 및 코드를 던지면 예외 분석이 더 쉬워진다. 

### 호출자를 고려해  예외 클래스를 정의하라
오류를 처리하는 방법은 많다. 효율적으로 처리하는 방법을 알아야한다. 

> 아래 코드는 외부 라이브러리를 호출하는 코드이다. 
> 모든 예외를 호출하는 코드에서 예외처리를 하고있다. 

```java
ACMEPort port = new ACMEPort(12);

 try {
     port.open();
 } catch (DeviceResponseException e) {
     reportPortError(e);
     logger.log("Device response exception", e);
 } catch (ATM1212UnlockedException e) {
     reportPortError(e);
     logger.log("Unlock exception", e);
 } catch (GMXError e) {
     reportPortError(e);
     logger.log("Device response exception");
 } finally {
     ...
 }
```

> 다음 아래 코드를 보자 
> 호출 라이브러리를 한번 감쌌다(wrapper) 해당 라이브러리에서 발생하는 에러는 호출자에서 한번만 정리하면된다. 

```java
LocalPort port = new LocalPort(12);
 try {
     port.open();
 } catch (PortDeviceFailure e) {
     reportError(e);
     logger.log(e.getMessage(), e);
 } finally {
     ...
 }
 
```

```java
public class LocalPort {
     private ACMEPort innerPort;

     public LocalPort(int portNumber) {
         innerPort = new ACMEPort(portNumber);
     }

     public void open() {
         try {
             innerPort.open();
         } catch (DeviceResponseException e) {
             throw new PortDeviceFailure(e);
         } catch (ATM1212UnlockedException e) {
             throw new PortDeviceFailure(e);
         } catch (GMXError e) {
             throw new PortDeviceFailure(e);
         }
     }
     ...
 }

```

### null을  반환하지 마라 
* null을 반환하면 호출자가 null에 대한 책임을 지게 된다. 
* null 체크 소스가 많아지면 가독성이 떨어진다. 

> null 체크를 호출자가 하는 코드이다.
```java
public void registerItem(Item item) {
	if (item != null) {
		ItemRegistry registry = peristentStore.getItemRegistry();
		if (registry != null) {
			Item existing = registry.getItem(item.getID());
			if (existing.getBillingPeriod().hasRetailOwner()) {
				existing.register(item);
			}
		}
	}
}
```
> 아래와 같이 정리 하면 더 깔끔해진다.
```java
List<Employee> employees = getEmployees();
for(Employee e : employees) {
	totalPay += e.getPay();
}

public List<Employee> getEmployees() {
	if (..직원이 없다면..)
		return Collections.emptyList();
}
```

### null을 전달하지 마라 
* 인수로 null을 전달하는것은 무의미하다. 
* 값을 기대하여 인수를 받을려고 하기 때문이다. 

> null을 인수로 받는 다면 아래와 같이 null을 체크 하는 로직이 들어가야한다. 
```java
public class MetricsCaculator {
    public double xProjection(Point p1, Point p2) {
      if (p1 == null || p2 == null) {
        throw InvalidArgumentException("Invalid argument for MetericsCalculator.xProjection");
      } 
      return (p2.2 - p1.x) * 1.5;
    }    
}
```

* 대다수의 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다.
* 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다.


## 경계 
* 시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다. 
* 때로는 패키지를 사거나, 오픈 소스를 이용하는 경우가 있다. 
* 이런 외부 코드를 직접 개발한 코드와 경계를 깔끔하게 하는 방법에 대해 알아보자 

### 외부 코드 사용하기 

* 인터페이스 제공자는 사용자의 편의를 위해서 최대한 많은 기능을 주려고한다.
* 사용자인 개발자는 모든 기능은 필요하지 않고 내가 쓰고 싶은 기능만 알고싶어한다. 
* 이런 차이로 인해 시스템의 경계가 문제가 있을 수 있다. 

예시 - java.util.Map
* clear() void - map
* containsKey(Object key) boolean - Map
* containsValue(Object value) boolean - Map
* entrySet() set - Map
* equals(Object o) boolean - Map
* get(Object key)Object - Map
* getClass() Class<? extends Object> - Object
* hashCode() int - Map
* isEmpty() boolean - Map
* keySet() Set - Map
* notify() void - Object
* notifyAll() void - Object
* put(Object key, Object value) Object - Map
* putAll(Map t) void - Map
* remove(Object key) Object - Map
* size() int - Map
* toString() String - Object
* values() Collection - Map
* wait() void - Object
* wait(long timeout) void - Object
* wait(long timeout, int nanos) void - Object

> Map은 굉장히 다양한 인터페이스로 수많은 기능을 제공한다. 
> 유용한 기능들이지만 오용하면 위험하다. 

예를 들어서 
* Map을 인수 인자나 반환 리턴값으로 사용했을 경우 상위 코드에서 clear()라는 함수를 호출하는 순간 예상치 못한 상황이 벌어질 수 있다.
* Map에 특정 유형만 저장하기로 했는데 유형을 제한하지않아  예상지 못한 상황이 생길 수 있다.(Generics 지원이 없는 이전 시점..)

```java
    Map sensors = new HashMap();
    Sensor s = (Sensor)sensors.get(sensorId);
```
> 맵을 형변환을 통해 Snssor라는 특정 객체만 저장할 수 있다. 하지만 Sensor가 아닌 다른 객체로 캐스팅 후 저장도 가능한 상황이 있을 수 있다.

```java
    Map<String, Sensor> sensors = new HashMap<String, Sensor>();
    Sensor s = sensors.get(sensorId); 
```
> 그것을 방지할 수 있는 방법이 제네릭을 사용해서 방지 할 수 있다. 하지만 호출자에세 Map<String, Seonsor>를 사용하라는정보 까지 굳이 할 필요는 없다. 

```java
public class Sensors {
	private Map sensors = new Hashmap();

	public Sensor getById(string id) {
		return (sensor) sensors.get(id);
	}
}
```
> 위처럼 아예 함수로 Sensor 객체를 강제적으로 리턴하는 함수만 사용하게 한다면 문제는 해소 된다. 

> 이런 경계에 있는 인터페이스들은 코드 상에서 값을 리턴, 인수로 사용하는 방법은 적절하지 않다. 
> 인터페이스는 수정이 일어나지 않을거라 생각하지만 그렇지 않은 경우도 있다. 만약 수정이 일어나게되면 모든 코드에있는 부분을 수정해야 하는 대혼란이 온다.

### 경계 살피고 익히기
* 외부패키지를 사용하면 적은시간에 개발을 완료할 수 있을 만큼 생산성이 좋다.
* 외부 패키지 테스트는 개발자의 책임은 아니지만, 개발자가 개발하는 프로그램을 위해 사용할 코드를 테스트하는걸 권장한다.

* 하지만 외부 패키지는 많은 정보가 없으면 익히기도 어렵고 외부코드를 통합하기도 어렵다.
* 그래서 코드를 작성 하기 전에 테스트케이스를  작성해 코드를 외부패키지를 익히면서 테스트할 수 있다. 이것을 *학습테스트* 라고 부른다
* 학습테스트는 통제된 환경에서 API를  제대로 이해하는지  확인하는 것으로 외부패키지를 사용하려는 목적에 초첨을 맞춘다.

### log4j 익히기
학습테스트 절차를 log4j를 익힌다는 예를 들어 설명하겠다. 

1. 패키지에 대한 소개 페이지를 보고 테스트케이스 작성 
```java
// 화면에 "hello"를 출력하는 테스트 케이스이다.
 @Test
 public void testLogCreate() {
     Logger logger = Logger.getLogger("MyLogger");
     logger.info("hello");
 }
```

2. 테스트 케이스 실행 
   * Appender라는 뭔가가 필요하다는 에거 발생

3. 다시 문서정독 -> ConsoleAppender라는 클래스가 필요하다는걸 인식 
```java
@Test
 public void testLogAddAppender() {
     Logger logger = Logger.getLogger("MyLogger");
     ConsoleAppender appender = new ConsoleAppender();
     logger.addAppender(appender);
     logger.info("hello");
 }
```
> Appender에 출력 스트림이 없다는 사실을 발견

4. 구글링 후 수정
```java
 @Test
 public void testLogAddAppender() {
     Logger logger = Logger.getLogger("MyLogger");
     logger.removeAllAppenders();
     logger.addAppender(new ConsoleAppender(
         new PatternLayout("%p %t %m%n"),
         ConsoleAppender.SYSTEM_OUT));
     logger.info("hello");
 }
```
5. 테스트 케이스 정상 작동
6. 위의 사이클을 겪으면서 log4j에 대한 이해도가 높아졌을 것이다. 그 지식으로 몇개의 테스트 케이스를 작성
```java
public class LogTest {
     private Logger logger;

     @Before
     public void initialize() {
         logger = Logger.getLogger("logger");
         logger.removeAllAppenders();
         Logger.getRootLogger().removeAllAppenders();
     }

     @Test
     public void basicLogger() {
         BasicConfigurator.configure();
         logger.info("basicLogger");
     }

     @Test
     public void addAppenderWithStream() {
         logger.addAppender(new ConsoleAppender(
             new PatternLayout("%p %t %m%n"),
             ConsoleAppender.SYSTEM_OUT));
         logger.info("addAppenderWithStream");
     }

     @Test
     public void addAppenderWithoutStream() {
         logger.addAppender(new ConsoleAppender(
             new PatternLayout("%p %t %m%n")));
         logger.info("addAppenderWithoutStream");
     }
 }
```
7. 위의 지식들을 활용해 log4j르 접점이 되는 독자적인 클래스로 캡슐화 한다.

### 학습 테스트는 공짜 이상이다. 
* 필요한 지식만 확보하는 손쉬운 방법이다.
* 학습테스트는 이해도를 높여주는 정확한 실험이다.
* 패키지 새버전이 나온다면 학습 테스트를 돌려 차이가 있는지도 확인이 가능하다.


### 아직 존재하지 않은 코드를 사용하기
* 아직 존재하지 않은 코드와 개잘자가 구현하는 코드의 연결고리로 인터페이스를 사용한다.
* 개념 및 기능을 정의하고 구현체를 먼저 개발하면 병렬로 개발이 가능하며, 인터페이스로서 통제를 할수 있다. 
* 테스트 구현체를 따로 개발해서 테스트에도 유용하다.

### 깨끗한 경계 
* 위의 개념과 비슷하게 경계가 깨끗하면 향후 외부 패키지가 변경이 일어날 경우 수정이 용이하다.
* 그러기 위해서는 통제가 불가능한 외부 패키지를 통제가 가능한 우리 코드로 가공해서 쓰면 우리 코드에만 의존하게 되므로 이러한 Adapter같은 패턴을 권장한다.

