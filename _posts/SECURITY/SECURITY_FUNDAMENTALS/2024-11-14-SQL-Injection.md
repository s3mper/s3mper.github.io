---
layout: post
comments: true

title: "SQL Injection의 원리, 공격 유형과 대응 방안"
excerpt: "모의해킹 취업반 스터디 7기 5주차"

date: 2024-11-13
last_modified_at: 2026-09-09

categories: [SECURITY]
tags: [TIL, WEB, DATABASE, SECURITY]
---

<!-- markdownlint-disable MD004 MD025 MD033 -->

# SQL Injection

- 쿼리가 사용되는 로그인 흐름: [로그인 로직]({% post_url DEVELOPMENT/WEB/2024-11-03-WEB-Login-Logic %})
- 실습: [SQL Injection CTF 풀이]({% post_url SECURITY/CTF/2024-11-18-CTF-SQL-Injection-1 %})

---

## 목차

1. [개요](#1-개요)
2. [발생 원인](#2-발생-원인)
3. [작동 원리](#3-작동-원리)
    1. [정상적인 로그인](#31-정상적인-로그인)
    2. [취약한 로그인 쿼리](#32-취약한-로그인-쿼리)
4. [공격 유형](#4-공격-유형)
    1. [In Band SQLi](#41-in-band-sqli)
        1. [Union SQLi](#411-union-sqli)
        2. [Error Based SQLi](#412-error-based-sqli)
    2. [Inferential SQLi](#42-inferential-sqli)
        1. [Boolean Based Blind SQLi](#421-boolean-based-blind-sqli)
        2. [Time Based Blind SQLi](#422-time-based-blind-sqli)
    3. [Out of Band SQLi](#43-out-of-band-sqli)
    4. [Stacked Query](#44-stacked-query)
    5. [Second Order SQLi](#45-second-order-sqli)
5. [발생 가능한 위치](#5-발생-가능한-위치)
6. [영향](#6-영향)
7. [대응 방안](#7-대응-방안)
    1. [Prepared Statement 사용](#71-prepared-statement-사용)
        1. [취약한 코드](#711-취약한-코드)
        2. [안전한 코드](#712-안전한-코드)
    2. [Allow List 검증](#72-allow-list-검증)
    3. [입력값 형식 검증](#73-입력값-형식-검증)
    4. [최소 권한 적용](#74-최소-권한-적용)
    5. [오류 메시지 통제](#75-오류-메시지-통제)
        1. [사용자 응답](#751-사용자-응답)
        2. [서버 로그](#752-서버-로그)
    6. [안전한 저장 프로시저 사용](#76-안전한-저장-프로시저-사용)
    7. [단순 필터링에 의존하지 않기](#77-단순-필터링에-의존하지-않기)
    8. [ORM 사용 시 주의사항](#78-orm-사용-시-주의사항)
    9. [다중 방어 계층](#79-다중-방어-계층)
8. [출처](#출처)

---

## 1. 개요

**SQL Injection(이하 SQLi)**: 외부 입력값이 SQL 쿼리에 안전하지 않은 방식으로 결합될 때, 공격자가 원래 쿼리의 구조나 실행 의도를 변경할 수 있는 취약점이다.

공격자는 SQLi을 이용해 다음과 같은 행위를 시도할 수 있다.

- 인증 우회
- 데이터 조회 및 유출
- 데이터 추가·수정·삭제
- 데이터베이스 구조 파악
- 데이터베이스 권한 상승
- 데이터베이스 서비스 거부
- 데이터베이스 서버 또는 운영체제 침해

SQLi은 MITRE의 취약점 분류에서 `CWE-89`로 관리된다.

## 2. 발생 원인

SQLi의 핵심 원인은 **SQL 코드와 사용자 입력 데이터가 구분되지 않는 것**이다.

다음과 같이 사용자 입력을 문자열 연결로 쿼리에 직접 삽입하면 취약점이 발생할 수 있다.

```javascript
const sql =
    "SELECT * FROM member " +
    "WHERE user_id = '" + inputId + "' " +
    "AND user_pw = '" + inputPw + "'";
````

사용자 입력이 단순한 문자열이 아니라 SQL 문법의 일부로 해석될 수 있기 때문이다.

```mermaid
flowchart TD
    A["사용자 입력"] --> B["문자열로 SQL에 결합"]
    B --> C["완성된 SQL 실행"]
    C --> D{"입력값이 SQL 문법으로 해석되는가?"}
    D -->|아니요| E["정상 쿼리 실행"]
    D -->|예| F["쿼리 구조 변경"]
    F --> G["SQLi 발생"]
```

## 3. 작동 원리

### 3.1. 정상적인 로그인

사용자가 다음 정보를 입력했다고 가정한다.

```text
ID: admin
PW: 1234
```

애플리케이션에서 생성되는 SQL은 다음과 같다.

```sql
SELECT *
FROM member
WHERE user_id = 'admin'
  AND user_pw = '1234';
```

두 조건을 모두 만족하는 회원이 존재하면 로그인에 성공한다.

### 3.2. 취약한 로그인 쿼리

로그인 쿼리가 문자열 결합으로 작성되어 있다면 공격자가 따옴표(`'`, `"` 등), 논리 연산자(`AND`, `OR` 등), 주석(`#`, `--`, `/**/` 등) 등을 입력하여 조건식을 변경할 수 있다.

개념적으로 공격자는 입력값을 이용해 쿼리를 다음과 같은 형태로 바꾸려고 한다.

```sql
SELECT *
FROM member
WHERE user_id = ''
   OR '1' = '1'
   -- ' AND user_pw = '';
```

`'1' = '1'`은 항상 참이므로 전체 `WHERE` 조건이 참이 될 수 있다. 뒤의 비밀번호 조건이 주석 처리되면 비밀번호 검증도 수행되지 않는다.

## 4. 공격 유형

### 4.1. In-band SQLi

공격 요청과 결과 확인이 동일한 통신 채널에서 이루어지는 방식이다.

#### 4.1.1. UNION SQLi

`UNION` 연산자를 이용해 원래 조회 결과에 다른 `SELECT`문의 결과를 결합하는 방식이다.

```sql
SELECT title, content
FROM board
WHERE category = 'notice'

UNION

SELECT column1, column2
FROM another_table;
```

`UNION`을 사용하려면 일반적으로 다음 조건을 만족해야 한다.

* 두 `SELECT`문의 열 개수가 같아야 한다.
* 대응하는 열의 자료형이 서로 호환되어야 한다.
* 결과가 웹페이지 등에 출력되어야 한다.

관련 글: [Union SQLi]({% post_url SECURITY/SECURITY_FUNDAMENTALS/2024-11-20-Union-SQL-Injection %})

#### 4.1.2. Error-based SQLi

의도적으로 데이터베이스 오류를 발생시키고, 오류 메시지에 포함된 정보를 통해 데이터베이스 구조나 값을 추론하는 방식이다.

오류 메시지를 통해 다음과 같은 정보가 노출될 수 있다.

* DBMS 종류 및 버전
* 테이블 또는 열 이름
* SQL 구문의 일부
* 데이터 형식
* 내부 파일 경로
* 애플리케이션 구조

운영 환경에서는 상세한 데이터베이스 오류를 사용자에게 직접 반환하지 않아야 한다.

[Error-Based SQLi]({% post_url SECURITY/SECURITY_FUNDAMENTALS/2024-11-28-Error-Based-SQL-Injection %})

### 4.2. Inferential SQLi

조회 결과가 직접 출력되지 않지만, 서버 응답의 차이를 이용해 정보를 한 글자 또는 한 비트씩 추론하는 방식이다.

#### 4.2.1. Boolean-based Blind SQLi

참과 거짓 조건에 따라 응답 내용, HTTP 상태 코드, 페이지 길이 등이 달라지는 것을 이용한다.

```sql
조건이 참인 경우  → 정상 페이지
조건이 거짓인 경우 → 다른 페이지 또는 빈 결과
```

판단에 활용되는 응답 차이는 다음과 같다.

* 특정 문구의 존재 여부
* 응답 본문 길이
* HTTP 상태 코드
* 리다이렉션 여부
* 조회 결과 개수

[Blind SQLi]({% post_url SECURITY/SECURITY_FUNDAMENTALS/2024-11-28-Blind-SQL-Injection %})

#### 4.2.2. Time-based Blind SQLi

조건이 참일 때 데이터베이스가 일정 시간 동안 응답을 지연하도록 만들고, 응답 시간을 측정해 참과 거짓을 판단하는 방식이다.

DBMS별 대표적인 지연 함수는 다음과 같다.

| DBMS       | 대표 지연 방식                        |
| ---------- | ------------------------------- |
| MySQL      | `SLEEP()`                       |
| PostgreSQL | `pg_sleep()`                    |
| SQL Server | `WAITFOR DELAY`                 |
| Oracle     | `DBMS_PIPE.RECEIVE_MESSAGE()` 등 |

`sleep(1) and '1'='1`
네트워크 지연도 응답 시간에 영향을 줄 수 있으므로 기준 응답을 여러 번 측정하고 반복 가능한 차이가 있는지 확인해야 한다.

### 4.3. Out-of-band SQLi

애플리케이션의 HTTP 응답이 아니라 DNS 또는 외부 네트워크 요청과 같은 별도의 통신 경로를 통해 결과를 확인하는 방식이다.

다음 조건이 필요할 수 있다.

* 데이터베이스 서버의 외부 통신 허용
* 관련 DBMS 기능 활성화
* 데이터베이스 계정에 필요한 권한 부여
* 방화벽 또는 DNS 정책 허용

일반적인 웹 애플리케이션 환경에서는 네트워크 정책과 권한 제한으로 인해 사용할 수 없는 경우가 많다.

### 4.4. Stacked Query

하나의 입력에서 여러 SQL 문을 연속으로 실행시키는 방식이다.

```sql
SELECT ...;
UPDATE ...;
```

### 4.5. Second-order SQLi

입력 시점에는 SQL이 실행되지 않고 데이터베이스에 저장되지만, 이후 해당 값이 다른 쿼리에 결합될 때 SQLi이 발생하는 방식이다.

```mermaid
sequenceDiagram
    participant U as 사용자
    participant A as 애플리케이션
    participant D as 데이터베이스

    U->>A: 조작된 값 입력
    A->>D: 값 저장
    Note over D: 즉시 실행되지 않음
    A->>D: 저장된 값으로 동적 SQL 생성
    D-->>A: 변조된 SQL 실행
```

저장된 데이터도 항상 안전하다고 가정할 수 없으며, 다시 쿼리에 사용할 때 매개변수화해야 한다.

## 5. 발생 가능한 위치

SQLi은 DB와 상호작용하는 모든 페이지에서 발생할 수 있다. 따라서 SQL 쿼리에 영향을 주는 모든 외부 입력이 진단 대상이 될 수 있다.

* URL 쿼리 파라미터
* HTTP 요청 본문
* 로그인 및 검색 입력창
* 정렬 조건
* 필터 및 카테고리
* 페이지 번호
* 쿠키
* HTTP 헤더
* JSON 또는 XML 요청값
* 파일명과 메타데이터
* 데이터베이스에 저장되었다가 다시 사용되는 값
* REST API 및 GraphQL 인자

## 6. 영향

SQLi 취약점이 실제로 악용되면 다음과 같은 피해가 발생할 수 있다.

| 보안 요소 | 발생 가능한 영향                     |
| ----- | ----------------------------- |
| 기밀성   | 개인정보, 계정정보, 인증정보 유출           |
| 무결성   | 데이터 추가·수정·삭제                  |
| 가용성   | 대량 연산, 테이블 삭제, 잠금으로 인한 서비스 장애 |
| 인증    | 로그인 및 접근제어 우회                 |
| 권한    | 데이터베이스 권한 상승                  |
| 시스템   | DBMS 기능을 통한 파일 접근 또는 명령 실행    |

실제 영향은 다음 요소에 따라 달라진다.

* 애플리케이션 DB 계정의 권한
* DBMS 종류와 설정
* 네트워크 접근 정책
* 다중 쿼리 허용 여부
* 오류 메시지 노출 여부
* 운영체제 연동 기능의 활성화 여부

## 7. 대응 방안

### 7.1. Prepared Statement 사용

가장 중요한 대응 방법은 SQL 구조와 사용자 입력을 분리하는 것이다.

다만 다음과 같은 동적 SQL 요소는 Prepared Statement만으로 처리하기 어려울 수 있다.

* 테이블 이름
* 열 이름
* `ASC`, `DESC`와 같은 정렬 방향
* SQL 키워드
* 동적으로 생성되는 `ORDER BY` 절

이러한 값은 외부 입력을 그대로 사용하지 말고 서버에서 미리 정의한 값으로 매핑해야 한다.

#### 7.1.1. 취약한 코드

```javascript
const sql =
    "SELECT * FROM member " +
    "WHERE user_id = '" + inputId + "' " +
    "AND user_pw = '" + inputPw + "'";
```

#### 7.1.2. 안전한 코드

Node.js의 `mysql2`를 사용하는 예시는 다음과 같다.

```javascript
const sql = `
    SELECT member_id, user_id, password_hash
    FROM member
    WHERE user_id = ?
`;

const [rows] = await connection.execute(sql, [inputId]);
```

Prepared Statement를 사용하면 입력값은 SQL 코드가 아니라 하나의 데이터로 처리된다.

비밀번호는 평문으로 SQL에서 직접 비교하지 않고, 애플리케이션에서 안전한 비밀번호 해시 알고리즘을 이용해 검증해야 한다.

```javascript
const isValid = await argon2.verify(
    rows[0].password_hash,
    inputPw
);
```

권장되는 비밀번호 저장 알고리즘은 다음과 같다.

* Argon2id
* scrypt
* bcrypt
* PBKDF2

### 7.2. Allow-list 검증

Prepared Statement는 값에는 사용할 수 있지만 테이블명, 열 이름, 정렬 방향 등의 SQL 구조에는 일반적으로 사용할 수 없다.

정렬 방향을 외부 입력으로 받는 경우 다음과 같이 허용 목록을 사용한다.

```javascript
const allowedSortOrders = {
    asc: "ASC",
    desc: "DESC"
};

const order = allowedSortOrders[inputOrder] ?? "ASC";

const sql = `
    SELECT post_id, title
    FROM post
    ORDER BY created_at ${order}
`;
```

입력값이 허용 목록에 없다면 기본값을 사용하거나 요청을 거부한다.

### 7.3. 입력값 형식 검증

입력값이 예상한 형식과 범위를 만족하는지 검증한다.

```javascript
const postId = Number.parseInt(req.params.id, 10);

if (!Number.isInteger(postId) || postId < 1) {
    return res.status(400).send("잘못된 요청");
}
```

입력값 검증은 Prepared Statement를 대체하는 방법이 아니라 추가적인 방어 계층이다.

### 7.4. 최소 권한 적용

애플리케이션에서 사용하는 데이터베이스 계정에는 필요한 권한만 부여한다.

예를 들어 조회 전용 기능의 DB 계정에는 다음 권한을 부여하지 않는다.

* `INSERT`
* `UPDATE`
* `DELETE`
* `DROP`
* `CREATE`
* `FILE`
* 관리자 권한

기능별로 데이터베이스 계정을 분리하면 SQLi이 발생하더라도 피해 범위를 줄일 수 있다.

### 7.5. 오류 메시지 통제

상세한 데이터베이스 오류를 사용자에게 직접 노출하지 않는다.

#### 7.5.1. 사용자 응답

```text
요청을 처리하지 못했습니다.
```

#### 7.5.2. 서버 로그

```text
SQLSTATE, 요청 식별자, 발생 위치, 내부 예외 정보
```

로그에는 비밀번호, 세션 토큰, 주민등록번호 등의 민감정보를 기록하지 않아야 한다.

### 7.6. 안전한 저장 프로시저 사용

저장 프로시저도 매개변수를 안전하게 사용하면 SQLi 방어에 도움이 될 수 있다.

그러나 저장 프로시저 내부에서 문자열을 연결해 동적 SQL을 생성하면 동일하게 SQLi이 발생할 수 있다.

```sql
SET @query = CONCAT(
    'SELECT * FROM member WHERE user_id = ''',
    input_id,
    ''''
);
```

따라서 저장 프로시저 사용 자체를 SQLi 방어로 간주해서는 안 된다.

### 7.7. 단순 필터링에 의존하지 않기

다음과 같은 문자열을 제거하는 블랙리스트 방식만으로는 충분하지 않다.

```text
OR
UNION
SELECT
--
'
```

우회 가능한 원인은 다음과 같다.

* 대소문자 차이
* 공백 및 주석 변형
* 인코딩
* DBMS별 문법 차이
* 동등한 함수 및 연산자
* 문자열 조합

입력값 이스케이프 역시 문자 집합, SQL Mode, DBMS에 따라 문제가 발생할 수 있으므로 주요 방어 방법으로 사용하지 않는다.

### 7.8. ORM 사용 시 주의사항

ORM을 사용한다고 해서 SQLi이 자동으로 모두 방지되는 것은 아니다.

다음 기능을 사용할 때는 별도로 확인해야 한다.

* Raw Query
* 문자열로 생성한 조건식
* 동적 정렬
* 동적 테이블 또는 열 이름
* ORM 고유 쿼리 언어
* 저장 프로시저 호출

ORM에서도 외부 입력은 바인딩 변수로 전달해야 한다.

### 7.9. 다중 방어 계층

효과적인 대응은 다음 방식을 함께 적용하는 것이다.

1. Prepared Statement
2. Allow-list 입력값 검증
3. 최소 권한
4. 안전한 오류 처리
5. 민감정보 비노출
6. 보안 로깅 및 모니터링
7. 코드 리뷰와 정적 분석
8. 정기적인 취약점 진단

---

## 출처

- MDN. 「SQL Injection」. [MDN](https://developer.mozilla.org/ko/docs/Glossary/SQL_Injection)
- OWASP. 「SQL Injection」. [OWASP](https://owasp.org/www-community/attacks/SQL_Injection)
- OWASP. 「SQL Injection Prevention Cheat Sheet」. [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- OWASP. 「Query Parameterization Cheat Sheet」. [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html)
- MITRE CWE-89. 「Improper Neutralization of Special Elements used in an SQL Command」. [MITRE CWE-89](https://cwe.mitre.org/data/definitions/89.html)
- PortSwigger Web Security Academy. 「SQL Injection」. [PortSwigger Web Security Academy](https://portswigger.net/web-security/sql-injection)
- PortSwigger Web Security Academy. 「SQL Injection Labs」. [PortSwigger Web Security Academy](https://portswigger.net/web-security/all-labs#sql-injection)

<div class="obsidian-links" style="display: none;">

[\[SECURITY\] SQL Injection Tip](2024-12-04-SQL-Injection-Tip.md)
[\[CTF\] CTF 풀이: Cookie, SQL Injection](../CTF/2024-11-18-CTF-SQL-Injection-1.md)
[\[CTF\] CTF 문제 풀이: SQL Injection w. UNION](../CTF/2024-11-25-CTF-SQL-Injection-2.md)
[\[CTF\] CTF 문제 풀이: SQL Injection](../CTF/2024-11-26-CTF-SQL-Injection-3.md)
[\[CTF\] CTF 문제 풀이: SQL Injection](../CTF/2024-11-28-CTF-SQL-Injection-4.md)
[\[CTF\] CTF 문제 풀이: Error Based SQL Injection](../CTF/2024-11-28-CTF-SQL-Injection-5.md)
[\[CTF\] CTF 문제 풀이: Blind SQL Injection](../CTF/2024-11-29-CTF-SQL-Injection-6.md)
[\[CTF\] CTF 문제 풀이: SQL Injection](../CTF/2024-12-09-CTF-SQL-Injection-7.md)
[\[CTF\] Dreamhack 문제 풀이: Really Not SQL](../CTF/2026-07-24-Really-Not-SQL.md)
[\[SECURITY\] WEB 취약점](2025-02-21-WEB-Secure-Weak-Points.md)

</div>
