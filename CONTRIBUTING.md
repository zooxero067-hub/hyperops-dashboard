# HyperOps 대시보드 — 월별 데이터 추가 가이드

> 매월 자사몰 대시보드 데이터를 추가하고 배포하는 방법을 설명합니다.  
> 수정할 파일은 **`deploy/index.html` 단 하나**입니다.

---

## 목차

1. [사전 준비](#1-사전-준비)
2. [필요한 데이터 파일 (CSV 추출)](#2-필요한-데이터-파일-csv-추출)
3. [index.html 수정 — 3곳 추가](#3-indexhtml-수정--3곳-추가)
4. [Git 커밋 & 배포](#4-git-커밋--배포)
5. [배포 확인](#5-배포-확인)

---

## 1. 사전 준비

### 처음 설정 (최초 1회)

```bash
# 레포 클론
git clone https://github.com/zooxero067/hyperops-dashboard.git
cd hyperops-dashboard

# 이후 매월 작업 전 최신 코드 받기
git pull origin main
```

---

## 2. 필요한 데이터 파일 (CSV 추출)

자사몰 분석 플랫폼(카페24 / 에이스카운터 등)에서 아래 리포트를 CSV로 내려받습니다.  
**기간: 해당 월 1일 ~ 말일**

| # | 리포트 이름 | 사용 위치 |
|---|------------|----------|
| 1 | 매출 종합 분석 (일별) | 일별 매출 차트 |
| 2 | 전체 방문자 수 (일별) | 일별 방문자 차트 |
| 3 | 구매단계 분석 | 퍼널(전환율) 차트 |
| 4 | 연령별 매출 분석 | 연령대 바 차트 |
| 5 | 지역별 매출 분석 | 지역 파이 차트 |
| 6 | 재구매까지 걸린 시간 | 재구매 바 차트 |
| 7 | 유입도메인 (상세) | 유입채널 차트 |
| 8 | 검색엔진 (상세) | 검색엔진 차트 |
| 9 | 처음 방문 / 재방문 구매 | 신규 vs 재방문 도넛 차트 |
| 10 | 상품별 매출 분석 | 상품별 TOP 리스트 |
| 11 | 시간대별 평균 접속 수 | 시간대별 유입 차트 *(없으면 생략 가능)* |

> 💡 시간대별 데이터가 없으면 3단계에서 `hourly: null`로 설정하면 해당 섹션이 자동으로 숨겨집니다.

---

## 3. index.html 수정 — 3곳 추가

`deploy/index.html`을 에디터(VS Code 등)로 열고 아래 3곳에 데이터를 추가합니다.

---

### 3-1. MONTH_DATA 객체에 새 월 데이터 추가

파일 내에서 `const MONTH_DATA` 를 검색합니다.  
기존 가장 최근 키 **바로 앞**에 새 키를 삽입합니다.

```javascript
"2026_04": {                          // ← YYYY_MM 형식
  hasPrev: true,                      // 전월 비교 여부 (항상 true)
  label: "2026년 4월",

  // ── KPI ─────────────────────────────
  revenue:   20530000,                // 총 매출 (원)
  orders:    501,                     // 주문 건수
  visitors:  43791,                   // 방문자 수
  convRate:  "1.14%",                 // 구매전환율
  avgOrder:  40977,                   // 객단가 (신규회원 없을 때)
  // newMembers: 120,                 // 신규회원 있으면 avgOrder 대신 사용

  // ── 일별 데이터 (1일~말일) ────────────
  dailyLabels:   [1,2,3, ...],        // 일 숫자 배열
  dailyVisitors: [1200, 980, ...],    // 일별 방문자
  dailyRevenue:  [450000, 320000, ...], // 일별 매출

  // ── 전월 데이터 (비교용) ─────────────
  prevDailyLabels:   [1,2,3, ...],
  prevDailyVisitors: [1100, 900, ...],
  prevDailyRevenue:  [400000, 280000, ...],

  // ── 퍼널 ────────────────────────────
  funnel: {
    visit: 43791, productView: 18000,
    cart: 5000, purchase: 501
  },

  // ── 연령대 ──────────────────────────
  ageLabels:  ["10대","20대","30대","40대","50대","60대+"],
  ageRevenue: [120000, 5000000, 8000000, 4000000, 2000000, 500000],

  // ── 지역 ────────────────────────────
  regionLabels:  ["서울","경기","부산","인천","기타"],
  regionRevenue: [8000000, 6000000, 2000000, 1500000, 3000000],

  // ── 재구매 주기 ─────────────────────
  repurchaseLabels: ["7일 이내","8~14일","15~30일","31~60일","61~90일","90일+"],
  repurchaseData:   [120, 80, 60, 40, 20, 10],

  // ── 유입 채널 ───────────────────────
  channelLabels: ["직접","네이버","인스타그램","카카오","기타"],
  channelData:   [12000, 18000, 7000, 4000, 2791],

  // ── 검색엔진 ────────────────────────
  searchLabels: ["네이버","구글","다음","기타"],
  searchData:   [15000, 2000, 800, 200],

  // ── 신규 vs 재방문 ──────────────────
  hasVisitType: true,                 // 데이터 있으면 true
  visitTypeData: [280, 221],          // [신규구매, 재방문구매]

  // ── 시간대별 (없으면 null) ───────────
  hourly: {
    labels: ["0시","1시","2시","3시","4시","5시","6시","7시",
             "8시","9시","10시","11시","12시","13시","14시","15시",
             "16시","17시","18시","19시","20시","21시","22시","23시"],
    newVisit:    [20,10,8,5,4,6,30,80,120,200,350,400,380,360,340,320,300,280,310,350,400,380,250,100],
    returnVisit: [10,5,4,3,2,3,15,40,60,100,180,200,190,180,170,160,150,140,155,175,200,190,125,50]
  },

  // ── 상품별 TOP ───────────────────────
  topProducts: [
    { name: "상품명 A", revenue: 5000000, orders: 120 },
    { name: "상품명 B", revenue: 3500000, orders: 85 },
    { name: "상품명 C", revenue: 2800000, orders: 68 },
    { name: "상품명 D", revenue: 2100000, orders: 51 },
    { name: "상품명 E", revenue: 1500000, orders: 36 },
  ],
},
```

---

### 3-2. 월별 카드 HTML 추가

파일 내에서 `tab-jasamol` 을 검색합니다.  
기존 카드들 **맨 앞**에 아래 HTML을 삽입합니다.

```html
<div class="month-card" data-month="2026_04" onclick="showDashboard('2026_04')">
  <div class="month-card-header">
    <span class="month-label">2026년 4월</span>
    <span class="badge up">▲ 전월 대비</span>  <!-- 매출 증가면 up, 감소면 down -->
  </div>
  <div class="month-card-stats">
    <div class="stat">
      <span class="stat-label">매출</span>
      <span class="stat-value">2,053만원</span>
    </div>
    <div class="stat">
      <span class="stat-label">주문</span>
      <span class="stat-value">501건</span>
    </div>
    <div class="stat">
      <span class="stat-label">방문자</span>
      <span class="stat-value">43,791명</span>
    </div>
  </div>
</div>
```

---

### 3-3. 관리자 인사이트 섹션 추가

파일 내에서 `insights_2026` 을 검색해서 가장 최근 섹션 **바로 앞**에 삽입합니다.

```html
<div id="insights_2026_04_jasamol" class="insights-section reveal">
  <h3 class="insights-title">2026년 4월 마케팅 인사이트</h3>

  <div class="insight-summary">
    <!-- 한 줄 요약 -->
    <p>📌 [이번 달 핵심 한 줄 요약을 여기에 작성]</p>
  </div>

  <div class="insight-block positive">
    <h4>✅ 긍정 지표</h4>
    <ul>
      <li>항목 1</li>
      <li>항목 2</li>
    </ul>
  </div>

  <div class="insight-block warning">
    <h4>⚠️ 주의 지표</h4>
    <ul>
      <li>항목 1</li>
    </ul>
  </div>

  <div class="insight-block action">
    <h4>🎯 개선 제안</h4>
    <ul>
      <li>제안 1</li>
      <li>제안 2</li>
    </ul>
  </div>

  <div class="insight-block summary">
    <h4>💬 총평</h4>
    <p>[전반적인 총평 작성]</p>
  </div>
</div>
```

---

## 4. Git 커밋 & 배포

```bash
# 변경사항 확인
git diff deploy/index.html

# 스테이징 & 커밋
git add deploy/index.html
git commit -m "feat: 2026년 4월 자사몰 대시보드 추가"

# push → Vercel 자동 배포
git push origin main
```

> `git push` 후 약 **1~2분** 내에 자동 배포됩니다.

---

## 5. 배포 확인

- 대시보드 URL: **https://hyperops-dash.vercel.app**
- Vercel 배포 상태: https://vercel.com/dashboard

### 체크리스트

- [ ] 새 월 카드가 목록 맨 앞에 표시되는가
- [ ] KPI 카드 숫자가 올바른가
- [ ] 일별 차트 x축 레이블이 현재 월 기준인가
- [ ] 전월 비교 바 그래프가 표시되는가
- [ ] 시간대별 섹션 — 데이터 있으면 표시, null이면 숨겨지는가
- [ ] 관리자 로그인 후 인사이트 섹션이 보이는가
- [ ] 모바일에서 가로 스크롤이 없는가

---

## 참고

- 관리자 로그인: 푸터 클릭 → ID `otmo` / PW `ico20190201!`
- 만원 단위 표시: `revenue` 값이 10,000원 이상이면 자동으로 "만원" 단위로 표시됨
- 전월 비교 바: `max(전월, 현재) = 100%` 기준으로 자동 계산됨
- 문의: 이주영 ([@zooxero067](https://github.com/zooxero067))
