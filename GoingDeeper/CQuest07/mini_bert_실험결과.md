# Mini BERT 프로젝트 정리

## 1. 프로젝트 개요

| 항목 | 내용 |
|---|---|
| 목표 | vocab_size 8000, 파라미터 ~1M의 mini BERT 모델 제작 및 학습 |
| 데이터 | 한국어 위키백과 (kowiki.txt), 128,000건 사용 |
| 학습 | 10 Epoch, batch_size 64, CosineSchedule |
| 환경 | PyTorch, SentencePiece, JupyterLab |

---

## 2. BERT 핵심 개념 요약

### BERT란?
- **B**idirectional **E**ncoder **R**epresentations from **T**ransformers
- 텍스트를 양방향으로 읽어 문맥을 파악하는 사전학습 언어모델
- Google이 2018년 발표, 이후 NLP 분야의 표준이 됨

### GPT와의 차이

| 항목 | BERT | GPT |
|---|---|---|
| 읽는 방향 | 양방향 (←→) | 단방향 (→) |
| 구조 | Encoder | Decoder |
| 학습 방식 | MLM + NSP | 다음 단어 예측 |
| 강점 | 이해(Understanding) | 생성(Generation) |
| 대표 제품 | 구글 검색 | ChatGPT |

### 임베딩 레이어 구조
BERT는 3가지 임베딩을 합산해서 입력을 만듦

```
Token Embedding    (단어 자체의 의미)
+ Position Embedding (몇 번째 위치인지, 학습 가능한 파라미터)
+ Segment Embedding  (문장 A=0, 문장 B=1)
= 최종 임베딩 → Encoder 입력
```

---

## 3. 데이터 전처리

### (1) MLM 마스킹 규칙

```
전체 토큰의 15% 선택
        │
        ├── 80% → [MASK] 토큰으로 교체
        ├── 10% → 랜덤 토큰으로 교체
        └── 10% → 원래 토큰 그대로 유지
```

- 띄어쓰기 단위(word-level)로 마스킹 → subword 일관성 유지
- 예: "대한" "민국" 중 하나만 마스킹하면 너무 쉬움 → 단어 단위로 묶어서 처리

### (2) NSP 쌍 생성

```
[CLS] + 문장A + [SEP] + 문장B + [SEP]
  └─ Segment: A=0, B=1
```

- 50% → 실제 이어지는 문장 (is_next=1, TRUE)
- 50% → A/B 순서 swap (is_next=0, FALSE)

### (3) Instance 구조

```python
{
    "tokens"    : ['[CLS]', '▁추적', ..., '[SEP]', ...],
    "segment"   : [0, 0, ..., 1, 1],
    "is_next"   : 1,          # NSP 정답
    "mask_idx"  : [3, 7, 11], # 마스킹된 위치
    "mask_label": ['▁비가', '▁손님', '▁많아']  # MLM 정답
}
```

### (4) 데이터셋 규모

| 항목 | 값 |
|---|---|
| 전체 corpus | kowiki.txt |
| 사용 건수 | 128,000건 |
| 시퀀스 길이 | 128 토큰 |
| 저장 방식 | np.memmap (메모리 효율화) |

---

## 4. 모델 구조

### Mini BERT Config

| 파라미터 | 값 | BERT-base 비교 |
|---|---|---|
| d_model | 64 | 768 |
| n_layer | 4 | 12 |
| n_head | 4 | 12 |
| d_ff | 256 | 3072 |
| n_vocab | 8007 | 30522 |
| n_seq | 128 | 512 |
| 총 파라미터 | 725,120 (0.73M) | 110M |

### 전체 모델 구조

```
PreTrainModel
    │
    ├── BERT
    │     ├── SharedEmbedding    (Token, Weight Sharing)
    │     ├── PositionEmbedding  (Learned, 학습 가능)
    │     ├── nn.Embedding(2)    (Segment A=0, B=1)
    │     ├── LayerNorm
    │     └── EncoderLayer × 4
    │           ├── MultiHeadAttention (4 heads)
    │           │     └── ScaleDotProductAttention
    │           ├── LayerNorm × 2
    │           ├── PositionWiseFeedForward (GELU)
    │           └── Dropout × 2
    │
    └── PooledOutput (NSP Head)
          ├── Linear(64→64) + tanh
          └── Linear(64→2)
```

---

## 5. 학습 설정

| 항목 | 값 |
|---|---|
| Optimizer | Adam (lr=1e-4) |
| LR Scheduler | CosineSchedule (warmup + cosine decay) |
| Epochs | 10 |
| Batch size | 64 |
| Max LR | 2.5e-4 |
| NSP Loss | CrossEntropyLoss |
| MLM Loss | CrossEntropyLoss (PAD 제외, ×20 스케일링) |

### MLM Loss ×20 이유
MLM loss 스케일이 NSP보다 작아지는 경향이 있어서 균형을 맞추기 위해 20배 증폭

---

## 6. 학습 결과 (Mini BERT, 10 Epoch)

| Epoch | NSP Loss | MLM Loss | NSP Acc | MLM Acc |
|---|---|---|---|---|
| 1 | 0.6931 | - | 62.7% | 7.3% |
| 10 | 0.5789 | 17.60 | 61.2% | 6.8% |

