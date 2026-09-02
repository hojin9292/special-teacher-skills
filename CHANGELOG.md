<!--
SPDX-FileCopyrightText: 2026 science-teacher-skills contributors
SPDX-License-Identifier: Apache-2.0
-->

# Changelog

이 프로젝트는 [Keep a Changelog](https://keepachangelog.com/ko/1.1.0/) 형식을 따르고,
버전은 [SemVer](https://semver.org/lang/ko/)를 따른다. 수치를 주장하는 항목은 반드시
`evals/runs/`의 실채점 기록을 가리킨다 — 측정 없이 수정 없다.

## [Unreleased]

### 추가 — `sped-lesson-differentiation` 스킬

특수교육(기본교육과정) 수업 차별화 스킬 신설. 계보:
`anthropics/k12-teacher-skills` → 본 저장소의 `ko12-lesson-differentiation`(과학 전용) →
`sped-lesson-differentiation`(특수교육 전용, 2축 개편).

- 축을 범용 1~5 지원 위계에서 **성취수준(A/B/C) × 자극-반응 pool 조합** 2축으로
  교체 — 국립특수교육원이 성취기준마다 이미 제공하는 pool(자극 조각×반응 조각으로
  개별 학생 문장을 짓는 체계)을 그대로 쓴다.
- `references/data/pool-all-subjects/`에 11개 교과(통합교과·국어·사회·수학·과학·
  체육·음악·미술·실과·진로와 직업·선택교과) 총 687개 성취기준의 pool 데이터를
  수록. 과학 고등학교는 국가 성취수준 자료집(A/B/C) 별도 수록
  (`references/data/science-achievement-levels-hs.md`).
- 렌더 엔진(`scripts/*.py`, `render_all.sh`, `theme.css`)과
  `references/curriculum-kr-mcp.md`는 `ko12-lesson-differentiation`과 사본 동일 —
  CI의 "렌더러 사본 동일" 게이트에 세 번째 스킬로 편입.
- 이식 경위·재현 절차(hwp 변환 스크립트 포함)는
  [docs/sped-lesson-differentiation-porting-notes.md](docs/sped-lesson-differentiation-porting-notes.md).
- 남은 일: 공통교육과정(통합학급)용 초등 pool 데이터 미반영, R8 군집화 로직 실학급
  검증 전, evals 루브릭 pool 축 이관 전 — 위 문서의 "아직 안 된 것" 참고.

## [0.3.0-preview.1] — 2026-08-08

### 바뀜 — HWPX 렌더러 (두 스킬 공통, 사본 동일)

품질 루프 5회차(docs/hwpx-quality-loop.md)의 결과. 추정(est) 기반 명시 쪽 나눔이 추정
오차를 그대로 빈 쪽으로 바꾸는 문제(-706pt 실측)를 걷어내고 한글 네이티브 장치로 교체했다.

- 제목·지시문이 답란과 떨어지지 않도록 `keepWithNext`(다음 문단과 붙임) 도입 — est 확률
  장치가 아니라 레이아웃 시점 강제.
- 긴 표는 앵커드 분할 표(`pageBreak="CELL"` + `flowWithText`)로 한글이 직접 나누고 머리행
  반복(`repeatHeader`)이 실제로 동작. 4행 미만 표는 분할 금지(고아 답줄 방지).
- 스타일 체계 정의(바탕글/제목 1·2 등 7종) — F10 위반 38→0.
- 빈 답란 열 폭 가중치 개선(라벨 열이 답란 열보다 넓어지지 않게).
- 실측 캘리브레이션: USABLE_PT=730(PDF 실측), 줄 높이 15pt+7, 줄바꿈 폭 CHARS_EST≈41.

### 추가 — 검사·평가 계층

- `tests/check_hwpx_quality.py`에 F10(스타일 정의·사용), F11(표 높이 선언 정합),
  F12(빈 답란 열 폭) 신설.
- HWPX 시각 루브릭 V1~V10 신설(`evals/hwpx-format/`) 및 실채점: 5회차 수렴 후
  7패키지 68/68 = 100% (`evals/runs/2026-08-08-r5/`).
- 차별화 루브릭 27+2항목 1차 실채점: 3패키지 81/81 = 100%, 학습맵 실호출 + 대화 기록
  근거 (`evals/runs/2026-08-08-diff/`).
- 라이브 대화 평가 체계 신설(`evals/live-conversation/`): 교사 역할 시나리오 6종(S1~S6) +
  라이브 루브릭 L1~L7 — M 버킷(확인 질문 타이밍·마무리 3종·교사 언어·학습맵 호출 시점)의
  재현 가능한 측정 경로. 아직 실측 전.
- GitHub Actions CI(`.github/workflows/ci.yml`): 렌더러 사본 동일 게이트, 렌더 스모크
  2종(수업·차별화), DoD 파일럿 4종, HWPX 품질 검사기 전 파일럿 — 전부 표준 라이브러리
  전용이라 한글 없이 돈다. 한컴 실열림·PDF 실측 층은 로컬 품질 루프 담당.

### 추가 — 파일럿

- `pilot/random-elementary/` — 일반화 시험용 무작위 추첨 단원([6과03-03] 용해와 용액,
  초5) 수업 패키지 + 차별화 패키지. 첫 렌더가 전 검사 층과 실채점을 통과.

### 알려진 한계 (배포 노트용)

- 대화 행동(M 버킷)은 평가 체계만 준비됨 — 라이브 1회전 실측 전.
- 고등학교 대상 산출은 미검증.
- 2026-08-06 수업안 실채점의 확정 fail 5건(중학교 파일럿 내용)은 미수정 상태.
- 다른 한글 버전(구버전)에서 keepWithNext·앵커드 분할 표 스모크 미실행.

## [0.2.0-preview.1] — 2026-08-06

- 한국 이식 초판: ko12-lesson-planning / ko12-lesson-differentiation 스킬,
  학습맵 MCP 연동(`plugin/.mcp.json`), 파일럿 3종, evals 루브릭 한국판 보정 및
  1차 실채점(40항목 × 4패키지 96.1%, `evals/runs/2026-08-06/`).
