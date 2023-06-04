## IO에서 발생하는 병목현상

자바에서 입력과 출력은 스트림(stream)을 통해서 이루어진다. 일반적으로 IO라고 하면 파일 IO만을 생각할 수 있는데, 어떤 디바이스를 통해 이뤄지는 작업을 모두 IO라고 한다.

네트워크를 통해서 다른 서버로 데이터를 전송하거나, 전송 받는 것도 IO에 포함된다.

콘솔에 출력하는 것도 스트림을 통해서 출력하는 것이다.

```java
System.out.println("hello");
```

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

바이트 단위로 읽거나, 문자열 단위로 읽을때 중요한 것은 한번 연(open한) 스트림은 반드시 닫아 주어야 한다는 것이다. Stream을 닫지 않으면 리소스가 계속 점유되어 다른 작업에 사용할 수 없게 되고, 시스템의 성능에도 영향을 줄 수 있습니다. 

<br>

## IO에서 병목이 발생한 사례

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

이 소스는 어떤 경로를 확인하는 시스템의 일부이다. getRoute로 경로를 가져오기 위해 매번 configUrl을 DaoUtility에 넘겨준다.
DaoUtility에서는 요청이 올 때마다 config.xml 파일을 읽고 파싱하여 관련 DB 쿼리 데이터를 읽는다.

이 애플리케이션이 실제 운영된다면, 모든 요청이 올 때마다 파일에 있는 DB쿼리를 읽어서 서버에는 엄청난 IO가 발생할 것이며, 응답 시간이 좋지 않으리라는 점도 쉽게 예상할 수 있다.

<br>

## NIO란?

JDK 1.4부터 새롭게 추가된 NIO가 어떤 것인지 알려면 IO가 어떻게 작동하는지 알아야한다.

1. 파일 읽는 메서드를 자바에 전달
2. 파일명을 받은 메서드가 운영체제의 커널에게 파일을 읽어달라고 요청
3. 커널이 하드 디스크로부터 파일을 읽어 자신의 커널에 있는 버퍼에 복사하는 작업을 수행하는데 DMA에서 수행

> DMA(Direct Memory Access)는 컴퓨터 시스템에서 주변장치와 메인 메모리 간에 데이터 전송을 수행하는 기술입니다.

4. 자바는 커널의 버퍼를 사용하지 못하므로 JVM으로 해당 데이터를 전달
5. JVM에서 메서드에 있는 스트림 관리 클래스를 사용하여 데이터를 처리

DMA는 CPU의 개입 없이 데이터를 전송하므로 CPU는 다른 작업을 수행할 수 있지만, 여전히 하드 디스크의 속도에 따라 복사 작업에 시간이 소요되어 자바에서는 3번 복사 작업을 할 때에나 4번 전달 작업을 수행할 때 대기하는 시간이 발생할 수 밖에 없다. 이러한 단점을 보완하기 위해서 NIO가 탄생했다.

NIO를 사용한다고 IO에서 발생하는 모든 병목 현상이 해결되는 것은 아니지만, IO를 위한 여러 가지 새로운 개념이 도입되었다.

- 버퍼의 도입

- 채널 도입

- 문자열의 인코더와 디코더 제공

- Perl 스타일의 정규표현식에 기초한 패턴 매칭 방법 제공

- 파일을 잠그거나 메모리 매핑이 가능한 파일 인터페이스 제공

- 서버를 위해 복합적인 Non-blocking IO 제공

<br>

## DirectByteBuffer를 잘못 사용하여 문제가 발생한 

NIO를 사용할 때 ByteBuffer를 사용하는 경우가 있다. ByteBuffer는 네트워크나 파일에 있는 데이터를 읽어 들일때 사용한다. ByteBuffer 객체를 생성하는 함수 중에서 allocateDirect 함수는 데이터를 JVM에 올려서 사용하는 것이 아니라, OS 메모리에 할당된 메모리를 Native한 JNI로 처리하는 DirectByteBuffer 객체를 생성한다. 그런데 이 DirectByteBuffer 객체는 필요할 때 계속 생성해서는 안 된다.

> JNI : JVM Native Interface는 자바 애플리케이션과 네이티브 코드(C, C++, 어셈블리 등) 간의 상호 작용을 위한 인터페이스입니다.

``` java
psvm() {
  DirectByteBuffer check = new DirectByteBufferCheck();
  for (int loop = 1; loop < 1024000; loop++) {
    check.getDirectByteBuffer();
    if (loop % 100 == 0) {
      System.out.println(loop);
    }
  }
}

public ByteBuffer getDirectByteBuffer() {
  ByteBuffer buffer = ByteBuffer.allocateDirect(65536);
  return buffer;
}
```

getDirectByteBuffer 함수를 지속적으로 호출하는 간단한 코드다. getDirectByteBuffer 함수에서는 ByteBuffer 클래스의 allocateDirect 함수를 호출함으로써 DirectByteBuffer 객체를 생성한 후 리턴해준다.

이 예제를 실행하고 나서 GC 상황을 모니터링 해보면 거의 5~10초에 한 번씩 Full GC가 발생하는 것을 볼 수 있다.

그 이유는 DirectByteBuffer의 생성자 때문이다. 이 생성자는 java.nio 에 reserveMemory() 함수를 호출한다. 이 reserveMemory 함수에서는 JVM에 할당되어 있는 메모리보다 더 많은 메모리를 요구할 경우 System.gc() 함수를 호출하도록 되어 있다.

JVM에 있는 코드에 System.gc() 함수가 있기 때문에 해당 생성자가 무차별적으로 생성될 경우 GC가 자주 발생하고 성능에 영향을 줄 수 밖에 없다. 따라서, 이 DirectByteBuffer 객체를 생성할 때는 매우 신중하게 접근해야만 하며, 가능하다면 singleton 패턴을 사용하여 해당 JVM에는 하나의 객체만 생성하도록 하는 것을 권장한다.

<br>

## lastModified() 메서드의 성능 저하

JDK6까지 자바에서 파일 변경을 확인하기 위해 File 클래스에 있는 lastModified()라는 메서드를 사용했다.

### lastModifed() 프로세스 

1. System.getSecurityManager()메서드를 호출하여 SecurityManager객체를 얻어옴

2. 만약 null이 아닐경우 SecurityManager의 checkRead()메서드 수행

3. File 클래스 내부에 있는 FileSystem이라는 클래스의 객체에서 getLastModifiedTime()메서드를 수행하여 결과 리턴

그냥 보기에는 3단계이지만, 각각의 호출되는 메서드에서 호출되는 메서드들이 매우 많으며 이는 OS마다 상이하다

그러나 JDK7 부터는 watch관련 클래스로 인해 편하게 모니터링이 가능해져 해당 파일이 변경되었는지 주기적으로 확인할 필요가 없어졌다.


