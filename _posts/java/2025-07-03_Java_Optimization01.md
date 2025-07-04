### 1장 최적화와 성능 정의
#### 자바 성능 개요
- 자바의 초기 성능 관점은 '환경이 충분히 빠르다면, 원시 성능을 어느 정도 희생하더라도 개발자의 생산성을 높이는 것이 더 중요하다'라는 것
  - 이러한 실용성을 바탕으로 가장 눈에 띄는 특징은 관리되는 서브 시스템
  - 예를 들면 가비지 컬렉션을 통해 자동으로 메모리 관리하고 프로그래머는 메모리를 수동적으로 관리할 필요가 없다.
- 자바 성능 데이터의 특성상 통계적으로 정교한 접근이 필요하며, 단순한 기법으로는 잘못된 결과를 산출할 수 있다.

#### 성능을 위한 분류 체계
- 성능을 관측할 수 있는 지표
  - 처리량, 효율성, 지연 시간, 확장성, 수용량, 성능 저하, 활용도
- 대 부분의 성능 프로젝트에서 모든 지표를 동시에 최적화하는 것은 불가능하며 한번의 성능 반복에서는 일부 지표만 개선될 수 있으며, 이는 한번에 달성할 수 있는 최적의 범위일 수 있다.
- 처리량
  - 시스템이나 서브시스템이 수행할 수 있는 작업의 속도를 나타내는 지표로 처리된 작업 단위를 말한다.
  - 처리량 값이 의미있으려면 플랫폼의 설명이 포함되어야 한다
    - 하드웨어 사양, 운영체제 또는 소프트웨어 스택이 처리량에 중요한 영향, 단일 서버인지 클러스터인지도 고려해야 한다.
- 지연 시간
  - 시작에서 끝까지 걸리는 시간으로 정의되며, 단일 거래를 처리하고 결과를 확인하는 데 걸리는 시간
  - 작업 부하에 따라 다를 수 있으며, 작업 부하가 증가함에 따라 지연 시간을 나타내는 그래프를 생성
- 수용량
  - 시스템이 보유한 병렬 처리 작업량을 의미, 시스템에서 동시에 진행할 수 있는 작업 단위의 수를 나타낸다.
  - 수용량과 처리량은 밀접한 관련이 있으며 부하가 증가함에 따라 처리량에 영향을 미칠 수 있다.
- 활용도
  - 시스템 리소스의 효율적인 사용을 달성하는 것
    - 이상적으로 CPU는 작업 단위를 처리하는 데 사용하며 유휴 상태로 남거나 운영 체제 작업을 처리하는 데 시간을 보내지 않아야 한다.
  - CPU, 네트워크, 메모리, I/O 서브시스템과 같은 다른 자원 유형도 클라우드 네이티브 애플리케이션에서 관리할 중요한 자원이 된다.
- 효율성
  - 처리량을 사용된 자원으로 나누면 시스템의 전체 효율성을 측정할 수 있다.
- 확장성
  - 자원을 추가할 때 처리량이 어떻게 변하는지를 기준으로 정의하는 것이 유용
  - 이상적인 것은 자원 증가에 따라 처리량이 정확히 비례하여 증가하는 것
  - 하지만 실제로는 다양한 부하 상황에서 달성하기 어렵다.
- 성능 저하
  - 부하가 증가할 때 관찰되는 지연 시간 또는 처리량에 변화가 나타난다.
  - 자원의 활용도에 따라 달라진다
     - 추가 부하로 인해 발생할 수 있는 성능 저하
- 관측 가능한 지표들 간의 상관관계
  - 여러 성능 지표의 동작은 일반적으로 어떤 방식으로든 연결되어 있다.
  - 예를들면 일반적으로 시스템의 부하가 증가하면 활용도가 변하게 된다.
    - 그러나 시스템이 낮은 활용상태라면 부하를 증가해도 활용도가 눈에 띄게 증가하지 않을 수 있다.
    - 반대로 시스템이 이미 과부하라면 부하 증가의 영향이 다른 관측 지표에 나타날 수 있다.

#### 클라우드 시스템의 성능
- 클라우드 시스템은 거의 항상 분산 시스템으로, 클러스터의 여러 노드가 공유 네트워크 자원을 통해 상호 작용한다.
- 단일 노드 시스템의 복잡성에 더해 또 다른 수준의 복잡성을 다워야 함을 의미
- 고려 사항
  - 클러스터 내에서 작업이 어떻게 분배되는가?
  - 소프트웨어의 새 버전(새 구성)을 클러스터에 어떻게 배포할 것인가?
  - 노드가 클러스터에서 이탈하면 어떻게 되는가?
  - 새 노드가 클러스터에 추가되면 어떻게 되는가?
  - 새 노드가 잘못 구성된 경우에 어떻게 되는가?
  - 새 노드가 클러스터의 나머지와 다르게 동작하면 어떻게 되는가?
  - 클러스터 자체를 제어하는 코드에 문제가 발생하면 어떻게 되는가?
  - 클러스터가 의존하는 인프라의 구성 요소가 제한된 자원일 때, 확장성에 병목이 생기면 어떻게 되는가?
