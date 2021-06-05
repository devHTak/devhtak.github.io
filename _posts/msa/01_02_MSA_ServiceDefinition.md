---
layout: post
title: Micro Service Definition
summary: msa 서비스 정의
author: devhtak
date: '2021-06-05 14:41:00 +0900'
category: msa
---

#### 다양한 서비스 분리 기준

- Domain Driven Design
  - 서비스를 분해하는 방법 중 하나로 DDD 개념 이용
  - 논리 데이터 모델의 Subject Area 또는 그 하위 그룹 단위로 표현될 수 있음
  - Main Model <<Context Map>> <-> Sub Model <<Bounded Context>> (-> Dependency <<Sharked Kernel>>)
    - Bounded Context
      - 업무의 독립 단위
      - 업무적 연관성이 높은 기능/데이터를 하나의 Boundary로 구분
      - 서비스 예상 단위
    - Sharked Kernel
      - 두개 이상의 모델에서 공유되는 모듈/데이터
      - 연관 팀이 같이 설계하고 변경에 대해서 서로 협의하여 반영
      - 전체 공통 또는 업무 내 공통
  
- 업무 기능 관점
  - Buisiness Capability > Service
  - 
