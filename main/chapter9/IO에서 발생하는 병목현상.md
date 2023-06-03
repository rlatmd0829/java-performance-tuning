## IO란

자바에서 입력과 출력은 스트림(stream)을 통해서 이루어진다. 일반적으로 IO라고 하면 파일 IO만을 생각할 수 있는데, 어떤 디바이스를 통해 이뤄지는 작업을 모두 IO라고 한다.

네트워크를 통해서 다른 서버로 데이터를 전송하거나, 다른 서버로부터 데이터를 전송 받는 것도 IO에 포함된다.

### 주요 클래스

바이트 기반의 스트림을 처리하기 위한 클래스는 java.io.InputStream 하위 클래스들이다. (쓰는데 관련된 클래스들은 Input을 Output으로 바꾸어 사용하면 된다.)
- ByteArrayInputStream
- FileInputStream
- FilterInputStream
- ObjectInputStream
- PipedInputStream
- SequenceInputStream

문자열 기반 스트림을 읽기위한 클래스는 java.io.Reader 클래스의 하위 클래스들이다.

- BufferedReader
- CharArrayReader
- FilterReader
- FileReader
- InputStreamReader
- PipedReader
= StringReader

바이트 단위로 읽거나, 문자열 단위로 읽을때 중요한 것은 한번 연(open한) 스트림은 반드시 닫아 주어야 한다는 것이다. 스트림을 닫지 않으면 나중에 리소스가 부족해 질 수 있다.

### IO에서 병목이 발생한 사례

``` java
String configUrl;
public Vector getRoute(String type) {
  if(configUrl == null) {
    config = this.getClass.getResource("/xxx/config.xml");
  }
  obj = new DaoUtility(configUrl, "1");
  ...
}
```

이 소스는 어떤 경로를 확인하는 시스템의 일부이다. 경로 하나를 가져오기 위해서 매번 configUrl을 DaoUtility에 넘겨준다.
DaoUtility에서는 요청이 올 때마다 config.xml 파일을 읽고 파싱하여 관련 DB 쿼리 데이터를 읽는다.

이 애플리케이션이 실제 운영된다면, 모든 요청이 올 때마다 파일에 있는 DB쿼리를 읽어서 서버에는 엄청난 IO가 발생할 것이며, 응답 시간이 좋지 않으리라는 점도 쉽게 예상할 수 있다.

### NIO란?

JDK 1.4부터 새롭게 추가된 NIO가 어떤 것인지 알려면 IO가 어떻게 작동하는지 알아야한다.

1. 파일 읽는 메서드를 자바에 전달
2. 파일명을 받은 메서드가 운영체제의 커널에게 파일을 읽어달라고 요청
3. 커널이 하드 디스크로부터 파일을 읽어 자신의 커널에 있는 버퍼에 복사하는 작업을 수행하는데 DMA에서 수행
4. 자바는 커널의 버퍼를 사용하지 못하므로 JVM으로 해당 데이터를 전달
5. JVM에서 메서드에 있는 스트림 관리 클래스를 사용하여 데이터를 처리

자바에서는 3번 복사 작업을 할 때에나 4번 전달 작업을 수행할 때 대기하는 시간이 발생할 수 밖에 없다. 이러한 단점을 보완하기 위해서 NIO가 탄생했다.

NIO를 사용한다고 IO에서 발생하는 모든 병목 현상이 해결되는 것은 아니지만, IO를 위한 여러 가지 새로운 개념이 도입되었다.

- 버퍼의 도입

- 채널 도입

- 문자열의 인코더와 디코더 제공

- Perl 스타일의 정규표현식에 기초한 패턴 매칭 방법 제공

- 파일을 잠그거나 메모리 매핑이 가능한 파일 인터페이스 제공

- 서버를 위해 복합적인 Non-blocking IO 제공

### DirectByteBuffer를 잘못 사용하여 문제가 발생한 
