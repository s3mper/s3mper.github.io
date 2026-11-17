---
layout: post
comments: true

title: "한국어 텍스트 토큰화의 개념과 실습"
excerpt: "토큰화의 기준과 한국어의 특성을 이해, NLTK·KSS·KoNLPy 실습"

date: 2025-03-16
last_modified_at: 2026-11-16

categories: [ai]
tags: [til, nlp, ai, tokenization]
redirect_from:
  - /Korean-Tokenization/
guid: https://s3mper.github.io/Korean-Tokenization/
---

<!-- markdownlint-disable MD004 MD007 MD010 MD025 MD033 -->

# 토큰화(Tokenization)

---

## 목차

1. [토큰화 개념](#1-토큰화-개념)
	1. [말뭉치(corpus)와 토큰(token)](#11-말뭉치corpus와-토큰token)
	2. [토큰의 단위](#12-토큰의-단위)
2. [토큰화 기준과 고려 사항](#2-토큰화-기준과-고려-사항)
	1. [분석 목적과 모델에 따른 기준](#21-분석-목적과-모델에-따른-기준)
	2. [구두점과 특수 문자](#22-구두점과-특수-문자)
	3. [축약형과 다단어 표현](#23-축약형과-다단어-표현)
3. [영어 토큰화 예시: Penn Treebank](#3-영어-토큰화-예시-penn-treebank)
	1. [주요 처리 규칙](#31-주요-처리-규칙)
	2. [NLTK 실습](#32-nltk-실습)
4. [한글 토큰화의 특성과 어려움](#4-한글-토큰화의-특성과-어려움)
	1. [어절과 단어의 차이](#41-어절과-단어의-차이)
	2. [교착어와 형태소](#42-교착어와-형태소)
	3. [띄어쓰기와 비정형 표현](#43-띄어쓰기와-비정형-표현)
	4. [목적에 따른 토큰화 방식 선택](#44-목적에-따른-토큰화-방식-선택)
5. [품사 태깅](#5-품사-태깅part-of-speech-tagging)
	1. [품사 태깅의 개념](#51-품사-태깅의-개념)
	2. [형태소 분석과 품사 태깅의 관계](#52-형태소-분석과-품사-태깅의-관계)
6. [한글 토큰화 실습](#6-한글-토큰화-실습)
	1. [KSS를 이용한 문장 분리](#61-ksskorean-sentence-splitter를-이용한-문장-분리)
	2. [KoNLPy를 이용한 형태소 분석](#62-konlpy를-이용한-형태소-분석)
7. [관련 글](#관련-글)
8. [출처](#출처)

---

## 1. 토큰화 개념

**토큰화(tokenization)**: 주어진 코퍼스(corpus)에서 정해진 기준에 따라 토큰(token) 단위로 나누는 작업이다.

### 1.1. 말뭉치(corpus)와 토큰(token)

- **말뭉치(corpus)**: 언어를 분석하거나 자연어 처리 모델을 학습, 평가하기 위해 모아 놓은 언어 자료의 집합이다.
- **토큰(token)**: 텍스트를 처리하기 위해 나눈 개별 단위이다.

### 1.2. 토큰의 단위

토큰의 단위는 분석 목적과 모델에 따라 달라진다. 따라서 토큰화를 수행하기 전에 어떤 단위로 텍스트를 처리할 것인지 결정해야 한다.

토큰의 단위는 문장, 어절, 단어, 형태소, 문자, 서브워드(subword) 등이 있다.

| 단위            | 설명                            | 예시                                    |
| ------------- | ----------------------------- | ------------------------------------- |
| 문장            | 문장의 경계를 기준으로 구분한다.            | `오늘은 맑다.`, `산책을 한다.`                  |
| 어절            | 띄어쓰기를 기준으로 구분한다.              | `에디가`, `책을`, `읽었다`                    |
| 단어            | 언어의 단어 경계와 토큰화 규칙에 따라 구분한다.   | `I read books` → `I`, `read`, `books` |
| 형태소           | 뜻이나 문법적 기능을 가진 최소 단위로 구분한다.   | `책을` → `책`, `을`                       |
| 문자            | 문자 단위로 구분한다.                  | `사과` → `사`, `과`                       |
| 서브워드(subword) | 학습된 어휘와 분할 규칙에 따라 단어 내부를 나눈다. | `playing` → `play`, `ing`처럼 분할할 수 있다. |

서브워드의 분할 결과는 토크나이저의 어휘와 알고리즘에 따라 달라진다.

- 대표 서브워드 토큰화 알고리즘: BPE, WordPiece, Unigram

## 2. 토큰화 기준과 고려 사항

토큰화 기준을 정한다는 것은 **무엇을 하나로 유지하고 어디에서 나눌 것인지 결정한다는 의미**이다. 언어의 특성, 분석 목적, 입력 데이터의 형태, 사용할 모델을 함께 고려한다.

| 고려 요소 | 판단할 내용 | 예시 |
| --- | --- | --- |
| 분석 목적 | 어떤 정보를 보존해야 하는가? | 감성 분석에서는 느낌표나 이모지가 유용할 수 있다. |
| 모델과의 호환성 | 모델이 어떤 토큰화 방식으로 학습되었는가? | 사전학습 모델에 대응하는 토크나이저를 사용한다. |
| 언어의 특성 | 공백만으로 필요한 단위를 얻을 수 있는가? | `책을`을 `책`과 `을`로 나눌지 결정한다. |
| 특수 표현 | 숫자와 기호를 하나의 표현으로 유지할 것인가? | `3.14`, `AT&T`, 이메일 주소를 처리한다. |
| 복합 표현 | 여러 단어를 하나의 분석 단위로 묶을 것인가? | `New York`을 지명 하나로 취급한다. |

### 2.1. 분석 목적과 모델에 따른 기준

같은 문장도 분석 목적에 따라 필요한 정보가 달라진다.

예를 들어 `이 영화는 재미없다`에서 명사만 추출하면 `영화`라는 주제를 파악할 수 있다. 하지만 감성을 분석하려면 `재미없다`라는 서술어가 필요하다. 명사만 남기면 부정적인 평가가 사라질 수 있다.

### 2.2. 구두점과 특수 문자

구두점이나 특수 문자는 문장의 구조와 의미를 나타낼 수 있다. 따라서 분석 목적을 고려하지 않고 단순 제거해서는 안 된다.

- 구두점(punctuation): 마침표(`.`), 쉼표(`,`), 물음표(`?`), 세미콜론(`;`), 느낌표(`!`) 등과 같은 기호

| 표현            | 고려 사항                   |
| ------------- | ----------------------- |
| `오늘은 맑다.`     | 마침표가` 문장의 끝`을 나타낸다.     |
| `3.14`        | 마침표가` 소수점`으로 사용된다.      |
| `m.p.h.`      | 마침표가 `약어`에 포함된다.        |
| `AT&T`        | 기호가 `고유명사의 일부`이다.       |
| `$45.55`      | 통화 기호와 소수점이 `금액을 표현`한다. |
| `25/03/12`    | 슬래시가` 날짜의 구성 요`소를 구분한다. |
| `123,456,789` | 쉼표가 `자릿수를 구`분한다.        |

특히 마침표는 문장의 끝뿐 아니라 숫자, 약어, 주소 등에도 나타난다. 따라서 마침표만 기준으로 문자열을 나누면 문장 경계를 잘못 판단할 수 있다.

### 2.3. 축약형과 다단어 표현

축약형 및 접어(clitic)

- `I'm` → `I`, `'m`
- `doesn't` → `does`, `n't`

여러 단어로 이루어진 표현

- `New York`: 여러 단어로 이루어진 지명이다.
- `rock 'n' roll`: 여러 요소로 이루어진 관용적인 표현이다.

## 3. 영어 토큰화 예시: Penn Treebank

대표적인 영어 토큰화 예시: **Penn Treebank**
NLTK의 `TreebankWordTokenizer`는 Penn Treebank의 규칙을 바탕으로 정규표현식을 사용해 토큰화한다.

### 3.1. 주요 처리 규칙

| 입력           | 분할 예시         | 설명                               |
| ------------ | ------------- | -------------------------------- |
| `I'm`        | `I`, `'m`     | 축약된 형태를 분리한다.                    |
| `doesn't`    | `does`, `n't` | 부정 축약형을 분리한다.                    |
| `don't`      | `do`, `n't`   | 아포스트로피(`'`) 위치만 기준으로 나누는 것은 아니다. |
| `well-known` | `well-known`  | 일반적인 단일 하이픈(`-`) 연결어를 유지한다.      |
| `Hello!`     | `Hello`, `!`  | 구두점을 별도 토큰으로 분리한다.               |

### 3.2. NLTK 실습

1. NLTK 설치
	```bash
	python -m pip install nltk
	```
2. 실습 예제 코드
	```python
	from nltk.tokenize import TreebankWordTokenizer
	
	tokenizer = TreebankWordTokenizer()
	text = "I'm a well-known developer who doesn't give up."
	
	tokens = tokenizer.tokenize(text)
	print(tokens)
	```

## 4. 한글 토큰화의 특성과 어려움

한글에서는 구두점 처리와 함께 조사와 어미의 결합, 어절과 단어의 차이를 고려해야 한다.

### 4.1. 어절과 단어의 차이

**어절**: 문장을 구성하고 있는 각각의 도막이자, 띄어쓰기를 하는 단위.

```text
'철수가 학교에 갔다'
-> '철수가', '학교에', '갔다'
```

**단어**: 자립적으로(혼자서) 쓸 수 있는 말의 단위.

```text
'철수가 학교에 갔다'
-> '철수', '가', '학교', '에', '갔다'
```

### 4.2. 교착어와 형태소

**교착어**: 어근(실질 형태소)이나 어간에 고유한 의미나 문법적 기능을 가진 접사(조사, 어미, 접두사, 접미사 등)가 차례대로 붙어서(교착, 첨가되어) 단어를 이루고 문법적 관계를 나타내는 언어

e.g. 잡히셨겠더라 = `잡-` + `-히-` + `-(으)시-`, `-겠-`, `-더라`

<table>
  <tr>
    <td rowspan="2">어근</td>
    <td colspan="5">접사</td>
  </tr>
  <tr>
    <td>파생 접사</td>
    <td colspan="4">굴절 접사</td>
  </tr>
  <tr>
    <td>잡-</td>
    <td>-히-</td>
    <td>-(으)시-</td>
    <td>-었-</td>
    <td>-겠-</td>
    <td>-더라</td>
  </tr>
  <tr>
    <td colspan="2" rowspan="2">어간</td>
    <td colspan="3">선어말 어미</td>
    <td>어말 어미</td>
  </tr>
  <tr>
    <td colspan="4">어미</td>
  </tr>
</table>

**형태소(morpheme)**: 뜻을 가진 가장 작은 말의 단위. 자립성에 따라 다음과 같이 구분한다.

* **자립 형태소**: 접사, 어미, 조사와 상관없이 자립하여 사용할 수 있는 형태소. 그 자체로 단어가 된다.
	* e.g. 체언(명사, 대명사, 수사), 수식언(관형사, 부사), 감탄사 등
* **의존 형태소**: 다른 형태소와 결합하여 사용되는 형태소
	* e.g. 접사, 어미, 조사, 어간

e.g. 에디가 책을 읽었다

|구분|분석|
|---|---|
|어절|에디가 / 책을 / 읽었다|
|형태소|에디 / 가 / 책 / 을 / 읽 / 었 / 다|
|자립 형태소|에디, 책|
|의존 형태소|가, 을, 읽-, -었-, -다|

### 4.3. 띄어쓰기와 비정형 표현

실제 한국어 데이터에는 띄어쓰기 오류, 구어체, 신조어, 반복 문자 등이 포함될 수 있다. 이러한 표현은 토큰 경계를 판단하거나 사전에서 단어를 찾는 작업을 어렵게 만든다.

|유형|예시|고려 사항|
|---|---|---|
|띄어쓰기 누락|`오늘날씨가좋다`|공백만으로 어절을 구분하기 어렵다.|
|비표준 표기|`안녕하세여`|사전에 등록된 표기와 다를 수 있다.|
|반복 문자|`좋아아아`, `ㅋㅋㅋㅋ`|반복을 줄일지 보존할지 결정해야 한다.|
|신조어·전문 용어|새로운 서비스명이나 기술명|분석기 사전에 없을 수 있다.|

필요에 따라 띄어쓰기 교정, 반복 문자 정규화, 사용자 사전 등록 등을 적용한다.

### 4.4. 목적에 따른 토큰화 방식 선택

|목적|고려할 방식|
|---|---|
|명사 빈도 분석|형태소 분석 후 명사를 추출한다.|
|문법 정보 분석|형태소 분석과 품사 태깅을 사용한다.|
|간단한 어절 통계|공백 기반 토큰화를 사용한다.|
|사전학습 언어 모델 사용|해당 모델에 대응하는 토크나이저를 사용한다.|
|문자 수준의 패턴 분석|문자 또는 자모 단위를 고려한다.|

- 형태소 토큰화: 어휘와 문법 정보를 분리하는 데 유용하다.
- 서브워드 토큰화: 학습된 어휘와 분할 규칙을 사용한다.

## 5. 품사 태깅(Part-of-speech tagging)

### 5.1. 품사 태깅의 개념

**품사 태깅**: 문맥을 고려해 단어나 형태소에 명사, 동사, 조사 등의 품사 정보를 부여하는 작업

같은 표기라도 문맥에 따라 품사가 달라질 수 있다.

| 문장          | 대상  | 품사와 의미             |
| ----------- | --- | ------------------ |
| `못`을 박았다.   | 못   | 명사, 물체를 고정하는 도구    |
| 밥을 `못` 먹었다. | 못   | 부사, 해당 동작을 할 수 없었음 |

### 5.2. 형태소 분석과 품사 태깅의 관계

- **형태소 분석**: 표현을 구성하는 형태소와 그 구조를 분석하는 작업
- **품사 태깅**: 분석 단위에 품사 정보를 부여하는 작업

## 6. 한글 토큰화 실습

### 6.1. KSS(Korean Sentence Splitter)를 이용한 문장 분리

1. KSS 설치
	```bash
	python -m pip install kss
	```
2. KSS 실습
	```python
	from kss import Kss
	
	split_sentences = Kss("split_sentences")
	
	text = "오늘은 날씨가 좋다. 공원에서 책을 읽었다. 내일도 산책할까?"
	sentences = split_sentences(text, num_workers=1)
	
	for index, sentence in enumerate(sentences, start=1):
	    print(f"{index}: {sentence}")
	```

### 6.2. KoNLPy를 이용한 형태소 분석

KoNLPy는 여러 한국어 형태소 분석기를 Python에서 사용할 수 있도록 제공하는 라이브러리이다.

- 대표 예시: Okt(Open Korea Text), Mecab, Komoran, Hannanum, KKma(꼬꼬마)

1. KoNLPy 설치
	```bash
	python -m pip install konlpy
	```
2. Okt 형태소 분석기
	```python
	from konlpy.tag import Okt
	
	okt = Okt()
	text = "열심히 코딩한 당신, 연휴에는 여행을 가봐요"
	
	print("Okt 형태소 분석:", okt.morphs(text))
	print("Okt 품사 태깅:", okt.pos(text))
	print("Okt 명사 추출:", okt.nouns(text))
	```
3. Kkma 형태소 분석
	```python
	from konlpy.tag import Kkma
	
	kkma = Kkma()
	text = "열심히 코딩한 당신, 연휴에는 여행을 가봐요"
	
	print("Kkma 형태소 분석:", kkma.morphs(text))
	print("Kkma 품사 태깅:", kkma.pos(text))
	print("Kkma 명사 추출:", kkma.nouns(text))
	```

---

## 관련 글

- [자연어 처리의 기본 개념]({% post_url AI_AND_DATA/AI/2025-03-25-nlp %})
- [한글 텍스트 자연어 처리 실습 1]({% post_url AI_AND_DATA/AI/2025-03-26-korean-text-nlp-prac %})

## 출처

- GitHub. 「Open Korean Text Processor」. [GitHub](https://github.com/open-korean-text/open-korean-text)
- KoNLPy. 「tag Package」. [KoNLPy](https://konlpy.org/en/latest/api/konlpy.tag/)
- WikiDocs. 「텍스트 전처리(Text preprocessing)」. [WikiDocs](https://wikidocs.net/21694)
- jh2021. 「정규표현식(Regular Expression) with 파이썬」. [Tistory](https://jh2021.tistory.com/8)
- GitHub. 「한국어 소설 텍스트를 위한 자연어처리 라이브러리」. [GitHub](https://github.com/storidient/KoBookNLP)
- NLTK. 「Accessing Text Corpora and Lexical Resources」. [NLTK](https://www.nltk.org/book/ch02.html)
- NLTK. 「nltk.tokenize.treebank module」. [NLTK](https://www.nltk.org/api/nltk.tokenize.treebank.html)
- Hugging Face. 「Tokenization algorithms」. [Hugging Face](https://huggingface.co/docs/transformers/tokenizer_summary)
- GitHub. 「KSS: Korean String processing Suite」. [GitHub](https://github.com/hyunwoongko/kss)

<div class="obsidian-links" style="display: none;">

[\[AI\] 자연어 처리(NLP)란?](2025-03-25-NLP.md)
[\[AI\] 한글 텍스트 자연어 처리 실습](2025-03-26-Korean-Text-NLP-Prac.md)

</div>
