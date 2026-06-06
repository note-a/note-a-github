# SET.OS — 요구사항 정의서

**문서 버전:** v3.0  
**작성일:** 2026-06-06  
**대상 URL:** https://note-a.github.io/set-os.html  
**관련 파일:** `set-os.html` (4,571줄), `gbar.html`, `work-status.html`, `id-card.html`, `pilot.html`, `measure.html`

---

## 1. 제작 목적

### 배경 — SET 사진 작업이 온라인으로 이동한 이유

SET는 사진 작업이지만 사진으로 시작하지 않는다. 먼저 지시문(DIR)이 쓰인다. 어떤 조건 아래 어떤 것을 촬영할 것인지 — 그 조건이 현실에서 발견될 때 셔터를 누른다. 촬영이 끝나면 사진들로부터 문법이 추출되고, 문법은 다시 지시문이 된다. 이 순환이 SET의 구조다.

집합(Set)은 수학적 의미에 가깝다. 조건이 충족되면 반드시 귀속된다. 탈출이 없다. 사진은 미적 완성의 결과물이 아니라 지시문의 조건이 현실에 존재했다는 실행 기록이다.

**온라인 이동은 아카이빙이 아니다.** SET의 논리가 다른 방식으로 계속되는 것이다. 이 전제 아래 set-os.html은 세 가지 기능을 동시에 수행하도록 설계되었다.

1. **측정 장치**: 관객이 수동적으로 사진을 보는 것이 아니라, MEASURE를 통해 지시문 언어를 직접 입력하면 교집합이 생성된다. 관객의 검색이 교집합을 만들고, 그 기록은 작업 안에 누적된다. 측정하는 자도 측정된다.
2. **과정의 가시화**: 어떤 사진이 CORE(채택)·SUPPORT(보조)·DROP(탈락)되었는지, 어떤 조건이 반복 출현했는지를 데이터로 드러낸다. 선별된 결과만이 아니라 과정 전체가 공개된다.
3. **집합의 확장**: NULL.LOG는 검색 결과 없었던 단어를 자동 수집한다. ATTR.TEST는 외부 이미지를 SET의 문법 코드로 분석한다. 집합이 자신의 후속을 스스로 준비한다.

### 인터페이스 원칙

- 갤러리가 아닌 **터미널**: 사진을 감상하는 공간이 아니라 데이터에 접근하는 시스템
- CRT 미학(주황/검정, 스캔라인, 모노스페이스 폰트)은 장식이 아니라 "이 시스템은 측정 중이다"라는 작업 맥락의 시각적 번역
- 고정 해상도(1140×757px)의 단일 창 형식. 크기가 아니라 밀도로 정보를 전달

---

## 2. 시스템 구조

![SET.OS 전체 인터페이스](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-full.png)

```
body
 └─ div#shell  (뷰포트 전체 · CRT 배경 + scanline 오버레이)
     ├─ div#wrap [1140×757px]  grid-rows: 1fr / 84px / 1fr / 28px
     │    ├─ div#top-row       (GALLERY + LOG + 컨트롤 버튼)
     │    ├─ div#gbar          (GBAR iframe)
     │    ├─ div#bot-row       (NOTICE + VIEWER_LOG + ID CARD + WORK + SET.TEXT)
     │    └─ div#botbar        (상태 바 + 네비게이션)
     ├─ aside.lr-rail          (좌측 레일 · fixed 포지션)
     ├─ aside.rr-rail          (우측 레일 · fixed 포지션)
     ├─ div#bg                 (배경 그라디언트 레이어)
     ├─ div.crt                (CRT 스캔라인 :: before 오버레이)
     └─ div#bg-orbs            (7개 애니메이션 구체)
aside.lr-rail / aside.rr-rail — JS place() 함수가 #wrap BoundingClientRect 기준으로 fixed 배치
                                scale(__hudScale × 1.2) · transform-origin 각 레일 기준점
```

**데이터 소스 전체**

| 소스 | URL | 용도 |
|------|-----|------|
| `admin-contents.json` | `./admin-contents.json` | 사진 메타데이터 + RULE 항목 |
| `published-posts.json` | GitHub raw URL | LOG 폴더 목록 |
| GitHub Commits API | `api.github.com/repos/note-a/note-a-github/commits?per_page=50` | WORK STATUS |
| `localStorage set_null_log` | 로컬 | PILOT NULL.LOG 검색어 누적 |
| `localStorage ws_commits` | 로컬 (5분 TTL) | GitHub API 응답 캐시 |
| `localStorage ws_posts` | 로컬 (5분 TTL) | posts 응답 캐시 |

---

## 3. 패널별 기능 명세

---

### 3-1. GALLERY 패널 (`SET.VIEW`)

![GALLERY](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-gallery.png)

**DOM:** `#top-row > .pnl:nth-child(1)` → `.pnl-in` → `#gly-1` / `#gly-2`  
**탭 전환:** `glSwitch(n)` 함수 — `gly-1`(ANALYSIS), `gly-2`(CONTACT) 중 하나에 `.active`

---

#### ① ANALYSIS 탭 (기본 뷰)

**데이터: `GL_FRAMES` 배열**

| 상태 | 내용 |
|------|------|
| 초기값 | 10개 seed 객체 `{seed, dir, n, total, status, date, exif}` |
| fetchData() 이후 | `admin-contents.json` photos 파싱 → Fisher-Yates 셔플 후 n/total 재설정 |
| 필드 | `{src, dir, id, n, total, status, date, exif}` |
| src 해석 | `resolveUrl(f)`: `http` / `/` / `images/` 시작이면 그대로, 아니면 `images/` 프리픽스 |
| 썸네일 | `thumbUrl(u)`: `/t/` 서브디렉토리의 동일명 파일 |
| verdict | `(p.verdict\|\|'—').toUpperCase()` → `CORE` / `SUPPORT` / `DROP` |
| 자동 전환 | `window._gvaTimer`: 8초마다 `glAThumbClick(glIdx+1)` |

**좌측 — `#gva-wrap` (사진 + CRT 시각화)**

사진 렌더링:

| 요소 | 설명 |
|------|------|
| `#gva-bg` (img) | 현재 GL_FRAMES[glIdx].src 이미지. filter: `brightness(1.0) contrast(0.95) grayscale(1) sepia(1) saturate(2.2) hue-rotate(-8deg)` → 주황/세피아 톤 |
| 클릭 | `gvaAccessGranted()` → "ACCESS GRANTED" 오버레이 1.8초 표시 → 1.5초 후 gallery1.html 새 탭 |
| hover | `gvaHoverIn()` / `gvaHoverOut()` → filter `saturate(2.4) brightness(1.15)` 강화 |

오버레이 레이어 (z-index 낮→높):

