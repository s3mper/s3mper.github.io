---
layout: post
comments: true

title: "한국어 텍스트 토큰화의 개념과 실습"
excerpt: "토큰화의 기준과 한국어의 특성을 이해, NLTK·KSS·KoNLPy 실습"

date: 2025-03-16
last_modified_at: 2026-11-16

categories: [AI]
tags: [TIL, NLP, AI, TOKENIZATION]
---

<!-- markdownlint-disable MD004 MD007 MD010 MD025 MD033 -->

# 토큰화(Tokenization)

---

## 목차

1. [](#1-토큰화-개념)
    1. [](#11-말뭉치corpus와-토큰token)
    1. [](#12-토큰의-단위)
1. [](#2-토큰화-기준과-고려-사항)
    1. [](#21-분석-목적과-모델에-따른-기준)
    1. [](#22-구두점과-특수-문자)
    1. [](#23-축약형과-다단어-표현)
1. [](#3-영어-토큰화-예시-penn-treebank)
    1. [](#31-주요-처리-규칙)
    1. [](#32-nltk-실습)
1. [](#4-한글-토큰화의-특성과-어려움)
    1. [](#41-어절과-단어의-차이)
    1. [](#42-교착어와-형태소)
    1. [](#43-띠어쓰기와-비정형-표현)
    1. [](#44-목적에-따른-토큰화-방식-선택)
1. [](#5-품사-태깅part-of-speech-tagging)
    1. [](#51-품사-태깅의-개념)
    1. [](#52-형태소-분석과-품사-태깅의-관계)
1. [](#6-한글-토큰화-실습)
    1. [](#61-kss를-이용한-문장-분리)
    1. [](#62-konlpy를-이용한-형태소-분석)
1. [](#관련-글)
1. [](#출처)
---

## 1. 토큰화 개념

**토큰화(tokenization)**: 주어진 코퍼스(corpus)에서 정해진 기준에 따라 토큰(token) 단위로 나누는 작업이다.

### 1.1. 말뭉치(corpus)와 토큰(token)

- **말뭉치(corpus)**: 언어를 분석하거나 자연어 처리 모델을 학습, 평가하기 위해 모아 놓은 언어 자료의 집합이다.
- **토큰(token)**: 텍스트를 처리하기 위해 나눈 개별 단위이다.

### 1.2. 토큰의 단위

토큰의 단위는 분석 목적과 모델에 따라 달라진다. 따라서 토큰화를 수행하기 전에 어떤 단위로 텍스트를 처리할 것인지 결정해야 한다.

토큰의 단위는 문장, 어절, 단어, 형태소, 문자, 서브워드(subword) 등이 있다.

| 단위 | 설명 | 예시 |
| --- | --- | --- |
| 문장 | 문장의 경계를 기준으로 구분한다. | `오늘은 맑다.`, `산책을 한다.` |
| 어절 | 띄어쓰기를 기준으로 구분한다. | `에디가`, `책을`, `읽었다` |
| 단어 | 언어의 단어 경계와 토큰화 규칙에 따라 구분한다. | 영어 `I read books` → `I`, `read`, `books` |
| 형태소 | 뜻이나 문법적 기능을 가진 최소 단위로 구분한다. | `책을` → `책`, `을` |
| 문자 | 문자 단위로 구분한다. | `사과` → `사`, `과` |
| 서브워드(subword) | 학습된 어휘와 분할 규칙에 따라 단어 내부를 나눈다. | `playing` → `play`, `ing`처럼 분할할 수 있다. |

서브워드의 실제 분할 결과는 토크나이저의 어휘와 알고리즘에 따라 달라진다.

- 대표 서브워드 토큰화 알고리즘: BPE, WordPiece, Unigram

## 2. 토큰화 기준과 고려 사항

토큰화 기준을 정한다는 것은 무엇을 하나로 유지하고 어디에서 나눌 것인지 결정한다는 의미이다. 언어의 특성, 분석 목적, 입력 데이터의 형태, 사용할 모델을 함께 고려한다.

| 고려 요소 | 판단할 내용 | 예시 |
| --- | --- | --- |
| 분석 목적 | 어떤 정보를 보존해야 하는가? | 감성 분석에서는 느낌표나 이모지가 유용할 수 있다. |
| 모델과의 호환성 | 모델이 어떤 토큰화 방식으로 학습되었는가? | 사전학습 모델에 대응하는 토크나이저를 사용한다. |
| 언어의 특성 | 공백만으로 필요한 단위를 얻을 수 있는가? | `책을`을 `책`과 `을`로 나눌지 결정한다. |
| 특수 표현 | 숫자와 기호를 하나의 표현으로 유지할 것인가? | `3.14`, `AT&T`, 이메일 주소를 처리한다. |
| 복합 표현 | 여러 단어를 하나의 분석 단위로 묶을 것인가? | `New York`을 지명 하나로 취급한다. |

### 2.1. 분석 목적과 모델에 따른 기준

같은 문장도 분석 목적에 따라 필요한 정보가 달라진다.

### 2.2. 구두점과 특수 문자

구두점이나 특수 문자는 문장의 구조와 의미를 나타낼 수 있다. 따라서 분석 목적을 고려하지 않고 단순 제거해서는 안 된다.

- 구두점(punctuation): 마침표(`.`), 쉼표(`,`), 물음표(`?`), 세미콜론(`;`), 느낌표(`!`) 등과 같은 기호

| 표현 | 고려 사항 |
| --- | --- |
| `오늘은 맑다.` | 마침표가 문장의 끝을 나타낸다. |
| `3.14` | 마침표가 소수점으로 사용된다. |
| `m.p.h.` | 마침표가 약어에 포함된다. |
| `AT&T` | 기호가 고유명사의 일부이다. |
| `$45.55` | 통화 기호와 소수점이 금액을 표현한다. |
| `25/03/12` | 슬래시가 날짜의 구성 요소를 구분한다. |
| `123,456,789` | 쉼표가 자릿수를 구분한다. |

특히 마침표는 문장의 끝뿐 아니라 숫자, 약어, 주소 등에도 나타난다. 따라서 마침표만 기준으로 문자열을 나누면 문장 경계를 잘못 판단할 수 있다.

### 2.3. 축약형과 다단어 표현

* 접어(clitic): 단어가 줄임말로 쓰일 때 생기는 형태 e.g. I'm -> m
* 하나의 단어이지만 중간에 띄어쓰기가 존재하는 경우: New York, rock 'n' roll

## 3. 영어 토큰화 예시: Penn Treebank

모든 언어와 작업에 공통으로 적용되는 단일 표준 토큰화 방법은 없다.

영어의 대표적인 예가 **Penn Treebank** 방식이다. NLTK의 `TreebankWordTokenizer`는 Penn Treebank의 규칙을 바탕으로 정규표현식을 사용해 토큰화한다. 축약형을 분리하고, 여러 구두점을 별도 토큰으로 처리한다.

### 3.1. 주요 처리 규칙

| 입력 | 분할 예시 | 설명 |
| --- | --- | --- |
| `I'm` | `I`, `'m` | 축약된 형태를 분리한다. |
| `doesn't` | `does`, `n't` | 부정 축약형을 분리한다. |
| `don't` | `do`, `n't` | 아포스트로피 위치만 기준으로 나누는 것은 아니다. |
| `well-known` | `well-known` | 일반적인 단일 하이픈 연결어를 유지한다. |
| `Hello!` | `Hello`, `!` | 구두점을 별도 토큰으로 분리한다. |

### 3.2. NLTK 실습

```python
from nltk.tokenize import TreebankWordTokenizer

tokenizer = TreebankWordTokenizer()
text = "I'm a well-known developer who doesn't give up."

tokens = tokenizer.tokenize(text)
print(tokens)
```

## 4. 한글 토큰화의 특성과 어려움

영어 토큰화에서는 축약형과 구두점 처리 규칙을 확인할 수 있다. 한글에서는 여기에 더해 조사와 어미의 결합, 어절과 단어의 차이를 고려해야 한다.

### 4.1. 어절과 단어의 차이

어절: 띄어쓰기 단위가 되는 단위  
어절 토큰화는 한국어 NLP에서 지양되고 있다. 한국어는 교착어이어서 어절 토큰화와 단어 토큰화는 같지 않기 때문이다.

### 4.2. 교착어와 형태소

교착어: 조사, 어미 등을 붙여서 말을 만드는 언어  

한국어 토큰화에서는 형태소(morpheme)란 개념을 반드시 이해해야 한다.  
형태소(morpheme): 뜻을 가진 가장 작은 말의 단위

* 자립 형태소: 접사, 어미, 조사와 상관없이 자립하여 사용할 수 있는 형태소. 그 자체로 단어가 된다. e.g. 체언(명사, 대명사, 수사), 수식언(관형사, 부사), 감탄사 등
* 의존 형태소: 다른 형태소와 결합하여 사용되는 형태소 e.g. 접사, 어미, 조사, 어간

e.g. 에디가 책을 읽었다  
어절 토큰화 -> '에디가', '책을', '읽었다'
형태소 단위로 분해  

* 자립 형태소: 에디, 책
* 의존 형태소: -가, -을, 읽-, -었-, -다

-> 한국어에서는 형태소 토큰화를 수행해야 한다.

### 4.3. 띠어쓰기와 비정형 표현

한국어는 영어권 언어와 비교해 띄어쓰기가 어렵고 잘 지켜지지 않는 경향이 있다. 그 이유는 여러 견해가 있으나, 가장 기본적인 견해는 한국어의 경우 띄어쓰기가 지켜지지 않아도 글을 쉽게 이해할 수 있는 언어라는 점이다.

### 4.4. 목적에 따른 토큰화 방식 선택

## 5. 품사 태깅(Part-of-speech tagging)

단어는 표기는 같지만 품사에 따라서 단어의 의미가 달라지기도 한다.
e.g. 못(명사): 망치를 사용해 목재 따위를 고정하는 물건  
못(부사): '먹다', '하다' 등과 같은 동작 동사를 할 수 없다는 의미를 나타냄  
품사 태깅(part-of-speech tagging): 단어 토큰화 과정에서 각 단어가 어떤 품사로 쓰였는지를 구분하는 작업

### 5.1. 품사 태깅의 개념

### 5.2. 형태소 분석과 품사 태깅의 관계

## 6. 한글 토큰화 실습

### 6.1. KSS를 이용한 문장 분리

한국어의 경우 KSS(Korean Sentence Splitter)를 추천함  
`pip install kss`  
KSS를 통해서 문장 토큰화 진행하기

```python
import kss
text = '~'
result = kss.split_sentences(text)
```

### 6.2. KoNLPy를 이용한 형태소 분석

KoNLPy를 통해서 사용할 수 있는 형태소 분석기: Okt(Open Korea Text), Mecab, Komoran, Hannanum, KKma(꼬꼬마)

1. okt 형태소 분석기

```python
from konlpy.tag import Okt
okt = Okt()
kkma = Kkma()
print('OKT 형태소 분석 :',okt.morphs("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))
print('OKT 품사 태깅 :',okt.pos("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))
print('OKT 명사 추출 :',okt.nouns("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))
```

1) morphs: 형태소 추출  
2) post: 품사 태깅(Part-of-speech tagging)
3) nouns: 명사 추출  

1. KKma(꼬꼬마) 형태소 분석기

```python
from konlpy.tag import Kkma
print('꼬꼬마 형태소 분석 :',kkma.morphs("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))
print('꼬꼬마 품사 태깅 :',kkma.pos("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))
print('꼬꼬마 명사 추출 :',kkma.nouns("열심히 코딩한 당신, 연휴에는 여행을 가봐요"))
```

---
각 형태소 분석기는 성능과 결과가 다르게 나오기 때문에, 형태소 분석기의 선택은 사용하고자 하는 필요 용도에 어떤 형태소 분석기가 가장 적절한지를 판단하고 사용해야 함.

## 관련 글

- [자연어 처리의 기본 개념]({% post_url AI_AND_DATA/AI/2025-03-25-NLP %})
- [한글 텍스트 자연어 처리 실습 1]({% post_url AI_AND_DATA/AI/2025-03-26-Korean-Text-NLP-Prac %})

## 출처

- GitHub. 「okt github」. [GitHub](https://github.com/open-korean-text/open-korean-text)
- KoNLPy. 「KoNLPy tag Package」. [KoNLPy](https://konlpy.org/en/latest/api/konlpy.tag/)
- WikiDocs. 「텍스트 전처리(Text preprocessing)」. [WikiDocs](https://wikidocs.net/21694)
- jh2021. 「정규표현식(Regular Expression) with 파이썬」. [Tistory](https://jh2021.tistory.com/8)
- GitHub. 「한국어 소설 텍스트를 위한 자연어처리 라이브러리」. [GitHub](https://github.com/storidient/KoBookNLP)
- NLTK. [Accessing Text Corpora and Lexical Resources](https://www.nltk.org/book/ch02.html).
- NLTK. [nltk.tokenize.treebank module](https://www.nltk.org/api/nltk.tokenize.treebank.html).
- Hugging Face. [Tokenization algorithms](https://huggingface.co/docs/transformers/tokenizer_summary).
- KoNLPy. [tag Package](https://konlpy.org/en/latest/api/konlpy.tag/).
- Open Korean Text. [공식 GitHub 저장소](https://github.com/open-korean-text/open-korean-text).

<div class="obsidian-links" style="display: none;">

[\[AI\] 자연어 처리(NLP)란?](2025-03-25-NLP.md)
[\[AI\] 한글 텍스트 자연어 처리 실습](2025-03-26-Korean-Text-NLP-Prac.md)

</div>
