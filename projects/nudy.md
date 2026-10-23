# nudy — 한 부위씩 잡아주는 AI 화장 코치

> 2026.09 – 2026.10 · 6인 팀 · 하이미디어 생성형 AI 과정 파이널 프로젝트 · 서비스 운영 종료
> **② 가이드 모듈 담당 + 레퍼런스 화장 입히기(정답사진)**
> [레포](https://github.com/1DAN-2ZO/support-yr-makeup) · 커밋 149건(main, 머지 포함)

따라 하고 싶은 화장 사진을 올리면 그 화장을 내 맨얼굴에 옮긴 "정답사진"을 만듭니다. 그다음 부위별로 어디에 그릴지 카메라 화면 위에 테두리로 보여 주고, 다 그린 뒤 찍은 사진과 비교해 피드백을 줍니다. 팀은 기능을 화면이 아니라 모듈 6개(촬영·가이드·채점·서버·모델·앱)로 나눴고, 그중 ② 가이드를 맡았습니다.

## 맡은 일

| 구분 | 내용 | 근거 |
|---|---|---|
| 레퍼런스 화장 입히기 | FLUX.2 klein 4B로 눈두덩·볼·입술(이후 눈썹 추가)만 다시 그려 정답사진 생성. 얼굴형·피부는 원본 유지 | [a62c0be](https://github.com/1DAN-2ZO/support-yr-makeup/commit/a62c0be3c893682cb1a201c1075328ec6cda85a6) · [4205cc2](https://github.com/1DAN-2ZO/support-yr-makeup/commit/4205cc284e78e89efd82b7071285d1cd6a04b3c8) |
| 정답사진 서버 | 지시문 임베딩 캐싱으로 크래시 해소·속도 2배. 무손실 webp 반환. 팀원이 학습한 화장 LoRA를 서버에 연결 | [304cd3d](https://github.com/1DAN-2ZO/support-yr-makeup/commit/304cd3deb47bcbeeaa733f146bec09a29d2e3e3a) · [302ffad](https://github.com/1DAN-2ZO/support-yr-makeup/commit/302ffadf304a82b0e6442a6d16d8d01e676e5f0f) · [75d15ab](https://github.com/1DAN-2ZO/support-yr-makeup/commit/75d15ab8aa9b6578848ef9cc68257bdc88759d29) |
| GPU 배포 | 팀원이 만든 GPU 서버 자동배포 워크플로에 LoRA 받기(SHA-256 확인)·로드 확인 단계를 추가. LoRA를 못 끼워도 서버는 계속 뜸 | [462f7cc](https://github.com/1DAN-2ZO/support-yr-makeup/commit/462f7cce5e9d177cf893b8961122262131fab2db) |
| 부위별 가이드 생성 | 맨얼굴과 정답사진의 차이로 입술·눈썹·라이너·섀도·블러셔 영역을 떼어 가이드 모양으로 변환 | [0ee7594](https://github.com/1DAN-2ZO/support-yr-makeup/commit/0ee7594cddb9f3f108f4d1c50cc1974d588bd922) · [1324712](https://github.com/1DAN-2ZO/support-yr-makeup/commit/13247129b4c2f3ca7d65d781019e17f6fea9c258) |
| 실시간 가이드 | 그릴 자리 테두리·시작점·화살표를 카메라 화면에 표시. 얼굴 메시에 고정해 고개를 돌려도 따라오게 함 (TypeScript) | [05afbc2](https://github.com/1DAN-2ZO/support-yr-makeup/commit/05afbc2170674895b08f181b42587dbcead1e9a7) · [96425f1](https://github.com/1DAN-2ZO/support-yr-makeup/commit/96425f1156d502acfaac90f43df84b7aac461e22) |
| 기술 결정 문서 | 가이드 모듈의 기술 결정 18개를 근거와 되돌리기 조건까지 문서화 | [597b407](https://github.com/1DAN-2ZO/support-yr-makeup/commit/597b407a8e728c5420a43e6e51f026b2c9585313) |

## 트러블슈팅

### 1. 정답사진 서버가 죽고 느림

- **환경**: RTX 3060 12GB
- **원인**: 모델을 부분씩 GPU로 올리는 `enable_model_cpu_offload`가 매번 PC 메모리와 GPU 사이로 옮기다 Windows access violation으로 죽었습니다. 10/07 하루에만 3번 죽었습니다.
- **해결**: 지시문은 고정이므로, 글 해석 부품(7.5GB)은 처음 한 번만 돌려 임베딩을 파일로 캐시했습니다. GPU에는 그리는 부품과 그림 변환 부품(7.4GB)만 상주시켰습니다.
- **결과**: 장당 15~18초 → 8.6초, 켜고 첫 장도 9초, 최대 VRAM 8.7GB. 6장을 연달아 만들어도 죽지 않았고, 같은 사진·seed의 결과는 전과 같았습니다(색 차 0.00).

### 2. 실시간 가이드 끊김

- **원인**: 보기 전용 화면에서도 촬영 품질 측정(43ms)이 매 프레임 돌고 있었습니다. 얼굴 찾기(31ms)보다 무거웠습니다.
- **해결**: 보기 모드에서는 측정을 건너뛰게 하고, 단계별 시간을 보는 개발용 측정 도구(`?perf`)를 만들었습니다.
- **결과**: 프레임 처리 76.3ms → 31.8ms. 1080p 가짜 카메라 + 헤드리스 크롬에서 CPU를 4배 느리게 해 폰을 흉내 낸 조건입니다. PC에서는 28.7ms → 11.8ms ([fd8c38d](https://github.com/1DAN-2ZO/support-yr-makeup/commit/fd8c38d9ad892846a52f857be0c056efcd76556c)).

### 3. 고개를 크게 돌리면 먼 쪽 눈 테두리가 이탈

- **원인**: 큰 각도에서는 눈꺼풀 랜드마크 자체가 틀어졌습니다.
- **해결**: 정면에서 잡은 눈 틀을 현재 점에 맞추고, 눈썹 비율로 폭을 보정했습니다. 맞출 수 없는 각도에서는 테두리를 흐리게 했습니다.
- **결과**: 각도 차 40~59°에서도 테두리가 붙는 것을 PC에서 확인했습니다 ([e3919b3](https://github.com/1DAN-2ZO/support-yr-makeup/commit/e3919b35b1e14b30ffe3b986098d08122f8adb1e)).

### 4. 입을 벌리면 얼굴 각도를 잘못 계산

- **원인**: 턱처럼 표정에 따라 움직이는 점까지 각도 계산에 쓰였습니다.
- **해결**: 움직이지 않는 강체 점 13개로 채점과 게이트의 기준을 통일했습니다 ([d7adeac](https://github.com/1DAN-2ZO/support-yr-makeup/commit/d7adeac81ce27e09f8262446474f6adf7fe29f68)).
- **결과**: 입을 크게 벌린 152프레임에서 직전 프레임 대비 흔들림 중앙값이 턱 기준 8.0° → 코끝 기준 2.9°로 줄었습니다 (PC 웹캠, [a12c3d4](https://github.com/1DAN-2ZO/support-yr-makeup/commit/a12c3d47ad05521a48185753fb4162e75e44a2f3)).

## 개발 방식

Claude Code와 함께 작업했고, 모든 커밋에 공동작성으로 표기되어 있습니다. 측정값과 시험 조건은 커밋 메시지에 남겼습니다.

## 사용 기술

`TypeScript` `Vite` `Vitest` `MediaPipe Face Landmarker` `Canvas 2D` `Python` `FLUX.2 klein 4B` `diffusers` `LoRA(peft)` `Gradio` `GitHub Actions(self-hosted)`