| z-index | 요소 | 역할 |
|---------|------|------|
| 3 | `#gva-grid` | 18×18px 격자 `rgba(255,74,22,.048)` 배경 이미지 |
| 4 | `#gva-sweep` | 52px 수직 sweep bar. animation: `gva-sweep 4.8s linear infinite` (left:-52px → 100%) |
| 4 | 텍스트 오버레이 | 좌상단 `#gva-dir`("PROJECT SET"), 우상단 `#gva-date`/`#gva-exif`, 하단 `#gva-cls`(판정), `#gva-ctr`(인덱스) |
| 5 | `.gva-trail` | 타깃 이동 전 잔상 박스 최대 3개, 3초 fade out |
| 5 | `#gva-hline` | 2px 수평 스캔라인. animation: `gva-hline 3.4s linear infinite` (top:-2px → 100%) |
| 6 | `.gva-corner.tl/.tr/.bl/.br` | 4개 코너 브래킷 18×18px. animation: `gva-bktpulse 3.5s ease-in-out infinite` (16→10→16px 크기 펄스) |
| 6 | `#gva-tick` | 8×8px 틱 커서, 타깃 중심 위치 추적 |
| 6 | `#gva-ping` | 40×30px ring. animation: `gva-ping 2s ease-out infinite` (scale 1→2.8, opacity 사라짐) |
| 6 | `#gva-measure-svg` | 타깃(`#gva-target`)↔보조 감지(`#gva-detect2`) 사이 대시 측정선(`#gva-mline`) + 거리 라벨(`#gva-mdist`). 거리 = `sqrt(dx²+dy²) × 0.31` (단위: m) |
| 7 | `#gva-target` | 72×54px 주황 테두리 박스, 흰 코너. 2800ms마다 `moveTarget()` 랜덤 이동. transition: `left 2.2s cubic-bezier`, `top 1.8s` |
| 7 | `#gva-detect2` | 18×14px 보조 감지 박스. 4200ms마다 `moveDet2()` 랜덤 이동 |
| 8 | `#gva-crosshair` SVG | `buildCrosshair()`로 생성. 수평/수직 arms (GAP=14px, ARM=min(W,H)×38%), 12px 간격 틱 마크, 두 개 링 (r=ARM×0.55, r=ARM×0.78), 중심 dot |
| 9 | `#gva-glitch` | animation: `gva-glitch 6s step-end infinite` — 간헐적 수평 노이즈 바 |
| 10 | `#gva-scan-texture` | `repeating-linear-gradient` 스캔라인 텍스처. animation: `gva-flk 7s linear infinite` (명도 깜빡임) |
| 15 | `#gva-lock-msg` | "LOCK ACQUIRED · Xm" 하단 알림. animation: `gva-lock-show 2.4s` |
| 20 | `#gva-access` | "ACCESS GRANTED" 텍스트. 기본 opacity:0. 클릭 시 `.active` → animation: `gva-access-in 1.8s + gva-access-flk 1.8s` |

**우측 — DIST scatter 패널 (`#gl-scatter`, SVG)**

`buildGlScatter()`로 생성. ViewBox: `0 0 160 280`

| 요소 | 설명 |
|------|------|
| 데이터 | `ALL_DIRS` 배열 — 45개 DIR 코드와 사진 수 `{code, n}` |
| X축 | `TEMPORAL AXIS →` — 시간축 (DIR 순서 = 촬영 순서) |
| Y축 | `FORMAL DEPTH` — 형태적 깊이 (DIR 인덱스 기반 수직 배치) |
| 점 크기 | `Math.max(2.5, d.n / maxN × 10)` — 사진 수에 비례 |
| 점 색상 CORE | `rgba(230,175,0,1)` — 골드 |
| 점 색상 SUP | `rgba(255,90,60,1)` — 코랄 |
| 점 색상 DROP | `rgba(210,20,20,1)` — 레드 |
| 점 색상 pending | `rgba(180,180,180,.22)` — 그레이 |
| 상태 결정 | GL_FRAMES에서 DIR별 CORE/SUPPORT/DROP 카운트 → 최댓값. 데이터 없으면 인덱스 기반 (0-14=CORE, 15-29=SUP, 30+=DROP) |
| drift 애니메이션 | 각 점: seed=42 기반 pseudo-random, `sin/cos` 파형으로 ±4~11px 진동 |
| 스캔라인 | y축 따라 천천히 내려가는 수평선 (opacity pulse) |
| 범례 | 하단: CORE(골드) · SUP(코랄) · DROP(레드) · 보류(그레이) |

**네비게이션 및 상태**

| 요소 | 설명 |
|------|------|
| `glAThumbClick(idx)` | 인덱스 범위 조정 후 `#gva-bg.src`, 텍스트 오버레이 업데이트 |
| 키보드 | `ArrowLeft` → 이전, `ArrowRight` → 다음 |
| `#gva-cls` | 현재 사진 판정: `■ CORE` / `■ SUPPORT` / `■ DROP` |
| `#gva-ctr` | 현재 인덱스 / 전체 수 (`001 / 021` 형식) |
| `#gva-dir` | DIR 코드 ("PROJECT SET") |
| `#gva-date` | 촬영일 (filename에서 정규식 `/\/(\d{4})\/(\d{2})\/(\d{2,4})\//` 추출) |
| `#gva-exif` | EXIF 정보 (`f/2.8 · 1/250s · ISO 200` 형식) |

---

#### ② CONTACT 탭 (`#gly-2`)

`buildCs2Grid()`로 생성.

| 요소 | 설명 |
|------|------|
| 레이아웃 | 6열 그리드 `#cs2-grid` |
| 데이터 | GL_FRAMES 또는 CS_DATA(정적 폴백) |
| 셀 클래스 | `.cs2-cell.core` / `.sup` / `.drop` |
| 사진 | `.cs2-img img` — 동일한 세피아 filter 적용 |
| 하단 바 | `.cs2-bar` — 번호 + 배지 (`C`=CORE, `S`=SUP, `—`=DROP) |
| CORE 강조 | `outline: 1px solid rgba(255,74,22,.6)` |
| DROP 처리 | `img opacity: 0.4` |

---

### 3-2. LOG 패널 (`HUD.XT2`)

![LOG ICON 뷰](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-log.png)

![LOG LIST 뷰](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-log-list.png)

**DOM:** `#top-row > .pnl:nth-child(2)` → `#hud-pnl`

**헤더 (`h-title`)**

| 요소 | 설명 |
|------|------|
| `h-logo-main` | `<LOG>` |
| `h-logo-sub` | `LOG` |
| LIVE 배지 | 녹색 dot pulse 애니메이션 |
| ERR 배지 | 빨간 dot |
| 탭 `■ ICON` | `data-v="icon"` — `.h-folders` 표시 |
| 탭 `☰ LIST` | `data-v="list"` — `.h-list` 표시 |
| 검색창 | `.h-search input` placeholder "SEARCH..." — 현재 바인딩 없음 (UI만 존재) |

---

#### ① ICON 뷰 (`.h-folders`, 4열 그리드)

**데이터 소스:**

| 소스 | URL | 필터 조건 |
|------|-----|---------|
| `published-posts.json` | `https://raw.githubusercontent.com/note-a/note-a-github/main/published-posts.json` | `p.publishedSection === 'rule'` |
| `admin-contents.json` | `./admin-contents.json` | `rule.items` 배열 → `.fixed=true` 표시 |
| 렌더 순서 | `admRule.concat(pubRule)` | — |

