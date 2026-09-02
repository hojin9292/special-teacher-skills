# special-teacher-skills

**한국 특수교육 기본교육과정 기반 수업 차별화 Claude 스킬 (v0.4.0-preview.1)**

Anthropic의 [k12-teacher-skills](https://github.com/anthropics/k12-teacher-skills)를 한국
특수교육 체제로 이식했습니다. 원래는 과학 전용 일반학급 수업 설계 스킬로 시작했으나,
국립특수교육원의 「기본 교육과정 평가자료」(성취기준마다 자극-반응 pool을 이미 갖춘
11개 교과 완비 자료)를 발견하면서 **특수교육 기본교육과정 수업 차별화** 단일 스킬로
재편했습니다. 과학 전용 일반학급 스킬(`ko12-lesson-planning`, `ko12-lesson-differentiation`)은
그 과정에서 제거했습니다 — 이 저장소의 목적은 특수교사를 위한 도구이기 때문입니다.

## 무엇을 만드나

스킬 1종, `sped-lesson-differentiation`이 들어 있습니다.

이미 있는 수업(기본교육과정 또는 공통교육과정/통합학급)을 **성취수준(A/B/C) × 자극-반응
pool 조합** 2축으로 차별화합니다.

- **성취수준(A/B/C)** — 2022 개정 교육과정 공통의 3범주(지식·이해 / 과정·기능 / 가치·태도)
  체계를 따릅니다.
- **자극-반응 pool 조합** — 범용 1~5 지원 위계가 아니라, 국립특수교육원이 성취기준마다
  이미 제공한 pool(자극 조각 × 반응 조각으로 개별 학생 문장을 짓는 체계)에서 고릅니다.
  11개 교과(통합교과·국어·사회·수학·과학·체육·음악·미술·실과·진로와 직업·선택교과) 총
  687개 성취기준이 완비돼 있습니다.

교사용 차별화 수업안 1종 + 학생용 학습지 3종(학급 학생들의 실제 수준·자극-반응 프로필을
대표 3그룹으로 군집화)을 한 턴에 편집 가능한 **한글(HWPX) 문서**로 만듭니다. 네 문서는
하나의 소스에서 렌더되므로 서로 어긋날 수 없습니다.

통합학급(공통교육과정) 수업을 각색해야 하면 한국 교육과정 학습맵 MCP(초등·중등)를 씁니다
— 연결이 안 돼 있어도 완전 동작합니다.

이식 경위·데이터 변환 절차(hwp 원자료 파싱 포함)는
[docs/sped-lesson-differentiation-porting-notes.md](docs/sped-lesson-differentiation-porting-notes.md)에
있습니다.

## 설치

Claude Code에서 두 줄이면 됩니다.

```
/plugin marketplace add hojin9292/special-teacher-skills
/plugin install special-teacher-skills@special-teacher-skills
```

터미널에서 하려면 같은 인자로 `claude plugin marketplace add …` / `claude plugin install …`을
쓰면 됩니다. 설치 후 Claude Code를 재시작하면 `sped-lesson-differentiation` 스킬과 학습맵
MCP 2종이 함께 올라옵니다.

저장소를 clone해서 쓰려면 clone한 폴더의 경로를 `marketplace add`에 그대로 넘깁니다.

학습맵 MCP 2종은 `plugin/.mcp.json`에 번들되어 있어 `npx`로 자동 실행됩니다 — 따로 설치할
것이 없습니다. 문서 렌더에는 Python 3만 있으면 됩니다 — 추가 패키지 없이 표준
라이브러리만 씁니다.

## 사용

```
이 학생은 기본교육과정을 배우는데, 국어 수업을 우리 반 학생별 성취수준에 맞게 차별화해 줘
```

```
우리 반은 편차가 커요. 철수는 만지고 체험해야 반응하고, 영희는 B수준이에요.
```

각색할 원본 수업(대화에 있거나, 업로드하거나, 없으면 주제·성취기준·학년부터)과 학급
학생들의 (성취수준, 선호하는 자극/반응 방식)을 알려주면 됩니다. 진단 자료가 전혀 없어도
pool의 대표 조합으로 기본값 진행하며, 그 사실을 첫 응답에 밝힙니다.

산출은 **한글(HWPX) 문서**입니다 — 표준 라이브러리만으로 OWPML을 직접 생성하므로 추가로
설치할 것이 없습니다.

## 데이터 출처

| 데이터 | 출처 |
|---|---|
| 기본교육과정 성취기준·성취수준·자극-반응 pool (11개 교과, 687개 성취기준) | 국립특수교육원 「기본 교육과정 평가자료」(공개 hwp 문서) |
| 과학 고등학교 국가 성취수준(A/B/C) | 국가 성취수준 자료집 |
| 공통교육과정(통합학급) 중·고 성취기준·선수관계·전이 | [korean-secondary-learning-map-mcp](https://github.com/raphysicst-create/korean-secondary-learning-map-mcp) |
| 공통교육과정(통합학급) 초등 성취기준·선수관계 | [korean-elementary-learning-map-mcp](https://github.com/taehyeonglim/korean-elementary-learning-map-mcp) |

성취기준·성취수준 원문(국가 발간 공공 데이터)은 그대로 인용하되, 정규식 기반 자동 파싱이라
문장 경계가 살짝 어긋난 항목이 있을 수 있습니다 — 이상하면 원문 hwp와 대조하도록 스킬
자체가 보수적으로 안내합니다.

이 플러그인은 시판 교과서·지도서·학습지의 활동·지문·삽화·문항을 재현하지 않습니다. 교사가
출판사를 확언하지 않으면 출판사명을 산출물이나 대화 어디에도 쓰지 않습니다.

## 원저작 표기

원본은 Anthropic, PBC와 Learning Commons의 `k12-teacher-skills` v0.6.0이며 Apache-2.0으로
배포됩니다. 이 저장소도 Apache-2.0을 따르며, 파생 파일에 SPDX 헤더와 원본 경로를 남겼습니다.
자세한 내용은 [LICENSE](LICENSE)를 보세요.

원본에서 바꾼 것은 다섯 지점입니다 — 대상 학생(일반학급 → 특수교육 기본교육과정), 차별화
축(범용 지원 위계 → 성취수준 × 자극-반응 pool 2축), 표준 조회(Learning Commons Knowledge
Graph → 국립특수교육원 pool 데이터 + 한국 학습맵 MCP), 교사 대면 언어, 교사 전달 형식
(docx → 한글 HWPX). 산출 스키마와 내부 HTML 렌더러는 원본(및 그 과학 전용 한국 이식판)
그대로라 업스트림을 계속 추적할 수 있습니다.

## 로드맵

| 단계 | 내용 | 상태 |
|---|---|---|
| 1 | 과학 전용 일반학급 파일럿 — 골격, 학습맵 연결, HWPX 산출, 수업 차별화 포팅 | ✅ 완료 (2026-08-03), 이후 제거 |
| 2 | 특수교육 기본교육과정 전환 — 성취수준×자극-반응 pool 2축 재설계, 11개 교과 687개 성취기준 데이터 이식, 저장소 특수교육 전용 재편 | ✅ 완료 (2026-09-02) |
| 3 | 공통교육과정(통합학급) 초등 pool 데이터 반영, R8 군집화 실학급 검증, evals 루브릭 pool 축 이관 | 진행 전 |

설계 근거와 결정 기록은 [DESIGN.md](DESIGN.md)에 있습니다.
