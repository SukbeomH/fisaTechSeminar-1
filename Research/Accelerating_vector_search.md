# SIMD 명령어로 벡터 검색 가속화

[원문](https://www.elastic.co/kr/blog/accelerating-vector-search-simd-instructions)

**작성자: Chris Hegarty**

2023년 6월 27일

오랫동안, 자바 플랫폼에서 실행되는 코드는 HotSpot C2 컴파일러에서 여러 스칼라 연산을 SIMD(Single Instruction Multiple Data) 벡터 명령어로 자동으로 벡터화하는 슈퍼워드 최적화의 이점을 누려왔습니다. 이는 훌륭하지만, 이러한 최적화는 어느 정도 취약하고 자연스러운 복잡성 한계를 가지며, 자바 플랫폼 사양(예: 부동소수점 연산의 엄격한 순서 지정)에 의해 제약을 받습니다. 이러한 최적화가 여전히 유용하지 않다는 것은 아니지만, 코드의 형태를 명시적으로 정의하는 것이 성능에 상당히 더 나은 결과를 가져오는 경우도 있습니다. 루씬(Lucene)에서 벡터 검색을 지원하는 저수준 원시 연산은 그런 경우 중 하나입니다.

이 기사에서는 루씬의 벡터 검색에서 사용되는 저수준 원시 연산이 어떻게 SIMD 명령어(예: x64의 AVX 명령어, AArch64의 NEON 명령어)로 안정적으로 컴파일되는지, 그리고 이것이 성능에 어떤 영향을 미치는지 살펴봅니다.

#### 저수준 원시 연산

루씬의 벡터 검색 구현의 핵심에는 두 벡터 간의 유사성을 찾을 때 사용되는 세 가지 저수준 원시 연산이 있습니다: 내적(dot product), 제곱(square), 코사인 거리(cosine distance). 각 연산에는 부동소수점과 이진 변형이 있습니다. 간결함을 위해, 여기서는 하나의 원시 연산인 내적(dot product)에 대해서만 살펴보겠습니다. 인터페이스는 단순하며 다음과 같습니다:

```java
/** 주어진 float 배열의 내적을 계산합니다. */
float dotProduct(float[] a, float[] b);
```

지금까지 이러한 원시 연산은 성능을 고려해 핸드 언롤(hand unrolled) 루프와 같은 기존에 알려진 기술을 사용하여 스칼라 구현으로 지원되었습니다. 이는 자바 코드를 작성하는 경우 가능한 한 최선의 방법이며, 나머지는 HotSpot JIT(Just-In-Time) 컴파일러가 최선을 다해 자동 벡터화를 수행하게 맡깁니다. 여기에 내적의 단순화된 스칼라 구현이 있으며, 언롤링은 제거되었습니다(실제 구현은 [여기](https://github.com/apache/lucene)에서 볼 수 있습니다):

```java
public float dotProduct(float[] a, float[] b) {
  float res = 0f;
  for (int i = 0; i < a.length; i++) {
    res += a[i] * b[i];
  }
  return res;
}
```

최근 변경된 것은 JDK에서 런타임에 SIMD 명령어로 안정적으로 컴파일될 수 있는 계산을 표현하는 API를 제공한다는 점입니다. 이는 OpenJDK의 Project Panama Vector API입니다. 물론, 실제 런타임에 생성되는 명령어는 하위 플랫폼이 지원하는 바에 따라 다르지만(예: AVX2 또는 AVX 512), API는 이를 고려하여 구조화되어 있습니다. 다시, 여기에 단순화된 내적 코드가 있지만 이번에는 Panama Vector API를 사용합니다:

```java
static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_PREFERRED;

public float dotProductNew(float[] a, float[] b) {
  int i = 0;
  float res = 0f;
  FloatVector acc = FloatVector.zero(SPECIES);
  int upperBound = SPECIES.loopBound(a.length);
  for (; i < upperBound; i += SPECIES.length()) {
    FloatVector va = FloatVector.fromArray(SPECIES, a, i);
    FloatVector vb = FloatVector.fromArray(SPECIES, b, i);
    acc = acc.add(va.mul(vb));
  }
  // reduce
  res = acc.reduceLanes(VectorOperators.ADD);
  // tail
  for (; i < a.length; i++) {
    res += b[i] * a[i];
  }
  return res;
}
```

코드가 약간 더 장황해졌지만, 런타임에 하드웨어로 어떻게 매핑되는지를 직관적으로 이해하기 쉬워졌습니다. 벡터 연산이 코드에 명시적으로 나타나 있기 때문입니다. `VectorSpecies`는 우리의 경우 float 타입의 요소와 벡터의 비트 크기(형태)를 포함합니다. 선호되는 `species`는 플랫폼에 최적화된 최대 비트 크기의 요소입니다. 먼저, 입력값을 반복하며 한 번에 `SPECIES::length` 요소를 곱하고 누적하는 루프가 있습니다. 그런 다음, 누산 벡터를 줄입니다. 마지막으로 남아 있는 "꼬리" 요소들을 처리하는 스칼라 루프가 있습니다.

이 코드를 AVX 512를 지원하는 CPU에서 실행하면 HotSpot C2 컴파일러가 AVX 512 명령어를 생성하는 것을 볼 수 있습니다. Advanced Vector Extensions (AVX)는 널리 사용 가능하며, 예를 들어 Intel의 Ice Lake 마이크로아키텍처와 이를 기반으로 한 클라우드 컴퓨팅 인스턴스(GCP 또는 AWS)를 사용할 수 있습니다. AVX 512 명령어는 내적 계산을 한 번에 16개의 값으로 수행합니다; 512 비트 크기 / 32비트 값당 = 루프 반복당 16개 값. AVX2를 지원하는 CPU에서 동일한 코드의 루프 반복은 반복당 8개 값을 처리합니다. 유사하게, NEON(128비트)은 반복당 4개의 값을 처리합니다.

이것을 확인하려면 생성된 코드를 살펴보아야 합니다. 이제 재미가 시작됩니다!

#### 네이티브로 전환

다음은 AVX 512를 지원하는 Rocket Lake에서 실행된 내적(dotProduct)에 대한 HotSpot C2 컴파일러의 디스어셈블리입니다.

아래 코드 스니펫은 주 루프 본문을 포함하며, rcx 및 rdx 레지스터는 각각 첫 번째와 두 번째 float 배열의 주소를 가리킵니다.

1. 먼저, `vmovdqu32` 명령어가 512비트의 패킹된 더블워드 값을 메모리 위치에서 `zmm0` 레지스터로 로드합니다. 이는 첫 번째 float 배열의 오프셋에서 16개의 값을 가져옵니다.
2. 다음으로, `vmulps` 명령어가 이전에 `zmm0`에 로드된 패킹된 단일 정밀도 부동소수점 값을 두 번째 float 배열의 오프셋에서 가져온 해당 패킹된 더블워드 값과 곱하고, 결과 부동소수점 값을 `zmm0`에 저장합니다.
3. 세 번째로, `vaddps`는 `zmm0`에서 16개의 패킹된 단일 정밀도 부동소수점 값을 `zmm4`와 더하고, 그 결과를 `zmm4`에 저장합니다. `zmm4`는 우리의 루프 누산기입니다.
4. 마지막으로, 루프 카운터를 증가시키고 확인하는 작은 계산이 있습니다.

```asm
0x00007f6b3c7243a4:   vmovdqu32 zmm0,ZMMWORD PTR [rcx+r10*4+0x10];*invokestatic load {reexecute=0 rethrow=0 return_oop=0}
0x00007f6b3c7243af:   vmulps zmm0,zmm0,ZMMWORD PTR [rdx+r10*4+0x10];*invokestatic reductionCoerced {reexecute=0 rethrow=0 return_oop=0}
0x00007f6b3c7243ba:   vaddps zmm4,zmm4,zmm0            ;*invokestatic binaryOp {reexecute=0 rethrow=0 return_oop=0}
0x00007f6b3c7243c0:   add    r10d,0x10
0x00007f6b3c7243c4:   cmp    r10d,eax
0x00007f6b3c7243c7:   jl     0x00007f6b3c7243a4
```

디스어셈블리의 정확한 세부 사항에 너무 신경 쓰지 마세요. 이는 "후드 아래에서" 무슨 일이 일어나는지 감을 주기 위해 제공된 것이며, 이해에 반드시 필수적인 것은 아닙니다. 실제로는 C2가 루프를 언롤링하여 한 번에 4번 반복하게 됩니다.

```asm
0x00007f74a86fa0f0:   vmovdqu32 zmm0,ZMMWORD PTR [rcx+r8*4+0xd0]
0x00007f74a86fa0fb:   vmulps zmm0,zmm0,ZMM

WORD PTR [rdx+r8*4+0xd0]
0x00007f74a86fa106:   vmovdqu32 zmm2,ZMMWORD PTR [rcx+r8*4+0x90]
0x00007f74a86fa111:   vmulps zmm2,zmm2,ZMMWORD PTR [rdx+r8*4+0x90]
0x00007f74a86fa11c:   vmovdqu32 zmm3,ZMMWORD PTR [rcx+r8*4+0x10]
0x00007f74a86fa127:   vmulps zmm3,zmm3,ZMMWORD PTR [rdx+r8*4+0x10]
0x00007f74a86fa132:   vmovdqu32 zmm4,ZMMWORD PTR [rcx+r8*4+0x50]
0x00007f74a86fa13d:   vmulps zmm4,zmm4,ZMMWORD PTR [rdx+r8*4+0x50]
0x00007f74a86fa148:   vaddps zmm1,zmm1,zmm3
0x00007f74a86fa14e:   vaddps zmm1,zmm1,zmm4
0x00007f74a86fa154:   vaddps zmm1,zmm1,zmm2
0x00007f74a86fa15a:   vaddps zmm1,zmm1,zmm0
0x00007f74a86fa160:   add    r8d,0x40
0x00007f74a86fa164:   cmp    r8d,r11d
0x00007f74a86fa167:   jl     0x00007f74a86fa0f0
```

우리는 반복당 더 많은 레지스터와 명령어를 사용하고 있습니다. 멋지군요! 게다가 루씬 코드는 루프를 4배 더 언롤링합니다(음... 언롤링이 꽤 많군요).

#### 빠른가요?

이러한 저수준 연산을 재작성한 영향력을 평가하기 위해, 자바 코드의 마이크로벤치마크를 수행하는 일반적으로 받아들여지는 방법인 JMH를 사용합니다. 여기서는 로버트 뮤어(Robert Muir)의 매우 멋지고 편리한 벤치마크 세트를 사용하여 이전과 이후의 코드 변형을 빠르게 비교할 수 있었습니다.

SIMD는 데이터 병렬성을 제공하므로 처리하는 데이터가 많을수록 잠재적 이익이 큽니다. 우리의 경우, 이는 벡터의 차원 크기와 직접적으로 관련이 있습니다. 차원 크기가 클수록 더 큰 이익이 기대됩니다. AVX 512를 지원하는 CPU(예: Intel Core i9-11900F @ 2.50GHz)에서 1024차원의 벡터로 내적의 부동소수점 변형을 실행할 때의 성능을 살펴보겠습니다:

```plaintext
Benchmark                                (size)   Mode  Cnt   Score   Error   Units
FloatDotProductBenchmark.dotProductNew     1024  thrpt    5  25.657 ± 2.105  ops/us
FloatDotProductBenchmark.dotProductOld     1024  thrpt    5   3.320 ± 0.079  ops/us
```

벤치마크는 마이크로초당 연산을 측정하므로 클수록 좋습니다. 여기서 새로운 내적이 이전 것보다 약 8배 더 빠르게 실행되는 것을 볼 수 있습니다. 부동소수점과 이진의 다양한 저수준 원시 연산에서도 유사한 성능 향상을 볼 수 있습니다:

```plaintext
Benchmark                                (size)   Mode  Cnt   Score   Error   Units
BinaryCosineBenchmark.cosineDistanceNew    1024  thrpt    5  10.637 ± 0.068  ops/us
BinaryCosineBenchmark.cosineDistanceOld    1024  thrpt    5   1.115 ± 0.008  ops/us
BinaryDotProductBenchmark.dotProductNew    1024  thrpt    5  22.050 ± 0.007  ops/us
BinaryDotProductBenchmark.dotProductOld    1024  thrpt    5   3.349 ± 0.041  ops/us
BinarySquareBenchmark.squareDistanceNew    1024  thrpt    5  16.215 ± 0.129  ops/us
BinarySquareBenchmark.squareDistanceOld    1024  thrpt    5   2.479 ± 0.032  ops/us
FloatCosineBenchmark.cosineNew             1024  thrpt    5   9.394 ± 0.048  ops/us
FloatCosineBenchmark.cosineOld             1024  thrpt    5   0.750 ± 0.002  ops/us
FloatDotProductBenchmark.dotProductNew     1024  thrpt    5  25.657 ± 2.105  ops/us
FloatDotProductBenchmark.dotProductOld     1024  thrpt    5   3.320 ± 0.079  ops/us
FloatSquareBenchmark.squareNew             1024  thrpt    5  19.437 ± 0.122  ops/us
FloatSquareBenchmark.squareOld             1024  thrpt    5   2.355 ± 0.003  ops/us
```

우리는 모든 원시 연산 변형에서, 그리고 다양한 작은 차원부터 큰 차원 크기에서(이것은 여기서는 표시되지 않았지만, Lucene PR에서 볼 수 있습니다) 상당한 개선을 봅니다. 이는 모두 훌륭하지만, 이러한 변화가 고차원 워크로드에 어떻게 연결되는지 어떻게 알 수 있을까요?

#### 확대해서 보기

마이크로벤치마크는 저수준 원시 연산이 어떻게 수행되는지를 이해하는 데 중요하지만, 이것이 매크로 수준에서 어떤 영향을 미치는지 살펴봐야 합니다. 이를 위해 엘라스틱서치의 벡터 검색 벤치마크인 SO Vector와 Dense Vector를 볼 수 있습니다. 두 벤치마크 모두 상당한 개선을 보이지만, 더 높은 벡터 차원을 가진 SO Vector가 더 흥미로워 보이므로 이를 살펴보겠습니다.

SO Vector 벤치마크는 768차원의 200만 벡터로 벡터 검색 성능을 테스트하며, kNN과 필터링을 포함합니다. 벡터는 StackOverflow 게시물 덤프에서 파생된 데이터 세트를 기반으로 합니다. 엘라스틱서치는 GCP에서 싱글 노드 클러스터로 실행되며, 8 vCPU, 16GB RAM, 1x300GiB SSD 디스크가 있는 맞춤형 n2 인스턴스에서 실행됩니다. 노드는 Ice Lake에 고정된 CPU 플랫폼을 가지고 있습니다.

벤치마크에서 기존의 상당한 변동성이 있지만, 전반적으로 긍정적인 개선을 확인할 수 있습니다:

- 색인 처리량이 약 30% 개선되었습니다.
- 병합 시간이 약 40% 감소했습니다.
- 쿼리 지연 시간이 크게 개선되었습니다.

#### 하지만 Panama Vector API는 인큐베이팅 중인가요?

Project Panama에서 개발된 JDK Vector API는 꽤 오랫동안 인큐베이팅 중입니다. 인큐베이팅 상태는 품질에 대한 반영이 아니라, OpenJDK에서 진행 중인 다른 흥미로운 작업(예: 값 타입)에 대한 의존성의 결과입니다. 루씬은 여기서 "최전선"에 있으며, JDK의 비최종 API를 활용하는 새로운 방법을 가지고 있습니다. JDK 버전별 API가 포함된 "apijar"을 빌드함으로써 이를 해결할 수 있었습니다. 이는 우리가 가볍게 고려하지 않는 실용적인 접근 방식입니다. 대부분의 인생의 일처럼, 이것도 트레이드오프입니다. 우리는 잠재적인 이익이 유지 비용을 초과할 때만 비최종 JDK API 채택을 고려합니다. 루씬은 여전히 이러한 저수준 원시 연산의 스칼라 변형을 가지고 있습니다. 구현 버전은 시작 시 선택할 수 있으며(루씬 변경 로그 참조), JDK 20 및 향후 출시될 JDK 21에서는 더 빠른 Panama 구현이 제공되며, 이전 JDK 또는 사용 불가능한 경우에는 스칼라 구현으로 되돌아갑니다. 다시 말하지만, 최신 JDK 버전만 지원하는 것은 잠재적 이익과 유지 비용을 균형 있게 고려한 실용적인 선택입니다.

#### 정리

이제 Panama Vector API를 사용하여 하드웨어 가속 SIMD 명령어를 안정적으로 활용하는 자바 코드를 작성할 수 있습니다. Lucene 9.7.0에서는 벡터 검색에서 사용되는 저수준 원시 연산의 대체 빠른 구현을 활성화할 수 있는 기능을 추가했습니다. 엘라스틱서치 8.9.0에서는 이 빠른 구현이 기본적으로 활성화되므로, 별도의 설정 없이 성

능 개선 혜택을 얻을 수 있습니다. 우리는 벡터 검색 벤치마크에서 상당한 성능 개선을 보았으며, 이는 사용자 워크로드에도 반영될 것으로 기대합니다.

SIMD 명령어는 새로운 것이 아니며 오랫동안 사용되어 왔습니다. 항상 그렇듯이, 이 변경 사항이 여러분의 환경에서 어떤 영향을 미칠지 알아보기 위해 자체적인 성능 벤치마크를 수행하는 것이 필요합니다. AVX 512는 멋지지만, 악명 높은 "다운클로킹"에 시달릴 수 있습니다. 전반적으로, 우리는 이 변경 사항으로 전반적으로 긍정적인 개선을 보았습니다.

마지막으로, 이 개선을 이끌어낸 즐겁고 생산적인 협력에 대해 루씬 PMC 멤버인 로버트 뮤어와 우베 신들러에게 감사를 표하고자 합니다. 그들이 없었다면 이 일은 일어나지 않았을 것입니다.
