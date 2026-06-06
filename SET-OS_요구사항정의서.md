# SET.OS — 요구사항 정의서

**문서 버전:** v2.0  
**작성일:** 2026-06-06  
**대상 URL:** https://note-a.github.io/set-os.html  
**관련 파일:** `set-os.html`, `gbar.html`, `work-status.html`, `id-card.html`, `pilot.html`, `measure.html`

---

## 1. 제작 목적

### 배경 — SET 사진 작업이 온라인으로 이동한 이유

SET는 사진 작업이지만 사진으로 시작하지 않는다. 먼저 지시문(DIR)이 쓰인다. 어떤 조건 아래 어떤 것을 촬영할 것인지 — 그 조건이 현실에서 발견될 때 셔터를 누른다. 촬영이 끝나면 사진들로부터 문법이 추출되고, 문법은 다시 지시문이 된다. 이 순환이 SET의 구조다.

집합(Set)은 수학적 의미에 가깝다. 조건이 충족되면 반드시 귀속된다. 탈출이 없다. 사진은 미적 완성의 결과물이 아니라 지시문의 조건이 현실에 존재했다는 실행 기록이다.

**온라인 이동은 아카이빙이 아니다.** SET의 논리가 다른 방식으로 계속되는 것이다. 이 전제 아래 set-os.html은 세 가지 기능을 동시에 수행하도록 설계되었다.

1. **측정 장치**: 관객이 수동적으로 사진을 보는 것이 아니라, MEASURE를 통해 지시문 언어를 직접 입력하면 교집합이 생성된다. 관객의 검색이 교집합을 만들고, 그 기록은 작업 안에 누적된다. 측정하는 자도 측정된다.

2. **과정의 가시화**: 어떤 사진이 CORE(채택)·SUPPORT(보조)·DROP(탈락)되었는지, 어떤 조건이 반복 출현했는지를 데이터로 드러낸다. 선별된 결과만이 아니라 과정 전체가 공개된다.

3. **집합의 확장**: MEASURE 외에도 두 가지 기능이 추가된다. ATTR.TEST는 SET 바깥의 이미지를 업로드하면 SET의 문법 코드 중 어디에 가장 가까운지 분석해 반환한다 — 문법이 측정 도구가 된다. NULL.LOG는 어디에도 귀속되지 못한 검색어와 이미지를 자동 수집하고, 이것들이 다음 작업 Exclusion의 데이터가 된다 — 집합이 자신의 후속을 스스로 준비한다.

SET는 미결인 채로 공개되어 있다. 종결을 향하지 않는다. 유예(Suspension) — 측정이 지속된다. set-os.html은 그 측정의 인터페이스다.

### 인터페이스 원칙

- 갤러리가 아닌 **터미널**: 사진을 감상하는 공간이 아니라 데이터에 접근하는 시스템
- CRT 미학(주황/검정, 스캔라인, 모노스페이스 폰트)은 장식이 아니라 "이 시스템은 측정 중이다"라는 작업 맥락의 시각적 번역
- 고정 해상도(1140×757px)의 단일 창 형식. 크기가 아니라 밀도로 정보를 전달

---

## 2. 시스템 구조

![SET.OS 전체 인터페이스](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-full.png)

```
┌───────────────────────────────────────────────────────────────────────┐
│  LEFT RAIL          │        MAIN AREA (1140×757px)         │ RIGHT RAIL │
│                     │                                        │            │
│  F.01 Intersection  │  TOP ROW:                              │  F.04      │
│  (집합도)           │   <GALLERY>  │  <LOG>  │  [CTRL]       │  SVA RADAR │
│                     │                                        │            │
│  F.02 CONTOUR FIELD │  GBAR (중간 타이틀바, iframe)          │  ANALYSIS  │
│  (등고선)           │                                        │            │
│                     │  BOTTOM ROW:                           │  PILOT     │
│  F.03 DATA          │   [MEASURE 입장] + [VIEWER_LOG.EXE]   │  CHANNEL   │
│  (측정값)           │   [ID CARD]  [WORK STATUS]  [SET.TEXT] │            │
└───────────────────────────────────────────────────────────────────────┘
```

**데이터 소스:** `admin-contents.json` (사진 메타데이터), `published-posts.json` (포스트 목록), GitHub Commits API

---

## 3. 패널별 기능 명세

---

### 3-1. LEFT RAIL — 집합 시각화

![LEFT RAIL 전체](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-left-rail.png)

#### F.01 — Intersection (집합도)

![F.01 Intersection](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-lr-f01.png)