- 클라우드 시스템에서는 단일 JVM 환경과 다른 두가지 중요한 측면이 존재한다.
  - 클라우드에서 배포 가능한 코드의 단위가 애플리케이션의 JVM이 아닌 컨테이너라는 점
  - 서비스에서 클라우드 제공업체를 사용하는 방식에 따라 효율성이나 활용도가 서비스 운영 비용에 직접적인 영향을 미친다.

### 2장 성능 테스트 방법론
#### 성능 테스트 종류
- 지연 테스트
  - 시작부터 끝까지의 트랜잭션 시간은 얼마나 걸리는지 확인
- 처리량 테스트
  - 현재 시스템 용량으로 몇개의 동시 거래를 처리할 수 있는 지 확인
  - 지연 테스트와 이중적인 관계가 있다.
    - 예를들면 지연테스트를 수행할 때는 지연 결과 분포를 생성하는 동안 동시 트랜잭션 수를 명확히 명시하고 제어하는 것이 중요
    - 마찬가지로 처리량 테스트를 수행할 때는 지연이 과도하게 증가하지 않도록 주의해야한다.
  - 최대 처리량을 지연 분포가 갑자기 변화하는 순간(시스템의 한계점)을 관찰해서 결정
- 스트레스 테스트
  - 시스템의 한계점은 무엇인지 확인
  - 일정한 상태의 트랜잭션 상태에 두고, 동시 트랜잭션 수를 천천히 증가시켜 시스템의 성능이 저하되기 시작하는 지점을 찾는 방식으로 진행
- 부하 테스트
  - 예상되는 비즈니스 이벤트 전에 수행되며 시스템이 특정 부하를 처리할 수 있는지 확인
- 내구성 테스트
  - 시스템을 장시간 실행했을 때 어떤 성능 이상이 발견될 수 있는 지 확인
  - 메모리 누수, 캐시 오염, 메모리 단편화 등이 포함되며 오랜 시간동안 진행된다.
- 용량 계획 테스트
  - 추가 리소스를 투입했을 때 시스템이 예상대로 확장되는 지 확인
  - 스트레스 테스트는 현재 시스템이 견딜 수 있는 한계를 찾는것이 목적이라면 용량 계획 테스트는 더 미래지향적으로 업그래이드 된 시스템이 어떤 부하를 처리할 수 있는지를 찾는 것을 목표로한다.
- 성능 저하 테스트
  - 시스템이 부분적으로 장애가 발생했을 때 어떻게 처리되는 지 확인
  - 관찰 항목으로 트랜잭션 지연 부포와 처리량이 포함

#### 모범 사례 개론
- 성능 조정 작업에서 집중할 지점을 결정할 때, 세가지 규칙을 고려하는 것이 좋다.
  - 중요한 요소가 무엇인지 파악하고, 이것을 측정하는 방법을 찾으세요.
  - 최적화하기 쉬운 것보다 중요한 것을 최적화하세요.
  - 가장 큰 영향을 미치는 요소부터 최적화를 시작하세요.
- 탐다운 성능
  - 애플리케이션 전체의 성능 동작을 분석하는 접근 방식을 일반적으로 탑다운 성능이라 한다.
  - 측정 또는 최적화할 항목을 명확히 이해하며, 이러한 성능 분석 작업이 전체 소프트웨어 개발 수명 주기에서 어떤 역할을 하는 지 파악해야 한다.
- 테스트 환경 성능
  - 가능한 프로덕션 환경과 모든 면에서 동일하게 구성되어야 한다.
  - 애플리케이션 서버 뿐만 아니라 웹 서버, 디비, MQ 등이 포함되며 부하를 처리할 수 없는 서드파티 네트워크 서비스 같은 요소는 현실적인 성능 테스트를 위해 목 서비스로 대체해야한다.
- 성능 요구사항 식별
  - 성능을 평가할 때 사용하는 지표를 특정 요소에만 한정해서는 안된다.
  - 비기능 요구사항, 최적화하려는 주요 대상이 된다.
- 소프트웨어 개발 수명 주기와 성능 테스트
  - 성능 테스트, 성능 저하 테스트를 SDLC의 중요한 부분으로 인식하고 지속적으로 수행
- 자바 특유의 문제들
  - JVM 특성상 성능 엔지니어가 추가적으로 고려해야할 몇가지 복잡한 요소가 있다.
  - JVM의 동적 자가 관리 기능에서 비롯되며 메모리 영역의 동적 튜닝과 JIT 컴파일이 대표적

#### 성능 안티 패턴
- 지루함
- 이력 부풀리기
- 사회적 압박
- 이해 부족
- 문제에 대한 오해 또는 문제 자체의 부재

#### JVM 성능을 위한

#### 통계해성

#### 인지적 편향과 성능 테스트
 
#### 출처
