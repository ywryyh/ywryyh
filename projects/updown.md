# UpDown — 뉴스가 시세를 움직이는 모의투자 게임

> 2026.04.06 – 05.08 · 6인 팀 · IBM x RedHat AX Academy 중간 프로젝트
> **기획서 · 뉴스–시세 연동 · 뉴스·차트 API · 거래 패널 모달**
> [레포](https://github.com/jinwoong96/IBM-Redhat-6th-midterm-team-project) (개인 브랜치 `yoonho`)

가상의 하루가 지날 때마다 새 뉴스가 나오고, 그 뉴스에 따라 다음 날 주가가 오르내리는 모의투자 게임입니다. 뉴스를 읽고 종목을 사고팔아 수익률 순위를 겨룹니다.

## 맡은 일

| 구분 | 내용 | 근거 |
|---|---|---|
| 기획 | 기획서 작성. 뉴스에 따라 실전에 가까운 주가 변동이 일어나도록 규칙 기획 (금융경제학 전공 활용) | 팀 Notion 역할표 |
| 데이터 | 실제 종목 60개(종목코드·업종), 뉴스 90개, 뉴스–업종 등락 규칙 273개(뉴스별 영향 업종과 등락 범위), 종목별 30일치 초기 시세 | 팀 Notion 더미 데이터(xlsx). 레포에는 데이터 파일 없음 |
| 뉴스–시세 엔진 | 사용자가 안 본 뉴스를 무작위로 배정 → 그 뉴스의 영향 업종을 찾음 → 업종별 등락 범위 안에서 무작위 등락 → 사용자별 다음 날 시세 생성 | [c1b1ce1](https://github.com/jinwoong96/IBM-Redhat-6th-midterm-team-project/commit/c1b1ce14f82600fcb64dcb833466202ca758e2f2) · [1a0f99d](https://github.com/jinwoong96/IBM-Redhat-6th-midterm-team-project/commit/1a0f99da5ac5b200679601535ed9c9ce2bf4cbe6) |
| 뉴스·차트 API | 사용자별 뉴스 목록, 지난 뉴스 다시 보기, 일자별 시세 차트 데이터 | [0195333](https://github.com/jinwoong96/IBM-Redhat-6th-midterm-team-project/commit/0195333acdfaa1ad519e39e733e9bc37df1d57da) · [2e6b84c](https://github.com/jinwoong96/IBM-Redhat-6th-midterm-team-project/commit/2e6b84c0ffe1f3999d867a5f67ab7b39f2df4561) |
| 거래 패널 | 매수·매도 모달 (React) | [9722c2d](https://github.com/jinwoong96/IBM-Redhat-6th-midterm-team-project/commit/9722c2d2d14ca1337a3d585643b5612165275486) (머지 커밋에 포함) |
| 발표 | 차트·뉴스 시연 | 팀 발표 스크립트 |

## 시세가 만들어지는 방식

`services/news_user.py` 기준입니다.

1. 사용자가 지금까지 본 뉴스를 빼고 남은 뉴스 중 하나를 무작위로 고르고, 다음 날 뉴스로 저장합니다.
2. 그 뉴스에 걸린 영향 규칙(업종, `flu_min`~`flu_max`)을 불러옵니다.
3. 종목마다 다음과 같이 다음 날 시세를 만듭니다.
   - 영향 업종이면 그 범위 안에서 등락률을 뽑습니다.
   - 영향이 없으면 −3%~+3% 안에서 뽑습니다.
   - 다음 날 시가는 전날 종가로 하고, 등락률로 종가를 계산합니다.
   - 고가·저가는 시가·종가보다 0~3% 더 벌어지게 잡습니다.
4. 사용자별 시세는 최근 31일치만 남기고 지웁니다.

같은 뉴스라도 사용자마다 다른 날에 나오고 등락 폭도 다르게 뽑힙니다. 그래서 사용자마다 시장이 따로 움직입니다.

## 사용 기술

`Python` `FastAPI` `SQLAlchemy` `MySQL` `React` `Redux Toolkit`