| 항목 | 내용 |
|------|------|
| 역할 | SET와 SET:Inclusion 두 작업의 교집합 구조를 SVG 벤 다이어그램으로 시각화 |
| 구현 | SVG 기반 동적 집합 다이어그램. 원 두 개의 겹침 면적이 교집합 |
| 데이터 | 현재 정적. 추후 실제 채택률 데이터 연동으로 교집합 면적 동적 변화 가능 |
| 상태 | 헤더 클릭 시 패널 접기/펼치기 |

#### F.02 — CONTOUR FIELD (등고선)

![F.02 CONTOUR FIELD](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-lr-f02.png)

| 항목 | 내용 |
|------|------|
| 역할 | 작업 밀도와 분포를 등고선 지형으로 표현 |
| 구현 | SVG path 기반 등고선 애니메이션 + 하단 단면 프로파일(SECTION A–A′) |
| 데이터 | 현재 정적 시각화 (개념적 밀도 표현). DIR별 촬영 밀도 연동 시 실측 분포 지도로 전환 가능 |
| 상태 | 헤더 클릭 시 패널 접기/펼치기 |

#### F.03 — DATA (측정값)

![F.03 DATA](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-lr-f03.png)

| 항목 | 내용 |
|------|------|
| 역할 | 전체 작업 수치 요약 |
| 표시 항목 | 집합 크기(｜A｜, ｜B｜, ｜A∩B｜, ｜A∪B｜), 경계 길이(∂A), 교집합 비율, 등고선 고도 밴드, 그래디언트 |
| 데이터 | admin-contents.json 집계값 또는 하드코딩 |

---

### 3-2. RIGHT RAIL — 분석·실험

![RIGHT RAIL 전체](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-right-rail.png)

#### F.04 — SVA RADAR (4축 형태학 분석)

![SVA RADAR](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-rr-radar.png)

| 항목 | 내용 |
|------|------|
| 역할 | 선택된 사진의 SVA(Sequential Vision Autopsy) 4축 분석값을 레이더 차트로 표시 |
| 4축 구성 | DS (도메인 구조) / SD (피사체 세부) / SA (표면 분석) / KV (핵심 변수) |
| 구현 | SVG 기반 레이더 차트. GALLERY에서 사진 클릭 시 해당 사진의 분석값으로 업데이트 |
| 데이터 | admin-contents.json의 sva 필드 |
| 원칙 | 미학적·감정적 해석 완전 배제. 형태학적 수치만 표시 |

#### ANALYSIS — 분석 데이터

![ANALYSIS](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-rr-analysis.png)

| 항목 | 내용 |
|------|------|
| 역할 | 선택된 사진의 상세 데이터 표시 |
| 표시 항목 | 이미지 ID, 촬영 회차(round), 인덱스(n/전체), DS·SD·SA·KV 수치, KV 비중 바, 종합 점수(score), 판정(CORE/SUPPORT/DROP) |
| 연동 | GALLERY 사진 선택 시 실시간 업데이트 |

#### PILOT CHANNEL

![PILOT CHANNEL](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-rr-pilot.png)

| 항목 | 내용 |
|------|------|
| 역할 | SET의 문법 체계 바깥에 있는 이미지와 언어를 위한 실험 공간. "공집합 채널" |
| 탭 구성 | **NULL.LOG** / **ATTR.TEST** / MANUAL |
| 접근 방식 | 패널 하단 "OPEN PILOT.EXP" 버튼 → pilot.html 전체 페이지 이동 |

**NULL.LOG 탭**

| 항목 | 내용 |
|------|------|
| 역할 | MEASURE에서 검색 결과가 없었던 단어(공집합 검색어)를 자동 수집·표시 |
| 동작 | measure.html의 공집합 발생 시 localStorage(`set_null_log`)에 키워드 + 횟수 누적. 3회 이상 = 지시문 후보로 분류 |
| 의미 | 이 데이터는 다음 작업 **Exclusion**의 입력 데이터가 된다. NULL.LOG에 쌓이는 것들이 집합의 바깥을 정의하고, 그 바깥이 다음 작업의 시작 조건이 된다 |
| 표시 | 날짜·키워드·횟수 목록. 3회↑ 항목은 시각적 구분 처리 |

**ATTR.TEST 탭**