**각 폴더 아이템 (`.fld-item`)**

| 요소 | 설명 |
|------|------|
| SVG 아이콘 | 경로: `M3 8 H17 L20 12 H43 V32 H3 Z` — 폴더 실루엣 |
| `--fc` CSS 변수 | 10가지 주황 계열 색상 중 순환 배정 |
| `.fld-item.fixed` | 채워진(filled) 폴더 아이콘 — admin rule 항목 |
| `.fld-name` | 폴더명, 너비 초과 시 ellipsis |
| 클릭 | `openPost(p)` → `crtNav('page-log.html?post='+encodeURIComponent(key))` |

**정적 초기 폴더 26개** (JS 로드 전 HTML에 내장):
*Unjudgmentable grammer, figure background grammer, space structure grammer, objects closeup grammer, figure forward grammer, figure background DIR, figure forward DIR, objects closeup DIR, Unjudgmentable DIR, instruction table, 2차 작업 가이드, Meetings and work directions, data Dashboard 2025 등*

---

#### ② LIST 뷰 (`.h-list` / `.h-rows`)

| 컬럼 | 내용 |
|------|------|
| NO | 행 번호 |
| 체크 | 선택 체크 |
| FILENAME | 파일명 |
| 썸네일 | 미리보기 이미지 |
| DATE | `DD/MM/YYYY` |
| SYS INFO | 시스템 메타정보 |
| × | 닫기 |

- `.h-row.sel`: 선택 시 배경 `rgba(255,74,22,.14)`
- 정적 샘플 14행 내장 (`File:Dir:XXXX/X` 형식)
- 클릭 → `openPost()` 동일

---

#### ③ HUD Footer (`.h-foot`)

| 영역 | 설명 |
|------|------|
| `.hf-left` (130px) | Progress bar 2개 (`hpfill` animation 22%→68% 왕복) + path 텍스트 |
| `#logStream2` | 실시간 로그 스트림. `addLine()` 300~1700ms 간격 실행. 형식: `HH:MM:SS.mmm [LV] [SRC] msg`. 최대 14줄 유지. SRC: NET/FS/SYS/DIR/IDX/AUTH/CACHE/IO. 가중치: OK(5)/INF(4)/WRN(3)/ERR(1) |
| `#logCount2` | 누적 로그 카운트 |
| `.hf-right` (142px) | 버튼 행 + 상태 인디케이터 |

**HUD Status Bar (`.h-status`)**: `HS.09` · `DIR:0345` · `ERR:02 WARN` · `SET XT2`

---

#### ④ 중앙 컨트롤 패널 (72px 컬럼)

| 버튼 | 기능 |
|------|------|
| ▲ UP (`.ctrl-btn.amber`) | `scrollHud(-1)` → h-folders 또는 h-rows 100px 위로 |
| ✕ EXIT (`.ctrl-btn.exit`) | onclick 없음 (시각적 UI) |
| ▼ DOWN (`.ctrl-btn.teal`) | `scrollHud(1)` → 100px 아래로 |

---

### 3-3. LEFT RAIL — 집합 시각화

![LEFT RAIL 전체](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-left-rail.png)

**DOM:** `aside.lr-rail`  
`position:fixed; width:240px; height:448px; transform-origin:top right`  
배치: `#wrap` 왼쪽 가장자리 기준, `scale(__hudScale × 1.2)`  
1560px 이하 미디어쿼리에서 `display:none` → JS `injectRailCSS()`로 override

**패널 접기:** `.lr-h` 클릭 → `.lr-dp`에 `.collapsed` 토글 → `width:14px`, `.lr-b` visibility:hidden, `.arr` 화살표 회전

---

#### F.01 — Intersection / 집합도

![F.01 Intersection](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-lr-f01.png)

**SVG** `svg-set` · ViewBox: `18 17 184 167`

**무엇을 시각화하는가:** SET(작업 A)와 SET:Inclusion(작업 B) 두 작업의 교집합 구조. 두 원이 얼마나 겹치는지가 두 작업의 공유 문법 비율을 나타낸다.

| 요소 | 설명 |
|------|------|
| 배경 그리드 | 20px 간격 `rgba(255,74,22,.05)` 선 |
| X축 텍스트 | -40 ~ +40 (5개) — 집합 공간 좌표축 |
| 중심 십자선 | 원점 기준 수평/수직선 |
| `#lrcA` (원 A) | 중심 `(84+sin(t×0.5)×3, 100)`, r=50±4, 색상 `#ff7a2e`, stroke-dasharray="3 4" — **SET (A)** |
| `#lrcB` (원 B) | 중심 `(122, 100)`, r=50±4, 색상 `#b07bff` — **INCL (B)** |
| `#lrhA` | A 원 내부 해칭 패턴 (A와 동기화) |
| 교집합 표시 | 타원 `cx=103, cy=100, rx=22, ry=36` radialGradient 채움 → **A∩B 영역** |
| `A∩B = 32` 레이블 | 교집합 크기 텍스트 + 라인 `(103,100)→(160,60)` |
| `#lrtrace` | r=2.4 추적점 — 원 A 테두리를 공전 (`ang=t×1.2`), 좌표 `#lrtraceXY` 표시 |

**동적 업데이트 (60ms 인터벌, `t += 0.05`):**

| 요소 | 변화 |
|------|------|
| `set-pa` | 원 A 둘레 (~314.2 ± random 1.6) |
| `set-pd` | 전 프레임 대비 변화량 (+/-) |
| `set-int` | 교집합 크기 (30~34 랜덤) |
| `set-ratio` | 미니 바 너비 = `int / 5125 × 100`% (전체 합집합 대비 비율) |

---

#### F.02 — CONTOUR FIELD / 등고선

![F.02 CONTOUR FIELD](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-lr-f02.png)

**SVG** `svg-cont` · ViewBox: `0 0 220 200`

**무엇을 시각화하는가:** 작업의 밀도와 분포. 등고선이 촘촘할수록 해당 영역에 사진이 집중되어 있음을 나타낸다. 지형도 형식으로 작업 밀도를 공간화한다.

| 요소 | 설명 |
|------|------|
| 등고선 12레벨 | `blob(r, seed, t)` 함수 — 30개 점 불규칙 폐곡선 (sin/cos 파형 조합). 반경 10px~111.5px |
| 색상 | `rgba(255,106,30, 0.55 - i×0.03)` — 외곽일수록 투명 |
| 고도 레이블 | 3의 배수 레벨 (+12, +24, +36) 텍스트 |
| 전체 회전 | `rotate(sin(t×0.2)×4)` — 느리게 흔들림 |
| 북쪽 방향 | (198,24) 위치 화살표 |
| 중심 | 십자 + "PK +148" 레이블 (피크 고도) |
| `#lrcscan` | 수평 스캔라인 y=14→186 주기 이동 |

**SECTION A-A' 단면 (`svg-prof`)** · ViewBox: `0 0 300 30`

