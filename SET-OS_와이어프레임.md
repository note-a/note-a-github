# SET.OS — 와이어프레임 레퍼런스

**문서 버전:** v1.0  
**작성일:** 2026-06-06  
**도구:** wireframe.html — DOM 어노테이션 오버레이 (실시간 인터랙티브)  
**원본:** `wireframe.html` (GitHub Pages)

> **읽는 법:** 각 패널의 와이어프레임 캡쳐 → DOM 선택자 + 색상 범례 순으로 구성됨  
> 컬러 테두리 = DOM depth 레벨 (깊을수록 다른 색)  
> 실선 = visible · 점선 = hidden(CSS display:none 또는 visibility:hidden)

---

## 전체 구조 (body)

![전체 구조](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-full.png)

```
body
 └── div#shell                     (배경 + 전체 wrapper)
      └── div#wrap [1140×757]      (메인 OS 창)
           ├── div#top-row         (상단 패널 행)
           ├── div#gbar            (중간 타이틀바 · iframe)
           ├── div#bot-row         (하단 패널 행)
           └── div#botbar / #mode-row  (상태바)
aside.lr-rail                      (좌측 레일 · fixed)
aside.rr-rail                      (우측 레일 · fixed)
div#bg + div.crt                   (배경 레이어 · 애니메이션)
div#bg-orbs                        (배경 구체 · setInterval 33ms)
```

| 선택자 | 설명 | 크기 |
|--------|------|------|
| `div#shell` | 전체 컨테이너. CRT 배경 + scanline 효과 포함 | 뷰포트 전체 |
| `div#wrap` | 메인 OS 인터페이스 창. grid-rows: 1fr / 84px / 1fr / 28px | 1140×757px |
| `aside.lr-rail` | 좌측 레일. fixed 포지션, `place()` 함수로 배치 | ~160×700px |
| `aside.rr-rail` | 우측 레일. fixed 포지션, `place()` 함수로 배치 | ~160×700px |
| `div#bg` | 배경 그라디언트·이미지 레이어 | 뷰포트 전체 |
| `div.crt` | CRT 스캔라인 오버레이 (::before pseudo) | 뷰포트 전체 |
| `div#bg-orbs` | 7개 애니메이션 구체 (.orb + .orb-i) · 7.5–12.5s 주기 | 뷰포트 전체 |

---

## LEFT RAIL

![LEFT RAIL](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-lr.png)

| 선택자 | 설명 |
|--------|------|
| `aside.lr-rail` | 좌측 레일 컨테이너. flex column |
| `.lr-rail .lr-dp:nth-child(1)` | F.01 SET MAP — SVG 벤 다이어그램 |
| `.lr-rail .lr-dp:nth-child(2)` | F.02 CONTOUR — SVG 등고선 애니메이션 |
| `.lr-rail .lr-dp:nth-child(3)` | F.03 DATA — 수치 요약 테이블 |

> 미디어쿼리 `@media (max-width:1560px)`에서 `display:none`. JS `place()` 함수가 `#wrap` 위치 기준으로 left/top 계산 후 fixed 배치.

---

### F.01 — SET MAP (Intersection)

| 항목 | 값 |
|------|-----|
| DOM | `.lr-rail .lr-dp:nth-child(1)` |
| 구현 | SVG 벤 다이어그램 (두 원 겹침) |
| 헤더 | `.lr-h` — 클릭 시 `lr-dp` toggled collapsed |
| 콘텐츠 | `.lr-b` 내 SVG |

### F.02 — CONTOUR FIELD (등고선)

| 항목 | 값 |
|------|-----|
| DOM | `.lr-rail .lr-dp:nth-child(2)` |
| 구현 | SVG path 애니메이션 + SECTION A–A′ 단면 |
| 헤더 | `.lr-h` — 클릭 시 토글 |

### F.03 — DATA (측정값)

| 항목 | 값 |
|------|-----|
| DOM | `.lr-rail .lr-dp:nth-child(3)` |
| 구현 | 데이터 행 목록 (｜A｜, ｜B｜, ｜A∩B｜, 경계길이 등) |

---

## RIGHT RAIL

![RIGHT RAIL](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-rr.png)

| 선택자 | 설명 |
|--------|------|
| `aside.rr-rail` | 우측 레일 컨테이너. flex column |
| `.rr-rail .lr-dp:nth-child(1)` | F.04 SVA RADAR — 레이더 차트 |
| `.rr-rail .lr-dp:nth-child(2)` | ANALYSIS — 분석 데이터 |
| `.pilot-pnl` | SET-NULL.CHANNEL — PILOT 채널 패널 |

### F.04 — SVA RADAR

![SVA RADAR](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-svaradar.png)

