에러 메시지에 자세한 정보를 출력하는 함수

서브쿼리(DB Version 출력) 실행결과가 포함된 에러

```sql
UTL_INADDR_GET_HOST_NAME((…))
UTL_INADDR_GET_HOST_ADDRESS((…))
ORDSYS_SRD_DICOM_GETMAPPINGXPATH((…, user, user))
CTXSYS.DRITHSX.SN(user, (…))
```

1. SQLi 취약점 존재 여부
2. Error Based SQLi
	1. 테이블 정보 추출
		1. 테이블 개수 확인
		2. 테이블 1개씩 추출 → 모든 테이블 추출
		3. 테이블 이름 확인
	2. 컬럼 정보 추출
		1. 컬럼 개수 확인
		2. 컬럼 1개씩 추출 → 모든 컬럼 추출
		3. 컬럼 이름 확인
	3. 데이터 정보 추출
		1. 데이터 정보 추출
		2. 데이터 1개씩 추출 → 모든 데이터 추출
		3. 데이터 이름 확인
3. 원하는 데이터 탈취

테이블 목록 개수 먼저 확인하기
```SQL
SELECT COUNT(TABLE_NAME)
FROM ALL_TABLES
```

첫번째 테이블명 출력
```sql
SELECT TABLE_NAME
FROM (
	SELECT TABLE_NAME, ROWNUM AS RNUM
	FROM ALL_TABLES
)
WHERE RNUM=1
```

RNUM이 아니면 행을 제거함
→ 고정해주는 작업