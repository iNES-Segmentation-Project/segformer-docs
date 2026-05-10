# E2: Focal Loss (MLP + Focal)

| | |
|---|---|
| **도메인 / Domain** | CamVid (11 class) |
| **Decoder** | MLP |
| **Loss** | Focal |
| **학습 전략** | scratch |
| **파라미터 수** | — |
| **역할** | loss 실험 |

## 성능 / Performance

| 지표 | Val Best | Test |
|------|----------|------|
| mIoU | 0.6503 | 0.5669 |
| Val→Test 하락폭 | — | −0.0834 |

## Per-class IoU (Test)

| Class | IoU |
|-------|-----|
| Sky | 0.9274 |
| Building | 0.7837 |
| Pole | 0.2477 |
| Road | 0.9373 |
| Pavement | 0.7426 |
| Tree | 0.7016 |
| SignSymbol | 0.2208 |
| Fence | 0.3250 |
| Car | 0.6863 |
| Pedestrian | 0.2769 |
| Bicyclist | 0.3867 |

## 학습 곡선 / Training Curves

<!-- No assets available yet -->

## 분석 요약 / Summary

E0 대비 유일한 변경은 CE loss를 Focal loss로 교체한 것이다. Val best mIoU는 E0 대비 +0.0134로 개선되었으나, test mIoU는 오히려 −0.0013으로 역전된다. 이는 E0~E5 중 유일하게 val-test 방향이 뒤집히는 이상 징후로, Focal loss가 train/val 분포의 어려운 샘플에 과적합하는 경향을 반영한다. Val→Test 하락폭(0.0834)도 E0~E4 중 가장 크다. Focal loss 단독 적용은 CamVid 도메인에서 일반화 신뢰도가 낮다.

The only change from E0 is replacing CE with Focal loss. Despite a val improvement of +0.0134, test mIoU reverses to −0.0013 below E0 — the only val-to-test direction flip in the entire E0–E5 series. The largest val→test drop (0.0834) also occurs here. Focal loss appears to over-specialize toward hard samples in the training distribution without generalizing to the held-out test set.
