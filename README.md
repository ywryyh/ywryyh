# 양윤호

팀 프로젝트에서 백엔드, AI 서빙, 실시간 비전 모듈을 맡아 왔습니다.

음성 키오스크의 결제 API, 서버 코드 없이 SQL만으로 짠 게임 백엔드,
얼굴 위에 화장 가이드를 그리는 실시간 모듈까지 — 맡은 부분이 실제로 돌아가는 데까지 끌고 가는 걸 중요하게 생각합니다.

<br>

## 프로젝트

### nudy — 화장 코치 모바일 웹 · 파이널 프로젝트

> 2026.09 – 2026.10 · 6인 · 서비스 운영 종료 · [저장소](https://github.com/ywryyh/support-yr-makeup) · [원본](https://github.com/1DAN-2ZO/support-yr-makeup)

따라 하고 싶은 화장 사진을 올리면 그 화장을 내 맨얼굴에 옮긴 그림을 만들고, 부위별로 어디에 얼마나 그릴지 보여 준 뒤, 다 하고 찍은 사진과 견줘 피드백을 줍니다.

**부위 가이드 모듈 담당** · 커밋 149건 · PR 46건 병합

- 얼굴 랜드마크 위에 눈썹·라이너·섀도·입술·블러셔를 그릴 자리를 테두리·시작점·화살표로 표시하는 가이드를 사진과 실시간 카메라 양쪽에 구현
- 실시간 가이드의 떨림을 잡기 위해 테두리를 얼굴 그물(mesh)에 고정하고, 입을 벌리거나 고개를 크게 돌려도 점이 튀지 않도록 보정
- 촬영 품질 게이트 — 각도·조명·안정 조건을 재되, 3초 이상 막히면 확인받기 단계로 넘겨 사용자가 갇히지 않게
- 예상 메이크업 생성 서버(FLUX.2 klein) 안정화 — 지시문 임베딩을 재사용해 **2배 빠르게**, 무손실 반환으로 라이너 꼬리 뭉개짐 해결, LoRA 적용
- 기술 결정 18개를 근거와 되돌리기 조건까지 문서화

`TypeScript` `Vite` `MediaPipe` `Python` `FastAPI` `FLUX.2` `Docker` `GitHub Actions`

<br>

### VINUS — 시각장애인을 위한 음성 주문 키오스크

> 2026.06 – 2026.07 · [백엔드](https://github.com/ywryyh/VINUS_backend) · [AI 서버](https://github.com/ywryyh/VINUS_ai) · [프론트엔드](https://github.com/ywryyh/VINUS_frontend) · [원본](https://github.com/vinus-final-project)

화면을 보기 어려운 사용자가 말로 카페 메뉴를 주문하고 결제까지 하는 키오스크입니다. 음성은 Whisper로 받아쓰고, 규칙으로 해석되는 발화는 즉시 처리하며, 해석되지 않는 자유 발화는 LLM에 넘깁니다.

**결제 · 데이터 모델 · 배포 · LLM 연동 담당** · 커밋 47건

- 결제 API 초안 작성과 보완
- DB 모델·스키마와 세션 로그 스키마 초안 설계
- GitHub Actions 배포 파이프라인 작성
- AI 서버의 LLM 서비스 연동과 RAG·LLM 1차 테스트, transformers 버전 충돌 해결
- 프론트엔드 초기 프로젝트 세팅

`Python` `FastAPI` `SQLAlchemy` `MySQL` `Whisper` `ChromaDB` `transformers` `Docker`

<br>

### 집가 — 술자리 미니게임 앱

> 2026.08 · [서비스](https://zip-ga-frontend.vercel.app) · [백엔드](https://github.com/ywryyh/ZipGa_Backend) · [프론트엔드](https://github.com/ywryyh/ZipGa_Frontend) · [원본](https://github.com/1DAN-2ZO)

술자리 인원이 QR로 한 방에 모여 미니게임으로 순위를 가리고, 못한 사람에게는 카카오T가 뜹니다. 앱 이름이 그대로 벌칙 대사입니다 — "집 가."

**백엔드 단독 설계·구현** · 백엔드 커밋 6건 중 5건

- 서버 코드와 Edge Function 없이 **Supabase Postgres만으로** 백엔드 구성
- 테이블 4개, 행 수준 보안(RLS)으로 클라이언트는 자기 방만 읽게, 쓰기는 전부 RPC를 거치게
- 게임 중에는 통신하지 않고 판이 끝날 때 점수만 제출하는 구조 — 재제출은 기본키로 차단
- 폰마다 다른 시계를 보정하는 `server_now()` 추가

`PostgreSQL` `PL/pgSQL` `Supabase` `Realtime`

<br>

### 모의주식 게임 — 중간 팀 프로젝트

> 2026.04 – 2026.05 · [저장소](https://github.com/ywryyh/IBM-Redhat-6th-midterm-team-project) · [원본](https://github.com/jinwoong96/IBM-Redhat-6th-midterm-team-project)

뉴스 이벤트에 따라 종목이 오르내리는 모의투자 게임입니다.

**뉴스 기능 담당** · 커밋 12건

- 뉴스 발생에 따른 카테고리별 종목 등락 로직
- 사용자가 아직 안 본 뉴스를 무작위로 배정하고, 지난 뉴스를 다시 볼 수 있게
- 매수·매도 모달

`React` `Redux Toolkit` `FastAPI` `SQLAlchemy` `MySQL` `JWT`

<br>

## 다뤄본 기술

| 분야 | |
|---|---|
| 백엔드 | Python · FastAPI · SQLAlchemy · MySQL · PostgreSQL · Supabase |
| AI · 비전 | Whisper · LLM · RAG(ChromaDB) · MediaPipe · FLUX.2 |
| 프론트엔드 | TypeScript · React · Redux Toolkit · Vite |
| 배포 · 협업 | Docker · GitHub Actions · 모듈 브랜치 + PR 리뷰 |
