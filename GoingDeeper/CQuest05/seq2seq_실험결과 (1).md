# 한국어→영어 Seq2seq 번역기 실험 결과

## 📌 프로젝트 개요

한국어를 영어로 번역하는 **Attentional Seq2seq 모델** 실험 결과를 정리한 문서입니다.

- **데이터셋**: korean-english-park (jungyeul/korean-parallel-corpora)
- **모델 구조**: GRU 기반 Encoder-Decoder + Attention
- **학습 환경**: CUDA GPU, PyTorch 2.7.1

---

## ⚙️ 실험 설정 비교

| 구분 | 실험 1 | 실험 2 | 실험 3 |
|------|--------|--------|--------|
| Attention | Bahdanau | Luong | Luong |
| 한국어 토큰화 | 공백 기준 | 공백 기준 | **Mecab 형태소 분석** |
| 학습 데이터 수 | 70,141개 | 70,141개 | 61,915개 |
| 한국어 사전 크기 | 69,181개 | 69,181개 | **24,244개** |
| 영어 사전 크기 | 26,573개 | 26,573개 | 24,341개 |
| Embedding 차원 | 256 | 256 | 256 |
| Hidden 차원 | 512 | 512 | 512 |
| Batch Size | 64 | 64 | 64 |
| Epochs | 20 | 20 | 20 |
| Optimizer | Adam (lr=1e-3) | Adam (lr=1e-3) | Adam (lr=1e-3) |

---

## 📉 Training Loss 비교

| Epoch | 실험 1 (Bahdanau) | 실험 2 (Luong) | 실험 3 (Luong + Mecab) |
|-------|------------------|----------------|------------------------|
| 1 | 5.6300 | 5.6420 | **5.5857** |
| 2 | - | - | 4.5304 |
| 3 | - | - | 3.9397 |
| 4 | - | - | 3.5258 |
| 5 | 3.4174 | 3.4461 | **3.2391** |
| 6 | - | - | 3.0466 |
| 7 | - | - | 2.9029 |
| 8 | - | - | 2.7915 |
| 9 | - | - | 2.7025 |
| 10 | 2.7482 | 2.7850 | **2.6243** |
| 11 | - | - | 2.5630 |
| 12 | - | - | 2.5064 |
| 13 | - | - | 2.4570 |
| 14 | - | - | 2.4144 |
| 15 | 2.4462 | 2.4871 | **2.3756** |
| 16 | - | - | 2.3398 |
| 17 | - | - | 2.3099 |
| 18 | - | - | 2.2789 |
| 19 | - | - | 2.2519 |
| 20 | 2.3143 | 2.2818 | **2.2264** |

---

## 🔤 번역 결과 비교

### K1) 오바마는 대통령이다.

| 구분 | 번역 결과 |
|------|----------|
| 정답 | obama is the president . |
| 실험 1 (Bahdanau + 공백) | obama is approaching the washington post . |
| 실험 2 (Luong + 공백) | obama is poised to be released in washington . |
| 실험 3 (Luong + Mecab) | obama displayed the head of the president . |

### K2) 시민들은 도시 속에 산다.

| 구분 | 번역 결과 |
|------|----------|
| 정답 | people live in the city . |
| 실험 1 (Bahdanau + 공백) | the locals is buried in the town of `<unk>` . |
| 실험 2 (Luong + 공백) | the city s locals was blown out . |
| 실험 3 (Luong + Mecab) | citizens are cities , and the city s fear is greatest modern . |

### K3) 커피는 필요 없다.

| 구분 | 번역 결과 |
|------|----------|
| 정답 | coffee is not needed . |
| 실험 1 (Bahdanau + 공백) | no . |
| 실험 2 (Luong + 공백) | it s not a little thing . |
| 실험 3 (Luong + Mecab) | **the coffee cannot be coffee .** ✅ |

### K4) 일곱 명의 사망자가 발생했다.

| 구분 | 번역 결과 |
|------|----------|
| 정답 | seven people were killed . |
| 실험 1 (Bahdanau + 공백) | **seven people were killed in the blast .** ✅ |
| 실험 2 (Luong + 공백) | the deaths occurred at . |
| 실험 3 (Luong + Mecab) | the emergency decree was killed in the air force . |

---

## 📊 전체 실험 요약

| 구분 | 실험 1 | 실험 2 | 실험 3 |
|------|--------|--------|--------|
| Attention | Bahdanau | Luong | Luong |
| 토큰화 | 공백 | 공백 | **Mecab** |
| 한국어 사전 | 69,181 | 69,181 | **24,244** |
| Epoch 20 Loss | 2.3143 | 2.2818 | **2.2264** |
| 최고 번역 | K4 (사망자) | - | K3 (커피) |
| 훈련 시간 | - | - | **94.2분** |

---

## 🔍 결론 및 분석

### 핵심 발견

- **형태소 분석이 가장 큰 성능 개선 요인**
  - 한국어 사전 크기를 69,181개 → 24,244개로 약 **65% 감소**
  - 조사가 분리되어 동일 어근의 토큰이 통합됨
- Bahdanau vs Luong 차이보다 **전처리 품질이 더 큰 영향**을 미침
- 실험 3에서 커피 예문에 **coffee** 단어가 정확히 등장 → 형태소 분석 효과 확인
- 실험 1이 K4 예문에서 가장 자연스러운 번역 생성

### 향후 개선 방향

- Mecab 정식 설치 후 재실험 (현재 python-mecab-ko 대체 사용 중)
- Epoch 추가 (30~50 epoch)
- Embedding / Hidden 차원 확대 (512 / 1024)
- Beam Search 디코딩 적용
- Teacher Forcing 비율 조정
