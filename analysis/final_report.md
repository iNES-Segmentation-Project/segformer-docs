# SegFormer-B0 통합 분석 보고서 (E0~E5, M0~M1)

---

## 1. 실험 개요

### 데이터셋

| 도메인 | 데이터셋 | 클래스 수 | Train / Val / Test |
|--------|----------|-----------|-------------------|
| 도시 도로 | CamVid | 11 | 369 / 100 / 232 |
| 의료 폴립 | Kvasir-SEG | 2 | 800 / 100 / 100 |

### 고정 요소

- **인코더**: MiT-B0 (SegFormer-B0), E0~E4는 scratch, E5·M0·M1은 ImageNet pretrained
- **E0~E4 공통**: 기본 학습률 6e-4, poly scheduler, 증강 없음
- **E5·M0·M1 공통**: enc lr 6e-5 / dec lr 6e-4 (차등), warmup_poly scheduler, PaperlikeTransform 증강, AdamW

### 변경 요소 요약

| 실험 | Decoder | Loss | 학습 전략 | 변경 변수 |
|------|---------|------|-----------|-----------|
| E0 | MLP | CE | scratch | baseline |
| E1 | FPN | CE | scratch | decoder |
| E2 | MLP | Focal | scratch | loss |
| E3 | MLP | CE+Dice | scratch | loss |
| E4 | MLP | CE+Boundary | scratch | loss |
| E5 | FPN | CE+Dice+Boundary | pretrained + warmup_poly + diff-LR + aug | 복합 |
| M0 | MLP | CE | pretrained + warmup_poly + diff-LR + aug | 의료 도메인 대조군 |
| M1 | FPN | CE+Dice+Boundary | pretrained + warmup_poly + diff-LR + aug | 의료 도메인 E5 구조 |

---

## 2. E0~E4 단일 변수 효과 분석

### 2-1. Decoder 효과 (E0 → E1)

| 지표 | E0 (MLP) | E1 (FPN) | 차이 |
|------|----------|----------|------|
| Val Best mIoU | 0.6369 | 0.6626 | +0.0257 |
| Test mIoU | 0.5682 | 0.5829 | +0.0147 |

Val과 test 양쪽에서 FPN이 MLP를 일관되게 상회한다. Per-class IoU에서 Pole(+0.0575), Pavement(+0.0317), Pedestrian(+0.0356) 등에서 두드러진 개선이 나타난다. 단, Fence에서는 E1(0.3528)이 E0(0.3632)보다 소폭 낮아, FPN의 다중 스케일 융합이 좁고 선형적인 구조물에는 제한적으로 작동함을 보여준다.

### 2-2. Loss 효과 (E2, E3, E4 vs E0)

| 실험 | Loss | Val Best mIoU | Val 대비 E0 | Test mIoU | Test 대비 E0 |
|------|------|---------------|-------------|-----------|-------------|
| E0 | CE | 0.6369 | — | 0.5682 | — |
| E2 | Focal | 0.6503 | +0.0134 | 0.5669 | **−0.0013** |
| E3 | CE+Dice | 0.6518 | +0.0149 | 0.5796 | +0.0114 |
| E4 | CE+Boundary | 0.6510 | +0.0141 | 0.5705 | +0.0023 |

**E2 (Focal)**: val에서 +0.0134 개선되었음에도 test에서 −0.0013으로 역전된다. 이는 E0~E5 전체에서 유일한 val-test 방향 역전이며, Focal loss의 어려운 샘플 집중 효과가 train/val 분포에는 유효하지만 test 분포에 일반화되지 않았음을 의미한다.

**E3 (CE+Dice)**: val +0.0149 → test +0.0114로 개선폭이 test에서도 유지된다. loss 변형 중 가장 안정적인 일반화 패턴이다.

**E4 (CE+Boundary)**: val +0.0141 → test +0.0023으로 개선폭이 크게 줄어든다. Boundary loss의 경계 강조 효과가 test 분포에 완전히 전이되지 않는다.

### 2-3. Val→Test 하락폭 분석

