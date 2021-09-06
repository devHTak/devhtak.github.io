---
layout: post
title: Cookie, Session, Filter, Interceptor
summary: Spring MVC Basic
author: devhtak
date: '2021-09-06 21:41:00 +0900'
category: Spring
---

#### Cookie, Session

- stateless한 HTTP 프로토콜의 특징과 대비하게 Stateful한 상태를 위해 사용
- 쿠키와 세션의 차이점
  |-|쿠키|세션|
  |--|--|--|
  |저장 위치|클라이언트(=접속자 PC)|웹 서버|
  |저장 형식|text (key-value)|Object|
  |만료 시점|쿠키 저장시 설정(만료 시간 기준)|브라우저 종료시 삭제(기간 지정 가능)|
  |사용하는 자원(리소스)|클라이언트 리소스|웹 서버 리소스|
  |용량 제한|총 300개, 하나의 도메인 당 20개, 하나의 쿠키 당 4KB(=4096byte)|서버가 허용하는 한 용량제한 없음.|
  |속도|세션보다 빠름|쿠키보다 느림|
  |보안|세션보다 안좋음|쿠키보다 좋음|

#### Cookie 를 통한 로그인

- Server에서 로그인이 성공하면 HTTP 응답에 쿠키를 담아 브라우저에 전달
- 브라우저는 앞으로 해당 쿠키를 지속해서 보내준다.
- 쿠키에는 영속 쿠키와 세션 쿠키가 있다.
  - 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지
  - 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지
  - 브라우저 종료 시 로그아웃이 되야 하기 때문에 세션 쿠키이다.
  
- 예시
  ```java
  Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId());
  response.addCookie(idCookie);
  ```
  - 로그인 성공 시 쿠키를 생성하고 HttpServletResponse에 담는다.
  - 쿠키 이름은 memberId이고, 회원의 ID를 담아둔다.
  - 웹 브라우저는 종료 전까지 회원의 ID를 서버에 계속 보내줄 것이다.
  
  - @CookieValue를 통한 쿠키 조회
    ```java
    @GetMapping("/")
    public String homeLogin(@CookieValue(name="memberId", required="false", Long memberId, Model model) {
        if(memberId == null) return "home";
        Member loginMember = memberRepository.findById(memberId);
        if(loginMember == null) return "home";
        
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
    ```
    - @CookieValue 를 사용하면 편리하게 쿠키를 조회할 수 있다.
    - 로그인 하지 않은 사용자도 홈에 접근할 수 있기 때문에 required = false 를 사용한다
    
  - 로그아웃
    ```java
    @PostMapping("/logout")
    public String logout(HttpServletResponse response) {
        expireCookie(response, "memberId");
        return "redirect:/";
    }
    private void expireCookie(HttpServletResponse response, String cookieName) {
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0);
        response.addCookie(cookie);
    }
    ```
    - 로그아웃도 응답 쿠키를 생성하는데 Max-Age=0 를 확인할 수 있다. 해당 쿠키는 즉시 종료된다.

- 쿠키와 보안 문제
  - 보안 문제
    - 쿠키 값은 임의로 변경할 수 있다.
      - 클라이언트가 쿠키를 강제로 변경하면 다른 사용자가 된다.
      - 실제 웹브라우저 개발자모드 Application Cookie 변경으로 확인
      - Cookie: memberId=1 Cookie: memberId=2 (다른 사용자의 이름이 보임)
    - 쿠키에 보관된 정보는 훔쳐갈 수 있다.
      - 만약 쿠키에 개인정보나, 신용카드 정보가 있다면?
      - 이 정보가 웹 브라우저에도 보관되고, 네트워크 요청마다 계속 클라이언트에서 서버로 전달된다.
      - 쿠키의 정보가 나의 로컬 PC가 털릴 수도 있고, 네트워크 전송 구간에서 털릴 수도 있다.
    - 해커가 쿠키를 한번 훔쳐가면 평생 사용할 수 있다.
      - 해커가 쿠키를 훔쳐가서 그 쿠키로 악의적인 요청을 계속 시도할 수 있다.
  - 대안
    - 쿠키에 중요한 값을 노출하지 않고, 사용자 별로 예측 불가능한 임의의 토큰(랜덤 값)을 노출하고, 서버에서 토큰과 사용자 id를 매핑해서 인식한다. 그리고 서버에서 토큰을 관리한다.
    - 토큰은 해커가 임의의 값을 넣어도 찾을 수 없도록 예상 불가능 해야 한다.
    - 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 해당 토큰의 만료시간을 짧게(예: 30분) 유지한다. 또는 해킹이 의심되는 경우 서버에서 해당 토큰을 강제로 제거하면 된다.

#### Session 을 통한 로그인

- Cookie에 저장할 때 생기는 보안문제가 발생하기 때문에 중요한 정보는 서버에 저장해야 한다.
- 클라이언트와 서버는 추정 불가능한 임의의 식별자 값으로 연결해야 한다.
- 서버에 중요한 정보를 보관하고 연결을 유지하는 방법이 세션

- 세션 로그인 동작 방식
  - 로그인
    - 사용자가 loginId, password 정보를 전달하면 서버에서 해당 사용자가 맞는지 확인
  - 세션 생성
    - 세션 ID를 생성하는 데, 추정 불가능해야 한다.
    - UUID는 추정이 불가능하다.
    - 생성된 세션 ID와 세션에 보관할 값(memberA)을 서버의 세션 저장소에 보관한다.
  - 세션 id를 응답 쿠키로 전달
  - 클라이언트와 서버는 결국 쿠키로 연결이 되어야 한다.
    - 서버는 클라이언트에 mySessionId 라는 이름으로 세션ID 만 쿠키에 담아서 전달한다.
    - 클라이언트는 쿠키 저장소에 mySessionId 쿠키를 보관한다.
    - 여기서 중요한 포인트는 회원과 관련된 정보는 전혀 클라이언트에 전달하지 않는다는 것이다.
    - 오직 추정 불가능한 세션 ID만 쿠키를 통해 클라이언트에 전달한다.
  - 클라이언트의 세션id 쿠키 전달
    - 클라이언트는 요청시 항상 mySessionId 쿠키를 전달한다.
    - 서버에서는 클라이언트가 전달한 mySessionId 쿠키 정보로 세션 저장소를 조회해서 로그인시 보관한 세션 정보를 사용한다.

- 보안 이슈 해결
  - 쿠키 값을 변조 가능, 예상 불가능한 복잡한 세션Id를 사용한다.
  - 쿠키에 보관하는 정보는 클라이언트 해킹시 털릴 가능성이 있다. 세션Id가 털려도 여기에는 중요한 정보가 없다.
  - 쿠키 탈취 후 사용 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 세션의 만료시간을 짧게(예: 30분) 유지한다.
  - 해킹이 의심되는 경우 서버에서 해당 세션을 강제로 제거하면 된다.


#### 출처

- 인프런 강의: 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 (김영한님 강의)
- https://velog.io/@fortice/Spring-%EC%84%B8%EC%85%98-%EC%9D%B8%ED%84%B0%EC%85%89%ED%84%B0-%EC%BF%A0%ED%82%A4