| 항목 | 내용 |
|------|------|
| 역할 | 외부 이미지를 업로드하면 SET의 어떤 문법 코드(G-01~G-14 등)에 가장 가까운지 점수 순으로 분석 반환 |
| 동작 | 드래그 또는 클릭으로 이미지 업로드 → 문법 코드별 귀속 점수 표시 |
| 귀속 불가 | 어떤 문법으로도 설명되지 않는 경우 "귀속 불가" 반환. 이것도 유효한 결과 |
| 의미 | SET의 문법이 내부 분류 체계를 넘어 외부 이미지를 측정하는 도구가 된다 |
| 전체 기능 | pilot.html에서 완전 구현. PILOT CHANNEL 패널은 요약 진입점 |

---

### 3-3. TOP ROW — 메인 콘텐츠

![TOP ROW 전체](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-top.png)

#### `<GALLERY>` 패널 (Top-Left)

![GALLERY](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-gallery.png)

| 항목 | 내용 |
|------|------|
| 역할 | SET 사진 아카이브의 메인 뷰어 |
| 탭 구성 | **ANALYSIS**(기본) / **CONTACT**(컨택 시트) |

**ANALYSIS 뷰 (기본)**

| 항목 | 내용 |
|------|------|
| 좌측 | 현재 사진 1장 전체 표시. GALLERY·ANALYSIS 레이블, DIR코드, 날짜, EXIF 오버레이. 스캔라인·크로스헤어 등 CRT 스코프 효과 |
| 클릭 동작 | "ACCESS GRANTED" 애니메이션 표시 후 상세 모드로 전환 |
| 우측 (DIST) | 전체 DIR의 사진 분포를 산포도(scatter plot)로 시각화. 금색=CORE, 산호=SUPPORT, 빨강=DROP |
| 탐색 | ◄► 화살표로 사진 이동 |
| 메타데이터 | 좌하단: STATUS(CORE/SUPPORT/DROP), 인덱스 표시 |

**CONTACT 뷰**

| 항목 | 내용 |
|------|------|
| 역할 | 전체 사진을 컨택 시트 그리드로 표시 |
| 표시 | DIR 정보 + CORE/SUPPORT/DROP 수량 요약 헤더 |
| 데이터 소스 | `admin-contents.json` → GL_FRAMES 배열. 미로드 시 하드코딩 샘플 폴백 |

#### `<LOG>` 패널 (Top-Right)

![LOG ICON 뷰](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-log.png)

![LOG LIST 뷰](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-log-list.png)

| 항목 | 내용 |
|------|------|
| 역할 | 작업의 문법·지시문 문서 파일 브라우저. 작업 전 과정의 기록에 접근하는 인터페이스 |
| 뷰 구성 | **ICON 뷰**(기본, 폴더 그리드) / **LIST 뷰**(파일 목록) |
| 폴더 목록 | 문법(grammar) 폴더·지시문(DIR) 폴더·SVA 분석 폴더·작업 가이드 폴더 등 SET 작업 전 문서 구조 반영 |
| 검색 | 상단 검색창으로 폴더/파일 이름 검색 |
| 클릭 | page-log.html로 이동 |

**LOG 패널 내 LOG 엔트리 (LIST 뷰)**

| 항목 | 내용 |
|------|------|
| 역할 | RULE 카드 형식의 작업 원칙 목록 |
| 구성 | RULE_1~5: 작업 근본 원칙. 카테고리(SVA/GRAMMAR/DIR), STATUS(CORE), 프리뷰 텍스트 |
| 검색 연동 | 상단 검색창으로 엔트리 + 참조문헌 동시 검색 |

---

### 3-4. GBAR — 중간 타이틀바

![GBAR](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-gbar.png)

| 항목 | 내용 |
|------|------|
| 파일 | `gbar.html` (set-os.html에 `<iframe>` 삽입) |
| 역할 | 사이트 전체 주요 섹션 이동 + 세션 정보 + 작업 통계 |
| 탭 목록 | SET(gallery1.html) / SET:Inclusion(COMING SOON) / LOG(page-log.html) / Reference(reference.html) / MEASURE(measure.html) |
| 특이사항 | iframe 내에서 부모창 `mobComingSoon()` 호출로 fullscreen 오버레이 연동 |

---

### 3-5. BOTTOM ROW — 하단 영역

![BOTTOM ROW 전체](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-bot.png)

#### G-1. NOTICE + VIEWER_LOG.EXE

![G-1 NOTICE / G-2 VIEWER_LOG](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-notice.png)

**MEASURE 채팅 입장 버튼 (G-1)**

| 항목 | 내용 |
|------|------|
| 역할 | MEASURE.DB 검색 채팅방 입장 유도 |
| 구현 | CRT 스타일 시스템 공지 카드. "채팅방에 입장했습니다" + [입장 ▶] 버튼 |
| 연동 | 클릭 시 measure.html로 이동 |

