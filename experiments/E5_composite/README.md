# E5: Composite (FPN + CE+Dice+Boundary + pretrained + warmup_poly + diff-LR + aug)

| | |
|---|---|
| **도메인 / Domain** | CamVid (11 class) |
| **Decoder** | FPN |
| **Loss** | CE+Dice+Boundary |
| **학습 전략** | pretrained encoder + warmup_poly + diff-LR + PaperlikeTransform aug |
| **파라미터 수** | — |
| **역할** | 복합 실험 |

## 성능 / Performance

| 지표 | Val Best | Test |
|------|----------|------|
| mIoU | 0.8043 (epoch 83) | 0.7572 |
| Val→Test 하락폭 | — | −0.0471 |

## Per-class IoU (Test)

| Class | IoU |
|-------|-----|
| Sky | 0.9348 |
| Building | 0.8900 |
| Pole | 0.4707 |
| Road | 0.9682 |
| Pavement | 0.8570 |
| Tree | 0.8186 |
| SignSymbol | 0.5455 |
| Fence | 0.6776 |
| Car | 0.8576 |
| Pedestrian | 0.6137 |
| Bicyclist | 0.6956 |

## 학습 곡선 / Training Curves

<!-- No assets available yet -->

## 분석 요약 / Summary

E0~E4의 단일 변수 결과를 기반으로 최적 구성을 결합한 복합 실험이다. Decoder는 E1에서 검증된 FPN, Loss는 E3·E4를 종합한 CE+Dice+Boundary, 학습 전략은 scratch의 한계를 극복하기 위해 ImageNet pretrained encoder, warmup_poly 스케줄러, 인코더-디코더 차등 학습률, PaperlikeTransform 증강을 도입했다. Test mIoU 0.7572는 E0 대비 +0.1890, E0~E4 최고인 E1 대비 +0.1743의 향상이다. Epoch 10(warmup 완료 시점)에서 이미 val mIoU 0.7135를 달성하여 E0~E4의 최종값을 상회함으로써, 성능 향상의 주된 요인은 pretrained encoder의 전이 학습 효과로 추정된다. Val→Test 하락폭(0.0471)이 E0~E4 평균(0.077)보다 유의미하게 작으며, 소수 클래스(SignSymbol 0.5455, Pedestrian 0.6137, Fence 0.6776, Bicyclist 0.6956)의 대폭 향상이 두드러진다.

**체크포인트 비고**: 테스트 로그에서 `best checkpoint epoch: 83 val_mIoU: ?`로 기록되어 val mIoU가 누락 표기되었으나, train/val 로그 교차 확인으로 epoch 83의 val mIoU = 0.8043임이 검증되었다.

E5 combines the best elements from E0–E4: FPN decoder (E1), CE+Dice+Boundary loss (E3+E4), plus pretrained encoder, warmup_poly scheduler, differential learning rates, and augmentation to overcome scratch training limits. Test mIoU of 0.7572 (+0.1890 over E0) far exceeds any single-variable gain. The pretrained encoder is the dominant contributor: at epoch 10 (end of warmup), val mIoU already reaches 0.7135, surpassing the final val values of all E0–E4 runs. Minority class IoU improvements are dramatic and persist through test evaluation, and the val→test drop (0.0471) is the smallest in the series.