| 요소 | 설명 |
|------|------|
| `buildProfile(t)` | 가우시안 곡선 + sin 파형으로 수직 단면 생성 |
| `#lrpfscan` | 수직 스캔라인 (등고선 스캔 x좌표와 연동) |

**동적 업데이트 (50ms 인터벌, `t += 0.05`):**

| 요소 | 변화 |
|------|------|
| 등고선 경로 전체 | `blob()` 재계산으로 유기적 형태 변화 |
| `cont-grad` | `|sin(t×0.7)|×2.4` — 현재 경사 크기 |
| `cont-az` | `(t×8) % 360` 도 — 현재 방위각 |

---

#### F.03 — DATA / 측정값

![F.03 DATA](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-lr-f03.png)

**무엇을 시각화하는가:** F.01과 F.02의 수치 요약. SET 집합의 크기·경계·교집합 비율을 한눈에 파악하는 대시보드.

**4개 컬럼 (`lr-dcol`):**

**CARDINALITY (집합 크기)**

| 항목 | 값 | 설명 |
|------|-----|------|
| `\|A\|` | 2,847 (주황) | SET(A) 사진 총 수 |
| `\|B\|` | 2,310 (보라) | SET:Inclusion(B) 사진 총 수 |
| `\|A∩B\|` | `#set-int` 동적 (황금) | 교집합 크기 (30~34 변화) |
| `\|A∪B\|` | 5,125 | 합집합 총 수 |
| `\|A\B\|` | 2,815 | A에만 속한 수 |

**PERIMETER (경계)**

| 항목 | 값 | 설명 |
|------|-----|------|
| `∂A` | `#set-pa` 동적 | 원 A 둘레 (~314.2) |
| `Δ` | `#set-pd` 동적 | 전 프레임 대비 변화량 |
| ∩ RATIO 바 | `#set-ratio` 너비 | 교집합 / 합집합 비율 미니바 |

**ELEVATION BANDS**: `#cont-bands` — 등고선 레벨별 색상 바 목록

**GRADIENT (경사도)**

| 항목 | 값 | 설명 |
|------|-----|------|
| `∇mag` | `#cont-grad` 동적 | 현재 경사 크기 (`\|sin(t×0.7)\|×2.4`) |
| `azim` | `#cont-az` 동적 | 현재 방위각 (0~360°) |
| `pk` | 3 (고정) | 피크 수 |

---

### 3-4. RIGHT RAIL — 분석·실험

![RIGHT RAIL 전체](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-right-rail.png)

**DOM:** `aside.rr-rail`  
`position:fixed; width:240px; height:620px; transform-origin:top left`  
배치: `#wrap` 오른쪽 가장자리 기준, `scale(__hudScale × 1.2)`

---

#### F.04 — SVA RADAR

![SVA RADAR](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-rr-radar.png)

**SVG** `svg-rr-radar` · ViewBox: `0 0 210 210` · 중심 cx=105, cy=105, R=72

**무엇을 시각화하는가:** 선택된 사진의 SVA(Sequential Vision Autopsy) 형태학적 4축 분석값. 미학적·감정적 해석 없이 순수 형태학 수치만 레이더 차트로 표현.

**4개 축 (90도 간격, -90도 시작, 시계방향):**

| 축 | 방향 | 한국어명 | 설명 |
|----|------|---------|------|
| **DS** | 위 (0°) | 도메인 구조 | 프레임 지배 구조 — 화면을 지배하는 요소의 구조적 특성 |
| **SD** | 우 (90°) | 공간 배치 | 피사체와 공간의 분포·배치 방식 |
| **SA** | 아래 (180°) | 표면 이상 | 표면 텍스처·이상·노이즈 특성 |
| **KV** | 좌 (270°) | 고유 변칙 | 이 사진만의 고유한 변칙적 요소. 모든 축 중 최고 가중치(×0.35) |

**값 범위:** 0.00 ~ 1.00

**그리드 구조:**

| 요소 | 설명 |
|------|------|
| 4개 동심 정사각형 | r=R×k/4 (k=1~4) — 0.25, 0.50, 0.75, 1.00 눈금 |
| 4개 스포크 | 각 축 방향 직선 |
| 비율 눈금 텍스트 | 각 레벨에 0.25/0.50/0.75/1.00 표시 |
| 레이블 | 각 축 끝: 축코드(8px) + 한국어명(4.5px) |

**데이터 렌더링 (50ms 인터벌):**

| 요소 | 설명 |
|------|------|
| `cur[i]` | 4개 축 현재값. 3000ms마다 새 target 설정 → `cur[i] += (tgt[i]-cur[i]) × 0.08` (EMA 보간) |
| 전면 폴리곤 `#rr-3d` | `fill="rgba(255,122,46,.22)" stroke="#ff7a2e" stroke-width="1.4"` |
| 후면 폴리곤 | x+13, y-12 오프셋 → 3D 그림자 효과 |
| 측면 패널 4개 | 전·후면 연결 사각형, 각각 다른 투명도 (0.10/0.20/0.15/0.06) |
| 4개 축 도트 | 현재값 위치에 점 + 수치 텍스트 |
| ghost 폴리곤 `#rr-ghost` | 이전 값 잔상 (stroke-dasharray, fade) |
| score 계산 | `DS×0.2 + SD×0.2 + SA×0.25 + KV×0.35` |
| cuts 판정 | score×4 → index → `['DROP','DROP','SUPPORT','CORE']` |

---

#### ANALYSIS 패널

![ANALYSIS](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-rr-analysis.png)

**무엇을 시각화하는가:** 선택된 사진의 상세 수치 데이터. SVA RADAR의 숫자 버전.

**SPECIMEN 컬럼:**

| 요소 | ID | 설명 |
|------|-----|------|
| img ID | `#rr-id` | "img_000" 형식 |
| 촬영 회차 | `#rr-rnd` | "1차" 또는 "2차" |
| 인덱스 | `#rr-n` | "000/438" (현재/전체) |

**AXES 컬럼:**

| 요소 | ID | 설명 |
|------|-----|------|
| DS 수치 | `#rr-ds` | 0.00~1.00 |
| SD 수치 | `#rr-sd` | 0.00~1.00 |
| SA 수치 | `#rr-sa` | 0.00~1.00 |
| KV 수치 | `#rr-kv` | 0.00~1.00 (황금색 `.v.hi`) |
| KV 비중 바 | `#rr-kvbar` | 너비 = `cur[3]×100%`, 주황색 `#ff7a2e` |

**VERDICT 컬럼:**

| 요소 | ID | 설명 |
|------|-----|------|
| 종합 점수 | `#rr-score` | 0.00~1.00 |
| 판정 | `#rr-cut` | DROP / SUPPORT / CORE |

**연동:** GALLERY에서 사진 선택 시 → `admin-contents.json`의 `sva` 필드 (`{ds, sd, sa, kv}`)로 업데이트

---

#### PILOT CHANNEL (`.pilot-pnl`)

![PILOT CHANNEL](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-rr-pilot.png)