**VIEWER_LOG.EXE (G-2)**

| 항목 | 내용 |
|------|------|
| 역할 | MEASURE 검색 기록 피드 표시 |
| 구현 | 스크롤 가능한 터미널 로그 UI. 시스템 메시지·검색어·결과 히트가 시간순 나열 |
| 현재 상태 | 데모 피드(정적 샘플). 실제 사용자 검색 데이터 실시간 연동 미구현 |
| 확장 | 백엔드 연동 시 실시간 검색 기록 수신 가능 |

#### G-2b. VIEWER 3D + ID CARD

![G-2b VIEWER 3D + ID CARD](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-viewer-id.png)

| 항목 | 내용 |
|------|------|
| ID CARD 파일 | `id-card.html` (iframe 삽입) |
| 역할 | 작가/프로젝트 신원 카드 |
| 표시 항목 | 프로젝트명, 작가 정보, 작업 상태, 접속자 수 등 |

#### G-3. WORK STATUS

![G-3 WORK STATUS](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-work.png)

| 항목 | 내용 |
|------|------|
| 파일 | `work-status.html` (iframe 삽입) |
| 역할 | GitHub 커밋 기반 작업 타임라인. 실시간 작업 진행 상황 표시 |
| 데이터 | GitHub API(`note-a/note-a-github`) 커밋 피드 + `published-posts.json` |
| 표시 항목 | POSTS 수 / COMMITS 수 / 이번 주 커밋 / 최근 커밋 날짜 / 스파크라인 / 커밋 피드 |
| 캐시 | localStorage 5분 TTL. 레이트리밋 시 stale 캐시 폴백 |

#### G-4. SET.TEXT — 개념 텍스트 패널

![G-4 SET.TEXT](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-settext.png)

| 항목 | 내용 |
|------|------|
| 역할 | 작업 개념 서술 텍스트 상시 노출 |
| 내용 | SET의 구조, 측정 논리, MEASURE 의미, 유예 선언 등 작업 전체 서술 |
| 클릭 | 전체 보기 모드로 확장 |

---

### 3-6. 오버레이 & 모달

#### COMING SOON 오버레이

![COMING SOON](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-modal-coming.png)

| 항목 | 내용 |
|------|------|
| 역할 | SET:Inclusion 섹션 미공개 안내 fullscreen 오버레이 |
| 트리거 | GBAR의 "SET:Inclusion" 탭 클릭 → `mobComingSoon()` 호출 |
| 구현 | 전체 화면 오버레이. 클릭/ESC로 닫기 |

#### STATEMENT MODAL

![STATEMENT MODAL](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-modal-stmt.png)

| 항목 | 내용 |
|------|------|
| 역할 | 작업 성명서·작가 노트 전체 텍스트 열람 |
| 트리거 | STATUS BAR 우측 "STATEMENT" 버튼 클릭 |
| 구현 | 스크롤 가능한 오버레이 모달 |

#### DOCS MODAL

![DOCS MODAL](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-modal-docs.png)

| 항목 | 내용 |
|------|------|
| 역할 | 기술 문서 / 요구사항 정의서 열람 |
| 트리거 | STATUS BAR 우측 "DOCS" 버튼 클릭 |
| 구현 | 스크롤 가능한 오버레이 모달 |

---

### 3-7. STATUS BAR

