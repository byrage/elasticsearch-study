http://d2.naver.com/helloworld/1230
# JVM
- 자바 바이트코드를 실행하는 주체
- 플랫폼에 독립적이며 모든 JVM은 JVM 규격에 정의된대로 바이트코드를 실행한다.
- CPU나 운영체제의 종류와 무관하게 동일하게 동작할 것을 보장함

- 특성
 1. 스택 기반의 가상 머신
 - 단일 상속형태의 객체 지향 프로그래밍을 가상머신 수준에서 구현
 - 포인터를 지원하지만 C와 같이 주소값을 임의로 조작이 가능한 포인터 연산은 불가능
 - 가비지 컬렉션 사용
 - 자바 바이트코드 검증기(verifier)를 통해 스택 넘침, 명령어 피연산자의 타입 규칙 위반, 필드 접근 규칙 위반, 지역 변수의 초기화 전 사용 등 많은 문제를 실행 전에 검증하여 실행 시 안전을 보장하고 별도의 부담을 줄여줌

# GC

## stop-the-world
- GC를 실행하기 위해 JVM이 어플리케이션 실행을 멈추는 것
- stop-the-world 발생 시 GC를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춘다. GC 완료 이후에 중단했던 작업을 다시 시작함
- GC 튜닝이란 stop-the-world 시간을 줄이는 것
- System.gc() 메소드를 직접 호출하는 것은 시스템 성능에 매우 큰 영향을 끼치는것이므로 절대 사용하면 안된다.

## Overview
- 자바에서는 개발자가 메모리를 명시적으로 해제하기 않기 때문에 GC가 사용하지 않는 객체를 찾아 지우는 작업을 함.
- GC는 'weak generational hypothesis'라고 하는 두가지 가정 하에 만들어졌다.
 - 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
 - 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.
- Hotspot VM에서는 이 가정 하에 Young Generation / Old Generation 영역으로 구분한다.
 - Young Generation : 새롭게 생성한 객체의 대부분이 위치하는 공간. 대부분의 객체가 금방 접근 불가능 상태가 되기 때문에 Young 영역에 생성되었다가 사라진다. 이 영역에서 객체가 사라질 때 Minor GC가 발생한다고 한다.
 - Old Generation : Young 영역에서 살아남는 객체가 복사 되는 영역. 대부분 Young 영역보다 큰 크기로 할당되며, 크기가 크기 때문에 GC가 적게 발생한다. 이 영역에서 객체가 사라질 때 Major GC(Full GC)가 발생한다고 한다.
 - Permanent Generation (Method Area) : 객체나 억류된 문자열 정보를 저장하는 곳. 여기서 GC가 발생해도 Major GC의 횟수에 포함된다.
- Old 영역에 있는 객체가 Young 영역에 있는 객체를 참조하는 경우?
- Old 영역에는 512바이트 chunk로 되어있는 카드 테이블이 존재한다. 카드 테이블에는 Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 정보가 표시된다. Young 영역의 GC를 실행 할 때 Old 영역에 있는 모든 객체의 참조를 확인하는 것이 아니라, 카드 테이블만 뒤져서 GC 대상인지 식별한다.
- 카드 테이블은 write barrier를 사용하여 관리한다. write barrier는 Minor GC를 빠르게 할 수 있도록 하는 장치이다. write barrirer때문에 약간의 오버헤드는 발생하지만 전반적인 GC 시간은 줄어들게 된다.

## Young Generation
- 1개의 Eden 영역, 2개의 Survivor(to, from) 영역으로 구성된다.
### 처리 절차
- 새로 생성한 대부분의 객체는 Eden 영역에 위치한다.
- Eden 영역에서 GC가 한 번 발생한 후 살아남은 객체는 Survivor 영역 중 하나로 이동된다.
- Eden 영역에서 GC가 발생하면 이미 살아남은 객체가 존재하는 Survivor 영역으로 객체가 계속 쌓인다.
- 하나의 Survivor 영역이 가득 차게 되면 그 중에서 살아남은 객체를 다른 Survivor 영역으로 이동한다. 그리고 가득 찬 Survivor 영역은 아무 데이터도 없는 상태로 된다.
- 이 과정을 반복하다가 계속해서 살아남아 있는 객체는 Old 영역으로 이동하게 된다.