**DOM:** `.pilot-pnl` (right rail 최하단, height:180px)  
**역할:** SET의 문법 체계 바깥에 있는 이미지와 언어를 위한 실험 공간. "공집합 채널."

| 요소 | 설명 |
|------|------|
| 워터마크 | "PILOT" 텍스트 회전, `rgba(255,74,22,0.04)` |
| `.pilot-badge` | "PILOT" 배지 |
| 소개문 | "귀속 불가 이미지 실험소 — 공집합 채널 PILOT EXP" |

**3개 탭 (`.pl-nav span`):**

**NULL.LOG 탭 (`#pt-log`)**

| 항목 | 설명 |
|------|------|
| 데이터 소스 | `localStorage.getItem('set_null_log')` — JSON `{단어: 카운트, ...}` |
| 채우는 주체 | `measure.html`에서 검색 결과 없을 때 자동 기록 |
| 테이블 | DATE \| QUERY \| × |
| 강조 조건 | 카운트 ≥ 3이면 `.cand` (황금색) + `<span class="pl-cand-tag">후보</span>` |
| 의미 | 3회 이상 공집합 = 다음 작업 Exclusion의 입력 데이터 후보 |
| 빈 상태 | `#pt-log-empty` 표시 |

**ATTR.TEST 탭 (`#pt-attr`)**

| 항목 | 설명 |
|------|------|
| 드롭존 | `.pl-at-dz` 클릭 → `#pt-file.click()` |
| 지원 형식 | `accept="image/*"` — JPG, PNG, WEBP |
| `#pt-stat` | 상태 텍스트 ("READY" 기본) |
| 동작 | 이미지 업로드 → SET 문법 코드(G-01~G-14 등) 귀속 점수 분석 반환 |
| 귀속 불가 | "귀속 불가" 반환도 유효한 결과 |
| 전체 구현 | `pilot.html` — PILOT CHANNEL은 요약 진입점 |

**MANUAL 탭 (`#pt-manual`)**: NULL.LOG·ATTR.TEST 사용 안내문

**하단 상태바 (`.pl-stat`):**

| 탭 | lbl | v |
|----|-----|---|
| log | `NULL.LOG` | `0 REC` |
| attr | `ATTR.TEST` | `83 CODES` |
| manual | `MANUAL.TXT` | `안내` |

**하단 버튼:** "OPEN PILOT.EXP" → `crtNav('https://note-a.github.io/pilot.html')`

---

### 3-5. GBAR — 중간 타이틀바

![GBAR](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-gbar.png)

**DOM:** `div#gbar` → `<iframe src="gbar.html?v=14">` (height:84px)  
**레이아웃:** `.bar` — `grid-template-columns: 250px 370px minmax(0,1fr) 165px`

---

**세그먼트 1 — `.seg-id` (250px)**

| 요소 | 설명 |
|------|------|
| SVG 글리프 (46×46px) | 두 개 회전 헥사곤 (`.hex` 9s, `.hex2` 역방향 6s) + 레이더 라인 (2.4s) + 틱 (역 16s) |
| `.ttl[data-text="SET.OS"]` | "SET.OS" 타이틀, 글리치 animation `tglitch 4s` |
| `.sub` | "ARCHIVE TERMINAL v2.4" |
| `.boot` | Typewriter 애니메이션 — 6개 메시지 타입/삭제 루프: `['> mount /archive ... ok', '> index 4,210 frames', '> cull queue: 432', '> render farm online', '> node-77 linked', '> sync delta +18']` |

**세그먼트 2 — `.seg-nav` (370px, 네비게이션)**

| 항목 | 이름 | 링크 |
|------|------|------|
| `[0]` `.on` | **SET** | `gallery1.html` |
| `[1]` | **SET:Inclusion** | `parent.mobComingSoon()` → COMING SOON overlay |
| `[2]` | **LOG** | `page-log.html` |
| `[3]` | **Reference** | `reference.html` |
| `[4]` | **MEASURE** | `measure.html` |

**세그먼트 3 — `.seg-stats` (세션 정보)**

| 요소 | ID | 설명 |
|------|-----|------|
| ENTRY | `#g-entry` | 페이지 로드 시각 `HH:MM:SS` |
| SESSION | `#g-sess` (amber) | 로드 후 경과 시간, 1초 인터벌 업데이트 |
| STATUS | `#g-status` | 기본 "● ACTIVE" (녹색). 5600ms마다 → "◌ SCANNING" (amber) 900ms 후 복귀 |
| DEPT | `#g-dept` | "KR-09-A" (고정) |

**세그먼트 4 — `.seg-stmt` (Statement 발췌)**  
`// STATEMENT` 태그 (pulse dot) + 고정 영문 텍스트 2단락

---

### 3-6. BOTTOM ROW

![BOTTOM ROW 전체](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-bot.png)

**DOM:** `div#bot-row`  
**레이아웃:** `grid-template-columns: minmax(0,2.6fr) minmax(0,1fr)`  
**bot-left:** `grid-template-columns: 215px 200px minmax(0,1fr)`

---

#### G-1. SYSTEM NOTICE + G-2. VIEWER_LOG (`.bl-split`)

![G-1 NOTICE / G-2 VIEWER_LOG](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-notice.png)

**SYSTEM NOTICE (`.bl-split > .pnl:nth-child(1)`)**

| 요소 | 설명 |
|------|------|
| `.ceb-title` | "시스템 공지" |
| `.ceb-main` | "채팅방에 입장했습니다" |
| `.ceb-sub` | "MEASURE.DB 검색 채팅방" |
| `.ceb-go` | "[ 입장 ▶ ]" 버튼 — `<a href="measure.html" target="_blank">` |

**VIEWER_LOG.EXE (`.bl-split > .pnl:nth-child(2)` → `#vlWindow`)**

| 요소 | 설명 |
|------|------|
| 헤더 | "VIEWER_LOG.EXE" + 최소화 버튼 `_` (`#vlBody` display toggle) |
| `#vlDot` | 5px pulse dot |
| 텍스트 | "MEASURE SEARCH LOG" |
| `#vlCount` | "0 queries" (누적 카운트) |
| `#vlList` | 최대 5줄 유지. `list.prepend(e)` + lastElementChild 제거 |

**로그 행 포맷:**

| 컬럼 | 설명 |
|------|------|
| `.vlt` | 시간 (HH:MM:SS) |
| `.vlq` | 쿼리 텍스트 |
| 색상 나이 | 최신→오래됨: `#ff4a16` → `rgba(255,74,22,.62)` → `.38` → `.24` → `.13` |
| 진입 animation | `vlSlideIn .3s ease` |

**데모 피드 (자동 생성):**

| 항목 | 설명 |
|------|------|
| 쿼리 풀 | `['crash','floating','shadow','grid','FOGDIS','cluster','rupture','isolate','strata','dmgpat','tonedis','diptych','matcov','headobj','backpose']` |
| 초기 3개 | 450ms 간격으로 표시 |
| 이후 | 3400ms 랜덤 인터벌 |
| 풋터 | `/measure/search` + `#vlLastTime` (마지막 쿼리 시각) + 블링크 커서 `█` |

