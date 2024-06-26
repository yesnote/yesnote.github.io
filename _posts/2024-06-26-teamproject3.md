---
layout: single
title:  "[Beyond VVC] Transform Kernel Selection"
categories: [Team Research Project 2024]
tag: [VVC, BeyondVVC]
use_math: true
---

이 글은 ECM 12.0 기준 Transfrom Kernel 선택 방법의 내용을 정리한 기록입니다.


# Transform Kernel Selection

## IPM (Intra Prediction Mode)을 사용한 Transform Kernel Selection

1. LFNST (Low Frequency Non-Separable Transform)

    Primary transform 이후, 저주파 영역의 Transform 계수들에 대해 적용하는 Secondary transform입니다.

    **사용조건**: Dual tree의 Luma & Chroma 채널 및 Single tree의 Luma 채널에서 Transform skip이 아닐 때 사용합니다.

    블록 당 Transform kernel 세트 35가지, 세트 당 Pre-trained kernel 3가지가 있습니다.

    <img width="518" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/9d917092-f5a9-4b9f-bca3-58e4f64dd23d">


2. NSPT (Non-Separable Primary Transform)

    Primary transform과 Secondary transform을 대체하는 Primary transform입니다.

    **사용조건**: LFNST 사용 조건을 만족할 때, 특정 크기인 블록에 한해서 사용합니다.

    블록 당 Transform kernel 세트 35가지, 세트 당 Pre-trained kernel 3가지가 있습니다.

    <img width="518" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/04eb3414-1e5e-497d-b7ef-cb901a02f7ce">


## 블록의 정보를 활용한 Transform Kernel Selection

1. Implicit MTS (Multiple Transfrom Selection)

    블록의 크기 정보로 신호 없이 Transform kernel을 결정합니다.

    * $4 <=$ _Width_ $<= 16$: Horizontal transform kernel = DST-7

    * $4 <=$ _Height_ $<= 16$: Vertical transform kernel = DST-7

    **사용조건**: Luma 채널에서 LFNST와 MIP를 사용하지 않는 경우 적용합니다.

2. Explicit MTS

    블록의 크기와 IPM 정보로 80가지 Transform kernel 세트 후보 중 하나를 선택합니다.

    * Transform kernel 세트 후보 당 Transform kernel 6가지

    인코더: 후보 중 최적의 Transform kernel index (_MTS_idx_)를 신호합니다.

    <img width="1647" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/7a7c2ce1-25ec-4dfd-b843-0329de90733f">

    디코더: 신호받은 (_MTS_idx_)로 결정된 Transform kernel을 사용합니다.

    <img width="1615" alt="image" src="https://github.com/yesnote/yesnote.github.io/assets/173476188/3c6e182c-4292-4e90-8ddc-11038225bdd3">

    

