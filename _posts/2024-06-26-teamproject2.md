---
layout: single
title:  "[NNVC] Intra Prediction"
categories: [Team Research Project 2024]
tag: [NNVC, HEVC]
use_math: true
---

이 글은 NNVC Intra Prediction의 내용을 정리한 기록입니다.

JVET-T0073 기고서를 기준으로 작성했으며, 실제 채택된 기술과 내용이 다를 수 있습니다.


# Neural Network-Based Intra Prediction



## 구성

8가지 Neural network로 구성되며, 각 모델은 서로 다른 사이즈의 블록을 예측합니다.

_w_ x _h_ 사이즈의 블록 **$Y$** 에 대해, $f_{h,w}$ 는 $\tilde{X}$ 를 입력받아 $\tilde{Y}$, $grpIdx_{1}$, $grpIdx_{2}$ 를 출력합니다.

* $f_{h,w}$: _w_ x _h_ 사이즈의 블록을 예측하는 Neural network

* $\tilde{X}$: Context $X$ 를 전처리한 결과

<img width="1623" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/cfd14e4e-bec9-49f8-b96e-63df98afa71a">



## 현재 블록의 Context에 대한 전처리 (4단계)

1. 현재 블록 $Y$ 의 Context $X$ 의 참조 샘플을 $2^{b-8}$ 로 나눕니다.

    * _b_ : bitdepth = 10 in VVC

2. 복원된 참조 샘플 $\hat{X}$ 에서 $\hat{X}$ 의 평균 𝜇 를 뺍니다.

3. 복원되지 않은 참조 샘플 $X_{u}$ 를 255로 설정합니다.

    <img width="649" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/37f7183c-3b1c-414c-8d46-a66e17e4a708">


4. 입력 데이터 분할 여부 결정
    
    * $min(h, w) <= 8$ : Context는 따로 분할되지 않습니다.

        * $\tilde{X}$ : $n_{a}(n_{l} + 2w) + 2hn_{l}$ 사이즈의 벡터

    * $min(h, w) > 8$ : Context는 $X_{0}$ 과 $X_{1}$으로 분할됩니다.

        * $\tilde{X} = X_{0} ∪ X_{1}$

    <img width="755" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/4c91fd47-405d-449a-89b1-90e889b6ca30">



## Neural Network의 구조

* $min(h, w) <= 8$ : $f_{h,w}$ 는 Fully-connected

    <img width="1301" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/57b56279-edcb-43ac-8785-a5927c49af0c">


* $min(h, w) > 8$ : $f_{h,w}$ 는 Convolutional

    * $f_{h,w}^{0}$, $f_{h,w}^{1}$, $f_{h,w}^{f}$ 로 구성됨.

    <img width="1312" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/f3f614f2-f492-44ce-a896-eaaf479a4590">


    * $f_{h,w}^{0}$ 의 구조

    <img width="1396" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/d327ccfc-a0f4-4820-bc1a-1574feaa020a">


    * $f_{h,w}^{1}$ 의 구조

    <img width="1396" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/3ca630b2-fc26-42b9-9f5a-548a055759ed">


    * $f_{h,w}^{f}$ 의 구조

    <img width="1467" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/3bc45bdb-a59d-4667-94ed-642638b0b906">



## Neural Network의 예측자에 대한 후처리 (3단계)

1. 예측자에 평균 𝜇 를 더합니다.

2. 그 결과에 $2^{b-8}$ 를 곱합니다.

3. 그 결과를 0 ~ 1023 범위로 제한합니다.

    predicted block $\hat{Y} = min(max(4(reshape(\tilde{Y} + 𝜇), 0, 1023)$



## Neural Network의 예측자와 Context의 Transform

현재 블록의 Context $X$ 는 전처리 단계 전에 Down-sample 혹은 Transpose 될 수 있습니다.

현재 블록의 예측자 $\hat{Y}$ 는 후처리 단계 후에 Transpose 혹은 Up-sample 될 수 있습니다.

<img width="1548" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/31df0f71-f02e-4825-9bcf-cac4493464ed">


<img width="1180" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/df9fe373-3e08-4ca6-9b6f-157a10658478">



## LFNST Selection

$f_{h,w}$ 의 Fully-connected layer는 사이즈 14의 벡터 $P$를 출력합니다.

**Case0**: $grpIdx_{i}$ 는 NN 모드에 따라 직접적으로 추론됩니다.

* $grpIdx_{1} = argmax(P[0:6]), grpIdx_{2} = argmax(P[7:13])$ 

* $lfnstIdx ∈ {1, 2}$, $grpIdx_{lfnstIdx}$ 는 DCT-2를 적용한 Primary transform 계수에 적용될 LFNST를 결정합니다.

<img width="1399" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/d3e01a7e-0a9f-4f41-8c0a-2c46bcdd6f6c">


**Case1**: NN 모드가 $grpIdx_{i}$ 를 예측하고, 예측자는 비트스트림을 통해 전송됩니다.

* Neural Network는 인코더를 통해 $grpIdx_{lfnstIdx}$ 를 예측합니다.

    * $lfnstIdx ∈ {1, 2}$

* $grpPredIdx_{1} = argmax(P[0:6]), grpPredIdx_{2} = argmax(P[7:13])$ 

* $grpIdx_{lfnstIdx}$ 는 $grpPredIdx_{lfnstIdx}$ 를 통해 예측적으로 코딩됩니다.

<img width="843" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/01d0e198-83aa-468b-b35d-99da81764d88">



## Signaling of LFNST (Case1일 때만)

* 인코더

    <img width="1398" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/9e68f888-f74a-46dd-8a3f-10fdd7c59bad">


* 디코더

    <img width="1396" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/4c7871f7-1d7e-4998-9fee-42a1fd2f2ceb">



## Signaling of Neural Network-based Intra Prediction Mode

* Luma

VVC에서는 _nnFlag_를 통해 신호됩니다.

_w_ x _h_ 크기의 Luma 블록의 top-left 픽셀 (_x_, _y_)에 기반합니다.

(_h_, _w_) $∈ T$ && $x >= n_{l}$ && $y >= n_{a}$ 일 때, Neural network-based mode는 신호됩니다.

<img width="1498" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/f2a9fece-4151-4810-aaea-b24e3e59d581">


* Chroma

_w_ x _h_ 크기의 Chroma 블록의 top-left 픽셀 (_x_, _y_)에 기반하여, Collocated Luma 블록이 NN 모드로 예측됐을 때 -> DM = NN mode

그 외의 경우 -> DM = PLANAR



## 성능

<img width="1601" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/0319bff2-d245-412c-a465-b34c1e50b0e0">



# 참고자료

[JVET-T0073](https://www.jvet-experts.org/){: target="_blank"}


