# String vs StringBuffer vs StringBuilder


### String 
String은 불변하다는 특징을 가지고 있어서 수정을 하지못하고 새로운 String 인스턴스가 생성되고 전에 있던 String은 GC에 의해 사라지게 된다.
그래서 좋은 성능을 기대하기는 힘들다. (String 불변, StringBuffer, StringBuilder 가변)

### StringBuffer
StringBuffer는 동기화 키워드를 지원하여 멀티쓰레드 환경에서 안전하다(thread-safe) 참고로 String도 불변성을 가지기때문에 마찬가지로 멀티쓰레드 환경에서의 안정성(thread-safe)을 가지고있다.

### StringBuilder
StringBuilder는 동기화를 지원하지 않기때문에 멀티쓰레드 환경에서 사용하는 것은 적합하지 않지만 동기화를 고려하지 않는 만큼 단일쓰레드에서의 성능은 StringBuffer 보다 뛰어나다.


### 정리

JDK 5.0 이상을 사용한다면, 컴파일러에서 자동으로 StringBuilder로 변환하여 준다. 하지만 반복 루프를 사용해서 문자열을 더할때는 객체를 계속 추가한다는 사실에는 변함이없다. 그러므로 String 클래스를 쓰는 대신,
스레드와 관련이 있으면 StringBuffer를, 스레드 안전 여부와 상관이 없으면 StringBuilder를 사용하는것을 권장한다.
