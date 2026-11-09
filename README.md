# How Federated Learning Works

**연합 학습은 어떻게 동작하는가 — LINE 스티커 추천으로 보는 Federated Learning.** LINE에서 메시지를 입력하면 키보드 위에 스티커가 뜨는데, 그 순서를 정하는 모델은 사용자가 무엇을 눌렀는지 서버가 한 번도 보지 않은 채 학습됩니다. LINE의 Haruka Kikuchi가 Tech-Verse 2022에서 발표한 「[연합 학습을 이용한 LINE 스티커 추천](https://www.youtube.com/watch?v=v-VwByrji0c)」을 따라가며, 그 구조를 이루는 부품을 브라우저에서 직접 돌려 보는 인터랙티브 교육 자료입니다.

축이 되는 문장은 하나입니다. **연합 학습은 “데이터를 모으는 대신 모델을 모은다”이고, 그 대가는 통제권이다.**

🔗 **[데모 바로가기](https://dev-jonghoonpark.github.io/how-federated-learning-works/)**

> [how-ai-works](https://github.com/dev-jonghoonpark/how-ai-works) 시리즈의 일부입니다.
> 앞서 읽으면 좋은 자료: [How SGD Works](https://github.com/dev-jonghoonpark/how-sgd-works) — 미니배치 기울기 평균과 동기·비동기 SGD

## 다루는 내용

| 절 | 주제 | 인터랙티브 데모 |
|---|---|---|
| §1 | ML을 어디서 돌릴 것인가 — 서버 중심 ML · 온디바이스 추론 · 연합 학습 | **데이터 흐름 애니메이션** — 배치마다 무엇이 네트워크를 건너고 서버가 무엇을 보는지 |
| §2 | 적용 대상 — 스티커 자동 추천과 의미 태그, 스티커 프리미엄 | **입력 → 태그 → 스티커** — 「あり」에서 「ありえ」로 바뀔 때 태그가 바뀌는 것 |
| §3 | 하이브리드 2단계 — 후보 생성은 서버, 재정렬은 폰 | 1단계 vs 2단계 비교표 |
| §4 | 1단계 후보 생성 — 사용자·아이템 임베딩, 사용자 클러스터링 | **k-평균 클러스터 실험** — k에 따라 “내 top-N”과 클러스터 후보가 얼마나 겹치는가, 목록 수 환산 |
| §5 | 2단계 재정렬 — 폰 안에서 노출·클릭으로 학습 | **한 사용자의 폰** — 입력 시뮬레이션, 유휴 학습, 키보드 8칸 순위 변화와 기대 클릭 |
| §6 | 연합 학습 한 바퀴 — FedAvg, 비동기 수집 | **FedAvg 시뮬레이터** — K·C·E·ε를 바꿔 40라운드를 실제로 돌리고 중앙 집중·각자 로컬만·상한과 비교 |
| §7 | 플랫폼으로 만들기 — LFL의 네 칸 구조 | **추론 경로 / 학습 경로 단계 실행기** — 슬라이드의 시스템 구조도를 상자 단위로 따라가기 |
| §8 | 로컬 차등 프라이버시 — 클리핑과 가우시안 메커니즘, ε | **구별 불가능성 실험** — 해석적 가우시안 메커니즘으로 σ 계산, 공격자 정확도 Φ(S/σ), 평균의 노이즈 σ/√m |
| §9 | 결과(+5.6%)와 질의응답에서 나온 설계 결정 | **보조 기기 계산기** — 기기 수 × 업로드 횟수만큼 쌓이는 ε |
| §10 | 정리 + 헷갈리기 쉬운 것 6가지 | — |
| §11 | 확인 퀴즈 | 9문항 |

## 이 페이지가 답하는 질문

- 원본 로그를 안 보내는데 어떻게 모두의 로그로 학습한 것과 비슷한 모델이 나오는가?
- 왜 추천을 한 단계로 하지 않고 서버(후보 생성)와 폰(재정렬)으로 나누는가?
- 사용자 임베딩을 왜 클러스터링하는가? 그러면 개인화는 어디서 되찾는가?
- 폰마다 다른 모델이 생기는가? (아니다 — 모델은 공통, 개인화는 입력인 사용자 임베딩이 맡는다)
- 연합 학습이면 프라이버시가 보장되는가? (아니다 — 그래서 로컬 DP를 따로 넣는다)
- ε을 작게 하면 무엇을 잃고, 무엇으로 되찾는가? (평균의 노이즈 σ/√m — 사용자 수)
- iPad 같은 보조 기기에서는 왜 학습하지 않는가?

## 실행

빌드 과정 없는 단일 `index.html`입니다. 브라우저로 열면 됩니다.

```bash
open index.html
```

라이트/다크 테마를 지원하며, 수식 렌더링(KaTeX) 외에는 외부 라이브러리를 쓰지 않습니다.

## 숫자에 대하여

발표 내용(구조, 1,000,000 → 100 후보, 노출·클릭 신호, 가우시안 메커니즘, 사용자 ID 없는 업로드, A/B +5.6%, 보조 기기 미학습 등)은 발표 슬라이드와 질의응답을 따랐습니다. 데모의 사용자·스티커·클릭은 그 구조를 따라 만든 **합성 데이터**이며 LINE의 실제 데이터가 아닙니다. 재정렬 모델은 `s(u,i) = a·(e_u·e_i) + w·e_i + b`라는 파라미터 8개짜리 작은 모델이고, 시뮬레이션 사용자는 1단계가 아는 취향 + 1단계가 모르는 공통 방향 + 위치 편향으로 스티커를 누릅니다. 데모의 향상률(예: +16%)은 이 장난감 세계의 값이지 발표의 5.6%와 비교할 수 있는 값이 아닙니다. 모든 학습·평가·σ 계산은 브라우저 안에서 실제로 돌린 결과입니다.

## 참고

- Haruka Kikuchi, *A Sticker Recommendation Using Federated Learning*, Tech-Verse 2022 — [한국어판 영상](https://www.youtube.com/watch?v=v-VwByrji0c)
- H. B. McMahan 외 (2017), *Communication-Efficient Learning of Deep Networks from Decentralized Data* — [arXiv:1602.05629](https://arxiv.org/abs/1602.05629)
- K. Bonawitz 외 (2019), *Towards Federated Learning at Scale: System Design* — [arXiv:1902.01046](https://arxiv.org/abs/1902.01046)
- P. Kairouz, H. B. McMahan 외 (2021), *Advances and Open Problems in Federated Learning* — [arXiv:1912.04977](https://arxiv.org/abs/1912.04977)
- P. Covington, J. Adams, E. Sargin (2016), *Deep Neural Networks for YouTube Recommendations*
- B. Balle & Y.-X. Wang (2018), *Improving the Gaussian Mechanism for Differential Privacy* — [arXiv:1805.06530](https://arxiv.org/abs/1805.06530)
- Ú. Erlingsson 외 (2019), *Amplification by Shuffling* — [arXiv:1811.12469](https://arxiv.org/abs/1811.12469)
