# [4th] Oracle Developer Meetup | 미래의 Java와 손에 잡히는 Microservice!
## Java.next()
- 앞으로 6개월마다 릴리즈 될 자바의 가까운 미래에 대해 살펴보는 세션

## Java
- Java = 언어 + VM + 라이브러리 세가지가 강결합이 되어 있음
- 자바백서 : The Java Language Environment
- Open Innovation = 오픈소스 + 업계 + 커뮤니티. (업계) 여러 기업이 모여서 기능을 개발함
- 최근에 자바가 구식언어라는 얘기가 있지만 애초에 자바는 작은 구식언어로 설계됨 - 블루칼라(노동자)의 언어
- James Gosling : we preferred tried-and-tested things

## OpenJDK
- OpenJDK를 기준으로 Old Java와 Modern Java로 나뉜다고 생각하심
- 실제 구현 및 테스트를 통해 검증을 받으면 도입됨
- JCP : 승인기구, OpenJDK : 구현. 대부분의 오라클 개발자가 포함
- Java7부터 시작
### 개선흐름
- 언어의 현대화 : 현재는 가독성이 주요함. 멀티코어 지원(CDP 대응). 시스템 내부를 컨트롤하는 기능을 제공(기존의 JNI 대체)
- (MSA의 정수) Micro Services - Java the Unix way. 강추하심. OpenJDK가 이 발표 이후에 직간접적으로 많이 바뀌었다고

## Amber
- 자바의 언어를 개선하는 프로젝트
- Generics, Diamond type, switch-string, Lambda을 제공했음
### 지역변수 타입 추론
- var users = xxx.getVisitors();
- 동적 타입 언어와 다름. 컴파일 타임에 타입매칭을 체크하지 않는게 아니라 체크는 그대로 하면서 타입을 유지.
- lambda에도 타입추론 가능. 기존에도 타입추론으로 타입 생략이 가능하지만, 이후에는 어노테이션을 추가 할 수 있다.
- 멤버변수 타입을 추론할 경우 scope이 넓어져서 지역변수만 제공
- val대신 final var
### Raw string
- Backtick으로 처리
- 런타임에 후처리하는 특수문자들(Escape), 여러 줄의 문자
### Switch식
- Switch 문이 아닌 Switch 이후에 특정한 값이 반환되는 식
- (문)if statement vs (식)3항연산자
### 패턴매칭
- switch 식+타입비교+객체분해+값 비교
- 패턴매칭의 개념이 너무 커서 작은부분을 도입한것이 switch 식
- 자바에서의 객체 사용 용도 - 구조체(c), 값(primitve), DTO, 객체(행위-OOP에서 중요한것이 행위), 컴포넌트
- record keyword 한줄로 getter, setter 등등을 가지고 있는 객체
- switch에서 자식 타입을 비교해서 캐스팅까지 해줌
- 객체분해?? case에서 값을 가져옴

## Vahalla
- 기존자바타입에서는 reference type으로 객체 생성, 객체 참조하는 것이 비효율적임. GC
- 유리수객체를 저장하는 배열을 가진다고 생각하면 매우 비효율적임. 참조하지않는 배열을 가지는 것이 값타입
- 코드는 클래스 같지만 동작은 값처럼' : primitive / reference type간의 간극을 가깝게
- JVM을 많이 수정해야됨. 기존의 Enum의 경우에는 클래스에 몇가지 기능을 추가하는 방식으로 JVM을 수정하지 않으려고 했었음
- Immutable한 객체. concurrent 환경에 유리하고 GC가 효율적
- **제네릭에 primitive** 타입이 가능. 기존 제네릭은 컴파일러 기술이라 런타임에는 정보가 남아있지 않다.
- 런타임에 제니릭 특화된 클래스가 새로 생성됨. ArrayList${T=int}.class
### Value Based Class
- Java8에 도입된 것. JavaBean처럼 나름의 클래스의 규격을 지켜서 생성
- Optional, LocalDate 등등...
- VM에서는 먼저 Release하고, 언어는 추후 Release하지만 @jdk.incubator.mvt.ValueCapableClass 타입으로 값 가능 클래스 사용 가능
- incubator 어노테이션 사용 시 jvm 옵션이 필요.
- 컴파일시에는 동일하지만 클래스로딩시 DVC 타입으로 메모리에 탑재. (기존은 VCC)
- 참조타입으로 인해 생기는 성능저하를 해결

## Loom
### Fiber
- 경량 쓰레드. 컨티뉴에이션(Runnable 같은)을 관리
- 쓰레드보다 더 가벼우므로 동기식으로도 많은 태스크를 처리할 수 있다.