![STATUS BAR](https://raw.githubusercontent.com/note-a/note-a-github/main/screenshots/sc-statusbar.png)

| 항목 | 내용 |
|------|------|
| 위치 | 화면 최하단 고정 바 |
| 좌측 | 시스템 상태 메시지 (실시간 텍스트 롤링) |
| 우측 | STATEMENT / DOCS 버튼 |
| 역할 | 전체 인터페이스 상태 표시 + 주요 문서 진입점 |

---

## 4. 데이터 구조

### admin-contents.json

```json
{
  "photos": [
    {
      "dir": "FOGDIS",
      "status": "CORE",
      "date": "2025.03.14",
      "exif": "f/2.8 · 1/250s · ISO 200",
      "src": "이미지 URL",
      "sva": {
        "ds": 0.72,
        "sd": 0.58,
        "sa": 0.41,
        "kv": 0.88
      }
    }
  ]
}
```

### GL_FRAMES (런타임 사진 배열)

admin-contents.json 로드 후 GL_FRAMES 배열에 정규화. 미로드 시 하드코딩 샘플 폴백.

### localStorage 키

| 키 | 용도 |
|----|------|
| `set_null_log` | NULL.LOG 공집합 검색어 누적 (키워드: 횟수) |
| `work_status_cache` | WORK STATUS GitHub API 응답 캐시 (5분 TTL) |

---

## 5. 모바일 대응 (`@media max-width: 767px`)

| 항목 | 데스크탑 | 모바일 |
|------|----------|--------|
| 레이아웃 | 좌우 레일 + 메인 고정창 | 단일 컬럼 스크롤 |
| 네비게이션 | 중간 GBAR (iframe) | 상단 햄버거 버튼 → 슬라이드 드로어 |
| 좌우 레일 | 표시 | 미표시 (1025px 이하 숨김) |
| 태블릿(터치) | — | 768~1560px 터치 기기: 레일 표시 |
| COMING SOON | fullscreen 오버레이 | fullscreen 오버레이 동일 |

---

## 6. 제약사항

| 구분 | 내용 |
|------|------|
| **정적 호스팅** | GitHub Pages 기반. 서버 사이드 로직 없음. 모든 기능은 클라이언트 JS로 구현 |
| **GitHub API 레이트리밋** | 비인증 요청 60회/시간. localStorage 5분 TTL 캐시로 완화. 초과 시 stale 캐시 표시 |
| **VIEWER_LOG 실시간성** | 현재 데모 피드. 실제 실시간 채팅 구현 시 별도 백엔드(Firebase, Supabase 등) 필요 |
| **이미지 용량** | GitHub Pages 리포 용량 제한 1GB. 대용량 이미지 다수 시 외부 CDN 고려 필요 |
| **SVA 데이터** | 현재 하드코딩 샘플. AI 분석 파이프라인 없으면 수동 입력 필요 |
| **iframe 통신** | gbar.html ↔ set-os.html 간 `window.parent` 호출. 동일 오리진 한정 |
| **NULL.LOG 지속성** | localStorage 기반으로 브라우저/기기별 데이터 분리. 서버 공유 불가 |
| **고정 해상도** | 메인 영역 1140×757px 고정. 브라우저 zoom 대응 고려 필요 |
| **IE/구형 브라우저** | CSS Grid, requestAnimationFrame, localStorage 사용. IE 미지원 |

---

## 7. 확장성

### 단기 — 현재 구조 내 즉시 가능

- `admin-contents.json`에 사진 데이터 추가 → GALLERY·DIST·SVA RADAR 자동 반영
- `set_null_log` localStorage와 measure.html 공집합 로직 연동 → NULL.LOG 자동 누적 활성화
- PILOT CHANNEL 패널 → pilot.html 실제 ATTR.TEST 결과 데이터 연동

### 중기 — 구조 개선 필요

- **VIEWER_LOG 실시간화**: Firebase Realtime DB 또는 Supabase 연동으로 관객 검색 기록 실제 수집·공유
- **NULL.LOG 서버 공유**: 현재 localStorage 개인 저장 → 서버 DB 공유로 전체 방문자 공집합 집계
- **SVA 자동화**: AI 분석 API 연동. 신규 사진 업로드 시 DS/SD/SA/KV 값 자동 생성
- **MEASURE 검색 고도화**: 단순 문자열 매칭 → 형태학 코드 기반 의미론적 검색

### 장기 — 아키텍처 전환

- **Exclusion 작업 섹션 개설**: NULL.LOG 누적 데이터를 기반으로 다음 작업 Exclusion 아카이브 구성
- **SET:Inclusion 공개**: COMING SOON 해제 후 SET:Inclusion 아카이브 별도 구성
- **교집합 동적 시각화**: SET × SET:Inclusion 실제 채택 데이터로 F.01 Intersection 다이어그램 동적 렌더링
- **CMS 연동**: JSON 직접 편집 → Headless CMS 연동

---

## 8. 파일 구성

| 파일 | 역할 |
|------|------|
| `set-os.html` | 메인 OS 인터페이스 |
| `gbar.html` | 중간 타이틀바·네비게이션 (iframe) |
| `work-status.html` | 작업 현황 패널 (iframe) |
| `id-card.html` | 작가·프로젝트 신원 카드 (iframe) |
| `gallery1.html` | SET 갤러리 전체 페이지 |
| `page-log.html` | LOG 전체 페이지 |
| `reference.html` | 참조 문헌 전체 페이지 |
| `measure.html` | MEASURE 검색 전체 페이지 |
| `pilot.html` | PILOT CHANNEL 전체 페이지 (NULL.LOG + ATTR.TEST) |
| `admin-contents.json` | 사진 메타데이터 데이터 소스 |
| `published-posts.json` | 발행 포스트 목록 |

---

*SET.OS — 측정이 지속된다.*
