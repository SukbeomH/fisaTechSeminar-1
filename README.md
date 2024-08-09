# 기술 세미나 1회차
First TECH_SEMINAR at Woori FISA 

## RAG 구성을 위한 ElasticSearch 활용 및 ELK 소개

### 0. RAG: Retrieval Augmented Generation 🚀

**RAG (Retrieval-Augmented Generation)**
- 모델이 **학습하지 않은 문서**들을 **벡터화**하여 데이터베이스에 저장하고, 유저의 **입력 벡터와 유사도를 판별**하여 관련된 K개의 데이터를 검색합니다. 
- 모델은 이 **데이터를 참고**하여 출력 결과를 생성합니다. 
- *특징) 모델 자체를 변경하지 않음.*

**Fine-tuning**
- 모델의 일부 레이어(주로 출력에 가까운 레이어)를 고정하지 않고, 새로운 데이터를 사용하여 모델의 웨이트를 조정하는 기법입니다. 
- *특징) 모델의 웨이트를 업데이트하여 성능을 향상시킴.*
- **LoRa (Low-Rank Adaptation)** :: Fine-tuning의 하위 개념
  - 모델의 기존 파라미터를 전부 조정하지 않고, 새로운 저차원 랭크 레이어(A와 B)를 추가하여 이들만 학습시키는 기법입니다. 
  - *특징) 모델의 웨이트에 독립된 웨이트를 추가하여 효율적인 학습을 가능하게 함.*

### 1. ElasticSearch 🔍

> You Know, for Search

- ElasticSearch: 실시간 **분산 검색** 및 **분석 엔진**
  - **Apache Lucene** 기반 (분산 검색 엔진)
  - RESTful API 지원
  - 데이터를 JSON 문서로 저장
  - 벡터 검색, 텍스트 검색, 구조화된 데이터 검색 등 다양한 검색 기능 제공
  - **Tokenizer, Analyzer, Token Filter** 등 다양한 분석기능 제공
  - 데이터를 색인하고 검색하는 기능 제공 (ElasticSearch)
  - 데이터를 분석하고 시각화하는 기능 제공 (Kibana)

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

#### [Link to ELK](Research/ELK.md)

### 2. LangChain 🐦‍⛓️
- LangChain을 통해 Open AI, ElasticSearch 및 다양한 기술을 한 곳에서 활용
- **LangChain**: 다양한 기술을 연결하여 활용할 수 있는 플랫폼
  - **Open AI**: GPT-3, Codex 등 다양한 인공지능 기술 제공
  - **ElasticSearch**: 실시간 분산 검색 및 분석 엔진
  - **기타 기술**: 다양한 기술을 통합하여 활용 가능
- LLM을 활용한 다양한 앱을 손쉽게 만들 수 있는 프레임워크

#### [Link to LangChain](Research/LangChain.md)

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

### Prompt-tuning 🎨 for RAG

RAG는 AI를 통해 도출한 결과의 정확도를 높이고, 동시에 환각과 같은 문제를 억제하기 위한 방법입니다.

- 내부 데이터로 모델을 학습시키는 것이 아닌, **외부 데이터를 검색**하여 활용
  - **외부 데이터**: 검색을 통해 가져온 데이터
  - **내부 데이터**: 모델이 학습한 데이터
- 외부 데이터를 레퍼런스로 활용하기 때문에 모델을 조정할 때, 모델 자체를 Fine-tuning 하지 않고 그 외 요소를 조정하는 것이 효율적입니다.
  - **Fine-tuning**: 모델의 가중치, 파라미터 등을 조정하여 성능을 향상시키는 방식
  - 특정 데이터에 종속된 모델을 만들지 않고, **범용적인 모델**을 활용할 수 있습니다.
- **Prompt-tuning**: 프롬프트를 튜닝하여 모델의 성능을 향상시키는 방식
  - **프롬프트**: 모델에 입력되는 문장의 형태를 결정하는 요소
  - 프롬프트를 튜닝하여 모델이 원하는 결과를 도출하도록 유도할 수 있습니다.
- 범용 모델을 그대로 사용하면서도 프롬프트를 조정하여 **특정 데이터에 맞게 활용**할 수 있습니다.

### Prompt-tuning의 장점

- **범용 모델 활용**: 특정 데이터에 종속되지 않고, 범용적인 모델을 활용할 수 있습니다.
- **프롬프트 조정**: 프롬프트를 튜닝하여 모델의 성능을 향상시킬 수 있습니다.
- **데이터 종속성 감소**: 외부 데이터를 활용하기 때문에 내부 데이터에 종속되지 않습니다.

### Prompt-tuning의 단점

- **프롬프트 튜닝의 어려움**: 프롬프트를 튜닝하는 것이 어려울 수 있습니다.
- **모델의 이해도**: 모델의 이해도가 높아야 프롬프트를 효율적으로 튜닝할 수 있습니다.
- **시간과 비용**: 프롬프트 튜닝에 시간과 비용이 소요될 수 있습니다.
- **모호성**: 프롬프트의 모호성으로 인해 원하는 결과를 도출하기 어려울 수 있습니다.

### 정량적인 평가와 효율적인 프롬프트 튜닝

[Prompt-tuning](Research/Prompt-tuning.md)

---

# Reference 

- RAG 개요 
  - [Understanding retrieval augmented generation (RAG) | Elastic Snackable Series](https://youtu.be/OS4ZefUPAks)

- 사용방식 of RAG with ElasticSearch and langchain
  - [Building LLM applications with LangChain: ElasticON AI](https://youtu.be/V05ieC9o0jQ)

- 샘플코드 및 설명 (코랩)
  - [RAG with OpenAI and Langchain - part 1 - Elastic Daily Bytes S05E12](https://www.youtube.com/live/XgXtSdNFM6s)
  - [langchain-elasticsearch-RAG Github](https://github.com/ashishtiwari1993/langchain-elasticsearch-RAG)

- ELK 측 공식문서 about RAG
  - [What is retrieval augmented generation](https://www.elastic.co/kr/what-is/retrieval-augmented-generation)

- 벡터 DB가 필요한 이유
  - [Superb_AI Vector Store](https://blog-ko.superb-ai.com/vector-store/)

- 백터가 대체 뭔가? (딥러닝개요)
  - [Neural networks Series - 3Blue1Brown](https://youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi)

- DSPy와 TextGrad
  - [DSPy](https://github.com/stanfordnlp/dspy)
  - [TextGrad](https://github.com/zou-group/textgrad)
  - [TextGrad vs DSPy](https://medium.com/@jelkhoury880/textgrad-vs-dspy-revolutionizing-ai-system-optimization-through-automatic-text-based-58f8ee776447)
  - [2406.07496 TextGrad](https://arxiv.org/pdf/2406.07496)