
#### 출처

- 토비의 봄 TV 5회 스프링 리액티브 프로그래밍(1) - Reactive Streams
  (URL: https://www.youtube.com/watch?v=8fenTR3KOJo&list=PLv-xDnFD-nnmof-yoZQN8Fs2kVljIuFyC&index=10&ab_channel=TobyLee)

- 참고: [Java] Reactive Stream 이란? 
  (URL: https://sabarada.tistory.com/98)
   
- 
   
   
#### Reactive stream
   
- non-blocking backPressure(배압) 을 이용하여 비동기 서비스를 할 때 기본이 되는 스펙
- java의 RxJava, Spring의 WebFlux의 Core에 있는 Project Reactor 프로젝트 모두 해당 스펙을 따르고 있다.
- java 1.9 버전에 추가된 Flow 역시 reactive stream 스펙을 채택하여 사용하고 있다.
   
- blocking과 non-blocking
  - blocking: 자신의 수행 결과가 끝날 때 까지 제어권을 갖고 있는 것을 의미
  - non-blocking: 자신이 호출되었을 때 제어권을 바로 자신을 호출한 쪽으로 넘기며, 자신을 호출한 쪽에서 다른 일을 할 수 있도록 하는 것을 의미
   
- BackPressure(배압)
  - 한 컴포넌트가 부하를 이겨내기 힘들 때, 시스템 전체가 합리적인 방법으로 대응해야 한다. 
  - 과부하 상태의 컴포넌트에서 치명적인 장애가 발생하거나 제어 없이 메시지를 유실해서는 안 된다. 
  - 컴포넌트가 대처할 수 없고 장애가 발생해선 안 되기 때문에 컴포넌트는 상류 컴포넌트들에 자신이 과부하 상태라는 것을 알려 부하를 줄이도록 해야 한다. 
  - 이러한 배압은 시스템이 부하로 인해 무너지지 않고 정상적으로 응답할 수 있게 하는 중요한 피드백 방법이다.
  - 배압은 사용자에게까지 전달되어 응답성이 떨어질 수 있지만, 이 메커니즘은 부하에 대한 시스템의 복원력을 보장하고 시스템 자체가 부하를 분산할 다른 자원을 제공할 수 있는지 정보를 제공할 것이다.
   
