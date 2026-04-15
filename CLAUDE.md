# HyperOps 프로젝트 인스트럭션

## 프로젝트 개요
- 퍼포먼스 마케팅 데이터 시각화 대시보드
- URL: https://hyperops-26-01.netlify.app
- Netlify 사이트 ID: `e8aa38ec-06e4-434b-bf5a-220851e045df`
- 단일 HTML 파일: `deploy/index.html`
- 배포: `netlify deploy --prod --dir=deploy --site=e8aa38ec-06e4-434b-bf5a-220851e045df`

## 기술 스택
- 순수 HTML/CSS/JS (프레임워크 없음)
- Chart.js (데이터 시각화)
- Pretendard Variable + Plus Jakarta Sans + JetBrains Mono 폰트
- SHA-256 Web Crypto API (관리자 인증)
- IntersectionObserver (스크롤 reveal 애니메이션)
- 라이트/다크 모드 (`[data-theme="dark"]`)

## 관리자 기능
- 푸터 클릭 → 관리자 로그인 모달 → 마케팅 분석 보고서 페이지
- 관리자 ID: `otmo`, PW: `ico20190201!`

## 대시보드 템플릿 정책 (중요)
- **2026년 3월부터 이 대시보드 디자인/구조를 고정 템플릿으로 사용**
- 매월 새 데이터를 추가할 때는 `MONTH_DATA` 객체에 키(`2026_03` 등)를 추가하고, HTML에 대응하는 대시보드 섹션과 관리자 인사이트 섹션을 복사·추가하는 방식
- 디자인·레이아웃·컴포넌트 구조는 변경하지 않고 데이터만 교체

## 사용자 개선 요청 이력

### 완료
1. Toss/Apple 스타일 디자인 전면 개편
2. 전월 비교 바 그래프 색상 버그 수정 (`.compare-bar.prev`)
3. 월별 X축 레이블 버그 수정 (전월 데이터가 현재 월 레이블 덮어쓰는 문제)
4. 푸터 클릭 → 관리자 로그인 → 마케팅 보고서 플로우 구현
5. 2월 마케팅 분석 보고서 작성 (10년차 퍼포먼스 마케터 관점)
6. 마케팅 보고서 페이지 `.reveal` 섹션 노출 버그 수정 (`observeReveals()` 미호출)
7. 모바일 가로 스크롤 제거 (`overflow-x: hidden`, `minmax` 수정, 반응형 패딩)
8. 일별 트렌드 차트 분리 (방문자 / 매출 별도 단일축 차트)
9. 지역별 파이차트 내부 라벨 추가 (`chartjs-plugin-datalabels`)
10. 재구매 바차트 색상 그라데이션 (녹색→빨간색, 단기→장기)
11. 총 매출액 KPI 카드 만원 단위 표시 (57,270,370원 → 5,727만원)
12. 2026년 3월 자사몰 대시보드 추가 (매출 2,053만원, 주문 501건, 방문자 43,791명)
13. 시간대별 유입 분석 차트 — 최고/최저 시간대 색상 하이라이트 (동적 계산)
14. 신규 vs 재방문 도넛 차트 — 내부 비율 텍스트 추가 (ChartDataLabels)

## 매월 대시보드 추가 체크리스트

새 월 데이터 추가 시 아래 순서로 작업:

1. **CSV 파일 파악** — 필요한 데이터 파일 목록:
   - 매출 종합 분석 (일별 매출)
   - 전체 방문자 수 (일별 방문자)
   - 구매단계 분석 (퍼널)
   - 연령별 매출 분석
   - 지역별 매출 분석
   - 재구매까지 걸린 시간
   - 유입도메인(상세)
   - 검색엔진(상세)
   - 처음 방문_재방문 구매
   - 상품별 매출 분석
   - 시간대별 평균 접속 수 (없으면 `hourly: null` 설정 — JS null guard 처리됨)

2. **MONTH_DATA에 키 추가** — `"YYYY_MM": { ... }` 형식, 기존 키 앞에 삽입
   - `hasPrev: true`, `prevDailyLabels/Visitors/Revenue`에 전월 데이터 넣기
   - `hasVisitType`: 처음방문/재방문 데이터 있으면 true
   - `hasNewMembers`: 신규회원 일별 데이터 있으면 true (없으면 false)
   - `hourly`: 시간대별 CSV 있으면 객체, 없으면 null

3. **월별 카드 HTML 추가** — `tab-jasamol` 내 최상단에 삽입
   - 전월 대비 증감에 따라 `badge up` 또는 `badge down` 사용
   - 비교 기준: 바로 이전 달

4. **대시보드 HTML 섹션 추가** — `dashboard_YYYY_MM` id로 기존 섹션 앞에 삽입
   - KPI 5번째 카드: 신규회원 데이터 없으면 **객단가**로 대체
   - 전월 비교 바: max(prev, curr) = 100%, 작은 쪽은 비율로 계산
   - visitTypeChart canvas (hasVisitType true일 때 JS가 도넛 차트 그림)

5. **관리자 인사이트 섹션 추가** — `insights_YYYY_MM_jasamol` id로 기존 섹션 앞에 삽입
   - 한 줄 요약 / 핵심 수치 / 긍정 / 우려 / 개선 제안 / 총평 구조 유지

6. **배포** — `netlify deploy --prod --dir=deploy --site=e8aa38ec-06e4-434b-bf5a-220851e045df`

## 차트 컴포넌트 구현 규칙

- **시간대별 유입 분석**: 신규방문·재방문 바 모두 최고시간=빨강, 최저시간=초록, 나머지=파랑(신규)/어두운색(재방문)으로 동적 계산
- **신규 vs 재방문 도넛**: `plugins: [ChartDataLabels]` 로컬 등록, 비율 5% 이상 세그먼트에만 텍스트 표시
- **지역별 파이차트**: `plugins: [ChartDataLabels]` 로컬 등록, 비율 6% 이상 세그먼트에만 표시
- **재구매 바차트**: `repColors` 배열로 단기(녹색) → 장기(빨강) 그라데이션 적용

## 주의사항
- `.reveal` 클래스는 `opacity:0`으로 시작하며 IntersectionObserver로 `.visible` 추가해야 표시됨
- 동적으로 `display:none`에서 보여주는 페이지는 반드시 `observeReveals(el)` 재호출 필요
- 차트 x축 레이블은 항상 현재 월 기준으로 사용 (전월 데이터로 덮어쓰지 말 것)
- KPI won 타입 값 >= 10,000원은 자동으로 만원 단위로 표시됨
- `chartjs-plugin-datalabels` CDN 포함됨 (파이차트에만 `plugins: [ChartDataLabels]`로 로컬 등록)
- `hourly: null`이면 시간대별 섹션 자동 숨김 (JS null guard 처리됨)
