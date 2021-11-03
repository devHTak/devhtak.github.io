---
layout: post
title: SQL Tuning. index 기본 및 튜닝
summary: SQL Tuning
author: devhtak
date: '2021-10-24 21:41:00 +0900'
category: SQL
---

#### 인덱스 구조 및 탐색

- 미리보는 인덱스 튜닝

  - 데이터베이스 테이블에서 데이터를 찾는 방법 2가지
    - 테이블 전체 스캔
    - 인덱스 이용

  - 인덱스 튜닝의 핵심요소
    - 인덱스 스캔 효율화 튜닝
      - 인덱스 스캔과정에서 발생하는 비효율을 줄이는 것
    - 테이블 랜덤 액세스 최소화 튜닝
      - 테이블 액세스 횟수를 줄이는 방법
      - 인덱스 스캔 후 테이블 레코드를 액세스할 때 랜덤 I/O 방식을 사용

    ![image](https://user-images.githubusercontent.com/42403023/140036433-ca7cf8ec-0d6d-4f98-a6af-f9349a60c5d5.png)

    - 인덱스 스캔 효율화보다 테이블 랜덤 액세스 최소화 튜닝이 성능 향상에 더 좋다.
    - SQL 튜닝은 랜덤 I/O를 줄이는 것이 중요하다
    - index scan(Sequential 액세스)
      - 레코드간 논리적 또는 물리적인 순서를 따라 차례대로 읽어 나가는 방식
      - 인덱스 리프 블록에 위치한 모든 레코드는 포인터를 따라 논리적으로 연결돼 있고, 이 포인터를 따라 스캔하는 것은 Sequential 액세스 방식
    - Table random access(Random access)
      - Random 액세스는 레코드간 논리적, 물리적인 순서를 따르지 않고, 한 건을 읽기 위해 한 블록씩 접근하는 방식

- 인덱스 구조
  - B* Tree
    
    ![image](https://user-images.githubusercontent.com/42403023/140037570-a8c21e4b-6bf1-4ee5-8640-b50bd1b1a654.png)

    - 


#### 참고

- 조시형 저자의 친절한 SQL 튜닝 교재
- sequential & randomaccess: https://m.cafe.daum.net/Oracle-/DHl0/94?listURI=%2FOracle-%2F_rec
