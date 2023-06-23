---
layout: post
title: Java GC. G1GC와 ZGC
summary: Java Version
author: devhtak
date: '2023-06-23 21:41:00 +0900'
category: Java Study
---
#### Heap 구조
##### 1.7 이전에 구조
- 1.7 버전 이전 Heap 메모리 구조는 Eden, Survivor 0/1, Old Memory, Permanent Generation 으로 구성되어 있었다.
  - Eden: 새로 생성한 대부분의 객체 위치
  - S0/1: Eden 영역에서 GC 발생한 후 살아남은 객체들이 존재하는 곳, GC가 발생하는 경우 영역을 옮기게 된다.
  - Eden, S0/1 부분을 Young Generation이라 한다
  - Old Memory: Young Generation에 대한 GC가 반복되는 과정 속에 살아남은 객체가 특정 회수 이상(Age)이 되는 경우 이동
  - PermGen: Class/Method의 Meta 정보, static 변수/상수 저장
  - Eden + S0/1 + Old Memory의 크기는 JVM Heap(-Xms -Xmx) 설정을 줄 수 있다.
  - Eden + S0/1의 크기는 Young Gen(-Xmn) 설정을 줄 수 있다.
- Young Generation(Eden + S0/1)에서 GC가 일어나는 경우 Minor GC라 하고 Old Memory에서 GC가 발생하는 경우 Major GC라 한다.
  - Major GC가 발생하면 STW(Stop the world)라 하여 성능에 영향을 미치게 된다.
 
##### 1.8 에서 변경된 부분
- PermGen -> Metaspace 로 변경
  - Java의 클래스 로더가 로드한 클래스들의 Metadata가 저장되는 공간
  - PermGen과 Metaspace의 가장 큰 차이는 Heap 영역이 아닌 Native Memory 영역에 위치한다는 것
  - 차이점 1. Heap영역과 Native Memory
    - Heap 영역은 JVM 관리 영역이고, Native Memory는 OS 관리 영역으로 Native Memory는 OS가 자동으로 크기를 조절하기 때문에 개발자 영역 확보를 의식할 필요가 없게 되었다.(MAX 값을 걸 수 있다)
  - 차이점 2. static 변수/상수가 PermGen에 저장되는 것이 아닌 Heap에 할당되어 GC 대상이 될 수 있도록 함
 
#### GC 종류
- GC 튜닝의 목적은 STW 시간을 줄이는 것이다.
  - Old generation 영역은 Young Generation 영역보다 크게 할당되고, GC는 적게 일어나지만 시간이 길 수 밖에 없고 이는 성능에 영향을 미친다.
- Serial GC
  - Heap의 앞부분부터 확인하여 살아이는 것만 남기고(Sweep) 객체들이 연속되도록 압축(Compaction)
  - Mark-Sweep-Compaction 알고리즘
  - 이와 비슷하게 Parallel GC(Multi-Thread), Parallel Old GC, Concurrent Mark&Sweep GC 등으로 발전했다.
- G1GC
  - 기존에 메커니즘(YoungGen, OldGen)과는 많이 다르다
  - Eden, Survivor, Old 영역이 존재하지만 해당 영역을 고정된 크기가 아닌 전체 Heap 메모리 영역을 Region이라는 특정한 크기로 나누고 Region의 역할(Eden, Survivor, Old)로 동적으로 변동한다.
    - Region은 Heap Memory/2048 을 default로 지정되어 있다
  - 추가된 Region 역할
    - Humonogous: Region크기 50% 초가하는 큰 객체 저장하기 위한 공간
    - Avaliable/Unused: 아직 사용하지 않은 Region
  - Minor GC
    - 살아남은 객체를 Eden에서 Survivor Region으로 옮기고, Eden에 대한 영역을 Available Region으로 돌리는 형태
  - Concurrent Cycle(Full GC와 유사)
    - Initial Mark : Old Region에 존재하는 객체들이 참조하는 Survivor Region을 찾는다(STW)
    - Root Region Scan : 위에서 찾은 Survivor 객체들에 대한 스캔 작업을 실시한다
    - Concurrent Mark : 전체 Heap의 scan 작업을 실시하고, GC 대상 객체가 발견되지 않은 Region은 이후 단계를 제외
    - Remark : 애플리케이션을 멈추고(STW) 최종적으로 GC 대상에서 제외할 객체를 식별
    - Cleanup : 애플리케이션을 멈추고(STW) 살아있는 객체가 가장 적은 Region에 대한 미사용 객체를 제거
    - Copy : GC 대상의 Region이었지만, Cleanup 과정에서 완전히 비워지지 않은 Region의 살아남은 객체들을 새로운 Region(Available/Unused) Region에복사하여 Compaction을 수행
- ZGC
  - New Feature에서 보면 ZGC는 적은 메모리나 큰 메모리에서 STW 시간을 최대한 적게(10ms 이하)로 가겨가기 위해 제작되었다고 되어 있다.
  - Region을 단순화 한다
    - small(2mb), medium(32 mb), large(N*2mb)
  - Colored pointers, Load barriers 알고리즘 사용
    - Colored pointers: 객체를 가리키는 변수 포인터 64비트를 활용하여 Marking
      - Finalizable(finalizer를 통해 참조되는 Object의 Garbage), Remapped(재배치 여부 판단 Mark), Marked 0, Marked 1(Live Object)
    - Load Barriers
      - RemapMark와 RellocationSet을 확인하면서 참조와 Mark 업데이트(해당 bit를 바탕으로 메모리 재배치하는 과정에 STW 없이 재배치가 가능)
      - Flow
        - Mark Start STW : ZGC의 Root에서 가리키는 객체 Mark 표시
        - Concurrent Mark/Remap: 객체의 참조를 탐색하면서 모든 객체에 Mark 표시
        - Mark End STW : 새롭게 들어온 객체들에 대해 Mark 표시
        - Concurrent Pereare for Relocate: 재배치하려는 영역을 찾아 Relocation Set에 배치
        - Relocate Start STW : 모든 Root 참조의 재배치를 진행하고 업데이트
        - Concurrent Relocate: 이후 Load Barriers 를 사용하여 모든 객체를 재배치 및 참조 수정
    - 결론적으로 포인터를 활용하여 객체를 Marking하고 관리
