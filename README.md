# 🎮 네온 테트리스 — HTML 웹게임 만들고 GitHub Pages로 배포하기

> **강의노트** · 작성일 2026-10-07
> HTML 파일 하나로 웹게임을 만들고, GitHub에 올려서 누구나 접속할 수 있는 웹사이트로 배포하는 전 과정을 정리합니다.

| 항목 | 링크 |
|---|---|
| ▶ 게임 플레이 | https://higogo1.github.io/neon-tetris/ |
| 📦 저장소 | https://github.com/higogo1/neon-tetris |
| 🧱 함께 만든 벽돌깨기 | https://higogo1.github.io/neon-breakout/ |

---

## 📌 목차

1. [오늘의 학습 목표](#1-오늘의-학습-목표)
2. [전체 흐름 한눈에 보기](#2-전체-흐름-한눈에-보기)
3. [1단계 — 게임 만들기 (HTML · CSS · JavaScript)](#3-1단계--게임-만들기)
4. [2단계 — Git으로 기록하기](#4-2단계--git으로-기록하기)
5. [3단계 — GitHub에 올리기](#5-3단계--github에-올리기)
6. [4단계 — GitHub Pages로 배포하기](#6-4단계--github-pages로-배포하기)
7. [오늘 작업 기록](#7-오늘-작업-기록)
8. [📖 용어 정리](#8--용어-정리)
9. [복습 퀴즈](#9-복습-퀴즈)

---

## 1. 오늘의 학습 목표

- [x] HTML 파일 **하나**로 동작하는 웹게임 구조를 이해한다
- [x] `<canvas>`와 **게임 루프**로 화면을 그리는 원리를 안다
- [x] `git` 으로 작업을 **커밋**하고, `gh` 로 GitHub 저장소를 만든다
- [x] **GitHub Pages**로 무료 웹사이트를 배포한다

---

## 2. 전체 흐름 한눈에 보기

```
 ┌──────────────┐   git commit   ┌──────────────┐   git push   ┌──────────────┐   Pages   ┌──────────────────────┐
 │ index.html   │ ─────────────▶ │ 로컬 저장소   │ ───────────▶ │ GitHub 저장소 │ ────────▶ │ https://...github.io │
 │ (내 컴퓨터)   │                │ (.git 폴더)   │              │ (원격/remote) │           │ (누구나 접속 가능)     │
 └──────────────┘                └──────────────┘              └──────────────┘           └──────────────────────┘
      만들기                          기록하기                        올리기                       배포하기
```

> 💡 **핵심 한 줄** — *만들고(HTML) → 기록하고(commit) → 올리고(push) → 공개한다(Pages).*

---

## 3. 1단계 — 게임 만들기

### 3-1. 파일 구조

빌드 도구나 라이브러리 없이 **`index.html` 한 파일**에 전부 들어 있습니다.

```html
<!doctype html>
<html lang="ko">
<head>
  <style> /* CSS: 생김새 (색, 배치, 네온 효과) */ </style>
</head>
<body>
  <canvas id="board"></canvas>   <!-- 게임이 그려지는 도화지 -->
  <script> /* JavaScript: 게임 규칙과 동작 */ </script>
</body>
</html>
```

| 언어 | 역할 | 비유 |
|---|---|---|
| **HTML** | 화면에 무엇이 있는지 (뼈대) | 건물 골조 |
| **CSS** | 어떻게 보이는지 (디자인) | 인테리어 |
| **JavaScript** | 어떻게 움직이는지 (동작) | 전기·수도 |

> 파일 이름이 `index.html` 인 이유: 웹 서버는 주소 뒤에 파일명이 없으면 자동으로 `index.html` 을 보여줍니다.

### 3-2. 게임 루프 — 모든 게임의 심장

게임은 **1초에 약 60번** "상태 갱신 → 화면 그리기"를 반복합니다.

```js
function loop(t) {
  const dt = t - lastTime; lastTime = t;   // 지난 프레임 이후 흐른 시간
  dropAcc += dt;
  if (dropAcc >= dropInterval()) {         // 일정 시간이 쌓이면
    dropAcc = 0;
    블록을 한 칸 내리거나, 바닥이면 고정();
  }
  draw();                                  // 화면 다시 그리기
  requestAnimationFrame(loop);             // 다음 프레임 예약
}
```

### 3-3. 테트리스 핵심 로직

| 기능 | 원리 |
|---|---|
| **보드** | 10칸 × 20줄 2차원 배열. 빈칸은 `null`, 블록은 종류(`'T'` 등) |
| **충돌 판정** | 블록을 움직이기 *전에* 새 위치가 벽·바닥·다른 블록과 겹치는지 검사 |
| **회전** | 정사각 행렬을 90° 돌림: `out[c][n-1-r] = m[r][c]` |
| **벽 차기 (Wall Kick)** | 회전 후 막히면 좌우/위로 1~2칸 밀어서 재시도 |
| **7-bag** | 7종 블록을 한 묶음으로 섞어 뽑기 → 같은 블록이 몰리지 않음 |
| **고스트 블록** | 바닥까지 떨어뜨렸을 때 위치를 미리 테두리로 표시 |
| **줄 삭제** | 가득 찬 줄을 지우고 위에 빈 줄 추가 |
| **점수** | 1줄 100 · 2줄 300 · 3줄 500 · 4줄 800 (× 레벨) |
| **레벨** | 10줄마다 +1, 낙하 속도 `800ms × 0.85^(레벨-1)` |
| **최고 기록** | `localStorage` 에 저장 → 브라우저를 껐다 켜도 유지 |

### 3-4. 조작법

| 키보드 | 동작 | 모바일 버튼 |
|---|---|---|
| ← → | 좌우 이동 | ◀ ▶ (누르고 있으면 반복) |
| ↓ | 소프트 드롭 (+1점/칸) | ▼ |
| ↑ / X | 시계방향 회전 | ⟳ |
| Z | 반시계 회전 | — |
| Space | 하드 드롭 (+2점/칸) | ⤓ |
| C / Shift | 홀드 | H |
| P | 일시정지 | — |

---

## 4. 2단계 — Git으로 기록하기

```bash
git init -b main          # 이 폴더를 Git 저장소로 만들기 (기본 브랜치: main)
git add index.html        # 커밋할 파일 고르기 (스테이징)
git commit -m "Add neon tetris web game"   # 스냅샷 저장
```

> 💡 **커밋 = 게임의 세이브 포인트.** 언제든 이 시점으로 돌아올 수 있습니다.

---

## 5. 3단계 — GitHub에 올리기

`gh` (GitHub CLI) 한 줄로 **저장소 생성 + 연결 + 업로드**를 동시에 합니다.

```bash
gh repo create neon-tetris --public --source=. --remote=origin --push
```

| 옵션 | 의미 |
|---|---|
| `--public` | 공개 저장소 (무료 계정은 Pages 쓰려면 공개여야 함) |
| `--source=.` | 현재 폴더를 올림 |
| `--remote=origin` | 원격 저장소 별칭을 `origin` 으로 등록 |
| `--push` | 만들자마자 바로 push |

> ⚠️ 사전 준비: `gh auth login` 으로 GitHub 로그인이 되어 있어야 합니다. (`gh auth status` 로 확인)

---

## 6. 4단계 — GitHub Pages로 배포하기

GitHub API로 Pages를 켭니다. (웹에서는 *Settings → Pages → Branch: main / root* 와 같음)

```bash
gh api -X POST repos/higogo1/neon-tetris/pages \
  -f "source[branch]=main" -f "source[path]=/"
```

배포 상태 확인:

```bash
gh api repos/higogo1/neon-tetris/pages --jq .status   # → built 이면 완료
```

- 첫 배포는 보통 **1~2분** 걸립니다.
- 이후에는 `main` 에 **push 할 때마다 자동으로 재배포**됩니다.
- 주소 규칙: `https://<사용자명>.github.io/<저장소명>/`

---

## 7. 오늘 작업 기록

| 순서 | 작업 | 결과 |
|---|---|---|
| 1 | 벽돌깨기(`neon-breakout`) 제작 → GitHub 업로드 → Pages 배포 | ✅ [플레이](https://higogo1.github.io/neon-breakout/) |
| 2 | 진행 중 "취소" 요청 → 이미 배포 완료 상태라 현황만 안내, 변경 없음 | 저장소 유지 |
| 3 | 테트리스 스타일로 재요청 → 새 폴더 `neon-tetris` 에 별도 저장소로 제작 | ✅ |
| 4 | `neon-tetris` GitHub 업로드 → Pages 배포 → 상태 `built`, HTTP 200 확인 | ✅ [플레이](https://higogo1.github.io/neon-tetris/) |
| 5 | 이 README(강의노트) 작성 및 업로드 | ✅ |

> 📝 **배운 점** — `push` 와 Pages 배포는 *외부에 공개되는 작업*이라 되돌리려면 저장소 삭제/비공개 전환이 필요합니다. 공개 전에 한 번 더 확인하는 습관을 들이세요.

---

## 8. 📖 용어 정리

### 웹 기초

| 용어 | 한 줄 설명 |
|---|---|
| **HTML** | 웹페이지의 뼈대(구조)를 만드는 언어 |
| **CSS** | 웹페이지의 디자인(색·배치·효과)을 정하는 언어 |
| **JavaScript (JS)** | 웹페이지를 움직이게 하는 프로그래밍 언어 |
| **`<canvas>`** | JS로 도형·이미지를 직접 그릴 수 있는 HTML 도화지 요소 |
| **`index.html`** | 폴더 주소로 접속하면 자동으로 열리는 기본 페이지 |
| **localStorage** | 브라우저에 작은 데이터를 영구 저장하는 공간 (최고 기록 저장용) |
| **반응형 (Responsive)** | 화면 크기(PC/모바일)에 맞춰 레이아웃이 바뀌는 것 |

### 게임 개발

| 용어 | 한 줄 설명 |
|---|---|
| **게임 루프** | "갱신 → 그리기"를 계속 반복하는 게임의 핵심 구조 |
| **프레임 (Frame)** | 화면 한 장. 보통 1초에 60프레임 |
| **`requestAnimationFrame`** | 브라우저에게 "다음 화면 그릴 때 이 함수 불러줘"라고 예약하는 함수 |
| **충돌 판정** | 두 물체가 겹치는지 검사하는 것 |
| **테트로미노** | 정사각형 4개로 된 테트리스 블록 (I·O·T·S·Z·J·L 7종) |
| **월 킥 (Wall Kick)** | 회전이 막힐 때 블록을 살짝 밀어 회전을 성공시키는 기법 |
| **7-bag** | 7종 블록을 한 세트씩 섞어 공평하게 나눠주는 방식 |
| **고스트 블록** | 블록이 떨어질 위치를 미리 보여주는 반투명 표시 |
| **홀드 (Hold)** | 현재 블록을 보관하고 나중에 꺼내 쓰는 기능 |

### Git · GitHub

| 용어 | 한 줄 설명 |
|---|---|
| **Git** | 파일 변경 이력을 기록·관리하는 버전 관리 도구 |
| **저장소 (Repository, repo)** | Git이 관리하는 프로젝트 폴더 |
| **로컬 / 원격 (Local / Remote)** | 내 컴퓨터의 저장소 / 인터넷(GitHub)에 있는 저장소 |
| **커밋 (Commit)** | 현재 상태를 메시지와 함께 저장하는 스냅샷 |
| **스테이징 (`git add`)** | 다음 커밋에 넣을 파일을 고르는 단계 |
| **브랜치 (Branch)** | 독립적인 작업 줄기. 기본은 `main` |
| **origin** | 원격 저장소에 붙이는 기본 별칭 |
| **푸시 (Push)** | 로컬 커밋을 원격 저장소로 올리는 것 |
| **GitHub** | Git 저장소를 인터넷에 올려 공유하는 서비스 |
| **gh (GitHub CLI)** | 터미널에서 GitHub를 조작하는 공식 명령어 도구 |
| **Public / Private** | 누구나 볼 수 있는 저장소 / 나만 볼 수 있는 저장소 |

### 배포

| 용어 | 한 줄 설명 |
|---|---|
| **배포 (Deploy)** | 만든 결과물을 다른 사람이 쓸 수 있게 공개하는 것 |
| **GitHub Pages** | GitHub 저장소의 HTML을 무료 웹사이트로 띄워주는 기능 |
| **정적 사이트 (Static Site)** | 서버 계산 없이 파일(HTML/CSS/JS)만 보내주는 사이트 |
| **API** | 프로그램끼리 기능을 요청·응답하는 약속된 창구 |
| **HTTP 200** | "요청 성공" 응답 코드 (사이트가 정상 동작 중) |

---

## 9. 복습 퀴즈

<details>
<summary><b>Q1.</b> 게임 루프에서 매 프레임 반복하는 두 가지 일은?</summary>

**상태 갱신(update)** 과 **화면 그리기(draw)**
</details>

<details>
<summary><b>Q2.</b> <code>git commit</code> 과 <code>git push</code> 의 차이는?</summary>

`commit` 은 **내 컴퓨터**에 기록, `push` 는 그 기록을 **GitHub(원격)** 로 올리는 것
</details>

<details>
<summary><b>Q3.</b> 무료 계정에서 GitHub Pages를 쓰려면 저장소가 어떤 상태여야 할까?</summary>

**Public(공개)** 저장소
</details>

<details>
<summary><b>Q4.</b> 7-bag 방식을 쓰는 이유는?</summary>

완전 무작위면 같은 블록이 연속으로 나오거나 오랫동안 안 나올 수 있어서, 7개를 한 묶음으로 섞어 **공평하게** 분배하기 위해
</details>

<details>
<summary><b>Q5.</b> 게임을 수정한 뒤 웹사이트에 반영하려면?</summary>

`git add` → `git commit` → `git push` 만 하면 GitHub Pages가 **자동으로 재배포**
</details>