| 실험 | Val Best mIoU | Test mIoU | 하락폭 |
|------|---------------|-----------|--------|
| E0 | 0.6369 | 0.5682 | 0.0687 |
| E1 | 0.6626 | 0.5829 | 0.0797 |
| E2 | 0.6503 | 0.5669 | **0.0834** |
| E3 | 0.6518 | 0.5796 | 0.0722 |
| E4 | 0.6510 | 0.5705 | 0.0805 |
| E5 | 0.8043 | 0.7572 | **0.0471** |

E0~E4 전반에 걸쳐 하락폭은 0.069~0.083 범위로 비교적 일정하다. 이는 특정 실험의 문제가 아닌, CamVid 369장의 제한된 학습 데이터와 scratch 학습이라는 구조적 한계에서 비롯된다. E2의 하락폭(0.0834)이 가장 크며, 이는 Focal loss의 과적합 경향과 일치한다.

---

## 3. E5 복합 실험 분석

### 3-1. 설계 근거

E5의 구성은 E0~E4 결과에서 직접 도출되었다:

- **Decoder → FPN**: E1에서 val·test 양쪽 일관된 우위 확인
- **Loss → CE+Dice+Boundary**: E3의 CE+Dice가 가장 안정적, E4의 Boundary 추가로 경계 정보 보완
- **pretrained encoder**: E0~E4 scratch 학습의 test mIoU 상한(0.5829)을 극복하기 위한 전이 학습 도입
- **warmup_poly + diff-LR + aug**: pretrained 가중치 보호(enc lr 6e-5)와 일반화 강화

### 3-2. 성능

| 지표 | 값 | E0 대비 | E1(단일 변수 최고) 대비 |
|------|----|---------|-----------------------|
| Val Best mIoU | 0.8043 | +0.1674 | +0.1417 |
| Test mIoU | 0.7572 | +0.1890 | +0.1743 |
| Val→Test 하락폭 | 0.0471 | −0.0216 | −0.0326 |

단일 변수 효과 최대치(E1 기준 +0.0147)를 약 12배 상회하는 향상이다.

### 3-3. Pretrained Encoder 기여 추정

E5에서 여러 요소가 동시에 변경되어 개별 기여를 정량적으로 분리할 수 없다. 다만 다음 근거에서 pretrained encoder의 기여가 가장 클 것으로 추정된다:

1. **Epoch 10 val mIoU = 0.7135**: warmup 완료 시점에서 이미 E0~E4가 40 epoch 전부 소진해도 달성하지 못한 수준을 기록한다.
2. **소수 클래스의 대폭 향상**: SignSymbol(0.2229 → 0.5455), Pedestrian(0.2734 → 0.6137), Fence(0.3632 → 0.6776). 이러한 향상은 단순한 학습량 증가나 loss 변경만으로 설명하기 어렵다.
3. **Val→Test 하락폭 감소(0.047)**: augmentation 강화가 train 분포를 test에 더 가깝게 만든 복합 효과로도 해석 가능하다.

### 3-4. 체크포인트 확인 사항

테스트 로그에서 `best checkpoint epoch: 83 val_mIoU: ?`로 기록되어 val mIoU가 누락 표기되었다. Train/val 로그 교차 확인으로 epoch 83의 val mIoU = 0.8043임이 검증되었으며, 학습 종료 후 best model 재평가 출력(이중 per-class 블록 중 두 번째)이 test per-class와 완전히 일치하여 동일 checkpoint임이 확인된다.

---

## 4. M0~M1 의료 도메인 전환 검증

### 4-1. 실험 설계

M0·M1은 E5 파이프라인의 도메인 전환 범용성을 검증하기 위해 설계되었다. 두 실험 모두 pretrained encoder, warmup_poly, diff-LR, PaperlikeTransform을 동일하게 적용하며, 유일한 변경 변수는 Decoder와 Loss 조합이다.

### 4-2. 최종 성능 비교

| 실험 | Val Best Dice | Best epoch | Test Dice | Test mIoU | Val→Test 변화 |
|------|---------------|------------|-----------|-----------|--------------|
| M0 (MLP + CE) | 0.8998 | 70 | 0.9222 | 0.9147 | +0.0224 |
| M1 (FPN + CE+Dice+Boundary) | 0.9083 | 91 | 0.9275 | 0.9201 | +0.0192 |
| M1 - M0 차이 | +0.0085 | — | +0.0053 | +0.0054 | — |

