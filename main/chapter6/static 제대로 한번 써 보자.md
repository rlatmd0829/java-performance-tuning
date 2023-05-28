## static

static이라는 단어는 '정적인. 움직이지 않는'이라는 의미이다. 자바에서 static으로 지정했다면, 해당 메서드나 변수는 정적이다.

```java
public class VariableTypes {
  int instance Variable;
  static int classVariable;
  public void method(int parameter) {
    int localVariable;
  }
}
```
여기서 static으로 선언한 classVariable은 클래스 변수라고 한다.

100개의 VariableTypes 클래스의 인스턴스를 생성하더라도, 모든 객체가 classVariable에 대해서는 동일한 주소의 값을 참조한다.

static 변수는 객체를 생성해서 참조할 필요없이 직접 클래스를 참조하면 된다.

<br>

## static 초기화 블록

```java
public class StaticBasicSample2 {
  static String staticVal;
  static {
    staticVal = "Static Value";
    staticVal = StaticBasicSample.staticInt + "";
  }
  
  public static void main(String[] args) {
        System.out.println(StaticBasicSample2.staticVal);
  }
  
  static {
    staticVal = "Performance is important !!!";
  }
}
```

static 초기화 블록은 위와같이 클래스 어느곳에나 지정할 수 있다. 이 static 블록은 클래스가 최초 로딩될 때 수행되므로 생성자 실행과 상관없이 수행된다.

static 블록은 순차적으로 실행되므로 staticVal = Performance is important !!! 가 된다.


<br>

## static 특징


- static에 특징은 다른 jvm에서는 static이라고 선언해도 다른 주소나 다른 값을 참조하지만, 하나의 jvm이나 was인스턴스에서는 같은주소에 존재하는 값을 참조한다는 것이다.
- gc에 대상이 되지않음
- static을 잘못사용하면 여러쓰레드에서 하나의 변수에 접근할수있기 때문에 데이터가 꼬일 수 있다.
- 객체를 다시 생성한다고 해도 그값은 초기화 되지않고 해당 클래스를 사용하는 모든 객체에서 공유하게 된다.

<br>

## static 잘 활용하기

### 잘 사용하고 절대 변하지 않는 변수는 final static으로 선언하자

자주 사용되는 로그인 관련 쿼리들이나 간단한 목록 조회 쿼리를 final static으로 선언하면 적어도 1바이트 이상의 객체가 GC 대상에 포함되지 않는다.

### 코드성 데이터는 DB에서 한 번만 읽자

```java
public class CodeManager {
  private HashMap<String, String> codeMap;
  private static CodeDAO cDAO;
  private static CodeManager cm;
  
  static {
    cDAO = CodeDAO();
    cm = new CodeManger();
    if (!cm.getCodes()) {
      // 에러처리
    }
  }
  
  prviate CodeManager() {
  }
  
  public static CodeManger getInstance() {
    return cm;
  }
  
  private boolean getCodes() {
    try {
      codeMap = cDAO.getCodes();
      return true;
    } catch (Exception e) {
      return false;
    }
  }
  
  public boolean updateCodes() {
    return cm.getCodes();
  }
  
  public String getCodeValue(String code) {
    return codeMap.get(code);
  }
}
```

CodeManager 클래스는 코드 정보를 미리 담아놓는 클래스이다. 내부적으로 cm 객체를 static으로 선언하여, 생성자가 아닌 getInstance() 메서드를 통해서 CodeManager 클래스에 접근하도록 했다.

클래스 메모리가 로드되면 static 초기화 블록에서 cDAO 객체 및 cm 객체를 초기화하고, getCodes() 메서드를 호출한다.  코드 정보는 codeMap에 저장된다.

만약 코드가 수정되었을 때는 updateCodes() 메서드를 호출하여 코드 정보를 다시 읽어 오도록 해야한다. 그런데 이와 같은 클래스를 만들어서 코드를 가져올 때 서버 인스턴스가 하나만 있다면 걱정할게 없지만 
서로 다른 JVM에 올라가 있는 코드 정보는 수정된 코드와 상이하므로 그 부분에 대한 대책이 있어야한다.

<br>

## static 잘못 쓰면 이렇게 된다.

해당 예는 잘못된 쿼리 관리용 클래스이다.

```java
public class BadQueryManager {
  private static String queryURL = null;
  public BadQueryManager(String badUrl) {
    queryURL = badUrl;
  }
  public static String getSql(String idSql) {
    HashMap<String, String> document = reader.read(queryURL);
  }
}
```

여기서 getSql() 메서드와 queryURL을 static으로 선언한 것이 잘못된 부분이다.

웹환경이기 때문에 여러 화면에서 호출할 경우에 queryURL은 그때 그때 바뀌게 된다. static으로 선언되어있기 때문에 클래스 변수이므로 모든 스레드에서 동일한 주소를 가리키게 되어 문제가 발생한다.

<br>

## static과 메모리 릭

static으로 선언한 부분은 GC가 되지 않는다. Vactor나 ArrayList에 담을 때 해당 Collection 객체를 static으로 선언하면 어떻게 될까?

만약 지속적으로 해당 객체에 데이터가 쌓인다면, 더 이상 GC가 되지 않으면서 시스템은 OutOfMemoryError를 발생시킨다.

더 이상 사용가능한 메모리가 없어지는 현상을 메모리 릭(Memory Leak)이라고 한다.

메모리 릭이 발생했을 때 가장 크게 나타나는 현상은 사용가능한 메모리가 적어지는 것이다. 계속 반복되다 보면 WAS를 재기동해야하는 문제이다.

<br>

## 정리

static은 원리를 잘 알고 사용하면 시스템의 성능을 향상시킬 수 있는 마법의 예약어이다. 하지만 잘못사용한다면 시스템이 다운되거나 예기치 못한 결과가 나올 수도 있다.

static은 반드시 메모리에 올라가며 GC에 대상이 되지 않는다. 객체를 다시 생성한다고 해도 그 값은 초기화되지 않고 해당 클래스를 사용하는 모든 객체에서 공유하게 된다.
