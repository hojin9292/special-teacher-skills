# sped-lesson-differentiation — 이식 노트

원본 계보: `anthropics/k12-teacher-skills`(영어, below/at/above 3수준)
→ `hojin9292/special-teacher-skills`(한국어·과학 전용, 기초/보통/심화 3수준)
→ 본 스킬(특수교육 전용, **성취수준(A/B/C) × 자극-반응 pool 조합** 2축,
**11개 교과 전 학년군 데이터 완비**)

## 설계 변경 이력
1차 설계는 지원강도를 "1~5 촉진 위계"로 임의로 만들었는데, 업로드된
`기본 교육과정 평가자료_한글.zip`을 제대로 열어보니 국립특수교육원이 성취기준마다
**"성취기준별 성취수준 기술을 위한 풀(pool)"**(자극 조각×반응 조각 조합으로 개별
학생 문장을 짓는 체계)을 이미 만들어뒀다는 걸 뒤늦게 발견해서, 축 전체를 이 pool
구조로 교체했다.

## 데이터 현황 — 11개 교과 전부 완료 ✅

`기본 교육과정 평가자료_한글.zip`(11개 교과 hwp)을 전부 변환·파싱해
`references/data/pool-all-subjects/`에 교과별 마크다운으로 저장했다. **총 687개
성취기준.**

| 교과 | 성취기준 수 | 학년군 |
|---|---|---|
| 통합교과 | 30 | 초1-2 |
| 국어 | 62 | 초1-2~고 |
| 사회 | 49 | 초3-4~고 |
| 수학 | 148 | 초1-2~고 |
| 과학 | 78 | 초3-4~고 |
| 체육 | 56 | 초3-4~고 |
| 음악 | 52 | 초3-4~고 |
| 미술 | 46 | 초3-4~고 |
| 실과 | 23 | 초5-6 |
| 진로와 직업 | 56 | 중~고 |
| 선택교과(정보통신활용·생활영어·보건) | 87 | 중~고 |

각 성취기준마다 성취기준 원문·해설·**적용 시 고려사항(장애 유형별 구체 지침 —
예: 뇌병변장애·시각중복장애·장애 정도가 심한 학생을 위한 최대-최소 촉진법 등)**·
**자극-반응 pool**·교수학습 자료·공통교육과정 연계 코드가 들어있다. 과학 고등학교는
국가 성취수준 자료집(A/B/C)에서 뽑은 `science-achievement-levels-hs.md`도 별도로 있다.

**주의**: 정규식 기반 자동 파싱이라 일부 항목에 줄바꿈이 뭉개지거나 문장 경계가 살짝
어긋날 수 있다(특히 "해설"·"적용 시 고려사항" 끝부분에 다음 섹션 라벨의 앞글자가
한두 글자 붙는 경우가 있음). 실제 수업안 작성 시 이상하다 싶으면 원문 hwp를 대조하는
게 안전하다.

## 기술적으로 막혔다가 뚫은 것들 (재현 가능한 절차)

1. **`hwp5txt`는 표를 못 읽는다.** 성취기준·pool이 전부 표 안에 있어서 `hwp5html`로
   바꿔야 함.
2. **여는 대괄호 `[` 유실.** `hwp5html`이 만든 텍스트에서 성취기준 코드 앞 `[`가 종종
   사라진다. 정규식에 `\[?`로 선택 처리.
3. **국어 파일이 UnicodeDecodeError로 안 열림.** `pyhwp`의 BSTR 디코더가 잘못된
   UTF-16 서로게이트 문자를 만나면 예외를 던지는데, 이 문서에 그런 손상된 문자열이
   있었음. `hwp5.dataio.decode_utf16le_with_hypua`와 `BSTR.read`를 관대한(lenient)
   버전으로 몽키패치해서 해결(아래 스크립트).
4. **수학·선택교과가 `hwp5html`에서 10분 넘게 걸리다 죽음.** 원인은 `hwp5html`이
   HTML+이미지(bindata, 두 파일 다 2000개 안팎의 임베디드 이미지/수식 개체 보유)를
   전부 처리하려 해서 매우 느림. **해결**: `hwp5html` 대신 `hwp5proc xml`로 바이너리를
   원시 XML로만 덤프(이미지 레이아웃 처리 없음 — 훨씬 빠름, 수학 18.9MB가 250초 안에
   완료), 그 XML의 `<Text>...</Text>` 요소만 순서대로 이어붙여 텍스트를 복원한 뒤
   같은 파서로 처리.

### 재사용 가능한 변환 스크립트