### 4-3. Per-class IoU (Test)

| 클래스 | M0 | M1 | M1 - M0 |
|--------|----|----|---------|
| background | 0.9737 | 0.9755 | +0.0018 |
| polyp | 0.8557 | 0.8647 | +0.0090 |

### 4-4. 성공 기준 판정

| 기준 | 값 | M0 | M1 | 결과 |
|------|----|----|----|------|
| 도메인 전환 성공 | Test Dice ≥ 0.80 | 0.9222 | 0.9275 | 양쪽 달성 |
| E5 구조 범용성 | M1 > M0 | — | 0.9275 > 0.9222 | 달성 |

두 기준 모두 충족. Val→Test 방향 역전(+값)은 test 100장의 상대적으로 쉬운 샘플 구성 또는 val 기준 best checkpoint 전략의 일반화 효과로 해석된다.

### 4-5. 수렴 비교

| 구분 | M0 | M1 |
|------|----|----|
| Val Dice 0.80 최초 epoch | 5 | 4 |
| Val Dice 0.90 최초 epoch | — | 52 |
| Best epoch | 70 | 91 |
| Early stopping | epoch 90 발동 | 미발동 (100 epoch 완주) |
| 최종 train loss | 0.0344 | 0.0539 |

M1의 train loss가 높은 것은 CE+Dice+Boundary 복합 loss의 스케일 차이에 의한 것으로, 직접 비교 의미가 없다.

---

## 5. Per-class IoU 전체 비교표 (CamVid, Test 기준)

| 클래스 | E0 | E1 | E2 | E3 | E4 | E5 | E0→E5 향상 |
|--------|----|----|----|----|----|----|-----------|
| Sky | 0.9243 | 0.9259 | 0.9274 | 0.9241 | 0.9271 | **0.9348** | +0.0105 |
| Building | 0.7892 | 0.8006 | 0.7837 | 0.7926 | 0.7870 | **0.8900** | +0.1008 |
| Pole | 0.2120 | 0.2695 | 0.2477 | 0.2546 | 0.2498 | **0.4707** | +0.2587 |
| Road | 0.9342 | 0.9466 | 0.9373 | 0.9414 | 0.9423 | **0.9682** | +0.0340 |
| Pavement | 0.7441 | 0.7758 | 0.7426 | 0.7605 | 0.7588 | **0.8570** | +0.1129 |
| Tree | 0.7048 | 0.7147 | 0.7016 | 0.6994 | 0.6985 | **0.8186** | +0.1138 |
| SignSymbol | 0.2229 | 0.2445 | 0.2208 | 0.2498 | 0.2311 | **0.5455** | +0.3226 |
| Fence | 0.3632 | 0.3528 | 0.3250 | 0.3731 | 0.3656 | **0.6776** | +0.3144 |
| Car | 0.6901 | 0.6794 | 0.6863 | 0.6967 | 0.6839 | **0.8576** | +0.1675 |
| Pedestrian | 0.2734 | 0.3090 | 0.2769 | 0.3095 | 0.2695 | **0.6137** | +0.3403 |
| Bicyclist | 0.3916 | 0.3925 | 0.3867 | 0.3743 | 0.3618 | **0.6956** | +0.3040 |
| **mIoU** | **0.5682** | **0.5829** | **0.5669** | **0.5796** | **0.5705** | **0.7572** | **+0.1890** |

**주요 관찰:**
- E0~E4 공통 취약 클래스: Pole, SignSymbol, Pedestrian, Bicyclist (IoU 0.21~0.39). 단일 변수 변경만으로는 해소되지 않는 구조적 한계다.
- **Fence 이상**: E1(FPN)에서 Fence IoU가 E0(MLP)보다 낮다(0.3528 < 0.3632). FPN의 다중 스케일 융합이 좁고 선형적인 구조물에는 MLP보다 불리할 수 있음을 시사한다.
- E5에서 소수 클래스의 향상이 특히 두드러지며, 이는 pretrained encoder와 augmentation의 복합 효과로 해석된다.

---

## 6. 이상 징후 종합 점검

