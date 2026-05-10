# SegFormer-B0 기반 Semantic Segmentation 실험 문서

## Abstract | 초록

SegFormer-B0의 경량 인코더(MiT-B0)를 고정한 채 디코더 구조(MLP vs FPN)와 손실 함수(CE / Focal / Dice / Boundary 조합)를 체계적으로 변경하여 각 요소의 독립적 기여도를 분석한다. CamVid(11 class 도시 도로)와 Kvasir-SEG(2 class 의료 폴립) 두 도메인에서 실험을 수행하였으며, 단일 변수 원칙(E0~E5)과 도메인 전환 검증(M0~M1)을 통해 범용 파이프라인의 가능성을 확인한다.

## Methods | 실험 설계 원칙

- **인코더 고정 원칙**: MiT-B0 구조 및 ImageNet pretrained 가중치 고정
- **단일 변수 원칙**: 실험당 Decoder 또는 Loss 중 하나만 변경
- **공정 비교 원칙**: 동일 데이터셋·학습 스케줄·증강 조건 유지 (E0~E4)
- **E5**: E0~E4 결과에 근거한 복합 실험 (FPN + CE+Dice+Boundary + pretrained + warmup_poly + diff-LR + aug)

## Experiment Index | 실험 목록

| 실험 | 도메인 | Decoder | Loss | 주요 변수 | Test mIoU / Dice |
|------|--------|---------|------|-----------|-----------------|
| [E0](experiments/E0_baseline/) | CamVid | MLP | CE | baseline | 0.5682 |
| [E1](experiments/E1_fpn_ce/) | CamVid | FPN | CE | decoder | 0.5829 |
| [E2](experiments/E2_mlp_focal/) | CamVid | MLP | Focal | loss | 0.5669 |
| [E3](experiments/E3_mlp_ce_dice/) | CamVid | MLP | CE+Dice | loss | 0.5796 |
| [E4](experiments/E4_mlp_ce_boundary/) | CamVid | MLP | CE+Boundary | loss | 0.5705 |
| [E5](experiments/E5_composite/) | CamVid | FPN | CE+Dice+Boundary | 복합 | 0.7572 |
| [M0](experiments/M0_mlp_ce/) | Kvasir-SEG | MLP | CE | baseline (의료) | 0.9222 (Dice) |
| [M1](experiments/M1_fpn_composite/) | Kvasir-SEG | FPN | CE+Dice+Boundary | 복합 (의료) | 0.9275 (Dice) |

## Key Results | 핵심 결과

### 도메인 간 구조적 우위 비교 — Baseline(MLP+CE) vs Proposed(FPN+Compound)

(A) CamVid Val mIoU, (B) CamVid Test mIoU, (C) Kvasir Test Dice, (D) Kvasir Polyp IoU — 4개 패널을 한 줄에 배치하여 두 도메인에 걸친 구조적 우위를 단일 시선으로 확인한다. 각 패널에서 Baseline(MLP+CE, 회색)과 Proposed(FPN+Compound, 파랑)를 나란히 비교하며, bar 사이에 Δ 값을 녹색 배지로 표시(+0.1674, +0.1890, +0.0053, +0.0090). 모든 패널에서 Proposed가 Baseline을 상회한다.

![Cross-Domain Structural Advantage](figure/4panel_cross_domain_comparison.png)

---

### CamVid (E0~E5) — Test mIoU

| 실험 | Test mIoU | E0 대비 |
|------|-----------|---------|
| E0 (baseline) | 0.5682 | — |
| E1 (FPN+CE) | 0.5829 | +0.0147 |
| E2 (MLP+Focal) | 0.5669 | −0.0013 |
| E3 (MLP+CE+Dice) | 0.5796 | +0.0114 |
| E4 (MLP+CE+Boundary) | 0.5705 | +0.0023 |
| **E5 (복합)** | **0.7572** | **+0.1890** |

#### Val vs Test mIoU — 검증 기반 Loss 평가의 신뢰도

좌측 패널(A)에서 E0~E4 각 실험의 Val mIoU와 Test mIoU를 같은 축 위에 나란히 배치하고, 우측 패널(B)에서 E0 대비 Δ(Val, Test)를 표시하여 방향 일치 여부를 직관적으로 드러낸다. E2(Focal)만 Val +0.0134로 "개선"으로 보이지만 Test −0.0013으로 "열화"되는 역전 현상이 빨간 박스로 강조된다.

