# 세종시 감염병 현황 대시보드 — Claude 작업 메모

> 이 파일은 세션이 초기화되어도 기존 작업 내용을 복원할 수 있도록 작성된 컨텍스트 문서입니다.
> 새 작업이 완료될 때마다 "작업 로그" 섹션을 업데이트합니다.

---

## 1. 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **프로젝트명** | 세종시 감염병관리지원단(SJCIDC) 대시보드 웹 전환 |
| **목적** | Tableau Public 5페이지 대시보드 → 바닐라 HTML 웹으로 전환 (라이선스 비용 제거) |
| **주요 파일** | `dashboard.html` (단일 파일 — CSS + JS + GeoJSON 데이터 전부 내장, ~1,800줄) |
| **GitHub** | `https://github.com/glow3855/Sejong-City-Infectious-Disease-Dashboard.git` (branch: main) |
| **배포 예정** | GitHub Pages → sjcidc.or.kr 에 iframe 임베딩 |
| **데이터 구조** | `rawData` = inf_disease 원시 데이터(~1만건) 하드코딩, `KR_DATA` = 전국 비교용 |

---

## 2. 기술 스택 (확정)

| 역할 | 도구 | 비고 |
|------|------|------|
| **차트** | Chart.js 4.4.7 + chartjs-plugin-datalabels 2.2.0 | ECharts는 제거됨 (1b67e41) |
| **지도** | 순수 SVG choropleth (`renderMapSVG` 함수) | 세종시 읍면동 GeoJSON 내장 |
| **상태관리** | 바닐라 JS 전역 객체 | React/Zustand 미채택 |
| **빌드** | 없음 | 단일 HTML 파일 직접 편집 |

---

## 3. 핵심 JS 패턴 (코드 규칙)

새 코드 작성 시 반드시 이 패턴을 따를 것.

```js
// 필터 상태 — 전역 객체 (초기화 위치: 스크립트 상단)
const filterState = { year: '전체', month: '전체', grade: '전체', disease: '전체' };

// 차트 뷰모드 상태 — 전역 객체
const viewState = { c2: 'cnt', p3: 'rate', p4: 'cnt' };

// 데이터 흐름 (updateDashboard 내부)
// baseData = rawData 를 grade + disease 로 필터링
// filtered  = baseData 를 year + month 로 필터링

// 버튼 토글 헬퍼
const setActive = (groupSelector, activeBtn) => {
  document.querySelectorAll(groupSelector).forEach(b => b.classList.remove('active'));
  activeBtn.classList.add('active');
};
```

| 함수/변수 | 역할 |
|----------|------|
| `updateDashboard()` | 필터 변경 시 전체 리렌더 (filterState 기반) |
| `syncFilterUI()` | 모든 탭 필터 UI를 filterState와 동기화 |
| `window.renderC5(name)` | P5 감염병 상세 차트 (updateDashboard 외부에서 호출) |
| `DISEASE_ABBREV` + `processName()` | 감염병명 축약 매핑 |
| `initChart(id, config)` | Chart.js 래퍼 — 기존 차트 destroy 후 재생성 |

---

## 4. 탭 구성 (5개 페이지)

| 탭 | HTML ID | 필터 노출 | 특이사항 |
|----|---------|---------|---------|
| P1 주요 발생현황 | `#p1` | 연도+월+질병급+감염병명 | |
| P2 연도별·월별 | `#p2` | 연도+월+질병급+감염병명 | 뷰모드 토글: 신고건수/발생률 |
| P3 읍면동별 | `#p3` | 연도+월만 | grade/disease 필터 미노출 (의도적) |
| P4 인구집단별 | `#p4` | 연도+월+질병급+감염병명 | 뷰모드 토글 있음 |
| P5 감염병별 | `#p5` | 연도+질병급만 | month 필터 미노출; 테이블 행 클릭 → renderC5 연동 |

---

## 5. CSS 클래스 규칙

인라인 `style=""` 최소화 원칙 (현재 9개만 허용 — JS 템플릿 리터럴 내 동적 색상 등).