---

#### G-2b. VIEWER 3D + ID CARD

![G-2b VIEWER 3D + ID CARD](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-viewer-id.png)

**DOM:** `.bot-left > .pnl:nth-child(1)` → `<iframe src="id-card.html?v=7">` (200px 컬럼)

**헤더 (18px, 주황 배경 검정 텍스트):** `▶ VISITOR.ID-CARD` + badge + LIVE dot

**IDENTITY 섹션:**

| 요소 | ID | 주기 | 내용 |
|------|-----|------|------|
| NAME | `#v-name` | 4700ms | "UNKNOWN" + 글리치 6라운드 60ms |
| ID | `#v-id` | 3200ms | "VIS-2026-###" → sfx 순환: `'###','4A2','F91','??7','C0D','E38'` |
| ROLE | `#v-role` | 5800ms | `OBSERVER → VISITOR → UNKNOWN → ANALYST → SUBJECT` 순환 |
| CLR | `#v-clr` | 900ms | "GUEST" 색상 토글: `#d96a0e` ↔ `rgba(217,106,14,0.3)` |

**SYS MONITOR 섹션:**

| 요소 | ID | 설명 |
|------|-----|------|
| STATUS | `#v-status` | "● ACTIVE" 기본. 이상 시 "⚠ ANOMALY" |
| INTEG | `#v-intval` | 98.7% ±0.35 랜덤워크 (범위 95.5~99.9). 0.8% 확률로 이상 트리거 |
| INTEG 바 | `#v-intbar` | 동일 값 너비 % |
| THREAT | `#v-threat` | "◉ LOW" / "◉ MED" / "◉ HIGH" (이상 시 상승) |
| PKT/s | `#v-pkt` | 0000~9999 (75ms 인터벌 랜덤 업데이트) |

**아바타 캔버스 (`#c1`, 152×80):**

| 요소 | 설명 |
|------|------|
| 벡터 실루엣 | `headPath` + `hairPath` bezier 곡선으로 인물 실루엣 |
| 조명 | radialGradient 좌우측 독립 조명 |
| 호흡 pulse | `tick` 변수 sin파 |
| orbit particles | 26개 (랜덤 반경·속도·크기) |
| 인터벌 | 40ms |
| 스캔라인 | `scanline overlay` + `sweep light` |
| 코너 브래킷 | 4개 |
| 텍스트 | "SCAN:OK" (30프레임마다 깜빡임) |

**바코드 캔버스 (`#bc1`, 180×14):** 60개 바 (1px or 2px). 97% `#d96a0e`, 3% `#ff9a55` 노이즈. 좌→우 sweep

**풋터:** `#v-now` 현재 날짜 `YYYY.MM.DD`

---

#### G-3. WORK STATUS

![G-3 WORK STATUS](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-work.png)

**DOM:** `.bot-left > .pnl:nth-child(2)` → `<iframe src="work-status.html?v=10">`

**데이터 소스:**

| API | URL |
|-----|-----|
| GitHub Commits | `https://api.github.com/repos/note-a/note-a-github/commits?per_page=50` |
| Posts | `https://raw.githubusercontent.com/note-a/note-a-github/main/published-posts.json` |
| 캐시 key | `ws_commits` / `ws_posts` (localStorage, TTL 5분) |

**커밋 분류 `classify(msg)` → `[클래스, 라벨]`:**

| 패턴 | 클래스 | 라벨 |
|------|--------|------|
| fix\|bug\|revert\|error | `drop` | FIX |
| add\|feat\|new\|creat | `core` | ADD |
| update\|upd\|sync\|push | `sync` | UPD |
| rename\|move\|refactor | `tag` | MOV |
| style\|design\|css\|ui\|font | `cull` | STY |
| 기타 | `scan` | LOG |

**상단 stats strip (`.f-stats`, 4컬럼):**

| 항목 | ID | 설명 |
|------|-----|------|
| POSTS | `#s-core` (yellow) | `publishedSection==='rule'` 조건 posts 수 |
| COMMITS | `#s-sup` (amber) | `commits.length` (최대 50) |
| THIS WK | `#s-drop` | 7일 이내 커밋 수 |
| LAST | `#s-q` | 최신 커밋 날짜 `MM/DD` |

**NOW 진행 바:**

| 요소 | 설명 |
|------|------|
| `#now-tgt` | 최신 커밋 메시지 (38자 truncate) |
| `#now-bar` | 너비 = `freshPct%` = `100 - (경과일/7×100)` (최소 2%) |
| `#now-pct` | freshPct 텍스트 |

**THROUGHPUT 스파크라인 (`#spk`):**

| 요소 | 설명 |
|------|------|
| 바 수 | 28개 (NB=28) |
| 데이터 | 날짜별 커밋 수 집계 (최근 28일) |
| 마지막 바 | `.hi` — 황금색 + glow 효과 |
| `#spk-peak` | 최대값 표시 |

**커밋 피드 (`#f-list`):**

| 요소 | 설명 |
|------|------|
| 표시 수 | 최신 14개 |
| 각 행 | 시간경과 \| 분류 배지 \| 메시지 (52자) |
| opacity | `max(0.28, 1-i×0.06)` — 오래될수록 흐림 |
| 시간경과 | `timeAgo()`: just now / Xm ago / Xh ago / Xd ago |
| 갱신 주기 | 1분마다 재렌더 |

**LIVE 클럭:** `#f-clk` 현재 시각 HH:MM:SS (1초 인터벌)

---

#### G-4. SET.TEXT

![G-4 SET.TEXT](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-settext.png)

**DOM:** `#bot-row > .pnl` → `#stmt.stmt-scroll`

| 요소 | 설명 |
|------|------|
| `.lead` | "세계를 측정하려면 세계 바깥에 서야 한다. 그 자리는 없다." |
| 본문 단락 | SET 구조, 측정 논리, MEASURE 의미, 유예 선언 4개 단락 |
| `.closing` | "이 작업의 이름은 유예다." |
| `.stmt-caret` | 깜빡이는 커서 animation `1s step-end` |
| hover | `background: rgba(255,74,22,.05)` |
| click | `openStmt()` → STATEMENT 모달 열기 |

**숨겨진 PC통신 채팅 (`.pc`, `display:none`):**  
타이틀: "■ MEASURE.DB 대화방 ■" · 접속자 5명 · DIR 45개 · 사진 6,421장 · 선택 438장  
로그 형식: `HH:MM | [ID] │ 메시지`  
자동 랜덤 메시지: 14초~36초 간격, 4명 사용자  
명령어: `/search`, `/who`, `/help`, `/exit`

---

### 3-7. 오버레이 & 모달

#### COMING SOON 오버레이

![COMING SOON](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-modal-coming.png)

| 항목 | 설명 |
|------|------|
| DOM | `div#mob-toast` (`position:fixed; z-index:9000`) |
| 트리거 | `window.mobComingSoon()` 전역 함수 |
| 내용 | ASCII art (`SET INCLUSION` 텍스트) + "COMING SOON" + "TAP TO DISMISS" |
| 자동 닫힘 | 3500ms 후 `.show` 제거 |
| 클릭 닫힘 | 클릭 시 즉시 `.show` 제거 |
| gbar 연동 | gbar 내부에서 `window.parent.mobComingSoon()` 호출 |

