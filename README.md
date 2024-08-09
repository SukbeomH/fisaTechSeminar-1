# 기술 세미나 1회차
First TECH_SEMINAR at Woori FISA 

## RAG 구성을 위한 ElasticSearch 활용 및 ELK 소개

### 1. ElasticSearch 🔍
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

**Tokenizer**
- 역할: Tokenizer는 텍스트를 개별 토큰(단어)으로 분리하는 역할을 합니다.
- 작동 방식: 입력된 텍스트를 특정 기준(예: 공백, 구두점 등)에 따라 분리하여 토큰의 리스트를 생성합니다.
- 예시: "ElasticSearch is powerful"라는 문장이 주어지면, Tokenizer는 이를 ["ElasticSearch", "is", "powerful"]로 분리합니다.

**Analyzer**
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

#### [Link to ELK](ELK.md)

### 2. LangChain 🐦‍⛓️
- LangChain을 통해 Open AI, ElasticSearch 및 다양한 기술을 한 곳에서 활용
- **LangChain**: 다양한 기술을 연결하여 활용할 수 있는 플랫폼
  - **Open AI**: GPT-3, Codex 등 다양한 인공지능 기술 제공
  - **ElasticSearch**: 실시간 분산 검색 및 분석 엔진
  - **기타 기술**: 다양한 기술을 통합하여 활용 가능
- LLM을 활용한 다양한 앱을 손쉽게 만들 수 있는 프레임워크

#### [Link to LangChain](LangChain.md)

## Why RAG? 🔥

- RAG: Retrieval Augmented Generation
  - **Retrieval**: 검색을 통해 정보를 가져오는 과정
  - **Augmented**: 보강된, 확장된
  - **Generation**: 생성, 발생

- Fine-tuning을 통한 성능 향상
  - **Fine-tuning**: 사전 학습된 모델을 특정 데이터에 맞게 재학습하는 과정
  - **성능 향상**: 특정 데이터에 대한 이해도가 높아지고, 정확도가 향상됨

RAG의 경우에는 모델자체를 변경하지 않고, 필요한 데이터를 검색하여 활용함으로써 성능을 향상시킬 수 있습니다.
반면에 Fine-tuning의 경우에는 모델을 변경하여 성능을 향상시키는 방식입니다. (가중치, 파라미터 등 모델에서 변경 가능한 수치를 조정)

---

# Reference 

- RAG 개요 
  - [Understanding retrieval augmented generation (RAG) | Elastic Snackable Series](https://youtu.be/OS4ZefUPAks)

- 사용방식 of RAG with ElasticSearch and langchain
  - [Building LLM applications with LangChain: ElasticON AI](https://youtu.be/V05ieC9o0jQ)

- 샘플코드 및 설명 (코랩)
  - [RAG with OpenAI and Langchain - part 1 - Elastic Daily Bytes S05E12](https://www.youtube.com/live/XgXtSdNFM6s)

- ELK 측 공식문서 about RAG
  - [What is retrieval augmented generation](https://www.elastic.co/kr/what-is/retrieval-augmented-generation)

- 벡터 DB가 필요한 이유
  - [Superb_AI Vector Store](https://blog-ko.superb-ai.com/vector-store/)

- 백터가 대체 뭔가? (딥러닝개요)
  - [Neural networks Series - 3Blue1Brown](https://youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi)


