## HotSpot VM은 어떻게 구성되어 있을까?

자바를 만든 Sun에서는 자바의 성능을 개선하기 위해서 Just In Time(JIT) 컴파일러를 만들었고, 이름을 HotSpot으로 지었다.

여기서 JIT 컴파일러는 프로그램의 성능에 영향을 주는 지점에 대해서 지속적으로 분석한다. 분석된 지점은 부하를 최소화하고, 높은 성능을 내기 위한 최적화의 대상이 된다. 이 HotSpot은 자바 1.3 버전부터 기본 VM으로 사용되어 왔기 때문에, 지금 운영되고 있는 대부분의 시스템들은 모두 HotSpot 기반의 VM이라고 생각하면 된다. HotSpot VM은 세 가지 주요 컴포넌트로 되어 있다.

- VM(Virtual Machine) 런타임
- JIT(Just In Time) 컴파일러
- 메모리 관리자

## JIT Optimizer