#### STATEMENT MODAL

![STATEMENT MODAL](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-modal-stmt.png)

| 항목 | 설명 |
|------|------|
| DOM | `div#stmt-modal` (`position:fixed; z-index:200`) |
| 트리거 | `openStmt()` (SET.TEXT 클릭 또는 STATUS BAR 버튼) |
| 닫기 | `closeStmt()` (배경 클릭 또는 ESC) |
| 헤더 | "ARTIST STATEMENT" (12px, `#ffb020`) |
| 본문 섹션 | 집합과 세계 · 재현과 시선 · 제도 · 관찰자 · AI 작업 프로세스 (사진 분류/규칙 형성/패턴 분석/문법-지시문 변환/촬영 과정/셀렉 기준) · Inclusion · 시선의 구조 · 유예 |
| 영어 인용 | `.en` 클래스 — 이탤릭 |
| 예시 | `.ex` 클래스 — blockquote 스타일 |
| 각주 | `.note` — AI 사용 고지 |
| "MORE ▶" | `openDocs()` — DOCS 모달으로 연결 |

#### DOCS MODAL

![DOCS MODAL](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-modal-docs.png)

| 항목 | 설명 |
|------|------|
| DOM | `div#docs-modal` (`position:fixed; z-index:210` — STATEMENT 위) |
| 트리거 | `openDocs()` |
| 닫기 | `closeDocs()` (배경 클릭 또는 ESC) |
| 헤더 | "WORK_DOCUMENT.EXE — 작업 설명서" |
| 섹션 구성 | 0. 이 문서의 목적 / 1. 작업의 출발점과 역사 / 2. 작업의 핵심 개념 / 3. 작업의 방법론 / 4. 문법 생성 / 5. 지시문(DIR) / 7. 개념문 핵심 문장 |
| 핵심 용어 table | 집합·세계·둘레(Perimeter)·시선의 구조·언어·주체·사진·유예 (8행) |
| SVA 4축 table | DS(프레임 지배 구조) / SD(공간 배치) / SA(표면 이상) / KV(고유 변칙) |

---

### 3-8. STATUS BAR

![STATUS BAR](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-statusbar.png)

**DOM:** `div#botbar` (height:28px)  
**레이아웃:** `border-top: 1px solid var(--g4)`

**좌측 `.binfo`:** `\GH.24` · TR 23.3 · Slow  
**두 번째 `.binfo`:** ▲ ERR·03 · SECTOR·F (주황-빨간)

**`.mode-row` — 6개 `.mb` 네비게이션 버튼:**

| 인덱스 | 이름 | 색상 | 동작 |
|--------|------|------|------|
| `[0]` `.on` | **SET** | `#ff6a26` | `crtNav('gallery1.html')` |
| `[1]` | **SET:inclusion** | `#ffa01e` | `mobComingSoon()` |
| `[2]` | **LOG** | `#ff9a2e` | `crtNav('page-log.html')` |
| `[3]` | **Reference** | `#ffa820` | `crtNav('reference.html')` |
| `[4]` | **MEASURE** | `#ff7a1f` | `crtNav('measure.html')` |
| `[5]` | **ABOUT** | `#ffa824` | `data-sec="about"` |

각 버튼 `.mb`: `clip-path: polygon(8px 0, ...)` 오각형 형태. `.k` 스팬: 단축키 문자 (S/I/L/R/M/A)

**ACTIVATE 버튼:** 클릭 이벤트 없음, border glow 시각 효과만

**우측 `.binfo`:** "SET XT2" + `#clk2` 시:분 (1초 인터벌)

**STATEMENT / DOCS 버튼:** `openStmt()` / `openDocs()`

---

## 4. CRT 화면 전환 효과

**`window.crtNav(url)`** 전역 함수:

| 단계 | 설명 |
|------|------|
| 1 | `body.classList.add('crt-leaving')` |
| 2 | `#shell`에 `crtOff` animation 0.46s: `scale(1,1) → scale(1,.02) → scale(.001,.012)` + brightness 2.8→5 |
| 3 | `#crt-off`에 `crtOffBg` animation: 55% 지점부터 opacity 0→1 |
| 4 | animationend 또는 560ms fallback → `location.href = url` |

**페이지 진입 시:**

| 단계 | 설명 |
|------|------|
| 1 | `DOMContentLoaded` → `body.classList.add('crt-entering')` |
| 2 | `crtOn` animation: `scale(.001,.012) → scale(1,.02) → scale(1,1)` + brightness 역방향 |
| 3 | animationend 또는 800ms → `classList.remove('crt-entering')` + `dispatchEvent(new Event('resize'))` (레일 재배치) |

**접근성:** `prefers-reduced-motion` 시 animation 없이 즉시 이동

---

## 5. 데이터 구조

### admin-contents.json 스키마

```json
{
  "about": {
    "artistStatement": "string"
  },
  "project": {
    "photos": [
      {
        "filename": "images/2025/03/14/DSCF0001.jpg",
        "dir": "FOGDIS",
        "id": "img_001",
        "verdict": "CORE",
        "sva": { "ds": 0.72, "sd": 0.58, "sa": 0.41, "kv": 0.88 }
      }
    ]
  },
  "contactSheetPhotos": [
    { "filename": "...", "dir": "...", "verdict": "...", "title": "...", "keywords": "...", "date": "..." }
  ],
  "rule": {
    "items": [
      { "id": "RULE_1", "title": "...", "author": "...", "year": "...", "category": "...", "publishedSection": "rule" }
    ]
  }
}
```

**flatten 로직:** `data.project`를 재귀 탐색 → `filename` 키가 있는 노드 수집

### GL_FRAMES 빌드 과정

```
admin-contents.json 로드
 → flatten(data.project) → 사진 배열
 → Fisher-Yates 셔플
 → 각 항목에 n(현재 인덱스)/total(전체 수) 재설정
 → GL_FRAMES 배열 교체
 → glIdx=0, glAThumbClick(0) 호출
```

### resolveUrl / thumbUrl

```javascript
resolveUrl(f):
  f.startsWith('http') || f.startsWith('/') || f.startsWith('images/')
    ? f
    : 'images/' + f

thumbUrl(u):
  u.replace(/\/DSCF([^/]+)\.(jpg|JPG)$/, '/t/DSCF$1.jpg')
  // 썸네일은 /t/ 서브디렉토리의 동일명 파일
```

### published-posts.json 스키마

```json
[
  {
    "id": "string",
    "title": "string",
    "author": "string",
    "year": "string",
    "category": "string",
    "publishedSection": "rule"
  }
]
```

`publishedSection === 'rule'`인 항목만 LOG 패널에 사용

### localStorage 키

