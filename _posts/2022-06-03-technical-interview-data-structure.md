---
layout: post
title: 기술면접 준비 - 자료구조
author: sisipark
tags: [기술면접]
excerpt_separator: <!--more-->
---

# 기술 면접 준비


## 자료구조
<!--more-->

1. Heap
<details>
  <summary>답변보기 ▾</summary>
  <div markdown="1">
    완전 이진트리의 일종.
    여러 값 중 최대값과 최소값을 빠르게 찾아내도록 만들어진 자료구조
    우선순퀴 큐에 주로 사용된다
    느슨한 정렬상태를 유지한다
    최대 힙과 최소힙으로 구분
    배열을 이용해 구현한다
    - 왼쪽 자식의 인덱스 = (부모의 인덱스) * 2
    - 오른쪽 자식의 인덱스 = (부모의 인덱스) * 2 + 1
    - 오른쪽 자식의 인덱스 = (부모의 인덱스) * 2 + 1
    힙의 삽입과 삭제는 O(log n)

  </div>
</details>
<br>
2. 이진탐색트리
<details>
  <summary>답변보기 ▾</summary>
  <div markdown="1">
    왼쪽 자식은 부모보다 작고 오른쪽 자식은 부모보다 큰 이진트리
    중복된 노드가 없어야 함
    - 검색 목적 자료구조니까
    이진탐색트리의 순회는 중위순회(왼-루트-오)
    삽입,검색,삭제 O(log n) ~ O(n)
    - 편향트리(예를들어 왼쪽노드로만 죄다 몰린다면) 그땐 O(N)이다.
  </div>
</details>
<br>
3. 해쉬
<details>
  <summary>답변보기 ▾</summary>
  <div markdown="1">
    해시에 매핑하여 데이터를 저장하는 자료구조
    키 값을 해시함수 통해 해시값으로 만들고 이걸 인덱스로 사용하여 데이터 저장. 
    데이터가 많아지면 해시충돌 일어남
    그래도 쓰는 이유
    - 적은 자원으로 많은 데이터 효율적으로 관리하기 위해
    해시테이블 삽입,삭제,검색 시간복잡도는 O(1)
  </div>
</details>

<br>
4. 해시충돌 회피 방법
<details>
  <summary>답변보기 ▾</summary>
  <div markdown="1">
    Open Addressing
    - 다른 해시값에 저장
    - 추가 메모리 공간을 사용하지 않고 Hash table 배열의 빈 공간을 사용
    - Separating Chanining 방법에 비해 메모리 적게 사용

    Separating Chanining
    - 링크드리스트 (또는 레드블랙 트리) 사용
    - 해시 테이블이 원소 하나 담는게 아니라 링크드리스트 담음
  </div>
</details>