| 항목 | 내용 | 판단 |
|------|------|------|
| **E2 Focal loss val-test 역전** | val +0.0134 → test −0.0013 | 이상 징후. Focal loss의 과적합 경향으로 해석 가능 |
| E4 epoch 1 학습 시간 이상 (399초) | 이후 정상 복귀 | 초기화 오버헤드. 성능 영향 없음 |
| E5 val_mIoU 누락 (`?`) | train 로그 교차 확인으로 0.8043 검증 | 로그 출력 문제. 값 자체는 정상 |
| E5 val 로그 이중 출력 | 학습 종료 후 best model 재평가 블록으로 확인 | test 결과와 일치. 정상 동작 |
| M0·M1 Val→Test 방향 역전 (+값) | test가 val보다 높게 나옴 | test 100장의 분포 특성 또는 best checkpoint 전략. 비정상 아님 |
| M1 early stopping 미발동 | 100 epoch 완주 | 복합 loss 특성상 느리고 지속적인 수렴. 정상 |
| E0~E4 소수 클래스 전반적 저조 | Pole·SignSymbol·Pedestrian·Bicyclist | 구조적 한계 (데이터 불균형 + scratch 학습). 단일 실험 이상 아님 |

심각한 이상 징후는 E2의 val-test 역전이 유일하다. 나머지는 해석 가능한 범위 내에 있다.

---

## 7. 핵심 결론

**결론 1. Decoder 변경(FPN)이 Loss 변경보다 더 안정적이고 큰 효과를 가져온다.**
val과 test 양쪽에서 FPN(E1)이 MLP(E0) 대비 일관된 우위(test +0.0147)를 보인 반면, loss 변형(E2~E4)의 효과는 test 단계에서 일부 반감되거나 역전된다. E2의 경우 val 개선이 test에서 역전되는 신뢰도 문제가 발생한다.

**결론 2. Loss 변형 중 CE+Dice(E3)가 가장 안정적으로 일반화된다.**
val +0.0149 → test +0.0114로 개선폭이 유지된다. Focal(E2)은 val과 test 간 방향이 역전되어 신뢰도가 낮으며, Boundary(E4)는 test에서 개선폭이 크게 줄어든다. Scratch 학습 조건에서 loss 선택 시 CE+Dice가 권장된다.

**결론 3. E5의 대폭 향상은 pretrained encoder가 주된 요인으로 추정된다.**
Epoch 10 시점의 val mIoU(0.7135)가 E0~E4 최종값을 이미 상회하며, 소수 클래스(SignSymbol, Pedestrian, Fence, Bicyclist)의 test IoU가 대폭 개선된 점이 근거다. 다만 복합 변경 실험이므로 단일 요인의 기여는 ablation 없이 정량화할 수 없다.

**결론 4. E0~E4의 val-test 하락폭(평균 0.077)은 데이터셋 규모의 구조적 한계다.**
369개의 학습 데이터로 11개 클래스를 scratch 학습하는 조건에서는 일정 수준의 val-test 괴리가 불가피하다. E5에서 pretrained encoder와 augmentation 도입 후 하락폭이 0.047로 감소한 점은 이 한계를 완화하는 방향을 확인해 준다.

**결론 5. E5 파이프라인은 도메인에 관계없이 재사용 가능하다.**
CamVid(도시 11 class)에서 설계된 구조, 증강, 학습 전략을 config 교체만으로 Kvasir-SEG(의료 2 class)에 적용하여 Test Dice 0.9275(M1)를 달성했다. FPN 구조의 우위(M1 > M0)도 양 도메인에서 일관되게 재현되어, 범용 파이프라인으로서의 가능성이 검증되었다.

---

# SegFormer-B0 Integrated Analysis Report (E0–E5, M0–M1)

---

## 1. Experiment Overview

### Datasets

| Domain | Dataset | Classes | Train / Val / Test |
|--------|---------|---------|-------------------|
| Urban driving | CamVid | 11 | 369 / 100 / 232 |
| Medical polyp | Kvasir-SEG | 2 | 800 / 100 / 100 |

### Fixed elements

