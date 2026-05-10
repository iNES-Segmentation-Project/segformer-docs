# E3: CE+Dice Loss (MLP + CE+Dice)

| | |
|---|---|
| **도메인 / Domain** | CamVid (11 class) |
| **Decoder** | MLP |
| **Loss** | CE+Dice |
| **학습 전략** | scratch |
| **파라미터 수** | — |
| **역할** | loss 실험 |

## 성능 / Performance

| 지표 | Val Best | Test |
|------|----------|------|
| mIoU | 0.6518 | 0.5796 |
| Val→Test 하락폭 | — | −0.0722 |

## Per-class IoU (Test)

| Class | IoU |
|-------|-----|
| Sky | 0.9241 |
| Building | 0.7926 |
| Pole | 0.2546 |
| Road | 0.9414 |
| Pavement | 0.7605 |
| Tree | 0.6994 |
| SignSymbol | 0.2498 |
| Fence | 0.3731 |
| Car | 0.6967 |
| Pedestrian | 0.3095 |
| Bicyclist | 0.3743 |

## 학습 곡선 / Training Curves

<!-- No assets available yet -->

## 분석 요약 / Summary

E0 대비 CE loss에 Dice loss를 추가한 실험이다. Val +0.0149, test +0.0114로 val·test 양쪽에서 일관된 개선이 유지되어, loss 변형 중 가장 안정적인 일반화 패턴을 보인다. E2(Focal)와 달리 val-test 방향 역전이 없으며, Fence(0.3731), SignSymbol(0.2498), Pedestrian(0.3095)에서 E0·E2보다 우수하다. CE+Dice 조합은 영역 기반 overlap 최적화가 CE의 픽셀 단위 분류를 보완하여 전반적인 경계 정렬 품질을 개선한 것으로 해석된다.

Adding Dice loss to CE (vs E0) yields consistent val (+0.0149) and test (+0.0114) gains — the most stable generalization pattern among all loss variants. Unlike E2, there is no val-to-test reversal. The Dice term's region-level overlap optimization complements CE's pixel-level classification, improving boundary alignment across most classes. CE+Dice is the recommended loss choice when using the MLP decoder on CamVid.

