# E0: Baseline (MLP + CE)

| | |
|---|---|
| **도메인 / Domain** | CamVid (11 class) |
| **Decoder** | MLP |
| **Loss** | CE |
| **학습 전략** | scratch |
| **파라미터 수** | — |
| **역할** | baseline |

## 성능 / Performance

| 지표 | Val Best | Test |
|------|----------|------|
| mIoU | 0.6369 | 0.5682 |
| Val→Test 하락폭 | — | −0.0687 |

## Per-class IoU (Test)

| Class | IoU |
|-------|-----|
| Sky | 0.9243 |
| Building | 0.7892 |
| Pole | 0.2120 |
| Road | 0.9342 |
| Pavement | 0.7441 |
| Tree | 0.7048 |
| SignSymbol | 0.2229 |
| Fence | 0.3632 |
| Car | 0.6901 |
| Pedestrian | 0.2734 |
| Bicyclist | 0.3916 |

## 학습 곡선 / Training Curves

<!-- No assets available yet -->

## 분석 요약 / Summary

MiT-B0 인코더를 scratch 학습하고 MLP 디코더와 Cross-Entropy loss만 사용한 기본 구성이다. Val best mIoU 0.6369에서 test mIoU 0.5682로 0.0687 하락하며, 이는 E0~E4 전반에서 관찰되는 구조적 val-test 괴리의 기준점이 된다. Pole(0.2120), SignSymbol(0.2229), Pedestrian(0.2734) 등 소수 클래스에서 저조한 성능을 보이며, 이는 369장의 제한된 학습 데이터와 scratch 학습의 복합적 한계에서 비롯된다.

E0 is the reference configuration: MiT-B0 trained from scratch with MLP decoder and CE loss only. The val-to-test drop of 0.0687 mIoU sets the baseline for structural generalization gap observed across E0–E4. Minority classes (Pole, SignSymbol, Pedestrian) remain weak, reflecting the limits of training 11 classes from scratch on 369 images.
