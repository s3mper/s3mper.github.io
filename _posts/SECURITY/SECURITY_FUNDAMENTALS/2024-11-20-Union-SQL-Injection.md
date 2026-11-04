---
layout: post
comments: true
title: UNION SQL Injection 개념 정리
excerpt: MySQL과 ORACLE 실습 코드 정리
date: 2024-11-20
last_modified_at: 2026-11-03
categories:
  - SECURITY
tags:
  - TIL
  - WEB
  - DATABASE
  - SECURITY
---

<!-- markdownlint-disable MD010 MD025 MD033 -->

# UNION SQL Injection 개념 정리

- SQL Injection: [SQL Injection 정리]({% post_url SECURITY/SECURITY_FUNDAMENTALS/2024-11-14-SQL-Injection %})
- Blind SQLi: [Blind SQL Injection]({% post_url SECURITY/SECURITY_FUNDAMENTALS/2024-11-28-Blind-SQL-Injection %})

## 1. UNION SQL Injection이란?

UNION SQL Injection(이하 UNION SQLi): SQL Injection 중에서 `UNION` 구문을 사용하여 데이터를 탈취하는 방법을 말한다.
이 방식을 알기 위해서는 SQL의 `UNION` 구문에 대해 알 필요가 있다.

## UNION SQLi 공격 Point

1. DB를 사용하는 곳
2. 그 데이터가 화면에 나오는 곳곳
    1. 게시판
    2. 로그인, 아이디 중복체크 등

## ORDER BY

출력되는 데이터 정렬할 때 사용
`SELECT ~ ORDER BY [COLUMN]`

## UNION SQLi Process

1. SQLi 취약점 존재 여부를 확인한다.
	- 싱글쿼터(`'`), 파이프(`|`) 등과 같은 특수문자 검색 → 서버 반응 확인
2. UNION SQLi
	1. 컬럼 개수를 확인한다.
		- `ORDER BY` 구문을 사용
	2. 출력되는 컬럼 위치를 찾는다.
	3. 컬럼 데이터형을 확인한다.
		- `NULL`, 숫자형, 문자형 등을 조회
3. 데이터 탈취
	1. DB 이름을 확인한다.
		- MySQL

			```SQL
			SELECT DATABASE()
			```

	2. 테이블 이름을 확인한다.
		- MySQL

			```SQL
			SELECT table_name
			FROM information_schema.tables
			WHERE table_schema = [DB 이름]
			```

		- ORACLE

			```SQL
			SELECT TABLE_NAME
			FROM ALL_TABLES
			```

	3. 컬럼 이름을 확인한다.
		- MySQL

			```SQL
			SELECT column_name
			FROM information_schema.columns
			WHERE table_name = [테이블 이름]
			```

		- ORACLE

			```SQL
			SELECT COLUMN_NAME
			FROM ALL_TAB_COLUMNS
			WHERE TABLE_NAME=[테이블 이름]
			```

	4. 원하는 데이터를 추출한다.

---

## 출처

- RedirHub. 「Redirect Types and SEO Effects」. [RedirHub](https://www.redirhub.com/ko/blog/redirect-types-and-seo-effects-2)
- MDN. 「HTTP 리디렉션」. [MDN](https://developer.mozilla.org/ko/docs/Web/HTTP/Redirections)
- SQL Fiddle. 「Online SQL Compiler for Learning & Practice」. [SQL Fiddle](https://sqlfiddle.com/)
- Pentestmonkey. 「Oracle SQL Injection Cheat Sheet」. [Pentestmonkey](https://pentestmonkey.net/cheat-sheet/sql-injection/oracle-sql-injection-cheat-sheet)
- SK Shieldus. 「웹 취약점과 해킹 메커니즘 #2 SQL Injection 개요」. [네이버 블로그](https://blog.naver.com/sk_shieldus/222734386644)

<div class="obsidian-links" style="display: none;">

[\[SECURITY\] SQL Injection 정리](2024-11-14-SQL-Injection.md)
[\[CTF\] CTF 문제 풀이: SQL Injection w. UNION](../CTF/2024-11-25-CTF-SQL-Injection-2.md)

</div>
