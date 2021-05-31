---
layout: post
title: Kubernetes Logging and Monitoring
summary: Kubernetes
author: devhtak
date: '2021-05-31 21:41:00 +0900'
category: Container
---

#### 쿠버네티스 모니터링 시스템과 아키텍처

- 모니터링 서비스 플랫폼
  - 쿠버네티스를 지원하는 다양한 모니터링 플랫폼
    - Heapster(deprecated), Metrics Service
      - 쿠버네티스의 메트릭 수집 모니터링 아키텍처에서 코어메트릭 파이프라인 경량화
      - 힙스터를 deperecated하고 모니터링 표준으로 metics-server 도입
    - cAdvisor, 프로메테우스, EFK
  ```
  $ kubectl top node
  $ kubectl top pod
  ```
  - top으로 볼 수 있는 것은 한계가 있다. (history 저장, UI 등)

- 리소스 모니터링 도구
  - 쿠버네티스 클러스터 내의 애플리케이션 성능을 검사
  - 쿠버네티스는 각 레벨에서 애플리케이션의 리소스 사용량에 대한 상세 정보를 제공
  - 애플리케이션의 성능을 평가하고 병목 현상을 제거하여 전체 성능 향상을 도모
  - 리소스 메트릭 파이프라인
    - kubectl top 등의 유틸리티 관련된 메트릭들로 제한된 집합을 제공
    - 단기 메모리 저장소인 metrics-server에 의해 수집
    - metrics-server는 모든 노드를 발견하고 kubelet에 CPU와 Memory를 질의
    - kubelet은 kubelet에 통합된 cAdvisor를 통해 레거시 도커와 통합 후 metric-server 리소스 메트릭으로 노출
    - /metrics/resource/v1beta1 API를 사용
  - 완전한 메트릭 파이프라인
    - 보다 풍부한 메트릭에 접근
    - 클러스터의 현재 상태를 기반으로 자동으로 스케일링하거나 클러스터를 조정
    - 모니터링 파이프라인은 kubelet에서 메트릭을 가져옴
    - CNCF 프로젝트인 프로메테우스가 대표적
    - custom.metrics.k7s,io, external.metrics.k8s.io API를 사용
    
- 모니터링 아키텍처
  
  - 