주요 커스텀 클래스:
- `.card-header` — 제목+버튼 가로 배치
- `.btn-group` + `.btn-toggle` — 뷰모드 버튼 (active 클래스로 상태 표시)
- `.kpi-unit` / `.kpi-unit-sm` — KPI 카드 단위 텍스트
- `.scroll-container` — 테이블 스크롤 래퍼
- `.rank-scroll` — P3 읍면동 순위 스크롤
- `.hint-text` — 테이블 상단 안내 문구

---

## 6. 작업 로그

| 날짜 | 커밋 해시 | 내용 |
|------|----------|------|
| 2026-03-19 | c9c2082 | 지도 라벨 및 10세 단위 연령대 적용 |
| 2026-03-19 | a8a35d5 | 전국/세종 구성비 차트 연도 필터 비활성화 |
| 2026-03-19 | e53cf68 | P4 레이아웃 재설계, P5 감염병 전체 목록 + 월별 차트 추가 |
| 2026-03-19 | ad390b8 | P1 제목 동적화, c1_comp 연도 필터 복원 |
| 2026-03-20 | 2ea2c55 | KPI 카드 위치 교체, ageGroup10 매핑 수정 |
| 2026-03-20 | d7c4b78 | P4 뷰모드 토글, 축 여백 1.2배, 성별 합계 표시 |
| 2026-03-23 | cd2faf9 | 모든 차트 툴팁 한국어 형식 통일 (yy.mm → yy년mm월, 건/명/% 단위) |
| 2026-03-23 | 94452a0 | TOP5/구성비 차트 툴팁 단위 추가 |
| 2026-03-23 | a7ce6b6 | 전국/세종 구성비 차트에 연도 필터 적용 (KR_COMP_BY_YEAR 구조 추가) |
| 2026-03-23 | 147398f | KR_COMP_BY_YEAR → kr_rep.xlsx 실제 데이터로 교체 |
| 2026-03-23 | 8f91101 | kr_rep.xlsx 전국 데이터 하드코딩 반영 (2001-2026, 명칭 정규화) |
| 2026-03-23 | 038b75b | 읍면동별 지도 레이아웃 조정 (550px 고정, 비율 개선) |
| 2026-03-23 | b501188 | 읍면동별 지도 배율 확대, 라벨 겹침 방지 |
| 2026-03-23 | 6b94aa4 | 읍면동별 지도 제목 정제, 가로 범례 상단 배치 |
| 2026-03-23 | 1c46b0e | 읍면동별 지도 Flex 정렬, 여백 확대, 배율 1.5배, 범례 정리 |
| 2026-03-23 | db65a56 | 읍면동별 지도 범례 외부화, 잘림 방지, 색상 매핑 수정 |
| 2026-03-23 | 1b67e41 | ECharts 완전 제거 → SVG 지도 교체, DISEASE_ABBREV 분리, head 구조 수정 |
| 2026-03-23 | 2484ce8 | 필터 UI를 탭 내부 인라인으로 이동 (안 A) |
| 2026-03-23 | 159467e | Task 6: 필터가 모든 탭에 반영 (renderC5 grade/year 필터 적용, P2 top5 레이블 수정) |
| 2026-03-23 | a6e4dd7 | 코드품질 & UX 개선: 인라인 스타일 70→9개, viewState 통합, setActive 헬퍼, P5 더미데이터 제거, P3 지도 제목 동적화 |

---

## 7. 미완성 / 보류 항목

| 항목 | 상태 | 비고 |
|------|------|------|
| Task 5 (미구현 기능) | 보류 | 미구현 부분 많음, 사용자 요청으로 나중에 진행 예정 |
| prototype.html / anti.html / index.html | 결정 대기 | 삭제 여부 미결정 |

---

## 8. 작업 로그 업데이트 방법

새 작업이 완료되면 사용자가 "CLAUDE.md 업데이트해줘" 요청 시:
1. 위 **6. 작업 로그** 테이블에 새 행 추가
2. 변경된 패턴이 있으면 해당 섹션도 갱신
3. 완료된 보류 항목은 **7. 미완성 항목**에서 제거
