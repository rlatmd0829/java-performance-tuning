### static

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
여기서 static으로 선얺나 classVariable은 클래스 변수라고 한다.

100개의 VariableTypes 클래스의 인스턴스를 생성하더라도, 모든 객체가 classVariable에 대해서는 동일한 주소의 값을 참조한다.

static 변수는 객체를 생성해서 참조할 필요없이 직접 클래스를 참조하면 된다.

### static 초기화 블록

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



- static에 특징은 다른 jvm에서는 static이라고 선언해도 다른 주소나 다른 값을 참조하지만, 하나의 jvm이나 was인스턴스에서는 같은주소에 존재하는 값을 참조한다는 것이다.
- gc에 대상이 되지않음
- static을 잘못사용하면 여러쓰레드에서 하나의 변수에 접근할수있기 때문에 데이터가 꼬일 수 있다.
- 객체를 다시 생성한다고 해도 그값은 초기화 되지않고 해당 클래스를 사용하는 모든 객체에서 공유하게 된다.

### static 잘 활용하기

잘 사용하고 절대 변하지 않는 변수는 final static으로 선언하자