![Val vs Test mIoU Trustworthiness](figure/e0_e4_val_vs_test.png)

#### E0~E4 Per-class IoU Heatmap (CamVid Test Set)

좌측(A): E0~E4 × 11 classes 절대 IoU 히트맵. 공통 취약 클래스(Pole, SignSymbol, Pedestrian, Bicyclist)는 빨간 박스로 강조. 우측(B): E0 대비 Δ IoU diverging 히트맵. E1(FPN) 행을 초록 박스로 강조하여 클래스 전반에 걸친 일관된 개선을 시각화 — Pole +0.058, Pavement +0.032 등 셀 단위로 확인 가능.

![E0~E4 Per-class IoU Heatmap](figure/e0_e4_per_class_heatmap.png)

#### E0 → E5 Per-class IoU 향상 (CamVid Test)

좌측(A): 11개 클래스를 Δ 크기 오름차순으로 정렬하여 E0와 E5의 IoU를 그룹 bar로 비교. 우측(B): Δ IoU(E5 − E0) 수평 bar에 전체 평균 Δ +0.189를 빨간 점선으로 표시. 소수 클래스(Pedestrian, SignSymbol, Fence, Bicyclist, Pole)는 빨간 라벨로 강조되며, 이들의 개선 폭(Pedestrian +0.340, SignSymbol +0.323, Fence +0.314, Bicyclist +0.304, Pole +0.259)이 전체 평균을 압도적으로 상회한다.

![E0 to E5 Per-class IoU Improvement](figure/e0_to_e5_per_class.png)

---

### Kvasir-SEG (M0~M1) — Test Dice

| 실험 | Test Dice | Test mIoU |
|------|-----------|-----------|
| M0 (MLP+CE) | 0.9222 | 0.9147 |
| **M1 (FPN+CE+Dice+Boundary)** | **0.9275** | **0.9201** |

#### Kvasir-SEG 학습 수렴 비교 — M0 vs M1

2×2 서브플롯: (A) Val Dice, (B) Polyp IoU, (C) Training Loss, (D) Val mIoU. M0(회색)은 epoch 70에서 best Dice 0.8998 달성 후 개선 없어 epoch 90에서 early stopping. M1(파랑)은 epoch 91까지 best를 갱신(0.9083)하며 100 epoch을 완주하며, 연두색 점선으로 M0 early stop 지점을 표시한다. Polyp IoU(B)에서도 M1이 후반부로 갈수록 M0을 안정적으로 상회한다.

![Kvasir-SEG Training Dynamics](figure/m0_vs_m1_convergence.png)

## Conclusions | 핵심 결론

1. **Decoder 변경(FPN)이 Loss 변경보다 안정적이고 일관된 효과**를 보인다 (E1 > E0, val·test 모두).
2. **Loss 변형 중 CE+Dice(E3)가 가장 안정적**으로 일반화된다. Focal(E2)은 val→test에서 역전 현상 발생.
3. **E5의 대폭 향상은 pretrained encoder가 주된 요인**으로 추정된다 (epoch 10 val mIoU 0.7135 = E0~E4 최종값 상회).
4. **E5 파이프라인은 도메인에 무관하게 재사용 가능**하다 (CamVid→Kvasir-SEG config 교체만으로 동작).

## Repository Structure | 리포 구조

```
experiments/E{N}_{name}/README.md   ← 실험별 상세 결과
analysis/final_report.md            ← E0~E5 통합 분석
```

---

# SegFormer-B0 Semantic Segmentation — Experiment Docs

## Abstract

We systematically vary the decoder architecture (MLP vs FPN) and loss function (CE / Focal / Dice / Boundary combinations) while keeping the SegFormer-B0 encoder (MiT-B0) frozen. Experiments run on two domains: CamVid (11-class urban driving) and Kvasir-SEG (2-class medical polyp). A single-variable principle is strictly enforced for E0–E4; E5 is a compound experiment grounded in E0–E4 findings.

## Key Findings

- FPN decoder consistently outperforms MLP across val and test sets (+0.0147 mIoU on CamVid test).
- CE+Dice is the most stable loss combination; Focal loss shows val→test reversal.
- E5 (pretrained encoder + FPN + compound loss + augmentation) achieves 0.7572 test mIoU, +0.189 over baseline.
- The same pipeline transfers to the medical domain (Kvasir-SEG) with only a config change, achieving 0.9275 Dice.
