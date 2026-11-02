1. SQLI 가능 여부 확인
2. 데이터 개수 확인
3. 데이터 문자열 1개씩 추출
4. 데이터 1개 추출
5. 모든 데이터 추출
6. 원하는 데이터 추출

```SQL
SELECT SUBSTR('ABC', 1, 1)
FROM DUAL
// A
```

```SQL
SELECT ASCII(SUBSTR('abc', 1, 1))
FROM DUAL
// 97
```

이진탐색트리

