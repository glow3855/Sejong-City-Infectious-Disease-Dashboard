# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## 1. 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **프로젝트명** | 세종시 감염병관리지원단(SJCIDC) 대시보드 웹 전환 |
| **목적** | Tableau Public 5페이지 대시보드 → 바닐라 HTML 웹으로 전환 (라이선스 비용 제거) |
| **주요 파일** | `dashboard.html` (단일 파일 — CSS + JS + GeoJSON 데이터 전부 내장, ~1,880줄) |
| **GitHub** | `https://github.com/glow3855/Sejong-City-Infectious-Disease-Dashboard.git` (branch: main) |
| **배포 예정** | GitHub Pages → sjcidc.or.kr 에 iframe 임베딩 |
| **데이터 구조** | `rawData` = 감염병 원시 데이터 하드코딩 (컬럼: `신고일_yy`, `신고일_mm`, `질병급`, `감염병명`, `통계_emdb`, `성별`, `연령대5`), `KR_COMP_BY_YEAR` = 연도별 전국 구성비 (2001–2026) |
| **원본 파일** | `SJCIDC_inf_disease_26.02.xlsx` — `inf_disease` 시트(rawData 원본), `sj_pop` 시트(연도별 읍면동 인구); `kr_rep.xlsx` — 전국 대표값 참조용 (KR_RATE 등 상수 산출 원본) |

**실행 방법:** 빌드 없음 — `dashboard.html`을 브라우저에서 바로 열면 됨.

**Git 규칙:** 코드 편집 완료 시 반드시 `git add` → `git commit` → `git push origin main` 까지 수행한다. 사용자가 별도로 요청하지 않아도 push까지 자동 진행한다.

---

## 2. 기술 스택

| 역할 | 도구 | 비고 |
|------|------|------|
| **차트** | Chart.js 4.4.7 + chartjs-plugin-datalabels 2.2.0 | CDN 로드 |
| **지도** | 순수 SVG choropleth (`renderMapSVG` 함수) | 세종시 읍면동 GeoJSON 내장 |
| **상태관리** | 바닐라 JS 전역 객체 | React/Zustand 미채택 |
| **빌드** | 없음 | 단일 HTML 파일 직접 편집 |

---

## 3. 핵심 JS 패턴 (코드 규칙)

새 코드 작성 시 반드시 이 패턴을 따를 것.

```js
// 필터 상태 — 전역 객체 (초기화 위치: 스크립트 상단)
const filterState = { year: '전체', month: '전체', grade: '전체', disease: '전체' };

// 차트 뷰모드 상태 — 전역 객체 (p3 기본값 'cnt', 'rate'로 토글 가능)
const viewState = { c2: 'cnt', p3: 'cnt', p4: 'cnt' };

// 데이터 흐름 (updateDashboard 내부)
// baseData = rawData 를 grade + disease 로 필터링
// filtered  = baseData 를 year + month 로 필터링

// 버튼 토글 헬퍼 — updateDashboard() 내부에 정의됨 (전역 아님)
const setActive = (groupSelector, activeBtn) => {
  document.querySelectorAll(groupSelector).forEach(b => b.classList.remove('active'));
  activeBtn.classList.add('active');
};
```