- Survivor 영역 중 하나는 반드시 비어 있어야한다. 두 Survivor 영역에 모두 데이터가 존재하거나 모두 사용량이 0이라면 비정상적인 상황

## Old 영역 GC 방식
- Serial GC
- Parallel GC
- Parallel Old GC(Parallel Compacting GC)
- Concurrent Mark & Sweep GC
- G1(Garbage First) GC

### Serial GC (-XX:+UseSerialGC)
- PC가 싱글코어 일때만 사용하기 위해 만든 방식. 운영 서버에서 절대 사용하면 안된다.
- Minor GC : 앞서 설명한 방식을 사용한다.
- Major GC : mark-**sweep**-compact라는 알고리즘 사용.
 - Mark : Old 영역에 살아 있는 객체 식별
 - Sweep : Mark한 정보를 바탕으로 힙의 앞부분부터 확인하여 Mark되지 않은 객체 제거
 - Compactation : 각 객체들이 연속되게 쌓이도록 힙의 가장 앞부분 부터 채워서 객체가 존재하는 부분과 객체가 없는 부분으로 나눈다.

### Parallel GC (-XX:+UseParallelGC)
- Serial GC와 기본적인 알고리즘은 동일하다.
- GC를 처리하는 스레드가 여러개 일때 사용한다.
- Throughput GC라고도 부른다.

### Parallel Old GC(-XX:+UseParallelOldGC)
- JDK 5 update 6 부터 제공한 GC 방식이다.
- 앞서 설명한 Parallel GC와 비교해서 Old 영역의 GC 알고리즘만 mark-**summary**-compaction이라는것이 다르다.
 - Summary : 앞서 GC를 수행한 영역에 대해서 별도로 살아 있는 객체를 식별함.

### CMS GC (-XX:+UseConcMarkSweepGC)
- stop-the-world 시간이 매우 짧다. 모든 애플리케이션의 응답 속도가 매우 중요할때 CMS GC를 사용하며, Low Latency GC라고도 한다.
#### 작업 단계
- Initial Mark : 클래스 로더중에 가장 가까운 객체 중 살아 있는 객체만 찾는다.
- Concurrent Mark : 방금 살아있다고 확인한 객체에서 참조하는 객체들을 tracking 하면서 확인한다. 다른 스레드가 실행중인 상태에서 동시에 진행된다는 것이 특징이다.
- Remark : Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다.
- Concurrent Sweep : 쓰레기를 정리하는 작업. 이 작업 또한 다른 스레드가 실행중인 상태에서 동시 진행함
#### 단점
- 다른 GC 방식보다 메모리와 CPU를 더 많이 사용한다.
- Compaction 단계가 기본적으로 제공되지 않는다.
- Compaction을 실행하면 stop-the-world 시간이 길어지기 때문에 Compaction 작업이 얼마나 자주, 오랫동안 수행되는지 확인 필요

### G1 GC (-XX:+UseG1GC)
- Java6에서 early access, Java7에서 정식 제공
- 다른 GC와 달리 Young / Old 영역이 없다. Young -> Old 이동 단계가 사라진 방식
#### 작업 단계
- 바둑판의 각 영역에 객체를 할당하고 GC를 실행한다.
- 해당 영역이 꽉 차면 다른 영역에서 객체를 할당하고 GC를 실행한다.

https://plumbr.io/blog/garbage-collection/minor-gc-vs-major-gc-vs-full-gc
# Minor GC
1. JVM이 새로운 객체를 할당할 공간이 없으면 Minor GC가 항상 발생한다. ex) Eden 영역이 가득차고 있으면, 할당률이 늘어나고 Minor GC가 더욱 자주 실행된다.