> 모델이 작아서 MLM 성능이 낮지만 loss는 안정적으로 수렴함

---

## 7. 인퍼런스 결과

### MLM 테스트

| 문장 | 정답 | 예측 Top-1 | 평가 |
|---|---|---|---|
| 대한민국의 수도는 [MASK]이다 | 서울 | 하는 | ❌ |
| 조선은 [MASK]년에 건국 | 1392 | . | ❌ |
| 인공지능은 미래의 [MASK] | 핵심 | . | ❌ |

### NSP 테스트

| 문장 A | 문장 B | 예상 | 예측 | 확률 |
|---|---|---|---|---|
| 서울은 대한민국의 수도이다 | 인구는 약 천만 명이다 | TRUE | ✅ TRUE | 0.61 |
| 고양이가 낮잠을 자고 있다 | 우주선이 화성에 착륙했다 | FALSE | ❌ TRUE | 0.51 |
| 그는 열심히 공부했다 | 결국 시험에 합격하였다 | TRUE | ❌ FALSE | 0.46 |

---

## 8. Ablation Study — 모델 크기별 성능 비교

### 실험 조건

| 실험명 | d_model | n_layer | n_head | 파라미터 |
|---|---|---|---|---|
| Tiny (베이스라인) | 64 | 4 | 4 | 725K (0.73M) |
| Small | 128 | 4 | 4 | 1,851K (1.85M) |
| Medium | 128 | 6 | 4 | 2,248K (2.25M) |
| Large | 256 | 6 | 8 | 6,888K (6.89M) |

### 최종 성능 비교 (10 Epoch 기준)

| 실험명 | 파라미터 | NSP Loss | MLM Loss | NSP Acc | MLM Acc |
|---|---|---|---|---|---|
| Tiny | 725K | 0.5789 | 17.60 | 61.2% | 6.8% |
| Small | 1,851K | 0.5590 | 14.76 | 62.8% | 14.5% |
| Medium | 2,248K | 0.5559 | 14.43 | 63.1% | 14.9% |
| Large | 6,888K | 0.5347 | 12.77 | 64.3% | 20.7% |

### 모델별 실패 패턴 비교 ("대한민국의 수도는 [MASK]이다")

| 모델 | 예측 Top-5 | 실패 패턴 |
|---|---|---|
| Tiny | `.` `,` `의` `년` `이` | 문장부호/조사 → 문맥 전혀 못 읽음 |
| Small | `▁다음을` `▁뜻은` `▁가리키는` | 위키 문체 패턴 학습, 의미는 모름 |
| Medium | `▁다음을` `▁다른` `▁다음과` | Small과 유사한 수준 |
| Large | `▁대한민국의` `▁일본의` `▁러시아` | 국가명 등장 → 방향은 맞기 시작 |

---

## 9. 핵심 인사이트

### ① MLM이 NSP보다 모델 크기에 훨씬 민감하다
- NSP: Tiny→Large 61.2%→64.3% (+3.1%p)
- MLM: Tiny→Large 6.8%→20.7% (+13.9%p, 약 3배)
- NSP는 단순 이진 분류라 작은 모델도 어느 정도 풀 수 있지만,
  MLM은 진짜 언어 이해가 필요해서 모델 크기 영향이 큼

### ② 레이어 수(depth)보다 임베딩 차원(width)이 더 중요하다
- Small(128, 4layer) vs Medium(128, 6layer): MLM Acc 14.5% vs 14.9%
- 파라미터 40만 개 차이에도 성능 차이는 0.4%p에 불과
- 이 데이터 규모에서는 깊이보다 너비(d_model)가 핵심

### ③ 모델이 클수록 더 의미있는 방향으로 실패한다 (Scaling Law)
- Tiny: 문장부호/조사 → 문맥 無
- Large: 국가명/역사 단어 → 문맥 有, 정답은 못 맞히지만 방향은 맞음
- 이것이 스케일링 법칙(Scaling Law)의 핵심:
  모델이 커질수록 더 의미있는 표현을 학습

### ④ 모든 모델이 10 epoch에서도 수렴 중
- Large의 MLM Acc는 4~6 epoch 사이 급격히 점프
- 더 많은 epoch을 학습하면 격차가 더 벌어질 가능성 높음

---

## 10. 한계 및 개선 방향

| 한계 | 개선 방향 |
|---|---|
| 파라미터 부족 (최대 6.89M) | 최소 10M 이상으로 증가 |
| 학습 데이터 부족 (128K건) | 전체 kowiki 사용 (약 900K건) |
| Epoch 부족 (10 epoch) | 최소 50 epoch 이상 |
| NSP 성능 정체 (61~64%) | RoBERTa처럼 NSP 제거 실험 |
| vocab_size 8000 (작음) | 32000으로 증가 시 표현력 향상 |

---

## 11. 추가 Ablation Study 아이디어

1. **NSP 제거** — RoBERTa 논문 검증 (NSP가 오히려 해롭다는 주장)
2. **마스킹 비율 변화** — 0.10 / 0.15 / 0.20 / 0.30 비교
3. **MLM 80/10/10 비율 변화** — MASK 100% vs 현재 방식
4. **LR 스케줄러 변화** — CosineSchedule vs 고정 LR vs Linear Decay
