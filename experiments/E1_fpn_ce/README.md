# E1: FPN Decoder (FPN + CE)

| | |
|---|---|
| **도메인 / Domain** | CamVid (11 class) |
| **Decoder** | FPN |
| **Loss** | CE |
| **학습 전략** | scratch |
| **파라미터 수** | — |
| **역할** | decoder 실험 |

## 성능 / Performance

| 지표 | Val Best | Test |
|------|----------|------|
| mIoU | 0.6626 | 0.5829 |
| Val→Test 하락폭 | — | −0.0797 |

## Per-class IoU (Test)

| Class | IoU |
|-------|-----|
| Sky | 0.9259 |
| Building | 0.8006 |
| Pole | 0.2695 |
| Road | 0.9466 |
| Pavement | 0.7758 |
| Tree | 0.7147 |
| SignSymbol | 0.2445 |
| Fence | 0.3528 |
| Car | 0.6794 |
| Pedestrian | 0.3090 |
| Bicyclist | 0.3925 |

## 학습 곡선 / Training Curves

<!-- No assets available yet -->

## 분석 요약 / Summary

E0 대비 유일한 변경은 디코더를 MLP에서 FPN으로 교체한 것이다. Val best mIoU +0.0257, test mIoU +0.0147로 val·test 양쪽에서 일관된 향상이 확인된다. Pole(+0.0575), Pavement(+0.0317), Pedestrian(+0.0356) 등 세부 클래스에서 E0 대비 개선이 두드러지나, Fence(0.3528 < E0 0.3632)에서는 오히려 소폭 하락한다. FPN의 다중 스케일 특징 융합이 전반적으로 유효하지만, 좁고 선형적인 구조물(Fence)에서는 MLP 대비 이점이 제한될 수 있음을 시사한다.

The only change from E0 is swapping the decoder from MLP to FPN. Both val (+0.0257) and test (+0.0147) mIoU improve consistently, confirming the FPN decoder effect is reliable. Gains are notable in Pole, Pavement, and Pedestrian, but Fence IoU slightly drops below E0, suggesting FPN's multi-scale fusion may be less effective for narrow linear structures.

