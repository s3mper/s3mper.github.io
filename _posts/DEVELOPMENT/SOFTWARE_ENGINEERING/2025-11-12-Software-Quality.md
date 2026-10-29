---
layout: post
comments: true

title: "소프트웨어 품질의 개념과 품질 관리"
excerpt: "소프트웨어 품질의 개념과 품질 관리"

date: 2025-11-12
last_modified_at: 2026-10-29

categories: [SOFTWARE_DESIGN]
tags: [SOFTWARE_DESIGN]
---

# 소프트웨어 품질의 개념과 품질 관리

## 목차

1. [정의](#1-정의)
2. [관리 사이클 & 측정](#2-관리-사이클--측정)
    1. [관리 사이클](#21-관리-사이클)
    2. [측정](#22-측정)
3. [품질 관리(Quality Control)](#3-품질-관리quality-control)
4. [알고리즘 관점의 품질](#4-알고리즘-관점의-품질)

---

## 1. 정의

- **Philip Crosby(1926~2001)**: 품질=요구사항(명세) 적합성
	- 명확한 정의·측정·관리 강조.
- **W Edwards Deming(1900~1993)**: 품질=고객 만족
	- 리더십·두려움 제거·부서 장벽 해소 등 **14원칙**으로 지속개선.
- **Armand V. Feigenbaum(1920~2014)**: 품질은 고객이 결정
	- **전사적 품질관리(TQC)**, 목표는 **변동**.
- **Kaoru Ishikawa(1915~1989)**: 표준 충족만으론 부족
	- 품질 범위 **광범위**, **가격=품질 속성** 포함.
- **Joseph Moses Juran(1904~2008)**: **사용 적합성**
	- **품질 기획–관리–개선**의 구조화된 경영.
- **Walter Shewhart(1891~1967)**: **객관/주관** 두 측면을 모두 다뤄야 함

- **명세 적합성(Verification)**: 규격·요구사항 충족 여부.
- **고객 요구 충족(Validation)**: 사용 맥락에서의 만족·가치.

## 2. 관리 사이클 & 측정

### 2.1. 관리 사이클

![Alt Images](https://cdn.jsdelivr.net/gh/s3mper/s3mper.github.io@main/_image/2025-11-17-13-51-03.png?raw=true)
> PDA Cycle(Walter A. Shewhart)

![393x393](https://cdn.jsdelivr.net/gh/s3mper/s3mper.github.io@main/_image/2025-11-12-12-45-04.png?raw=true)
> PDCA Cycle(Dr. W. Edwards): 계획→실행→점검→개선, 카이젠의 핵심 도구.

### 2.2. 측정

> Measurement is the first step that leads to control and eventually to improvement.
> If you can’t measure something(to see), you can’t understand it.
> If you can’t understand it, you can’t control it.
> If you can’t control it, you can’t improve it.
> -James Harrington

- 측정→이해→통제→개선의 출발점
- 측정: 사물/사건에 숫자 부여하는 것이다.

## 3. 품질 관리(Quality Control)

![Alt Images](https://cdn.jsdelivr.net/gh/s3mper/s3mper.github.io@main/_image/2025-11-12-12-44-09.png?raw=true)
> 품질 관리: 입력(input) → 프로세스(process) → 출력(output)으로 전환되는 동안 품질을 일관되게 확보/통제

![470x210](https://cdn.jsdelivr.net/gh/s3mper/s3mper.github.io@main/_image/2025-11-12-12-59-59.png?raw=true)
> 품질 관리 프로세스: 품질 요구 정의 → 측정/점검 → 변동(문제) 식별 → 시정/예방조치 → 모니터링/표준화

![Alt Images](https://cdn.jsdelivr.net/gh/s3mper/s3mper.github.io@main/_image/2025-11-12-12-44-32.png?raw=true)
(a) 전통적 QC ↔ (b) **통계적 QC** 대비.

- 전통적 QC: 사후 검사, 합격/불합격 중심의 통제
- 통계적 QC: 샘플링과 관리도 등 통계 기법으로 변동을 관리해 예방 중심으로 전환

## 4. 알고리즘 관점의 품질

- **Correctness(정확성)**
- **Efficiency(효율성)**
	- ![Alt Images](https://cdn.jsdelivr.net/gh/s3mper/s3mper.github.io@main/_image/2025-11-12-12-45-23.png?raw=true)
	- 시간 (O(f(n)))
	- 공간 (memory)
- **Maintainability(유지보수성)**
	- Extendability(확장성)
- **Understandability(가독성)**
	- Structure(구조)