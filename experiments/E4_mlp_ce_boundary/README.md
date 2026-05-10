# E4: CE+Boundary Loss (MLP + CE+Boundary)

| | |
|---|---|
| **도메인 / Domain** | CamVid (11 class) |
| **Decoder** | MLP |
| **Loss** | CE+Boundary |
| **학습 전략** | scratch |
| **파라미터 수** | — |
| **역할** | loss 실험 |

## 성능 / Performance

| 지표 | Val Best | Test |
|------|----------|------|
| mIoU | 0.6510 | 0.5705 |
| Val→Test 하락폭 | — | −0.0805 |

## Per-class IoU (Test)

| Class | IoU |
|-------|-----|
| Sky | 0.9271 |
| Building | 0.7870 |
| Pole | 0.2498 |
| Road | 0.9423 |
| Pavement | 0.7588 |
| Tree | 0.6985 |
| SignSymbol | 0.2311 |
| Fence | 0.3656 |
| Car | 0.6839 |
| Pedestrian | 0.2695 |
| Bicyclist | 0.3618 |

## 학습 곡선 / Training Curves

<!-- No assets available yet -->

## 분석 요약 / Summary

E0 대비 CE loss에 Boundary loss를 추가한 실험이다. Val에서는 +0.0141로 E3(+0.0149)과 유사한 개선을 보이지만, test에서는 +0.0023으로 개선폭이 크게 줄어든다. Val→Test 하락폭(0.0805)도 E3(0.0722)보다 크다. Boundary loss의 경계 강조 효과가 val 분포에서는 유효하지만 test 분포에 완전히 일반화되지 않음을 보여준다. 또한 epoch 1에서 학습 시간이 399초로 비정상적으로 길었으나 이후 정상 복귀하여 성능에는 영향이 없다.

Adding Boundary loss to CE shows val gains (+0.0141) comparable to E3, but test improvement collapses to +0.0023, and val→test drop (0.0805) is larger than E3 (0.0722). Boundary emphasis during training does not fully transfer to the test distribution. Note: epoch 1 training time was anomalously long (399 s) but recovered normally afterward with no measurable impact on results.

