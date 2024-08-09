## ElasticSearch 🔍
> ElasticSearch is a **distributed**, **RESTful search and analytics engine** capable of solving a growing number of use cases.

- ElasticSearch: 실시간 **분산 검색** 및 **분석 엔진**
  - **Apache Lucene** 기반 (분산 검색 엔진)
  - RESTful API 지원
  - 데이터를 JSON 문서로 저장
  - 벡터 검색, 텍스트 검색, 구조화된 데이터 검색 등 다양한 검색 기능 제공
  - **Tokenizer, Analyzer, Token Filter** 등 다양한 분석기능 제공
  - 데이터를 색인하고 검색하는 기능 제공 (ElasticSearch)
  - 데이터를 분석하고 시각화하는 기능 제공 (Kibana)

> You Know, for Search

- **Tokenizer와 Analyzer**
  - **Tokenizer**: 문장을 단어로 분리하는 역할
  - **Analyzer**: 토큰화된 단어를 분석하는 역할

**Tokenizer** 🪓
- 역할: Tokenizer는 텍스트를 개별 토큰(단어)으로 분리하는 역할을 합니다.
- 작동 방식: 입력된 텍스트를 특정 기준(예: 공백, 구두점 등)에 따라 분리하여 토큰의 리스트를 생성합니다.
- 예시: "ElasticSearch is powerful"라는 문장이 주어지면, Tokenizer는 이를 ["ElasticSearch", "is", "powerful"]로 분리합니다.

**Analyzer** 🧪
- 역할: Analyzer는 Tokenizer와 여러 Token Filter를 조합하여 텍스트를 분석하고 처리하는 역할을 합니다.
- 작동 방식: Analyzer는 먼저 Tokenizer를 사용하여 텍스트를 토큰으로 분리한 후, 추가적인 처리를 위해 여러 Token Filter를 적용합니다. 이 과정에서 토큰의 형태를 변경하거나 불필요한 토큰을 제거할 수 있습니다.
- 예시: "ElasticSearch is powerful"라는 문장이 주어지면, Analyzer는 이를 ["elasticsearch", "powerful"]로 변환할 수 있습니다. 여기서 "is"는 불용어로 제거되고, 나머지 단어는 소문자로 변환되었습니다.

```json
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_tokenizer": {
          "type": "standard"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "my_tokenizer",
          "filter": ["lowercase", "stop"]
        }
      }
    }
  }
}
```

- 이 예제에서 my_tokenizer는 표준 Tokenizer를 사용하고, my_analyzer는 my_tokenizer를 사용하여 토큰을 분리한 후, **소문자 변환(lowercase)과 불용어 제거(stop) 필터**를 적용합니다.

## Logstash 📦
> Logstash is a free and open server-side **data processing pipeline** that **ingests data from multiple sources**, **transforms it**, and then **sends it** to your favorite "stash."

- Logstash: 서버 측 **데이터 처리 파이프라인**
  - 다양한 소스에서 데이터를 **수집**하고, **변환**한 후, **stash**로 전송하는 역할
    - **ElasticSearch**와 **Kibana**와 함께 사용하여 데이터를 수집, 처리, 시각화하는 역할
    - **Beats**와 함께 사용하여 로그 데이터를 수집하고 전송하는 역할
  - 다양한 **Input, Filter, Output** 플러그인을 제공하여 데이터 처리 파이프라인을 구성할 수 있음

**Input Plugin**
- 역할: 데이터를 수집하는 역할
- 예시: **Filebeat**, **Beats**, **JDBC**, **HTTP**, **Kafka** 등
- 설정: 데이터를 수집할 소스와 포맷을 지정
- 예시: Filebeat를 사용하여 로그 파일을 수집하고, JSON 형식으로 변환

**Filter Plugin**
- 역할: 데이터를 변환하거나 필터링하는 역할
- 예시: **Grok**, **Date**, **GeoIP**, **Mutate** 등
- 설정: 데이터를 변환할 방법과 필터링 조건을 지정
- 예시: Grok을 사용하여 로그 메시지를 구문 분석하고 필드로 추출
  - 입력: `2022-01-01 12:00:00 INFO Message`
  - 출력: `timestamp: 2022-01-01 12:00:00`, `level: INFO`, `message: Message`

