# VINUS — 시각장애인을 위한 음성 주문 키오스크

> 2026.06 – 2026.07 · 5인 팀 · IBM x RedHat AX Academy 파이널 프로젝트 · **배포 총괄 · LLM · 백엔드**
> [백엔드](https://github.com/vinus-final-project/VINUS_backend) · [AI 서버](https://github.com/vinus-final-project/VINUS_ai) · [프론트엔드](https://github.com/vinus-final-project/VINUS_frontend)

화면을 보기 어려운 사용자가 말로 카페 메뉴를 주문하고 결제까지 하는 키오스크입니다. 음성은 Whisper로 받아씁니다. 규칙으로 해석되는 발화는 백엔드가 바로 처리하고, 해석되지 않는 자유 발화와 추천 요청은 LLM 서버로 넘깁니다.

> 각 레포의 `main`은 최종본 한 커밋으로 덮여 있습니다. 그래서 아래 근거는 개인 브랜치(`yoonho`)와 팀원 브랜치에 남은 커밋으로 연결합니다.

## 맡은 일

| 구분 | 내용 | 근거 |
|---|---|---|
| LLM 선정 | 같은 주문 시나리오로 Gemma3-1B와 EXAONE 3.5 2.4B(GGUF)를 비교해 EXAONE을 채택. 평균 응답 6.63초 → 0.64초, 초당 토큰 15.2 → 30.6, 정확도 46.7% → 86.7%, 환각 3건 → 0건 | 팀 성능평가(6/23) 중 LLM 담당. Streamlit 대시보드 결과지 |
| LLM 서버 | 모델 로딩 → 프롬프트 생성 → 출력 파싱을 모듈로 나눔. 매 호출마다 FSM 상태·주문 중인 메뉴·장바구니를 프롬프트에 넣고, JSON 응답을 주문 FSM 이벤트로 돌려줌 | [1a5dbd4](https://github.com/vinus-final-project/VINUS_ai/commit/1a5dbd41fbe1662916bfaddf269ca7eb8e59542c) |
| RAG 연동 (공동) | ChromaDB에서 검색한 메뉴를 컨텍스트로 넣어 응답 생성 | [0a8791b](https://github.com/vinus-final-project/VINUS_ai/commit/0a8791b87de1bfe524fafab9d1bead09009110f6) |
| 서빙 방식 전환 | llama.cpp(GGUF)에서 transformers fp16 GPU 추론으로 교체 | 0a8791b(7/7) → [11c06b8](https://github.com/vinus-final-project/VINUS_ai/commit/11c06b87d2d807a9729462aabe61b302d2445bb9)(7/15) |
| 백엔드 | DB 모델·스키마 초안, 결제 API 초안·보완, 세션 로그 스키마, AI 서버 호출 클라이언트 | [f2708ef](https://github.com/vinus-final-project/VINUS_backend/commit/f2708ef8ae61806860a1fcd520d5acece397f6bc) · [582e06d](https://github.com/vinus-final-project/VINUS_backend/commit/582e06d0c6f458be7d44ee821a1085463b3a4358) · [8a6d3a4](https://github.com/vinus-final-project/VINUS_backend/commit/8a6d3a415dc46433a8f52cbcba751ff9a825c065) · [ee473ac](https://github.com/vinus-final-project/VINUS_backend/commit/ee473ac4f212e55f133bcd231cf4e6920479adda) |
| 배포 파이프라인 | 백엔드 `Dockerfile`·`.dockerignore`·GitHub Actions 워크플로 작성 → SHA 태그 + Task Definition 렌더링 + 안정화 대기 방식으로 개편. AI 서버 워크플로는 같은 구조로 팀원이 커밋 | [cdf7e15](https://github.com/vinus-final-project/VINUS_backend/commit/cdf7e159cc10998c7be7858d5c7dc6e032e1cd1f) · [f8ec621](https://github.com/vinus-final-project/VINUS_backend/commit/f8ec6211b60ddd9600b754ff58a2c21fecb92ca1) · [d1acc19](https://github.com/vinus-final-project/VINUS_backend/commit/d1acc199ed0de0c68d75af8896b520a2dd1f1bbf) |
| AWS 인프라 | 아래 배포 구조 전체. 태스크 정의 개정이 백엔드 30회·AI 40회, ECR 이미지가 38개·39개 쌓임 | 콘솔 작업이라 git 흔적 없음 (팀 Notion에 콘솔 캡처 보관) |

## 배포 구조

```
Route 53 (voice-in-us.com)
  └─ ALB  HTTPS 443 (ACM *.voice-in-us.com) · HTTP 80 → 301 HTTPS
       ├─ 대상 그룹 :8081 → ECS vinus-cluster     (EC2, T4)   FastAPI · WebRTC VAD · Whisper STT
       └─ 대상 그룹 :8001 → ECS vinus-ai-cluster  (EC2, L40S) EXAONE 3.5 2.4B · ChromaDB RAG
RDS MySQL (db.t3.small) · ECR vinus-backend / vinus-ai
GitHub Actions: main push → 이미지 빌드(SHA 태그) → ECR → Task Definition 새 개정 → ECS 서비스 배포
```

- 두 ECS 클러스터는 Auto Scaling 그룹(최소 0·최대 1)을 용량 공급자로 씀
- 발화부터 응답까지 단계별 측정값(팀 성능평가): STT 708ms, LLM 457~640ms, TTS(Amazon Polly) 51ms, VAD·RAG 각 1ms 미만 → 합계 1.2~1.4초

## 트러블슈팅

### 1. 배포가 끝나도 어떤 버전이 떠 있는지 모르고, 되돌릴 수도 없음

- **원인**: 처음 워크플로는 `latest` 태그 하나로 푸시한 뒤 `update-service --force-new-deployment`만 호출했습니다. 그래서 떠 있는 이미지가 어느 커밋인지 추적할 수 없었습니다. 또 명령이 성공하면 새 태스크가 실제로 떴는지와 상관없이 배포가 성공으로 끝났습니다.
- **해결**: 이미지에 커밋 SHA 태그를 붙였습니다. 현재 Task Definition을 받아 이미지만 바꾼 새 개정을 등록하고, `wait-for-service-stability`로 서비스가 안정될 때까지 기다리게 했습니다.
- **결과**: 개정 번호로 롤백할 수 있고, 새 태스크가 안 뜨면 워크플로가 실패로 끝납니다.

### 2. Task Definition 등록 단계에서 배포 실패

- **원인**: `describe-task-definition` 출력에는 `enableFaultInjection` 필드가 들어 있는데, 배포 액션이 새 개정을 등록할 때 이 필드를 지원하지 않았습니다.
- **해결**: 등록 전에 `jq 'del(.enableFaultInjection)'`로 그 필드를 뺐습니다. 개편 커밋 16분 뒤에 수정했습니다([d1acc19](https://github.com/vinus-final-project/VINUS_backend/commit/d1acc199ed0de0c68d75af8896b520a2dd1f1bbf)).

### 3. GPU 컨테이너에서 모델 로딩 실패

- **원인**: `transformers`·`tokenizers`·`huggingface_hub`·`accelerate`·`sentence-transformers`의 호환 범위가 서로 어긋났습니다.
- **해결**: 4.46과 5.x 계열을 오가며 한 패키지씩 맞춰 `transformers 5.1.0`에서 수렴시켰습니다([d6c85a8](https://github.com/vinus-final-project/VINUS_ai/commit/d6c85a8265b8053b40cf98991d96bbb95eefd079) … [ca60ad0](https://github.com/vinus-final-project/VINUS_ai/commit/ca60ad0ad395133e73e3bf9b87392390d03e1cee), 7/22~23).
- requirements 103개 전체 버전 고정은 그 전 통합 커밋(7/15)에서 이미 해 두었습니다.

### 4. LLM 출력 형식이 흔들리면 응답이 끊김

- **원인**: 모델이 JSON을 코드블록으로 감싸거나 앞뒤에 설명을 붙이면 파싱에 실패했습니다.
- **해결**: 코드블록 제거 → 중괄호 구간 추출 → 이벤트 단위 파싱(깨진 이벤트만 건너뜀)으로 처리하고, 최종 실패하면 안내 문구를 돌려주게 했습니다([1a5dbd4](https://github.com/vinus-final-project/VINUS_ai/commit/1a5dbd41fbe1662916bfaddf269ca7eb8e59542c)). 최종본에는 `//` 주석 제거 단계가 더해졌습니다.

## 사용 기술

`Python` `FastAPI` `SQLAlchemy` `Pydantic` `MySQL` `Transformers` `EXAONE 3.5` `llama.cpp` `ChromaDB` `Docker` `GitHub Actions` `AWS ECR · ECS(EC2) · ALB · Route 53 · ACM · RDS`