| 함수/변수 | 역할 |
|----------|------|
| `updateDashboard()` | 필터 변경 시 현재 활성 탭만 리렌더 (`activeTab = document.querySelector('.tab.active').dataset.target`로 판별, 비활성 탭은 스킵) |
| `syncFilterUI()` | 모든 탭 필터 UI를 filterState와 동기화 |
| `window.renderC5(name)` | P5 감염병 상세 차트 (updateDashboard 외부에서 호출) |
| `DISEASE_ABBREV` + `processName()` | 감염병명 축약 매핑 |
| `charts` | Chart.js 인스턴스 풀 전역 객체 — `initChart` 호출 시 기존 인스턴스 `destroy()` 후 새 인스턴스 저장 |
| `initChart(id, config)` | Chart.js 래퍼 — 기존 차트 destroy 후 재생성 (`charts` 전역 객체에 인스턴스 보관) |
| `renderMapSVG(mapData)` | GeoJSON 기반 SVG 코로플레스 지도 생성 (읍면동별 발생현황) |
| `getTopN(arr, n)` | 감염병명별 건수 집계 후 상위 N개 반환 |
| `window.getGlobalColor(name)` | 감염병명별 일관된 색상 반환 (`DISEASE_COLORS` 캐싱) |
| `POP_BASE` | 세종시 전체 인구 기준값 (391,122 — 2025년 기준) — 발생률 계산에 사용 |
| `REGION_POP` | 읍면동별 인구 상수 (2025년 기준, 24개 지역) — 폴백용 |
| `REGION_POP_BY_YEAR` | 연도별(2012-2026) 읍면동 인구 (`sj_pop` 시트 기준) — P3 발생률·tooltip에 사용 |
| `EMDB_MAP` | 법정동→행정동 매핑 (`집현동→반곡동`, `산울동→해밀동`, `가람동→한솔동`) — rawData의 통계_emdb가 법정동명으로 기록된 경우 처리 |
| `AGE_POP` | 10세 단위 연령대별 인구 상수 |
| `KR_COMP_BY_YEAR` | 연도별(2001-2026) 전국 감염병 구성비 데이터 (%) |
| `KR_RATE` | 연령대별 전국 발생률 기준값 (10만명당) — P4 비교용 |
| `sejongGeoJson` | 세종시 읍면동 GeoJSON 데이터 인라인 상수 (JS 섹션 상단, line ~870) |
| `PALETTE` / `EXTENDED_PALETTE` | 차트용 색상 배열 (6색 / 15색). 감염병별 고정 색상은 `DISEASE_COLORS` 캐싱 방식의 `getGlobalColor`가 담당 |
| `GRID_COLOR` | 차트 격자선 색상 (`#f1f5f9`) |
| `createSparkline(id, labels, data, color, unit)` | P1 KPI 카드 아래 스파크라인 생성 헬퍼 (`initChart` 래핑) |
| `formatSparkLabel(label)` | 스파크라인 X축 레이블 포맷 (연도.월 → 연도 또는 월 표시) |
| `getP3Pop(r)` | P3 블록 내부 로컬 함수 — 지역명 `r`에 대해 `REGION_POP_BY_YEAR[p3Year]` 조회, 없으면 최근 연도로 폴백 |

**탭 전환 패턴:** `.tab[data-target]` 클릭 → 모든 `.tab`·`.page`에서 `active` 제거 → 클릭된 `.tab`과 `document.getElementById(tab.dataset.target)` (.page)에 `active` 추가. `updateDashboard()`는 탭 전환 시에도 재호출됨.

**이벤트 처리 패턴:** 필터 변경과 버튼 클릭 모두 `document` 레벨 이벤트 위임 사용.
- 필터: `document.addEventListener('change', ...)` → `.tab-filter[data-key]` 감지 (`data-key`값이 `filterState` 키와 1:1 매핑)
- 버튼: `document.addEventListener('click', ...)` → `btn.id`로 분기

**초기화 흐름 (`DOMContentLoaded`):**
1. `rawData`에서 연도/월/질병급/감염병명 목록 추출 → `<select>` 옵션 생성
2. `filterState.year` = 최신 연도, `filterState.month` = 최신 연도의 최신 월
3. `syncFilterUI()` → `updateDashboard()` 최초 호출

**`질병급` 필터 주의:** `rawData`의 `질병급` 컬럼은 `'1'`, `'2'` 등 숫자 문자열. `filterState.grade`는 `'1급'` 형태로 저장하며, 비교 시 `replace('급', '')` 처리.

---

## 4. 탭 구성 (5개 페이지)

| 탭 | HTML ID | 필터 노출 | 특이사항 |
|----|---------|---------|---------|
| P1 주요 발생현황 | `#p1` | 연도+월+질병급+감염병명 | KPI 카드 + 스파크라인 |
| P2 연도별·월별 | `#p2` | 연도+월+질병급+감염병명 | 뷰모드 토글: 신고건수/발생률 |
| P3 읍면동별 | `#p3` | 연도+월만 | grade/disease 필터 미노출 (의도적); SVG 지도 + 순위 차트; 인구는 `getP3Pop(r)` 헬퍼로 연도별 동적 조회 |
| P4 인구집단별 | `#p4` | 연도+월+질병급+감염병명 | 뷰모드 토글 있음 |
| P5 감염병별 | `#p5` | 연도+질병급만 | month 필터 미노출; 테이블 행 클릭 → renderC5 연동 |

