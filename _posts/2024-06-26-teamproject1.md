---
layout: single
title:  "[Team Research Project 2024] HEVC Overview"
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





