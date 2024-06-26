---
layout: single
title:  "[Team Research Project 2024] HEVC Overview (Block Partitioning & Intra Prediction)"
categories: TRP2024
tag: [HEVC]
use_math: true
---

이 글은 HEVC의 간략한 내용 정리에 관한 기록입니다.


# Block Partitiong



## Coding Tree Unit (CTU)

비디오를 압축할 때, 하나의 픽쳐를 Raster scan order를 따라 CTU 단위로 분할합니다.

다양한 블록 사이즈를 가질 수 있습니다.
* 64x64, 32x32, 16x16

Y CTB, Cb CTB, Cr CTB 를 모두 통틀어 하나의 CTU로 정의합니다.
* CTB: Coding Tree Block

<img width="728" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/1c416b45-3612-4610-a304-27ba67ca57c1">



## Coding Unit (CU)

화면 내 예측 (Intra Prediction) 혹은 화면 간 예측 (Inter Prediction) 의 코딩 단위입니다.

CTU를 Quad-tree로 분할하여 얻으며, 다양한 블록 사이즈를 가질 수 있습니다.
* 64x64, 32x32, 16x16, 8x8

Y CB, Cb CB, Cr CB 를 모두 통틀어 하나의 CU로 정의합니다.
* CB: Coding Block

<img width="728" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/7ad012b4-8013-429f-9ab9-db5faa7f9bbf">


CTU는 Quad-tree 구조에 따라 CU로 분할되며, 하위 레벨의 CU로 분할될지 결정하는 정보는 _split_cu_flag_ 를 이용해 표현합니다.

_split_cu_flag_
* 0 -> split into sub-CUs
* 1 -> do not split into sub-CUs

<img width="1374" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/e1a2b83f-5bcc-4fc6-b995-9cab06b795ca">



## Prediction Unit (PU)

하나의 CU 내에서도 다른 PU라면 다른 예측 방법으로 예측 블럭을 생성할 수 있습니다.

하나의 CU 내에서는 예측 방법이 다를 순 있으나 화면 내 예측, 화면 간 예측을 혼합해서 사용하지는 않습니다.

<img width="1026" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/d7237581-b377-4cbb-80b9-bdeb0f3d5d6d">


PU의 분할 방법은 PU가 속한 CU의 코딩 모드 (화면 내 예측 혹은 화면 간 예측)에 따라 달라집니다.

<img width="1342" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/9f0965bd-3bab-4783-a0e2-70e2f897ddd4">



## Transform Unit (TU)

CU는 Quad-tree 구조에 따라 TU로 분할되며, 하위 레벨의 CU로 분할될지 결정하는 정보는 _split_transform_flag_ 를 이용해 표현합니다.

다양한 블록 사이즈를 가질 수 있습니다.
* 32x32, 16x16, 8x8, 4x4

만약 현재 CU가 화면 내 예측으로 코딩된 CU라면, **TU의 블록 크기는 절대 PU의 블록 크기보다 클 수 없습니다**.
* 복원되지 않은 영역을 변환해야하는 문제가 발생하기 때문에



# Intra Prediction

현재 블록 주변의 복원된 영역을 참조하여 현재 블록을 예측하는 과정입니다.

일정한 방향성을 가진 영역이나, 평평한 영역에서 효과적입니다.

영상의 첫번째 픽쳐, 혹은 중간에 Random access와 에러 방지를 위해 사용됩니다.

<img width="525" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/34e324b8-3510-4535-bed9-466947e92078">



## 픽쳐 내부에서

Top left to bottom right 순으로 CTU가 코딩됩니다.

코딩된 CTU는 복원되어 DPB (Decoded Picture Buffer) 에 저장되며, 이 방식은 인코더와 디코더 내에서 동일합니다.

저장된 CTU는 다음 CTU를 코딩할 때 참조됩니다.

<img width="811" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/3be1f52f-a300-4860-a080-c6fe28afd2d7">



## CTU 내부에서

Z-order (CU0 to CU12)에 따라 CU가 코딩됩니다.

CU의 코딩이 완료되면 복원된 블록은 DPB에 저장됩니다.

저장된 CU는 다음 CU를 코딩할 때 참조됩니다.

<img width="511" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/1551973e-78e8-4995-a6e6-a588a396d01c">



## CU 내부에서

화면 내 예측일 때, 예측 모드는 PU 단위로 선택됩니다.

선택된 예측 모드에 따라서 **TU 단위 (PU 단위가 아님에 주의)**로 예측이 진행됩니다.

RD-cost를 계산하고 비교하여, TU의 Quad-tree 구조를 결정하고 TU의 블록 사이즈가 결정됩니다.

<img width="1492" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/24a69c14-1a04-4f11-866f-b260bbe902f1">



## 참조 샘플 전처리

1. 참조 샘플 패딩 (Padding): 참조 샘플이 존재하지 않을 때 패딩을 진행합니다.

    <img width="1237" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/3eb0eea3-c1f9-43df-94a5-f0a4a9c0355e">