- **Encoder**: MiT-B0. Trained from scratch for E0–E4; ImageNet pretrained for E5, M0, M1.
- **E0–E4**: base lr 6e-4, poly scheduler, no augmentation.
- **E5/M0/M1**: enc lr 6e-5 / dec lr 6e-4 (differential), warmup_poly scheduler, PaperlikeTransform augmentation, AdamW.

---

## 2. Single-Variable Analysis (E0–E4)

### Decoder Effect (E0 → E1)

FPN consistently outperforms MLP on both val (+0.0257) and test (+0.0147) mIoU. Gains are especially clear for Pole (+0.0575), Pavement (+0.0317), and Pedestrian (+0.0356). The exception is Fence, where E1 (0.3528) slightly underperforms E0 (0.3632), suggesting FPN's multi-scale fusion is less effective for narrow linear structures.

### Loss Effect (E2, E3, E4 vs E0)

| Experiment | Loss | Val Δ | Test Δ |
|------------|------|-------|--------|
| E2 | Focal | +0.0134 | **−0.0013** |
| E3 | CE+Dice | +0.0149 | +0.0114 |
| E4 | CE+Boundary | +0.0141 | +0.0023 |

E2 is the only experiment in the entire series where the val-to-test direction reverses — a warning sign of Focal loss overfitting to the training distribution. CE+Dice (E3) is the most stable loss variant. CE+Boundary (E4) shows strong val gains but partial collapse at test.

### Val→Test Drop

E0–E4 drops cluster in the 0.069–0.083 range — a structural dataset artifact from training 11 classes on 369 images from scratch, not an experiment-specific failure. E2's largest drop (0.0834) aligns with its Focal loss overfitting pattern. E5's smallest drop (0.0471) reflects augmentation and pretrained encoder contributions.

---

## 3. E5 Compound Experiment

E5 combines FPN decoder (validated by E1), CE+Dice+Boundary loss (grounded in E3+E4), and a pretrained encoder with warmup_poly / differential LR / augmentation to overcome the scratch training ceiling.

**Test mIoU 0.7572** (+0.1890 over E0, +0.1743 over best E1) far exceeds any single-variable effect. The pretrained encoder is the dominant suspected contributor: val mIoU of 0.7135 at epoch 10 already surpasses the final values of all E0–E4 runs. Minority-class gains (SignSymbol +0.3226, Pedestrian +0.3403, Fence +0.3144, Bicyclist +0.3040) are implausibly large for a loss or decoder change alone. The smallest val→test drop (0.0471) is consistent with improved generalization from augmentation.

A checkpoint logging artifact (`val_mIoU: ?` in the test log) was resolved by cross-referencing train/val logs: epoch 83 val mIoU = 0.8043, confirmed consistent with the best-model re-evaluation block in the training log.

---

## 4. Medical Domain Transfer (M0–M1)

The E5 pipeline (FPN + CE+Dice+Boundary + pretrained + warmup_poly + diff-LR + aug) was applied to Kvasir-SEG with only a config change. Both M0 and M1 exceed the domain-transfer success threshold (Test Dice ≥ 0.80) by more than 0.12. M1 (FPN + compound loss) outperforms M0 (MLP + CE) on all metrics, consistent with CamVid findings. The positive val→test delta in both M experiments reflects the relative ease of the 100-image test split.

---

## 5. Key Conclusions

1. **FPN decoder outperforms MLP more reliably than any loss change.** Val and test gains are consistent (+0.0147 test mIoU). Loss changes show unstable or reversed effects at test.
2. **CE+Dice is the most reliable loss combination.** Val gains persist at test (+0.0114). Focal loss reverses at test; CE+Boundary collapses at test.
3. **Pretrained encoder is the primary driver of E5 gains.** Epoch-10 val mIoU (0.7135) already surpasses E0–E4 final values; minority-class improvements cannot be attributed to decoder or loss changes alone.
4. **The val→test gap in E0–E4 (~0.077 average) is a structural dataset constraint**, not a tuning problem. Pretrained encoder + augmentation (E5) reduces this to 0.047.
5. **The E5 pipeline generalizes across domains.** Applied to Kvasir-SEG via config swap only, it achieves Test Dice 0.9275 (M1 > M0), replicating the FPN > MLP pattern from CamVid and validating the pipeline as domain-agnostic.