| 키 | 형식 | TTL | 용도 |
|----|------|-----|------|
| `set_null_log` | `{"단어": 카운트, ...}` | 영구 | PILOT NULL.LOG 검색어 누적 |
| `ws_commits` | `{ts: timestamp, data: commits[]}` | 5분 | GitHub Commits API 캐시 |
| `ws_posts` | `{ts: timestamp, data: posts[]}` | 5분 | posts API 캐시 |

### 주요 전역 변수

| 변수 | 설명 |
|------|------|
| `GL_FRAMES[]` | 갤러리 프레임 배열 |
| `ALL_DIRS[]` | 45개 DIR 코드와 사진 수 `{code, n}` |
| `DIRS[]` | 상위 11개 DIR |
| `LOG_ITEMS[]` | RULE_1~RULE_5 (5개 SET Fundamental) |
| `REF_ITEMS[]` | 10개 참고문헌 (Barthes, Sontag 등) |
| `galPhotos[]` | fetch된 contactSheetPhotos |
| `glIdx` | 현재 갤러리 인덱스 |
| `window._gvaTimer` | 8초 자동전환 interval ID |

### 주요 전역 함수

| 함수 | 설명 |
|------|------|
| `init()` | 메인 초기화 (buildGlScatter, buildGvaRadar, buildCrosshair, tick, fetchData, bindEvents) |
| `fetchData()` | admin-contents.json 로드 |
| `crtNav(url)` | CRT 전환 후 이동 |
| `glSwitch(n)` | GALLERY 탭 전환 (1=ANALYSIS, 2=CONTACT) |
| `glAThumbClick(idx)` | 갤러리 사진 선택 |
| `buildCs2Grid()` | CONTACT sheet 빌드 |
| `buildGlScatter()` | DIST scatter 빌드 |
| `buildCrosshair()` | GVA 조준 레티클 SVG 생성 |
| `openStmt()` / `closeStmt()` | STATEMENT 모달 |
| `openDocs()` / `closeDocs()` | DOCS 모달 |
| `mobComingSoon()` | COMING SOON overlay (전역) |
| `scrollHud(dir)` | HUD 스크롤 (+1/-1) |
| `setHudView(v)` | HUD icon/list 전환 |

---

## 6. 반응형 대응

| 브레이크포인트 | 조건 | 동작 |
|--------------|------|------|
| `> 1560px` | 기본 | 좌우 레일 표시 |
| `1025~1560px` | 비터치 | 레일 `display:none` |
| `768~1560px` | `pointer:coarse` (터치기기) | 레일 `display:flex !important` |
| `≤ 767px` | 모바일 | 완전 모바일 레이아웃 |

**`window.__hudScale()` 스케일 계산:**
```
railsFit 조건이면 base=1800, 아니면 base=1140
s = Math.min(w/base, innerHeight/757, 1)
#wrap.style.transform = 'scale('+s+')'
레일: scale(s×1.2) + wrap BoundingClientRect 기준 fixed 재배치
```

**모바일 레이아웃 (`≤ 767px`):**

| 요소 | 데스크탑 | 모바일 |
|------|----------|--------|
| `#wrap`, `#gbar`, `#bot-row`, 레일 | 표시 | `display:none!important` |
| `#mob-main` | 숨김 | 표시 (개념문 텍스트) |
| `#mob-hdr` | 숨김 | 고정 헤더 64px + 햄버거 |
| `#mob-drawer` | 숨김 | 우측 슬라이드 nav (`translateX`) |

---

## 7. 제약사항

| 구분 | 내용 |
|------|------|
| **정적 호스팅** | GitHub Pages 기반. 서버 사이드 없음. 모든 기능 클라이언트 JS |
| **GitHub API 레이트리밋** | 비인증 60회/시간. localStorage 5분 TTL 캐시 완화. 초과 시 stale 캐시 |
| **VIEWER_LOG 실시간성** | 현재 데모 피드. 실시간 구현 시 Firebase/Supabase 필요 |
| **이미지 용량** | GitHub Pages 리포 1GB 제한. 대용량 시 외부 CDN |
| **SVA 데이터** | 현재 랜덤 시뮬레이션. 실제 AI 분석 파이프라인 없으면 수동 입력 |
| **iframe 통신** | gbar.html ↔ set-os.html 간 `window.parent`. 동일 오리진 한정 |
| **NULL.LOG 지속성** | localStorage 브라우저/기기별 분리. 서버 공유 불가 |
| **고정 해상도** | 메인 영역 1140×757px 고정. 브라우저 zoom 비대응 |
| **검색 기능** | HUD LOG 검색창 UI만 존재, 이벤트 바인딩 미구현 |
| **EXIT 버튼** | 상단 컨트롤 ✕ EXIT 버튼 onclick 미구현 |
| **ABOUT 버튼** | STATUS BAR ABOUT 버튼 동작 미구현 |

---

## 8. 확장성

### 단기 — 현재 구조 내 즉시 가능
- `admin-contents.json` 사진 추가 → GALLERY·DIST·SVA RADAR 자동 반영
- `set_null_log` localStorage + measure.html 공집합 로직 연동 → NULL.LOG 자동 누적
- PILOT CHANNEL → pilot.html 실제 ATTR.TEST 결과 연동
- HUD 검색창 이벤트 바인딩 추가 → 폴더/파일 실시간 필터

### 중기 — 구조 개선 필요
- **VIEWER_LOG 실시간화**: Firebase Realtime DB 또는 Supabase 연동
- **NULL.LOG 서버 공유**: localStorage → 서버 DB로 전체 방문자 공집합 집계
- **SVA 자동화**: AI 분석 API 연동. 신규 사진 업로드 시 DS/SD/SA/KV 자동 생성
- **GALLERY 연동**: GALLERY 사진 클릭 → SVA RADAR 실제 값으로 업데이트

### 장기 — 아키텍처 전환
- **Exclusion 작업 섹션**: NULL.LOG 데이터 기반 다음 작업 아카이브
- **SET:Inclusion 공개**: COMING SOON 해제 + 별도 아카이브 구성
- **교집합 동적 시각화**: 실제 채택 데이터로 F.01 Intersection 동적 렌더링
- **CMS 연동**: JSON 직접 편집 → Headless CMS

---

## 9. 파일 구성

| 파일 | 역할 |
|------|------|
| `set-os.html` | 메인 OS 인터페이스 (4,571줄) |
| `gbar.html` | 중간 타이틀바·네비게이션 (iframe, 257줄) |
| `work-status.html` | 작업 현황 패널 (iframe, 253줄) |
| `id-card.html` | 작가·프로젝트 신원 카드 (iframe, 238줄) |
| `gallery1.html` | SET 갤러리 전체 페이지 |
| `page-log.html` | LOG 전체 페이지 |
| `reference.html` | 참조 문헌 전체 페이지 |
| `measure.html` | MEASURE 검색 전체 페이지 |
| `pilot.html` | PILOT CHANNEL 전체 페이지 (NULL.LOG + ATTR.TEST) |
| `admin-contents.json` | 사진 메타데이터 데이터 소스 (현재 비어 있음) |
| `published-posts.json` | 발행 포스트 목록 |

---

*SET.OS — 측정이 지속된다.*