```python
# lenient_hwp5.py — pyhwp의 엄격한 UTF-16 디코더를 우회
import sys, re
import hwp5.dataio as dataio
_ILLEGAL_XML = re.compile(u'[\\x00-\\x08\\x0b\\x0c\\x0e-\\x1f\\ud800-\\udfff\\ufffe\\uffff]')
def _sanitize(s): return _ILLEGAL_XML.sub(u'', s)
_orig = dataio.decode_utf16le_with_hypua
def lenient_decode(b):
    try:
        return _sanitize(_orig(b))
    except UnicodeDecodeError:
        return _sanitize(b.decode('utf-16le', errors='replace'))
dataio.decode_utf16le_with_hypua = lenient_decode
from hwp5.dataio import UINT16, readn
def lenient_bstr_read(f):
    size = UINT16.read(f)
    if size == 0: return u''
    try:
        return dataio.decode_utf16le_with_hypua(readn(f, 2*size))
    except Exception:
        return u''
dataio.BSTR.read = staticmethod(lenient_bstr_read)

# 표까지 다 있는 작은 파일: hwp5html 그대로 사용
# from hwp5.hwp5html import main; sys.argv = ['hwp5html'] + sys.argv[1:]; main()

# 크고 느린 파일(이미지·수식 많음): hwp5proc xml로 원시 XML만 덤프 (훨씬 빠름)
from hwp5.hwp5proc import main
sys.argv = ['hwp5proc', 'xml'] + sys.argv[1:]
main()
```

### 파싱 스크립트 로직 (`references/data/`에 코드 자체는 안 넣음 — 데이터 생성 도구라서)

1. HTML 또는 XML에서 순수 텍스트만 추출(HTML은 태그 제거, XML은 `<Text>` 요소만 이어붙임).
2. `내용요소 및 성취기준` 문자열이 나오는 위치마다 새 항목 시작으로 간주(표 안에
   있던 라벨인데도 텍스트로는 안정적으로 남는 유일한 구분자).
3. 그 직전 600~700자에서 `\[?숫자+한글(1-4자)+두자리-두자리\]` 패턴의 **마지막**
   매치를 그 항목의 성취기준 코드로 삼는다(첫 매치를 쓰면 공통교육과정 연계용
   참조 코드를 잘못 잡는다 — 이번 작업에서 두 번 이 버그를 밟았다).
4. `성취기준 해설`, `성취기준 적용 시 고려사항`, `기술을 위한 풀`, `교수 ･ 학습을
   위한 참고자료`, `공통 교육과정 관련 성취기준` 같은 고정 라벨 사이 텍스트를 잘라낸다.

## 그대로 재사용한 것 (검증 완료)
- `scripts/*.py`, `scripts/render_all.sh`, `scripts/theme.css` — 렌더 엔진, 수정 없이
  재사용. `references/example_differentiation.json`(실제 [4과학01-01] pool 데이터
  기반)으로 렌더 테스트 통과.
- 3범주(지식·이해/과정·기능/가치·태도) — 2022 개정 교육과정 공통 틀.
- `references/curriculum-kr-mcp.md` — 공통교육과정(통합학급) 경로용. 포크에서 그대로.

## 새로 쓴 것
- `SKILL.md` — 기본/공통교육과정 두 경로 + pool 기반 용어, 11개 교과 완비 상태 반영.
- `references/differentiation-rules.md` — R1–R8을 **pool 축**으로 재작성.
- `references/data/pool-all-subjects/*.md` — 11개 교과 687개 성취기준 pool 데이터.
- `references/data/science-achievement-levels-hs.md` — 과학 고등학교 A/B/C.

## 아직 안 된 것
1. `공통 교육과정 평가자료_한글.zip`(초등 7과목, 일반 통합학급용)도 같은 pool 구조를
   갖고 있어서 `curriculum-kr-mcp.md`의 MCP 미연결 폴백으로 붙일 수 있는데 아직 미반영.
2. R8 군집화 로직(학생 프로필 → pool 조합 매칭)을 실제 학급 데이터로 검증 안 함.
3. evals — 포크의 `evals/*/rubrics/*.csv`를 pool 축에 맞게 새로 써야 함.
4. `ko12-lesson-planning`(신규 수업 생성) 스킬은 이번에 손대지 않음.
5. 자동 파싱 특성상 남아있는 자잘한 텍스트 경계 노이즈 — 수작업 정제하면 더 좋아지지만
   실사용에는 지장 없는 수준.

## 확인된 설계 결정
- 대상: 기본교육과정 + 공통교육과정(통합학급) 둘 다, 초·중·고 전체
- 축: 성취수준(A/B/C, 국가 3범주) × **자극-반응 pool 조합**(범용 척도 아님, 성취기준별)
- 산출물: 문서 3개 고정, 학생 프로필을 대표 3그룹(1/2/3모둠)으로 군집화