**Output Plugin**
- 역할: 데이터를 전송하는 역할
- 예시: **ElasticSearch**, **Kibana**, **File**, **TCP**, **UDP** 등
- 설정: 데이터를 전송할 대상과 포맷을 지정
- 예시: ElasticSearch에 데이터를 전송하고, 인덱스와 타입을 지정

## Kibana 📊
> Kibana lets you **visualize** your Elasticsearch data and **navigate** the Elastic Stack, so you can do anything from **tracking query load** to **understanding the way requests flow through your apps**.

- Kibana: ElasticSearch 데이터를 **시각화**하고 **탐색**할 수 있는 도구
  - **ElasticSearch**와 **Logstash**와 함께 사용하여 데이터를 시각화하고 분석하는 역할
  - **Dashboard**, **Visualization**, **Discover**, **Canvas** 등 다양한 기능 제공
  - **Machine Learning**을 통한 이상 징후 탐지, **시계열 분석**, **로그 분석** 등 다양한 분석 기능 제공
  - **ElasticSearch**의 데이터를 시각화하고 대시보드를 생성하여 데이터를 모니터링하는 역할

**Dashboard** 📈
- 역할: 시각화된 데이터를 모아서 **한 화면에 표시**하는 역할 (대시보드)
- 예시: 시계열 차트, 막대 그래프, 테이블 등 다양한 시각화 요소를 조합하여 대시보드 생성

**Visualization** 📉
- 역할: 데이터를 시각화하는 역할 **(요소 요소)**
- 예시: 막대 그래프, 파이 차트, 지도, 테이블 등 다양한 시각화 요소를 생성

**Discover** 🔍
- 역할: ElasticSearch의 데이터를 검색하고 탐색하는 역할
- 예시: 데이터를 검색하고, 필터링하고, 정렬하여 원하는 데이터를 찾는 기능

**Canvas** 🖼️
- 역할: 다양한 데이터를 시각화하여 대시보드를 생성하는 **Low-Code 툴**
- 예시: 이미지, 텍스트, 차트, 테이블 등 다양한 요소를 조합하여 대시보드 생성

## Beats 🎵
> Beats is the platform for single-purpose data shippers. They **ingest data** and **send it** to Elasticsearch.

- Beats: **단일 목적의 데이터 수집기**
  - 다양한 **Input**을 통해 데이터를 수집하고, **Output**을 통해 데이터를 전송하는 역할
  - **Filebeat**, **Metricbeat**, **Packetbeat**, **Heartbeat** 등 다양한 종류의 Beats 제공
  - **ElasticSearch**와 **Logstash**와 함께 사용하여 데이터를 수집하고 전송하는 역할
  - **Lightweight**, **Efficient**, **Scalable**한 특징을 가짐

**Logstash** 와 **Beats**의 차이점
- **Logstash**: 데이터를 수집하고, 변환하고, 전송하는 **데이터 처리 파이프라인**
- **Beats**: 단일 목적의 데이터 수집기로, 데이터를 수집하고 전송하는 역할
  - 단일 목적: **Filebeat** (로그 파일), **Metricbeat** (시스템 및 서비스 메트릭), **Packetbeat** (네트워크 데이터), **Heartbeat** (서비스 상태) 등

**Filebeat** 📁
- 역할: 로그 파일을 수집하고 전송하는 역할
- 설정: 로그 파일의 경로, 포맷, 타임스탬프 등을 지정

**Metricbeat** 📊
- 역할: 시스템 및 서비스 메트릭을 수집하고 전송하는 역할
- 설정: 메트릭을 수집할 대상, 주기, 모듈 등을 지정

**Packetbeat** 📦
- 역할: 네트워크 데이터를 수집하고 전송하는 역할
- 설정: 네트워크 트래픽을 수집할 대상, 프로토콜, 필터 등을 지정

**Heartbeat** 💓
- 역할: 서비스 상태를 모니터링하고 전송하는 역할
- 설정: 서비스 상태를 확인할 대상, 주기, 포트 등을 지정

