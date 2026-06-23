# LMS Academy — 인수인계 문서

> 정적(서버 불필요) 멀티페이지 웹사이트. 다크 테마(미드나잇 네이비).
> 다른 컴퓨터로 폴더째 옮겨서 바로 열고 수정·배포할 수 있습니다.

---

## 1. 이게 뭔가요

학습관리시스템(LMS) UI 목업을 **실제로 동작하는 정적 사이트**로 만든 것입니다.

- 순수 HTML + Tailwind CSS(CDN) 만 사용 → **빌드 과정·Node·npm 전부 불필요**
- 백엔드 없음(로그인·DB 없음). 화면과 페이지 이동, 제출 폼 인터랙션까지 동작
- 폰트/아이콘/이미지는 전부 외부 URL(Google) → **인터넷만 되면 어디서든 동일하게 보임**

---

## 2. 파일 구성

```
LMS/
├── index.html        ← 대시보드(홈). 첫 화면.
├── course.html       ← 강의 상세 + 공지 피드 (상단 메뉴 "Notices")
├── assignment.html   ← 과제 상세 (마감·배점·지시사항·루브릭 아코디언, 메뉴 "Tasks")
├── submit.html       ← 과제 제출 (드래그&드롭 업로드·파일목록·서약 체크)
└── HANDOFF.md        ← 이 문서
```

각 HTML은 **자기 완결형**입니다. `<head>` 안에 Tailwind 설정(색상 토큰·폰트)이 통째로 들어있어 파일 하나만 열어도 정상 렌더링됩니다.

---

## 3. 새 컴퓨터에서 여는 법

### 방법 A — 그냥 더블클릭 (가장 간단)
`index.html`을 더블클릭하면 브라우저에서 바로 열립니다.
※ 단, `file://`로 열면 일부 브라우저에서 보안정책상 동작이 제한될 수 있어 **방법 B 권장**.

### 방법 B — 로컬 서버로 띄우기 (권장)
폴더 안에서 터미널/명령창을 열고 아래 중 하나 실행:

```bash
# 파이썬이 있으면 (대부분 설치돼 있음)
python -m http.server 8801
#   → 브라우저에서  http://127.0.0.1:8801  접속

# Node가 있으면
npx serve .
```

> Windows PowerShell도 `python -m http.server 8801` 동일하게 동작합니다.
> 파이썬 명령이 안 먹으면 `py -m http.server 8801` 로 시도.

### (참고) Claude Code 미리보기 설정
이 PC에서는 `.claude/launch.json`에 `lms`(포트 8801)로 등록돼 있었습니다.
새 PC에서 Claude Code로 미리보기하려면 그 PC의 `.claude/launch.json`에 아래를 추가:

```json
{
  "name": "lms",
  "runtimeExecutable": "python",
  "runtimeArgs": ["-m", "http.server", "8801", "--bind", "127.0.0.1", "--directory", "LMS"],
  "port": 8801
}
```
(`--directory` 경로는 LMS 폴더 위치에 맞게 조정)

---

## 4. 디자인 시스템 (색상 토큰)

색은 **이름(토큰)으로** 쓰입니다. 색을 바꾸려면 각 HTML `<head>`의
`tailwind.config → theme.extend.colors` 객체에서 값만 바꾸면 그 페이지 전체에 반영됩니다.
**4개 파일의 colors 객체는 완전히 동일**하니, 톤을 바꾸려면 4개 파일을 같은 값으로 맞추세요.

| 토큰 | 값 | 용도 |
|---|---|---|
| `background` / `surface` | `#0a0f1a` | 페이지 배경(딥 네이비) |
| `surface-container-lowest` | `#0c121e` | 카드·사이드바 |
| `surface-container-low` | `#111827` | 한 단계 밝은 면 |
| `surface-container` | `#161f30` | 칩·작은 면 |
| `surface-container-high` | `#1d2840` | 강조 카드·아이콘 타일 |
| `surface-container-highest` | `#25324d` | 가장 밝은 면·토스트 |
| `on-surface` | `#e6ebf4` | 본문 텍스트(밝음) |
| `on-surface-variant` | `#aab4c6` | 보조 텍스트(회색) |
| `outline-variant` | `#2b3852` | 테두리 |
| `primary` | `#4f82f0` | 강조·버튼(애저 블루) + 흰 글자 |
| `primary-container` | `#1e40af` | 히어로·채움 영역(딥 블루) |
| `secondary` | `#5fc8ff` | 보조 포인트(시안) |
| `error` | `#ff9d94` | 경고·마감 임박 |
| `tertiary` | `#ffb86b` | 공지(앰버) |

- 폰트: 제목 **Plus Jakarta Sans**, 본문 **Inter** (Google Fonts CDN)
- 아이콘: **Material Symbols Outlined** (`<span class="material-symbols-outlined">아이콘이름</span>`)
- `<html class="dark">` 로 다크모드 고정. (다시 라이트로 가려면 색 토큰 자체를 라이트값으로 바꿔야 함)

---

## 5. 페이지 이동(내비게이션) 구조

```
index.html (대시보드)
 ├─ 사이드/하단 메뉴 Courses → index.html
 ├─                 Notices  → course.html
 ├─                 Tasks    → assignment.html
 ├─                 Profile  → index.html   ← (전용 프로필 페이지 아직 없음. 임시로 홈 연결)
 ├─ "Continue Study" / 강의카드 → course.html
 ├─ "Upload Now" / 우하단 + 버튼 → submit.html
 └─ 알림 종 아이콘 → course.html

course.html (강의/공지)
 └─ 공지의 "Go to Submission" → assignment.html

assignment.html (과제 상세)
 └─ 하단 "Start Submission" → submit.html

submit.html (제출)
 ├─ 파일 드롭/선택 → 목록에 추가
 ├─ 서약 체크박스 ON + 파일 1개 이상 → "Submit Assignment" 버튼 활성화
 └─ 제출 → 로딩 → 성공 토스트 → assignment.html 로 복귀
```

---

## 6. 알아둘 점 / TODO

- **Tailwind는 CDN 방식**입니다. 콘솔에 "cdn.tailwindcss.com should not be used in production" 경고가 뜨는데, 목업/내부용으론 무해합니다.
  실서비스로 배포하려면 Tailwind를 빌드해서 CSS 파일로 떨어뜨리는 게 좋습니다(선택).
- **이미지는 전부 Google `lh3.googleusercontent.com` URL**입니다. 인터넷이 없으면 이미지가 안 뜹니다.
  오프라인/영구보관이 필요하면 이미지를 내려받아 `images/` 폴더에 넣고 `src`를 로컬 경로로 바꾸세요.
- **Profile 전용 페이지 없음** — 현재 홈으로 연결. 필요하면 페이지 추가.
- 데이터는 전부 하드코딩(더미)입니다. 실제 강의/과제/사용자 연동은 백엔드 작업이 필요합니다.

---

## 7. 배포(원하면)

정적 사이트라 어디든 폴더만 올리면 됩니다.

- **GitHub Pages**: 새 저장소에 4개 HTML 올리고 Settings → Pages에서 브랜치 지정. `index.html`이 자동 첫 화면.
- **Netlify / Vercel**: 폴더 드래그&드롭만으로 즉시 배포.
- 정적 호스팅이면 빌드 설정 불필요(Build command 비움, Publish directory = LMS 폴더).

---

*작성: Claude Code · 다크 테마(미드나잇 네이비) 버전 기준*