| 항목 | 값 |
|------|-----|
| DOM | `.rr-rail .lr-dp:nth-child(1)` |
| 구현 | SVG 4축 레이더 차트 (DS / SD / SA / KV) |
| 연동 | GALLERY 사진 클릭 → `admin-contents.json` sva 값으로 업데이트 |

### ANALYSIS

![ANALYSIS](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-analysis.png)

| 항목 | 값 |
|------|-----|
| DOM | `.rr-rail .lr-dp:nth-child(2)` |
| 구현 | 이미지 ID / round / 인덱스 / DS·SD·SA·KV / score / 판정 |
| 연동 | GALLERY 사진 선택 시 실시간 업데이트 |

### SET-NULL.CHANNEL (PILOT)

![SET-NULL.CHANNEL](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-pilot.png)

| 항목 | 값 |
|------|-----|
| DOM | `.pilot-pnl` |
| 탭 | NULL.LOG / ATTR.TEST / MANUAL |
| 진입 | "OPEN PILOT.EXP" → `pilot.html` |

---

## TOP ROW

![TOP ROW](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-toprow.png)

```
div#top-row  [grid: 1fr 1fr 72px]
 ├── .pnl:nth-child(1)   SET.VIEW   (GALLERY)
 ├── .pnl:nth-child(2)   HUD.XT2    (LOG)
 └── .pnl:nth-child(3)   컨트롤     (UP/EXIT/DOWN)
```

| 선택자 | 이름 | 설명 |
|--------|------|------|
| `#top-row` | TOP ROW | grid 3열 컨테이너 |
| `#top-row>.pnl:nth-child(1)` | SET.VIEW | GALLERY 패널 (사진 뷰어 + DIST 스캐터) |
| `#top-row>.pnl:nth-child(2)` | HUD.XT2 | LOG 패널 (파일 브라우저 / 엔트리 리스트) |
| `#top-row>.pnl:nth-child(3)` | UP/EXIT/DOWN | LOG 스크롤 컨트롤 버튼 3개 |

### SET.VIEW (GALLERY)

![SET.VIEW](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-setview.png)

| 항목 | 값 |
|------|-----|
| DOM | `#top-row>.pnl:nth-child(1)` → `.pnl-in` → `#gly-1` |
| 탭 | ANALYSIS (기본) / CONTACT |
| ANALYSIS | `#gva-wrap` — 스코프 효과 + 현재 사진. `DIST 스캐터` — scatter plot |
| CONTACT | 컨택 시트 그리드 |
| 헤더 | `.ph` → `.ph-row1` (타이틀·탭) + `.ph-row2` (필터·탐색) |

### HUD.XT2 (LOG)

![HUD.XT2](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-hudxt2.png)

| 항목 | 값 |
|------|-----|
| DOM | `#top-row>.pnl:nth-child(2)` → `#hud-pnl` |
| 구조 | `.h-title` (헤더 + 검색) → `.h-folders` (폴더 그리드) → `#h-list` (LIST 뷰, hidden) |
| 헤더 | `.h-title-row1`: 로고·뱃지·메뉴 / `.h-title-row2`: 탭·검색창 |
| ICON 뷰 | `.h-folders` — `.fld-item` ×14 |
| LIST 뷰 | `#h-list` — `.h-row` 엔트리 목록 (기본 hidden) |

---

## GBAR

![GBAR](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-gbar.png)

| 항목 | 값 |
|------|-----|
| DOM | `div#gbar` |
| 구현 | `<iframe src="gbar.html">` 삽입 |
| 높이 | 84px (grid row 고정) |
| 탭 | SET / SET:Inclusion / LOG / Reference / MEASURE |
| 통신 | `window.parent.mobComingSoon()` 호출 (동일 오리진) |

---

## BOT ROW

![BOT ROW](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-botrow.png)

```
div#bot-row
 ├── .bot-left
 │    ├── .bl-split
 │    │    ├── .pnl:nth-child(1)   SYSTEM NOTICE (G-1)
 │    │    └── .pnl:nth-child(2)   VIEWER_LOG    (G-2)
 │    ├── .pnl:nth-child(1)        ID CARD        (G-2b · iframe)
 │    └── .pnl:nth-child(2)        WORK.STATUS    (G-3 · iframe)
 └── .pnl                          SET.TEXT       (G-4)
```

### SYSTEM NOTICE (G-1)

![SYSTEM NOTICE](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-notice.png)

| 항목 | 값 |
|------|-----|
| DOM | `.bl-split>.pnl:nth-child(1)` |
| 구현 | CRT 스타일 공지 카드 + `a.chat-enter-btn` |
| 연동 | 클릭 → `measure.html` |

### VIEWER_LOG (G-2)

![VIEWER_LOG](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-viewerlog.png)

| 항목 | 값 |
|------|-----|
| DOM | `.bl-split>.pnl:nth-child(2)` → `#vlWindow` |
| 구현 | 터미널 로그 UI. 스크롤 가능 피드 |
| 상태 | 현재 정적 데모. 백엔드 연동 시 실시간 |