## Elastic Stack 📚
> The Elastic Stack is a collection of **open-source products** for **search**, **observability**, and **security** use cases.

- Elastic Stack: **검색**, **감시**, **보안** 등 다양한 용도로 사용되는 **오픈 소스 제품 모음**
  - **ElasticSearch**, **Logstash**, **Kibana**, **Beats** 등 다양한 제품으로 구성
  - **검색**: ElasticSearch를 통해 데이터를 색인하고 검색하는 기능 제공
  - **감시**: Kibana를 통해 데이터를 시각화하고 모니터링하는 기능 제공
  - **보안**: Elastic Stack을 통해 데이터를 보호하고 보안을 강화하는 기능 제공
  - **오픈 소스**: Elastic Stack은 오픈 소스로 제공되어 커뮤니티와 협력하여 개발 및 운영 가능

## ElasticSearchStore 💭
> ElasticSearchStore is a **data store** that **provides** a **distributed**, **RESTful search and analytics engine**.

- ElasticSearchStore: **분산**, **RESTful 검색 및 분석 엔진**을 제공하는 **데이터 저장소**
  - **ElasticSearch**를 통해 데이터를 저장하고 검색하는 역할
  - **분산**: 여러 노드에 데이터를 분산하여 저장하고 검색하는 기능 제공
  - **RESTful API**: RESTful API를 통해 데이터를 검색하고 분석하는 기능 제공
  - **검색 및 분석**: 데이터를 색인하고 검색하며, 시각화하여 분석하는 기능 제공

**ElasticSearchStore**와 **ElasticSearch**의 차이점
- **ElasticSearchStore**: 데이터 저장소로서, 데이터를 저장하고 검색하는 역할
- **ElasticSearch**: 실시간 분산 검색 및 분석 엔진으로, 데이터를 색인하고 검색하는 역할
- **ElasticSearchStore**는 **ElasticSearch**를 포함하며, 데이터 저장소로서의 역할을 수행

# Why Use ElasticSearch For Vectorized Token Search? 🤔
> ElasticSearch is a **powerful search engine** that **supports** vectorized token search for **efficient similarity search**.
> By using **dense vector fields** and **vector similarity functions**, 
> ElasticSearch can perform **fast and accurate similarity search** on large-scale data.

- **ElasticSearch**: **효율적인 유사성 검색**을 위한 **벡터화된 토큰 검색**을 지원하는 **강력한 검색 엔진**
  - **밀집 벡터 필드**와 **벡터 유사성 함수**를 사용하여
  - **대규모 데이터**에 대한 **빠르고 정확한 유사성 검색**을 수행할 수 있음
  - **유사성 검색**: 벡터화된 토큰을 사용하여 유사한 문서를 검색하는 기능
  - **밀집 벡터 필드**: 벡터 데이터를 저장하고 검색하는 필드
  - **벡터 유사성 함수**: 벡터 간의 유사성을 계산하는 함수

**ElasticSearch**를 사용하는 이유
- **효율적인 유사성 검색**: 벡터화된 토큰을 사용하여 빠르고 정확한 유사성 검색을 수행
- **대규모 데이터**: 대규모 데이터에 대한 검색 및 분석을 지원
- **밀집 벡터 필드**: 벡터 데이터를 저장하고 검색하는 기능을 제공
- **벡터 유사성 함수**: 벡터 간의 유사성을 계산하는 함수를 제공

## What is vectorized token? 🤨
> A **vectorized token** is a **dense vector representation** of a **token** that **captures** its **semantic meaning** in a **high-dimensional space**.

- **벡터화된 토큰**: **고차원 공간**에서 **토큰의 의미**를 **캡처**하는 **밀집 벡터 표현**
  - **밀집 벡터**: 고차원 공간에서 토큰의 의미를 표현하는 밀집한 벡터
  - **의미**: 토큰의 의미를 수학적으로 표현한 값
  - **고차원 공간**: 다차원 공간에서 토큰의 의미를 표현