2. 모드에 따른 Intra Smoothing

    양자화 과정에서 발생하는 에러를 줄이기 위해 Low-band pass 필터를 사용합니다.

    HEVC는 적응적인 필터 적용을 지원합니다.

    $[ 1/4, 2/4, 1/4 ]$ 3-tap 필터를 사용합니다.

    <img width="1172" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/ee816eda-8c06-485f-bc61-6eef420b2715">



## 화면 내 예측자 생성

1. _Intra_Planar_ mode

    경계 샘플들로부터 수직적 그리고 수평적으로 표면의 수치값을 예측합니다.

    <img width="1399" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/574157eb-3c3a-4623-b46c-b3c1bdfedcdc">


2. _Intra_DC_ mode

    현재 블록의 참조 샘플들의 평균값으로 예측자를 생성합니다.

    예측자의 경계에서 발생하는 불연속성을 필터링함으로써 제거합니다.

    <img width="1444" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/84327673-96b8-42d6-b880-245a84acb7ca">


3. _Intra_Angular_ mode

    방향성을 고려하여 예측자를 생성합니다.

    2부터 34까지의 모드를 지원합니다.

    <img width="1410" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/a24e41f5-39d7-49bb-ade4-e606b01e2def">


    _Intra_Angular_ mode에서는 예측을 수행하기 위해 _IntraPredAngle_, _invAngle_ 을 사용합니다.

    <img width="1081" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/90d094cf-b2bd-4bbd-a3a6-e6844abb1b95">


    사용하는 예측 모드와 현재 TU 블록 사이즈를 고려하여, Side array에서 Main array로 복사될 참조 샘플의 수를 결정합니다.
    
    <img width="1178" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/8512c41c-d588-4141-8ac1-8c91354811f3">


    각 예측마다 _iIdx_, _iFact_가 결정됩니다.
    
    * _iIdx_: Main array에서 어느 값이 예측 샘플을 생성하는 데 사용될지 결정합니다.

    * _iFact_: _intraPredAngle_에 따라 참조 샘플들을 32 파트로 나누고, 그 중 값을 취합니다.

    <img width="784" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/7b7836a7-b535-4dcb-bf81-8b68d90bbb54">


    _Intra_Horizontal_ (10) mode와 _Intra_Vertical_ (26) mode에 한해서, 참조 샘플과 예측자의 불연속성을 필터링함으로써 제거합니다.

    <img width="1595" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/182b51d4-db81-4afe-b458-29debfafa05e">


    Chroma 성분의 화면 내 예측

    * _Intra_Planar_, _Intra_DC_, _Intra_Vertical_, _Intra_Horizontal_, _Intra_DM_

    * _Intra_DM_: Luma 성분에서 사용된 예측 모드와 동일한 예측 모드를 Chroma 성분을 예측하는데 사용합니다.



## 화면 내 예측 모드의 코딩

35가지의 모드를 적은 비트로 표현하기 위함입니다.

일반적으로, 이미지가 임의의 블록 사이즈로 분할되었을 경우, 하나의 블록과 그 주변의 블록들은 비슷한 이미지 특성을 지닙니다.

현재 PU의 모드는 현재 PU 블록의 왼쪽 PU 블록과 위쪽 PU 블록의 예측 모드에 기반하여 코딩됩니다.

* 왼쪽과 위쪽 블록의 예측 모드 -> Most Probable Mode (MPM)

<img width="852" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/a94a922c-faac-4876-8f93-006290c086e0">


**Luma 성분의 모드 코딩**

* MPM을 제외한 32가지 모드가 5비트를 사용해 코딩됩니다.

* MPM은 1~2비트를 사용해 코딩됩니다.

* 이렇게 코딩된 비트들은 모두 전송되지 않고, CABAC을 통해 코딩됩니다.

<img width="1643" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/0ae6f83b-94f1-4205-b1b1-2975750beef4">


* 첫번째 MPM은 왼쪽 PU의 모드에 따라 결정됩니다.

* 두번째 MPM은 위쪽 PU의 모드에 따라 결정됩니다.

* 세번째 MPM은 _Intra_Planar_, _Intra_DC_, _Intra_Vertical_ 모드 중 하나로 결정됩니다.

<img width="1267" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/574b573f-4918-4c29-9263-a6fd814205cf">



## Rough Mode Decision (RMD)

코딩의 효율은 유지하면서 빠른 속도로 예측 모드를 결정하기 위한 과정입니다.

1. 35가지 모드 중 RD-cost 계산을 통해 상위 N개의 후보 모드를 결정합니다.

    <img width="1170" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/1ec6b839-64ef-4972-92b8-66e78c0d875c">


2. 상위 N개의 모드와 3개의 MPM을 합친 (N+3)개의 후보 모드 중, Single-level RQT를 통해 최적의 모드를 결정합니다.

3. 선택된 최적의 모드에 대해 Multi-level RQT를 진행합니다.