### ID CARD (G-2b)

![ID CARD](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-idcard.png)

| 항목 | 값 |
|------|-----|
| DOM | `.bot-left>.pnl:nth-child(1)` → `iframe[src*="id-card"]` |
| 파일 | `id-card.html` |
| 표시 | 작가 정보 · 프로젝트명 · 상태 |

### WORK.STATUS (G-3)

![WORK.STATUS](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-workstatus.png)

| 항목 | 값 |
|------|-----|
| DOM | `.bot-left>.pnl:nth-child(2)` → `iframe[src*="work-status"]` |
| 파일 | `work-status.html` |
| 데이터 | GitHub Commits API + `published-posts.json` |
| 캐시 | localStorage 5분 TTL |

### SET.TEXT (G-4)

![SET.TEXT](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-settext.png)

| 항목 | 값 |
|------|-----|
| DOM | `#bot-row>.pnl` → `#stmt.stmt-scroll` |
| 구현 | 작업 개념 텍스트 상시 노출. 클릭 시 전체 확장 |

---

## STATUS BAR (#mode-row)

![STATUS BAR](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/wf-modebar.png)

| 항목 | 값 |
|------|-----|
| DOM | `div#botbar` → `.mode-row` |
| 구현 | `.mb` ×6 버튼 + `button.activate` |
| 위치 | `div#wrap` grid 마지막 행 (28px) |
| 우측 | STATEMENT / DOCS 버튼 |

---

## DOM 선택자 전체 색인

| 패널 | CSS 선택자 | 색상 (wireframe) |
|------|-----------|-----------------|
| 전체 | `body` | 🟥 `#ff4a16` |
| 메인 shell | `div#shell` | 🟧 `#e06010` |
| OS 창 | `div#wrap` | 🟫 `#cc5500` |
| LEFT RAIL | `aside.lr-rail` | 🟦 `#4ecdc4` |
| F.01 SET MAP | `.lr-rail .lr-dp:nth-child(1)` | 🟦 `#4ecdc4` |
| F.02 CONTOUR | `.lr-rail .lr-dp:nth-child(2)` | 🟦 `#4ecdc4` |
| F.03 DATA | `.lr-rail .lr-dp:nth-child(3)` | 🟦 `#4ecdc4` |
| TOP ROW | `div#top-row` | 🟨 `#f7dc6f` |
| SET.VIEW | `#top-row>.pnl:nth-child(1)` | 🟣 `#bb8fce` |
| HUD.XT2 | `#top-row>.pnl:nth-child(2)` | 🟣 `#bb8fce` |
| CTRL 버튼 | `#top-row>.pnl:nth-child(3)` | 🔵 `#85c1e9` |
| GBAR | `div#gbar` | 🟩 `#a8e6cf` |
| BOT ROW | `div#bot-row` | 🟨 `#f7dc6f` |
| .bot-left | `.bot-left` | 🟠 `#f0b27a` |
| SYSTEM NOTICE | `.bl-split>.pnl:nth-child(1)` | 🟡 `#f9e79f` |
| VIEWER_LOG | `.bl-split>.pnl:nth-child(2)` | 🟡 `#f9e79f` |
| ID CARD | `.bot-left>.pnl:nth-child(1)` | 🟢 `#abebc6` |
| WORK.STATUS | `.bot-left>.pnl:nth-child(2)` | 🟢 `#abebc6` |
| SET.TEXT | `#bot-row>.pnl` | 🟣 `#bb8fce` |
| RIGHT RAIL | `aside.rr-rail` | 🟦 `#4ecdc4` |
| SVA RADAR | `.rr-rail .lr-dp:nth-child(1)` | 🟦 `#4ecdc4` |
| ANALYSIS | `.rr-rail .lr-dp:nth-child(2)` | 🟦 `#4ecdc4` |
| PILOT CHANNEL | `.pilot-pnl` | 🔴 `#f1948a` |
| STATUS BAR | `#mode-row` | 🔵 `#85c1e9` |
| BG layer | `div#bg` | ⬛ `#555` |
| CRT overlay | `div.crt` | ⬛ `#444` |
| BG Orbs | `div#bg-orbs` | 🟤 `#a0522d` |

---

## 반응형 브레이크포인트

| 조건 | 영향 |
|------|------|
| `max-width: 1560px` | `.lr-rail`, `.rr-rail` → `display:none` |
| `max-width: 1025px` | 태블릿 모드. 레일 미표시 |
| `max-width: 767px` | 모바일 모드. 단일 컬럼 레이아웃 |
| JS `place()` 함수 | `#wrap` getBoundingClientRect 기준으로 레일 fixed 재배치 |

---

*SET.OS — 측정이 지속된다.*
