# M0: Medical Baseline (MLP + CE, Kvasir-SEG)

| | |
|---|---|
| **도메인 / Domain** | Kvasir-SEG (2 class: background / polyp) |
| **Decoder** | MLP |
| **Loss** | CE |
| **학습 전략** | pretrained encoder + warmup_poly + diff-LR + PaperlikeTransform aug |
| **파라미터 수** | 3.7M |
| **역할** | 의료 도메인 대조군 (기본 구조) |

## 성능 / Performance

| 지표 | Val Best | Test |
|------|----------|------|
| Dice (polyp) | 0.8998 (epoch 70) | 0.9222 |
| mIoU | — | 0.9147 |
| Val→Test 변화 | — | +0.0224 (test > val) |

## Per-class IoU (Test)

| Class | IoU |
|-------|-----|
| background | 0.9737 |
| polyp | 0.8557 |

## 학습 수렴

| 항목 | 값 |
|------|----|
| Val Dice 0.80 최초 달성 epoch | 5 |
| Best 달성 epoch | 70 |
| Early stopping | epoch 90 발동 (patience=20) |
| 최종 train loss | 0.0344 |

## 학습 곡선 / Training Curves

<!-- No assets available yet -->

## 분석 요약 / Summary

Kvasir-SEG 도메인에서 MLP 디코더와 CE loss만 사용한 기본 구조 대조군이다. Test Dice 0.9222, test mIoU 0.9147로 도메인 전환 성공 기준(Test Dice ≥ 0.80)을 0.12 이상 초과 달성했다. Val→Test 방향이 역전(+0.0224)되는 현상은 test 100장이 상대적으로 쉬운 샘플로 구성되어 있거나 val 기준 best checkpoint 전략이 test에서도 일반화된 결과로 해석된다. Epoch 70에서 best 달성 후 epoch 90에서 early stopping이 발동하여 정상 수렴한다.

M0 is the medical-domain control: MLP decoder with CE loss on Kvasir-SEG (2-class polyp segmentation). Test Dice of 0.9222 exceeds the domain-transfer success threshold (≥0.80) by over 0.12. The positive val→test delta (+0.0224) indicates the test split (100 images) contains relatively easier samples. ImageNet pretrain transfer is highly effective in this domain. Early stopping triggered at epoch 90 (patience=20 after best at epoch 70), confirming clean convergence.

