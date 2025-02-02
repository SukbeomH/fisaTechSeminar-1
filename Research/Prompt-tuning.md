## Prompt Engineering 

# Quantitative Prompt Engineering

모호하고 복잡한 프롬프트를 만들어야 할 때, 어떻게 프롬프트를 만들어야 할까?
어떤 단어를 사용하고 어떤 순서로 사용해야 할까?
언어적 지식과 느낌을 조합하는 것이 중요한가? 혹은 경험적인 방법이 더 중요한가?

## 신경망에 대한 비유

우리가 신경망을 구축할 때, 직접 손으로 튜닝한 실수(float) 리스트에 대해 수동으로 for-loop을 작성하지는 않습니다. 

- 대신, PyTorch와 같은 프레임워크를 사용하여 선언적인 레이어(예: Convolution 또는 Dropout)를 구성하고, 
- 그런 다음 옵티마이저(예: SGD 또는 Adam)를 사용하여 네트워크의 파라미터를 학습합니다.

> 마찬가지로 프롬프트 또한 정량적이고 기계적인 방식으로 조정할 수 있어야 합니다.
> 재현 가능하고, 효율적인 방식으로 프롬프트를 조정할 수 있어야 합니다.
> etc...

## DSPy :: Programming —not prompting— Foundation Models
"Declarative Self-improving Language Programs"
[DSPy is a framework for algorithmically optimizing LM prompts and weights](https://github.com/stanfordnlp/dspy)

> DSPy는 **알고리즘적으로** 언어 모델(LM)의 프롬프트와 가중치를 최적화하는 프레임워크

### 기존 Prompt 제작 방법

1. 문제를 **단계별로 나누기**
2. 각 단계가 독립적으로 **잘 작동할 때까지 LM 프롬프트 조정**하기 
3. 각 단계가 **잘 어우러지도록** 조정하기
4. 각 단계를 조정하기 위해 **합성 예제를 생성**하기
5. 비용 절감을 위해 이러한 예제를 사용하여 작은 LM들을 **미세 조정**하기. 

- **문제점**: 파이프라인, LM, 또는 데이터가 변경될 때마다 **모든 프롬프트(또는 미세 조정 단계)를 다시 조정해야 할 수 있음**

### DSPy의 개선점

더 체계적이고 강력한 프롬프트를 만들기 위해 DSPy는 두 가지를 수행합니다. 

- 첫째, **프로그램의 흐름(모듈)**과 각 단계의 **매개변수(LM 프롬프트와 가중치)**를 **분리**합니다. 
- 둘째, DSPy는 **새로운 최적화 도구**를 도입하는데, 
  - 이는 사용자가 극대화하고자 하는 지표에 따라 프롬프트나 LM 호출의 **가중치를 조정할 수 있는 LM 기반 알고리즘**입니다.
    - 마치 모델의 가중치를 조정하는 것처럼, 이 도구는 LM의 프롬프트와 가중치를 조정합니다. (Like Fine-tuning)

DSPy는 **더 높은 품질을 제공**하거나 **특정 실패 패턴을 피할 수 있게** 해줍니다.
  - DSPy 최적화 도구는 *동일한 프로그램을 각각의 LM에 대해* 다양한 지시사항, few-shot 프롬프트, 또는 가중치 업데이트(미세 조정)로 **"컴파일"**합니다. 
  - 보다 정량적이고 효율적인 방식으로 프롬프트를 조정할 수 있다는 것을 의미합니다.

### DSPy 특징

> DSPy는 문자열 기반의 프롬프트 트릭을 대체하는 올바른 범용 모듈(예: ChainOfThought, ReAct 등)을 제공합니다. 

- DSPy는 프로그램의 파라미터를 업데이트하는 알고리즘인 **범용 옵티마이저(BootstrapFewShotWithRandomSearch 또는 BayesianSignatureOptimizer)를 제공**합니다. 
  - 프롬프트 해킹과 일회성 합성 데이터 생성기를 대체 가능합니다.
    - **프롬프트 해킹**: 특정 프롬프트를 사용하여 모델을 조정하는 것
    - **일회성 합성 데이터 생성기**: 일회성 데이터를 생성하여 모델을 조정하는 것

**코드를 수정**하거나, **데이터를 변경**하거나, **어설션을 추가**하거나, **메트릭을 조정**할 때마다 프로그램을 다시 컴파일 가능합니다.

- **DSPy 옵티마이저는 무엇을 조정하나요?**
  - 각 옵티마이저는 다르지만, 모두 프로그램의 프롬프트나 LM(언어 모델) 가중치를 업데이트하여 메트릭을 최대화 합니다. 
    - **데이터를 검사**하고, 
    - 프로그램을 통해 **경로를 시뮬레이션하여 각 단계의 좋은/나쁜 예제를 생성**하며, 
    - 과거 결과를 바탕으로 각 단계에 대한 지시 사항을 제안하거나 개선하고, 
    - 자체 생성한 예제에 대해 **LM의 가중치를 미세 조정**하거나, 
    - 이러한 **여러 방법을 결합**하여 품질을 개선하거나 비용을 절감합니다. 
  - 프롬프트 엔지니어링, "합성 데이터" 생성, 또는 자기 개선을 위해 수동으로 거치는 대부분의 단계를 DSPy 옵티마이저가 일반화하여 임의의 LM 프로그램에 작용할 수 있게 합니다.

## TextGrad :: Automatic "Differentiation" via Text

[An autograd engine for textual gradients!](https://github.com/zou-group/textgrad)

**TextGrad**는 **텍스트를 통한 자동 미분**을 제공하는 **프레임워크**

> 대형 언어 모델(LLM)이 제공하는 텍스트 피드백을 통해 **역전파(backpropagation)를 수행**하며, 이를 통해 경사를 계산하고 모델을 최적화합니다.

> **TextGrad**는 사용자가 자신의 **손실 함수를 정의**하고 텍스트 피드백을 사용하여 이를 최적화할 수 있는 간단하고 직관적인 API를 제공합니다. 
> 이 API는 PyTorch API와 유사하게 설계되어 있어, 사용자가 자신의 사용 사례에 쉽게 적용할 수 있습니다.
> 간단히 **TextGrad**는 텍스트 기반의 자동 미분 엔진으로, **텍스트 피드백을 활용해 모델을 최적화**하는 새로운 방법을 제공합니다.

### TextGrad의 특징

**텍스트 기반의 "미분" 구현 :: 손실 함수의 기울기를 계산하는 방식**
- 이는 대형 언어 모델(LLM)의 능력을 활용해, 모델이 텍스트 기반의 피드백을 통해 스스로 개선할 수 있도록 돕습니다.

**역전파(backpropagation)를 통한 최적화**
- 역전파 알고리즘을 텍스트 피드백에 적용하여, 피드백을 바탕으로 모델의 파라미터를 업데이트합니다. 
- 이 과정에서 피드백은 **경사의 역할**을 하며, 모델의 학습을 유도합니다.
  - *경사: 손실 함수의 기울기를 나타내는 값*

**사용자 정의 손실 함수**
- 사용자가 모델 학습 과정에서 다양한 목표를 설정하고, 그 목표를 향해 모델을 효과적으로 조정할 수 있게 해줍니다.

**직관적이고 간단한 API**
- PyTorch와 유사한 구조로 설계되어 있어, PyTorch에 익숙한 사용자들이 쉽게 적응할 수 있습니다. 
- 이 API를 통해 사용자는 손실 함수를 정의하고, 텍스트 피드백을 받아 모델을 최적화하는 전체 과정을 간단하게 구현할 수 있습니다.

**실험 및 연구에 적합**
- 새로운 모델을 실험하고 텍스트 기반의 최적화 방법을 탐구하는 데 유용합니다. 
- 다양한 NLP 작업에 쉽게 적용할 수 있으며, 텍스트 피드백을 기반으로 모델 성능을 개선하는 새로운 접근 방식을 제시합니다.

### 언어모델은 파인튜닝을 하지 않더라도 InContextLearning 을 이용해서 결과를 향상시킬 수 있다.

- **InContextLearning**은 **사용자가 특정 문맥에서 모델을 학습**할 수 있도록 하는 기술입니다.
- 이를 통해 **모델이 특정 문맥에서 더 나은 결과를 얻을 수 있도록** 도와줍니다.

텍스트는 숫자와 다르게 쉽게 미분해서 역전파할 수 있는 대상이 아니지만, TextGrad는 LLM의 유연한 의미연산 능력 덕분에 쉽게 미분할 수 없는 대상을 최적화할 수 있습니다.

## DSPy와 TextGrad의 차이점

**TEXTGRAD**
- **텍스트 피드백을 통한 최적화**: TEXTGRAD는 LLM으로부터 받은 텍스트 피드백을 개별 구성 요소에 **역전파하여 수동 조정 없이 복합적인 AI 시스템을 최적화**합니다.
- **다양한 작업에의 적용성**: 질문 답변, 분자 최적화, 방사선 치료 계획 등 **다양한 작업에서 활용**됩니다.
- **성능 향상**: 예를 들어, Google-Proof 질문 답변 작업에서 GPT-4o의 제로샷 정확도를 51%에서 55%로 향상시켰습니다.
- **해석 가능성 향상**: 자연어 그라디언트를 제공하여 모델의 의사 결정 과정을 더 잘 이해할 수 있게 합니다.
- **PyTorch와의 유사성**: PyTorch의 구문과 추상화를 따르므로 유연하고 사용이 용이합니다.
- **컴퓨테이션 그래프 접근법**: AI 시스템을 변수들이 복잡한 함수 호출의 입력과 출력으로 구성된 계산 그래프로 취급합니다. LLM은 코드 스니펫부터 분자 구조에 이르기까지 다양한 자연어 제안을 제공하여 이 그래프 내의 변수를 최적화합니다.

> TextGrad는 프롬프트 개선 뿐만 아니라 다양한 분야에 적용이 가능한 장점이 있습니다.
> 기존에 경사하강법으로 미분 불가능했던 분야들을 LLM이 스스로 최적화할 수 있게 도와줍니다.
> 최적화에 있어서 TextGrad는 LLM의 텍스트 피드백을 활용하여 비교적 강한 연산 능력을 필요로 합니다.

**DSPy**
- **프로그래밍 모델** 도입: 언어 모델(LM) 파이프라인을 텍스트 변환 그래프로 추상화하고, 주어진 메트릭을 최대화하기 위해 이러한 파이프라인을 자동으로 최적화합니다.
- **모듈화 및 컴파일러 사용**: LM 호출을 파라미터화된 모듈로 표현하고, 컴파일러를 사용하여 이들 모듈의 조합을 최적화합니다.
- **성능 향상**: 수학적 단어 문제와 다중 단계 질문 답변 작업에서 각각 25%와 65% 이상의 성능 향상을 보였습니다.
- **작은 LM의 활용**: T5와 Llama2와 같은 작은 LM을 활용하여 전문가가 작성한 프롬프트 체인과 독점적인 LM을 사용하는 시스템과 유사한 성능을 달성합니다

> DSPy는 보다 단순한 프로그래밍 모델을 도입하여 LM 파이프라인을 최적화합니다.
> 가장 좋은 성능을 내기 위해 LM 호출을 모듈로 표현하고, 컴파일러를 사용하여 최적화합니다.
> 최적의 프롬프트를 만드는 것을 **일종의 머신러닝 최적화 문제**라고 보고, 
> 언어모델과 자동화된 프레임워크를 이용해 모델과 데이타에 따라 최적의 프롬프트를 찾아내는 자동화된 방법론을 제시합니다.

