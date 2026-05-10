# M1: Medical Composite (FPN + CE+Dice+Boundary, Kvasir-SEG)

| | |
|---|---|
| **도메인 / Domain** | Kvasir-SEG (2 class: background / polyp) |
| **Decoder** | FPN |
| **Loss** | CE+Dice+Boundary |
| **학습 전략** | pretrained encoder + warmup_poly + diff-LR + PaperlikeTransform aug |
| **파라미터 수** | 6.1M |
| **역할** | 핵심 실험 — E5 구조의 의료 도메인 검증 |

## 성능 / Performance

| 지표 | Val Best | Test |
|------|----------|------|
| Dice (polyp) | 0.9083 (epoch 91) | 0.9275 |
| mIoU | — | 0.9201 |
| Val→Test 변화 | — | +0.0192 (test > val) |

## Per-class IoU (Test)

| Class | IoU |
|-------|-----|
| background | 0.9755 |
| polyp | 0.8647 |

## 학습 수렴

| 항목 | 값 |
|------|----|
| Val Dice 0.80 최초 달성 epoch | 4 |
| Val Dice 0.90 최초 달성 epoch | 52 |
| Best 달성 epoch | 91 |
| Early stopping | 미발동 (100 epoch 완주) |
| 최종 train loss | 0.0539 |

## 학습 곡선 / Training Curves

<!-- No assets available yet -->

## 분석 요약 / Summary

CamVid E5 구조(FPN + CE+Dice+Boundary + pretrained + warmup_poly + diff-LR + aug)를 Kvasir-SEG에 그대로 적용한 핵심 실험이다. M0 대비 Test Dice +0.0053, polyp IoU +0.0090으로 차이는 작지만 방향성이 일치하며, 이는 CamVid에서의 패턴(FPN > MLP)과 동일하다. 파라미터 수는 3.7M → 6.1M으로 증가하였으며, CE+Dice+Boundary 복합 loss의 스케일 차이로 train loss(0.0539)가 M0(0.0344)보다 높게 나타나지만 이는 직접 비교 의미가 없다. Early stopping이 발동되지 않고 100 epoch을 완주한 것은 복합 loss 특성상 수렴이 느리고 지속적임을 반영한다.

M1 applies the E5 pipeline (FPN + CE+Dice+Boundary + pretrained + warmup_poly + diff-LR + aug) directly to Kvasir-SEG with only a config change. Test Dice of 0.9275 (+0.0053 over M0) and polyp IoU of 0.8647 (+0.0090) confirm that FPN + compound loss outperforms MLP + CE in the medical domain, consistent with CamVid findings. The higher train loss (0.0539 vs M0 0.0344) reflects compound loss scale differences, not overfitting. The model completed all 100 epochs without early stopping, reflecting the slower but steady convergence characteristic of multi-term losses.

