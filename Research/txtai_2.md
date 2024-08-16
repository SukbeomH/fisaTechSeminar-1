### 검색은 많은 애플리케이션의 기반입니다.
데이터가 쌓이기 시작하면 사용자는 이를 찾아내고 싶어합니다. 이는 인터넷의 기초이자 계속해서 해결되지 않는 성장하는 도전 과제입니다.

자연어 처리(NLP) 분야는 새로운 발전과 함께 빠르게 진화하고 있습니다. 대규모 일반 언어 모델은 놀라운 기능을 추가할 수 있는 흥미로운 새 능력을 제공합니다. 혁신은 매주 새로운 모델과 발전이 등장하면서 계속되고 있습니다.

**txtai**는 모든 애플리케이션에서 자연어 이해(NLU) 기반 검색을 가능하게 하는 올인원 임베딩 데이터베이스입니다.

### **txtai 소개**
**txtai**는 의미 기반 검색, LLM(대형 언어 모델) 오케스트레이션 및 언어 모델 워크플로우를 위한 올인원 임베딩 데이터베이스입니다.

임베딩 데이터베이스는 벡터 인덱스(희소 및 밀집), 그래프 네트워크, 관계형 데이터베이스의 결합체입니다. 이를 통해 SQL을 사용한 벡터 검색, 주제 모델링, 검색 증강 생성(RAG) 등을 수행할 수 있습니다.

임베딩 데이터베이스는 독립적으로 사용되거나 LLM 프롬프트를 위한 강력한 지식 소스로 작동할 수 있습니다.

다음은 주요 기능 요약입니다:

- 🔎 **벡터 검색**: SQL, 객체 스토리지, 주제 모델링, 그래프 분석, 멀티모달 인덱싱을 통한 검색
- 📄 **임베딩 생성**: 텍스트, 문서, 오디오, 이미지, 비디오에 대한 임베딩 생성
- 💡 **파이프라인**: LLM 프롬프트 실행, 질문 응답, 라벨링, 전사, 번역, 요약 등을 지원하는 언어 모델 기반 파이프라인
- ↪️ **워크플로우**: 파이프라인을 결합하여 비즈니스 로직을 집계하는 워크플로우. txtai 프로세스는 단순한 마이크로서비스 또는 다중 모델 워크플로우가 될 수 있음
- ⚙️ **개발 환경**: Python 또는 YAML로 빌드 가능. JavaScript, Java, Rust, Go용 API 바인딩 제공
- ☁️ **배포**: 로컬에서 실행하거나 컨테이너 오케스트레이션으로 확장 가능

**txtai**는 Python 3.8+, Hugging Face Transformers, Sentence Transformers, FastAPI로 빌드되었습니다. txtai는 Apache 2.0 라이선스 하에 오픈소스로 제공됩니다.

### **txtai 설치 및 실행**
**txtai**는 pip 또는 Docker를 통해 설치할 수 있습니다. 다음은 pip를 통해 설치하는 방법입니다.

```bash
pip install txtai
```

### **의미 기반 검색**
임베딩 데이터베이스는 의미 기반 검색을 제공하는 엔진입니다. 데이터는 임베딩 벡터로 변환되며, 유사한 개념은 유사한 벡터를 생성합니다. 이러한 벡터로 인덱스를 구축하여 동일한 의미를 가지는 결과를 찾을 수 있습니다.

임베딩 데이터베이스의 기본 사용 사례는 의미 기반 검색을 위한 근사 최근접 이웃(ANN) 인덱스 구축입니다. 다음 예제는 텍스트 항목의 소수를 인덱싱하여 의미 기반 검색의 가치를 보여줍니다.

```python
from txtai import Embeddings

data = [
  "US tops 5 million confirmed virus cases",
  "Canada's last fully intact ice shelf has suddenly collapsed, forming a Manhattan-sized iceberg",
  "Beijing mobilises invasion craft along coast as Taiwan tensions escalate",
  "The National Park Service warns against sacrificing slower friends in a bear attack",
  "Maine man wins $1M from $25 lottery ticket",
  "Make huge profits without work, earn up to $100,000 a day"
]

embeddings = Embeddings(path="sentence-transformers/nli-mpnet-base-v2")
embeddings.index(data)

for query in ("feel good story", "climate change", "public health story", "war", "wildlife", "asia", "lucky", "dishonest junk"):
  uid = embeddings.search(query, 1)[0][0]
  print(f"Query: {query} | Best Match: {data[uid]}")
```

위의 예제에서는 쿼리 텍스트가 데이터에 직접적으로 포함되어 있지 않음에도 불구하고 일치하는 항목을 찾을 수 있음을 보여줍니다. 이것이 토큰 기반 검색보다 트랜스포머 모델이 제공하는 진정한 힘입니다.

### **업데이트 및 삭제**
임베딩에 대한 업데이트와 삭제가 지원됩니다. 업서트(upsert) 연산을 통해 새로운 데이터를 삽입하거나 기존 데이터를 업데이트할 수 있습니다.

```python
# 기존 데이터 업데이트 예제
udata = data.copy()
udata[0] = "See it: baby panda born"
embeddings.upsert([(0, udata[0], None)])

# 데이터 삭제 예제
embeddings.delete([0])
```

### **영속성**
임베딩은 저장소에 저장하고 다시 불러올 수 있습니다.

```python
embeddings.save("index")
embeddings.load("index")
```

### **하이브리드 검색**
밀집 벡터 인덱스는 의미 기반 검색 시스템에서 가장 좋은 옵션이지만, 희소 키워드 인덱스도 여전히 가치가 있습니다. 하이브리드 검색은 밀집 및 희소 벡터 인덱스의 결과를 결합하여 최상의 검색 결과를 제공합니다.

### **컨텐츠 스토리지**
예제에서 다룬 텍스트를 외부 데이터 저장소에서 검색해야 하는 경우, 관련 데이터와 벡터 인덱스를 함께 저장할 수 있습니다.

### **주제 모델링**
의미 그래프를 통해 주제 모델링이 가능합니다. 그래프 인덱스를 사용하면 각 항목에 주제가 할당됩니다.

### **LLM 오케스트레이션**
txtai는 임베딩 데이터베이스뿐만 아니라 LLM 오케스트레이션도 지원합니다. RAG(검색 증강 생성) 파이프라인에서 txtai를 사용하여 문맥을 제공하고 LLM 프롬프트를 구동할 수 있습니다.

### **언어 모델 워크플로우**
언어 모델 워크플로우를 통해 지능형 애플리케이션을 구축할 수 있습니다. 워크플로우는 관계형 데이터베이스의 저장된 절차와 유사하게 임베딩 인스턴스와 함께 실행될 수 있습니다.