---

## 5. CSS 클래스 규칙

인라인 `style=""` 최소화 원칙 (JS 템플릿 리터럴 내 동적 색상 등 불가피한 경우만 허용).

주요 커스텀 클래스:
- `.card-header` — 제목+버튼 가로 배치
- `.btn-group` + `.btn-toggle` — 뷰모드 버튼 (active 클래스로 상태 표시)
- `.kpi-unit` / `.kpi-unit-sm` — KPI 카드 단위 텍스트
- `.scroll-container` — 테이블 스크롤 래퍼 (max-height: 600px)
- `.rank-scroll` — P3 읍면동 순위 스크롤 (height: 500px)
- `.hint-text` — 테이블 상단 안내 문구
- `.map-card` / `.map-wrap` / `.rank-chart-wrap` — P3 지도 레이아웃 컨테이너
- `.map-rgn` — SVG 지도 각 읍면동 path 요소 (`data-name`, `data-rate`, `data-cnt`, `data-pop` 속성 보유)
- `#map_tooltip` — JS로 동적 생성되는 지도 호버 툴팁 (position:fixed)

---

## 6. 데이터 검증 참고사항

- **rawData vs Excel 일치 여부**: `SJCIDC_inf_disease_26.02.xlsx`의 `inf_disease` 시트와 비교 시 실질적으로 완전 일치 (Excel 일부 셀에 trailing space 있으나 값 동일)
- **P3 외지 데이터 제외**: rawData의 `통계_emdb`가 세종시 외 지역(예: 충청남도 공주시, 대전광역시)인 경우 `REGION_POP`에 없어 P3 집계에서 제외됨 — 의도된 동작
- **GeoJSON 미포함 지역**: 나성동·해밀동·반곡동·어진동·집현동·산울동은 rawData에 데이터가 있으나 `sejongGeoJson`에 없어 지도에 표시 안 됨 (미해결 — GeoJSON 업데이트 필요)
- **법정동 기록 주의**: rawData의 `통계_emdb`에 집현동(→반곡동)·산울동(→해밀동)·가람동(→한솔동)이 법정동명으로 기록된 경우 있음 → `EMDB_MAP`으로 집계 시 행정동으로 변환
- **`REGION_POP_BY_YEAR` 연도 범위**: 2012~2026년. 특정 연도에 존재하지 않는 지역(신설 행정동)은 가장 최근 연도 값으로 폴백

---

## 7. 미완성 / 보류 항목

| 항목 | 상태 | 비고 |
|------|------|------|
| Task 5 (미구현 기능) | 보류 | 사용자 요청으로 나중에 진행 예정 |
| prototype.html / anti.html / index.html | 결정 대기 | 삭제 여부 미결정 |
| GeoJSON 미포함 지역 | 미해결 | 나성동·해밀동·반곡동·어진동 등 GeoJSON 업데이트 필요 |

---

## 8. 커스텀 스킬 (Slash Commands)

`.claude/commands/`에 4개 스킬 정의. 상세 사용법은 `skill_info.md` 참조.

| 스킬 | 용도 | 사용 예시 |
|------|------|----------|
| `/handoff` | 세션 전환 전 `SESSION_HANDOFF.md` 갱신 | `/handoff P3 지도 작업 중` |
| `/debug-tab [탭]` | 탭별 렌더링·데이터 흐름 진단 | `/debug-tab P3` |
| `/add-chart [탭] [종류] [설명]` | 패턴 준수하며 새 차트 추가 | `/add-chart p2 bar 월별 누적` |
| `/review [탭]` | 코드 리뷰 (기능·품질·데이터·UX) | `/review P4` |

---

## 9. 저장소 구조

- `.gitignore`는 **deny-all** 방식 (`*`로 전부 무시 후 `!`로 허용)
- Git 추적 파일: `dashboard.html`, `CLAUDE.md`, `.gitignore` 만 해당
- `SESSION_HANDOFF.md` — `/handoff` 스킬이 생성·갱신하는 세션 간 컨텍스트 전달 파일 (Git 미추적)
- `skill_info.md` — 스킬 사용법 문서 (Git 미추적)