## Vectorized Token means something in Large Language Models (LLM)? 🤖
**LLM** 모델을 사용하여 **토큰을 벡터화**하는 이유
- **의미적 유사성**: 토큰의 의미를 벡터로 표현하여 유사한 토큰을 검색
- **고차원 공간**: 다차원 공간에서 토큰의 의미를 표현하여 정확한 유사성 검색
- **밀집 벡터**: 밀집한 벡터로 토큰의 의미를 효율적으로 표현
- **정확성**: 정확한 유사성 검색을 위해 벡터화된 토큰을 사용
- **효율성**: 벡터화된 토큰을 사용하여 빠르고 효율적인 검색을 수행
- **유연성**: 다양한 유사성 검색 알고리즘을 적용하여 다양한 유사성 검색을 수행
- **확장성**: 대규모 데이터에 대한 유사성 검색을 지원

# Retrieval Augmented Generation (RAG) 🚀
> **RAG** is a **hybrid model** that combines **retrieval** and **generation** to **improve** the **performance** of **language understanding** tasks.

- **RAG**: **검색**과 **생성**을 결합하여 **언어 이해** 작업의 **성능**을 **향상**시키는 **하이브리드 모델**
  - **검색**: 검색을 통해 관련 문서를 검색하여 정보를 획득
  - **생성**: 생성을 통해 문장을 생성하여 정보를 제공
  - **언어 이해**: 언어 이해 작업을 수행하여 자연어 처리 성능을 향상
  - **성능**: 성능을 향상시키기 위해 검색과 생성을 결합

## 1. Retrieval 📚
> **Retrieval** is the process of **finding** and **extracting** relevant information from a **large collection** of data.
> By using **retrieval models** and **retrieval algorithms**, 
> we can **retrieve** information that is **relevant** to a given query.

- **검색**: **대량의 데이터**에서 **관련 정보**를 **찾아내는** 과정
  - **검색 모델**과 **검색 알고리즘**을 사용하여
  - **주어진 쿼리**에 **관련된** 정보를 **검색**할 수 있음
  - **정보 검색**: 검색을 통해 관련 정보를 찾아내는 과정
  - **검색 모델**: 정보 검색을 위한 모델
  - **검색 알고리즘**: 정보 검색을 위한 알고리즘

이 과정에서 **ElasticSearch**를 사용하여 **대량의 데이터**에서 **정보를 검색**하고 **추출**할 수 있음

## 2. Generation 🧠
> **Generation** is the process of **creating** new content based on **existing data** or **knowledge**.
> By using **generation models** and **generation algorithms**,
> we can **generate** new content that is **relevant** and **informative**.

- **생성**: **기존 데이터**나 **지식**을 기반으로 **새로운 콘텐츠**를 **생성**하는 과정
  - **생성 모델**과 **생성 알고리즘**을 사용하여
  - **관련**하고 **정보성** 있는 **새로운 콘텐츠**를 **생성**할 수 있음
  - **정보 생성**: 생성을 통해 새로운 콘텐츠를 생성하는 과정
  - **생성 모델**: 콘텐츠 생성을 위한 모델
  - **생성 알고리즘**: 콘텐츠 생성을 위한 알고리즘

이 과정에서 **GPT, Llama, Gemma**와 같은 **언어 생성 모델**을 사용하여 **새로운 콘텐츠**를 **생성**할 수 있음

## 3. Augmentation 🚀
> **RAG** model combines **retrieval** and **generation** to **improve** the **performance** of **language understanding** tasks.
> By using **retrieval** to **find** relevant information and **generation** to **create** new content,
> RAG can **answer questions**, **generate text**, and **perform language tasks** more **effectively**.

- **RAG** 모델은 **검색**과 **생성**을 결합하여 **언어 이해** 작업의 **성능**을 **향상**시킴
  - **검색**을 사용하여 **관련 정보**를 **찾고**, **생성**을 사용하여 **새로운 콘텐츠**를 **생성**
  - **질문에 답변**하고, **텍스트를 생성**하며, **언어 작업**을 **효과적**으로 수행
  - **언어 이해**: 언어 이해 작업을 수행하여 자연어 처리 성능을 향상
  - **성능**: 성능을 향상시키기 위해 검색과 생성을 결합
