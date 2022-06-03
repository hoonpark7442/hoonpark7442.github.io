---
layout: post
title: 기술면접 준비 - web
author: sisipark
tags: [기술면접]
excerpt_separator: <!--more-->
---

# 기술 면접 준비


## web
<!--more-->

1. 브라우져에 url검색하면?
<details>
  <summary>답변보기 ▾</summary>
  <div markdown="1">
    참고: https://devjin-blog.com/what-happen-browser-search/
    1. 웹브라우져가 URL을 해석한다.
    2. 문법에 맞으면 캐싱된 DNS 기록들을 뒤져 해당 url의 ip주소를 찾는다. 
    3. 없다면 ISP의 DNS 서버가 해당 ip 찾기 위해 다른 네임서버들에 DNS 쿼리를 날린다.(?)
    4. 브라우져가 올바른 ip주소를 얻게 되면 서버와 connection 빌드를 한다. http 요청의 경우 TCP를 사용한다. TCP 3way handshaking 통해 연결이 이루어진다.
    - 클라이언트 머신이 SYN 패킷을 서버에 보내고 connection을 열어달라고 물어본다
    - 서버가 새로운 connection을 시작할 수 있는 포트가 있다면 SYN/ACK 패킷으로 대답을 한다
    - 클라이언트는 SYN/ACK 패킷을 서버로부터 받으면 서버에게 ACK 패킷을 보낸다
    5. 브라우져가 웹서버에 HTTP 요청을 한다. 헤더에 부가적인 정보들을 담아 전송한다. 
    6. 서버가 요청을 처리하고 응답을 생성한다. 
    7. 응답을 전송한다. 요청한 웹페이지, status code, compression type, cache-control, 쿠키 등의 정보를 담아 보낸다. 
    8. 브라우져가 HTML 컨텐츠를 보여준다.
  </div>
</details>

<br>

2. 쿠키와 세션
<details>
  <summary>답변보기 ▾</summary>
  <div markdown="1">
    HTTP는 비연결지향, stateless가 특징이다. 그러므로 요청간 의존관계가 없다. 이전 요청과 현재 요청이 같은 사용자에 의한 것인지 알기 위해서는 상태를 유지해야한다.
    HTTP 프로토콜에서 상태를 유지하기 위한 기술로 세션과 쿠키가 있다. 

    쿠키
    - 유저의 브라우저에 저장되는 키와 밸류가 들어있는 값
    - 이름, 값, 유효시간, 경로 등을 포함
    - 클라이언트의 상태 정보를 브라우저에 저장하여 참조
    - 웹브라우저가 서버에 요청 -> 상태 유지하고 싶은 값을 쿠키로 생성 -> 서버가 응답할 때 HTTP 헤더(Set-Cookie)에 쿠키를 포함해서 전송 -> 전달받은 쿠키는 웹브라우저에서 관리하고 있다가, 다음 요청 때 쿠키를 HTTP 헤더에 넣어서 전송
    - 쿠키사용 예: 아이디 저장, 장바구니

    세션
    - 일정 시간 동안 같은 브라우져로부터 들어오는 요청을 하나의 상태로 보고 그 상태를 유지하는 기술
    - 즉 웹브라우저 통해 서버에 접속한 이후부터 브라우저 종료할 때 까지 유지되는 상태 
  </div>
</details>

<br>

3. REST API란
<details>
  <summary>답변보기 ▾</summary>
  <div markdown="1">
    REST 기반으로 서비스 API 구현한 것
    자원의 표현으로 상태를 주고 받는 것
    - 표현: URI
    - 상태: 정보
    자원의 행동: HTTP method

    REST란
    - HTTP URI(Uniform Resource Identifier)를 통해 자원(Resource)을 명시하고, HTTP Method(Post, Get, Put, Delete)를 통해 해당 자원에 대한 CRUD Operation을 적용하는 것을 의미한다. 

    REST API 설계 기본 규칙
    - URI는 정보의 자원을 표현해야 한다.
    - 자원에 대한 행위는 HTTP Method(GET, PUT, POST, DELETE 등)로 표현한다.
  </div>
</details>






