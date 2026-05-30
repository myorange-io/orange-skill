# 연결 단계 — GitHub · Vercel · Supabase

목표: 빈 Next.js 앱을 만들어 **GitHub·Vercel·Supabase에 연결**하고 **자동배포를 준비**한다. 이
단계가 끝나면 `git push`마다 자동 배포되도록 배선돼 있고, **곧바로 구현 단계로 넘어간다.** 빈
화면을 먼저 배포해 확인하는 단계는 없다 — 라이브 URL은 화면을 다 만든 뒤 구현 끝에서 받는다.

> 왜 지금 연결까지 하나: Supabase를 Vercel을 통해 연결하려면 Vercel 프로젝트가 먼저 있어야 한다.
> 그래서 구현 전에 연결·배선만 잡아둔다. 단, **실제 배포(라이브 URL 확인)는 구현이 끝난 뒤**에 한다.

## 1. Stitch 디자인 정리

Stitch 산출물은 **zip(HTML 코드)으로 받는다.** HTML이 있어야 레이아웃·요소·색을 정확히 읽어
디자인대로 구현할 수 있다 — PNG 스크린샷만으로는 추정에 그쳐 "Stitch와 다른 화면"이 나온다.

`design/` 폴더로 정리한다:
- 프로젝트 폴더에 Stitch **zip 파일이 있으면** `unzip`으로 압축을 푼다. 푼 내용물을 `design/`
  폴더 안으로 옮기고 (한 폴더로 감싸여 나오면 그 안의 내용을 `design/`로), 정리가 끝나면 zip은 지운다.
- 이미 `design/`에 **HTML 파일이** 있으면 (사용자가 직접 풀어 넣음) 그대로 쓴다.
- `design/`에 정리된 내용을 한 줄로 알린다 (예: "Stitch 화면 5개를 design/에 정리했어요").
- **PNG·이미지만 있고 HTML(zip)이 없으면**: 그대로 진행하지 말고 **zip을 다시 요청한다.**
  "Stitch는 코드(zip)로 받아야 디자인대로 정확히 만들 수 있어요. PNG만으론 화면이 어긋나기
  쉬워요. Stitch에서 그 화면을 연 상태로 **'프로젝트 다운로드'**를 눌러 zip을 받아서, 이 폴더에
  넣고 알려주세요." 라고 안내하고 **여기서 멈춰 기다린다.** (받은 PNG는 보조 참고로 둬도 된다.)
- zip도 `design/`도 전혀 없으면: "Stitch 프로토타입을 아직 안 만드셨나요?" 물어본다. 만들 거면
  phase-plan의 Stitch 안내로 잠깐 돌아간다. 건너뛸 거면 그대로 진행한다 (구현 단계에서
  화면을 글 설명만으로 만든다).

### 화면 ↔ 디자인 파일 매핑 (구현 정확도의 핵심)

Stitch 결과와 다른 화면이 구현되는 가장 큰 원인은 **빌드할 때 어느 디자인 파일이 그 화면인지
몰라서** PLAN.md 글 설명만 보고 만들기 때문이다. 그러니 지금 **각 화면이 어느 파일인지** 짝지어
`PLAN.md`에 박아둔다 (구현 단계가 이걸 보고 정확한 파일을 연다):

- `design/`의 각 HTML 파일을 연다. 파일명이 모호하면(`code.html` 등) 내용의 제목·헤딩을 보고
  어떤 화면인지 식별한다.
- 식별한 파일을 `PLAN.md` `## 화면`의 각 화면 항목에 `디자인:` 한 줄로 적는다.
  예: `- 디자인: design/screen-2-list.html` (해당 파일이 없으면 `디자인: 없음`).
- 어느 파일이 어느 화면인지 애매하면 사용자에게 한 번 확인한다 ("이 화면이 [파일명] 맞나요?").

### PLAN.md와 맞춰보기 — Stitch에서 늘거나 바뀐 게 없나

사용자는 Stitch에서 화면을 다듬다가 **새 화면·기능을 추가**하거나 **화면 구성을 바꾸는** 경우가
많다 (Stitch가 알아서 넣거나 바꾸기도 한다). 이걸 그냥 두면 `PLAN.md`(빌드의 기준)와 실제
디자인이 어긋난다. 정리한 `design/`을 `PLAN.md` `## 화면`과 **비교**해 차이를 모두 찾는다:

- `design/`의 HTML 파일(제목·헤딩·내용)을 훑어 화면 목록과 각 화면의 주요 요소(입력 칸·버튼·
  표·항목)를 뽑는다.
- `PLAN.md` `## 화면`과 대조해 아래 세 가지를 가려낸다:
  - **추가** — PLAN엔 없고 Stitch에만 있는 화면·기능.
  - **변경** — 같은 화면인데 구성이 다름 (입력 칸·항목·흐름·1순위 요소가 바뀜).
  - **누락** — PLAN엔 있는데 Stitch엔 없음 (사용자가 뺐거나 안 만든 것).
- 차이가 하나도 없으면 "PLAN.md와 Stitch 디자인이 일치해요"라고 한 줄로 알리고 다음(디자인
  토큰 다듬기)으로 넘어간다.

### PLAN.md에 반영하기

차이가 있으면 사용자에게 차이를 한 번에 보여주고, 항목별로 어떻게 할지 묻는다 (한 번의
AskUserQuestion으로 묶는다):

- **추가된 화면·기능**: "Stitch에 PLAN.md엔 없던 **[이름]**이 보여요." →
  **이번 MVP에 포함** / **다음으로 미루기** / **무시(안 만듦)**.
- **변경된 화면**: "**[화면]**이 PLAN.md와 달라요 — [무엇이 바뀌었는지]." →
  **Stitch대로 반영** / **PLAN.md 유지**.
- **누락된 화면**: "PLAN.md의 **[화면]**이 Stitch엔 없어요." →
  **그래도 만들기(PLAN 유지)** / **이 화면 빼기**.

사용자 선택에 따라 **`PLAN.md`를 직접 수정한다**:

- 추가-포함 / 변경-반영 → `## 화면`에 화면을 추가하거나 해당 화면의 `보임`·`동작`·`데이터`·
  `상태`를 Stitch에 맞게 고친다. 새 화면이면 `## 진행 상황`에 `- [ ] 구현: [화면]`을 추가하고,
  새 데이터가 필요하면 `## 데이터`도 보완한다.
- 추가-미루기 / 누락-빼기 → `## MVP 범위`의 '제외 — 다음에'로 옮기고, 빼는 화면은 `## 화면`과
  진행 체크리스트에서도 지운다.
- 무시 / PLAN 유지 / 누락-그래도 만들기 → `PLAN.md`는 그대로 둔다.

수정한 뒤 바뀐 `## 화면`·진행 체크리스트를 사용자에게 한 줄로 요약해 확인받는다. 이렇게
**PLAN.md를 빌드의 단일 기준으로 다시 맞춘다** — 이후 구현 단계는 이 PLAN.md를 따른다.

> **스코프는 사용자가 정한다.** Claude가 임의로 화면을 늘리거나 빼거나 바꾸지 않는다 — 차이를
> 보여주고 사용자가 고른 대로만 PLAN.md에 반영한다.

### 디자인 토큰 다듬기

`PLAN.md`의 `## 디자인`에는 이미 기본 토큰이 채워져 있다 (기획 단계에서). 이게 디자인의
바닥선이다. 여기서 할 일은 그 기본값을 Stitch 디자인으로 **다듬는** 것이다:

- `design/`에 Stitch 산출물이 **있고 기본값보다 나으면**: Stitch의 실제 값으로 `## 디자인`을
  덮어쓴다. 색은 **Stitch의 HTML/CSS 소스에서 실제 값을 딴다.** 강조색은 하나로 추린다.
- `design/`이 **없거나** Stitch 결과가 제네릭·어설프면: `## 디자인` 기본값을 **그대로 둔다.**
  기본값 자체가 깔끔한 디자인 시스템이다 — 억지로 바꾸지 않는다.

최종 `## 디자인`을 사용자에게 한 줄로 요약해 알린다.

## 2. 사전 점검·자동 설치

먼저 도구를 점검한다. **Supabase CLI는 없으면 자동으로 설치한다** (수강생이 가장 자주 막히는
지점이다 — `npx supabase`는 버전·네트워크 문제로 불안정해서, 전역 설치해 쓰는 게 안정적이다).

### 도구 점검 (없으면 안내 후 중단)

| 명령어 | 실패하면 |
|--------|----------|
| `node -v` | Node.js 미설치 → https://nodejs.org 설치 안내 후 중단 |
| `git --version` | Git 미설치 → https://git-scm.com 설치 안내 후 중단 |
| `gh auth status` | GitHub 미인증 → `gh auth login` 실행 안내 후 중단 |
| `vercel whoami` | Vercel 미인증 → `vercel login` 실행 안내 후 중단 |

이 넷 중 하나라도 실패하면 거기서 멈추고, 사용자가 해결한 뒤 다시 부르도록 안내한다.
**`gh auth login` / `vercel login`의 브라우저 승인 화면은 Claude가 크롬으로 대신 진행할 수
있다** — 같은 폴더 `browser-steps.md`를 따른다(로그인 자격증명은 사용자가 직접 입력).

### Supabase CLI 자동 설치 (macOS·Windows)

`supabase --version`으로 확인한다. **이미 있으면 이 절을 건너뛴다.** 없으면 **먼저 OS를
판별**한 뒤 해당 절차로 설치한다:

```bash
uname -s   # Darwin = macOS · MINGW*/MSYS*/CYGWIN* = Windows(Git Bash) · Linux = 리눅스
```

(`uname`이 없으면 Windows다. PowerShell/명령 프롬프트 환경일 수 있다.)

#### macOS — Homebrew

1. **Homebrew 확인.** `brew --version`이 실패하면 Homebrew부터 설치한다. 이 설치는 **사용자
   비밀번호 입력과 Enter 확인을 요구**하니, 시작 전에 "Homebrew를 설치할게요 — 터미널에 암호를
   입력하고 안내에 따라 Enter를 눌러 주세요"라고 알린다:

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

   설치 후 PATH에 반영한다. Apple Silicon(M1~)은 보통 다음이 필요하다 — 안내하고 실행한다:

   ```bash
   eval "$(/opt/homebrew/bin/brew shellenv)"
   ```

2. **Supabase CLI 설치:** `brew install supabase/tap/supabase`
3. `supabase --version`으로 확인한다.

#### Windows — Scoop

Windows는 Supabase가 공식 지원하는 **Scoop**으로 설치한다. Scoop 명령은 **PowerShell**에서
돌아간다 — 사용자에게 "PowerShell 창에서 아래를 실행해 주세요"라고 안내하고, 끝나면 돌아오게 한다.

1. **Scoop 확인.** `scoop --version`이 안 되면 PowerShell에서 Scoop을 먼저 설치하도록 안내한다:

   ```powershell
   Set-ExecutionPolicy -Scope CurrentUser RemoteSigned -Force
   irm get.scoop.sh | iex
   ```

2. **Supabase CLI 설치** (PowerShell):

   ```powershell
   scoop bucket add supabase https://github.com/supabase/scoop-bucket.git
   scoop install supabase
   ```

3. 설치 후 **새 터미널에서** `supabase --version`으로 확인한다 (PATH 반영을 위해 터미널을 새로 연다).
   Scoop이 막히면 https://github.com/supabase/cli/releases 에서 `.exe`를 받아 PATH에 넣어도 된다.

#### 공통 — 로그인 확인

설치가 끝나면 `supabase projects list`가 미인증으로 실패하면 `supabase login`을 실행하도록
안내한다 (브라우저로 토큰 인증, 1회). 이 명령이 성공해야 6단계에서 키를 가져올 수 있다.
**이 브라우저 인증도 Claude가 크롬으로 대신할 수 있다** — 같은 폴더 `browser-steps.md` 참고.
(`supabase login`이 토큰을 저장 못 하는 버그가 있으면 `troubleshooting.md`의 'login이 토큰을
저장 못 함' 항목대로 액세스 토큰 발급으로 우회한다 — 이 발급도 크롬으로 대신할 수 있다.)

> 어느 방법도 안 되면 임시로 `npx supabase`(느리지만 동작)로 진행한다 — 단, 이후 모든
> `supabase ...` 명령을 `npx supabase ...`로 바꿔 실행해야 한다.

## 3. Next.js 앱 만들기 (scaffold)

현재 폴더에 이미 `PLAN.md`·`design/`이 있으므로, 깨끗한 임시 폴더에서 만들어 옮겨온다:

```bash
TMP=$(mktemp -d)
npx create-next-app@latest "$TMP/app" --typescript --tailwind --app --eslint \
  --no-src-dir --import-alias "@/*" --use-npm --skip-install --disable-git --yes
cp -a "$TMP/app/." .
rm -rf "$TMP"
npm install
npm install @supabase/supabase-js
```

`.gitignore`에 `.env*`가 있는지 확인한다 (create-next-app 기본값에 포함됨). 없으면 추가한다 —
DB 키가 절대 커밋되지 않게 한다.

### shadcn/ui 설치

검증된 UI 컴포넌트 위에서 화면을 만들면 손으로 짠 마크업보다 결과가 훨씬 깔끔하다 — 입문자
앱에서 "AI가 대충 만든 티"가 나는 걸 막는 가장 큰 장치다.

```bash
npx shadcn@latest init --yes
npx shadcn@latest add button card input label select textarea table badge --yes
```

- 프롬프트가 뜨면 기본값(New York 스타일, neutral 베이스 색)을 고른다. 비대화형 플래그가
  불확실하면 `npx shadcn@latest init --help`로 확인한다.
- 구현 단계에서 컴포넌트가 더 필요하면 그때 `npx shadcn@latest add <이름>`으로 추가한다.
- 실패하면 같은 폴더 `troubleshooting.md`의 'shadcn/ui 설치 실패'를 본다.

### 디자인 토큰 적용 — shadcn 변수가 유일한 출처

`PLAN.md` `## 디자인`의 색·모서리를 `app/globals.css`의 shadcn CSS 변수에 **전부** 반영한다.
이렇게 해야 토큰의 출처가 하나(shadcn 변수)로 모이고, 화면마다 색이 어긋나지 않는다:

- 강조색 → `--primary` · 배경 → `--background` · 글자 → `--foreground`
- 카드/테두리 → `--card`·`--border` · 오류 상태색 → `--destructive` · radius → `--radius`
- `:root`(라이트 모드)에 반영한다. 색 형식은 globals.css가 쓰는 형식(보통 oklch)에 맞춰
  변환해 넣는다 — 형식만 맞추고 값은 `## 디자인`을 따른다.

**변수 값만** 바꾼다. `@import`·`@theme` 줄은 절대 건드리지 않는다 (Tailwind v4 파서가 깨진다).
나머지 변수(`--muted` 등)는 shadcn 기본값을 그대로 둔다.

### 한국어 폰트

`app/layout.tsx`의 본문 폰트를 **Noto Sans KR**로 바꾼다 — create-next-app 기본 폰트는 라틴
전용이라 한글이 OS 폴백으로 떨어져 의도하지 않은 모양이 된다. `next/font/google`로 불러온다:

```tsx
import { Noto_Sans_KR } from "next/font/google";
const notoSansKr = Noto_Sans_KR({ subsets: ["latin"], weight: ["400", "500", "700"] });
```

`<body>`의 `className`에 `notoSansKr.className`을 넣는다. (Stitch가 특정 폰트를 강하게
요구할 때만 다른 폰트를 쓴다 — 보통은 Noto Sans KR로 충분하다.)

### 임시 홈 화면

`app/page.tsx`를 기본 Next.js 화면 대신 앱 이름이 보이는 간단한 화면으로 바꾼다 —
create-next-app 기본 화면은 제네릭하니 치워둔다. 이건 스캐폴드가 컴파일되게 하는 **임시
홈**일 뿐이다 (구현 단계에서 실제 화면으로 대체된다). **이 화면을 따로 배포해 확인하지 않는다
— 연결이 끝나면 바로 구현으로 넘어간다.**

```tsx
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center gap-3 p-8 text-center">
      <h1 className="text-3xl font-bold">[앱 이름]</h1>
      <p className="text-muted-foreground">[PLAN.md 한 줄 소개]</p>
      <p className="text-sm text-muted-foreground">곧 만나요</p>
    </main>
  )
}
```

### CLAUDE.md 작성

프로젝트 루트에 `CLAUDE.md`를 만든다. 이 파일은 자동으로 읽혀, 나중에 세션이 바뀌거나
사용자가 직접 코드를 고칠 때도 Claude가 같은 규칙을 지키게 한다:

```markdown
# [앱 이름]

[PLAN.md 한 줄 소개]. Orange Skill로 만든 입문자 MVP다.

## 스택
Next.js (App Router, TypeScript) · Tailwind CSS v4 · shadcn/ui · Supabase · Vercel

## 규칙
- 화면·디자인은 `PLAN.md`를 따른다 — 특히 `## 디자인` 토큰과 각 화면의 `상태:` 명세.
- UI는 `components/ui/`의 shadcn/ui 컴포넌트를 우선 쓴다. 색·모서리는 `app/globals.css`의
  CSS 변수를 따른다.
- `app/globals.css`의 `@import` 줄은 건드리지 않는다 (Tailwind v4 파서가 깨진다). 본문 폰트는
  `app/layout.tsx`에 Noto Sans KR로 설정돼 있다 — 그대로 쓴다.
- URL 경로·slug·DB 키는 ASCII만 쓴다. 한글은 화면 표시용으로만.
- Supabase는 `lib/supabase.ts`의 클라이언트로 읽고 쓴다.
- LLM API 키는 서버 라우트(`app/api/...`)에서만 쓴다 — 브라우저에 노출 금지.
- `PLAN.md`에 없는 화면·기능을 임의로 더하지 않는다.
```

## 4. GitHub 레포 만들기

레포 이름은 `PLAN.md` 첫머리의 **기술 이름(slug)**을 쓴다 (`PLAN.md`에 slug가 없으면 폴더
이름을 소문자-하이픈으로 다듬는다).

```bash
git init
git add -A && git commit -m "기획 + Next.js 스캐폴드"
gh repo create <slug> --source=. --public --push
```

(비공개로 하려면 `--public` 대신 `--private`.)

## 5. Vercel 연결 (GitHub 자동배포 준비)

목표: `git push` → Vercel 자동 빌드·배포가 되도록 **연결만** 해둔다. **여기서 빈 화면을 먼저
배포해 확인하지 않는다** — 실제 배포와 라이브 URL 확인은 화면을 다 만든 뒤 구현 단계 마지막에
한다. 자동배포가 동작하려면 **Vercel GitHub App**이 필요하다.

1. Vercel 프로젝트에 연결한다 (프로젝트 이름 = slug): `vercel link --yes --project <slug>`
2. **Vercel GitHub App 확인** — 자동배포의 필수 조건이다.
   `gh api /user/installations --jq '.installations[].app_slug' 2>/dev/null`의 출력에 `vercel`이
   있으면 OK. 없거나 확인이 안 되면 https://github.com/apps/vercel 에서 설치해야 한다 (1회 —
   레포 접근 권한을 허용). **이 설치를 사용자에게 떠넘기지 말고, Claude가 크롬으로 직접 처리한다**
   — 같은 폴더 `browser-steps.md`의 'Vercel GitHub App 설치'를 따른다(연결이 안 되면 그때
   사용자에게 수동 안내). 끝나면 이어간다.
3. GitHub 레포를 Vercel에 연결한다: `vercel git connect --yes`

이걸로 이후 `git push`가 자동 배포된다. **여기서 `vercel --prod`로 먼저 배포하지 않는다** —
라이브 URL은 구현이 끝난 뒤 확인한다. push해도 배포가 안 되면 같은 폴더 `troubleshooting.md`의
'git push가 자동배포 안 됨'을 본다.

## 6. Supabase 연결 + 키 가져오기

Vercel CLI로 Supabase를 프로비저닝한다:

```bash
vercel integration add supabase --name <slug>-db \
  --environment production preview development
```

- 무료(Hobby/Free) 플랜을 고른다. 플랜·리전 선택을 물으면 무료 플랜과 가까운 리전을 고른다.
- 플래그가 불확실하면 `vercel integration add --help`로 확인한다.
- 명령이 실패하면 같은 폴더의 `troubleshooting.md` 'Supabase 연동 실패' 항목을 본다.

**키는 Supabase CLI로 직접 가져온다.** `vercel env pull`은 값이 비어 오는 동기화 버그가 잦아
키 확보용으로 쓰지 않는다 (2단계에서 `supabase login`을 해 둔 상태여야 한다):

```bash
supabase projects list --output json
```

- 출력에서 **방금 만든 프로젝트**(가장 최근 생성, 이름은 보통 `<slug>-db`)의 `reference_id`를 찾는다.
- 그 ref로 API 키를 가져온다: `supabase projects api-keys --project-ref <ref> --output json`
- 프로젝트 **URL**은 `https://<ref>.supabase.co` 다.
- **anon 키**는 출력의 `publishable`(또는 `anon`) 키를 쓴다 — 브라우저에서 안전하게 쓰는 공개
  키다. `secret`·`service_role` 키는 절대 쓰지 않는다.
- 로그인 오류가 나면 사용자에게 `supabase login` 실행을 안내한 뒤 다시 시도한다.

이어서 **프로젝트를 link** 한다 (구현 단계의 테이블 생성에 쓴다):

```bash
echo | supabase link --project-ref <ref>
```

DB 비밀번호를 물으면 빈 값으로 넘어간다 — 테이블 생성은 Management API(`db query --linked`)로
하므로 비밀번호가 필요 없다 (`echo |`가 그 빈 입력을 넣어 준다).

## 7. 환경변수와 클라이언트 코드

`.env.local`에 6단계에서 얻은 값으로 아래 두 줄을 쓴다:

```
NEXT_PUBLIC_SUPABASE_URL=https://<ref>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<anon/publishable 키>
```

배포된 앱도 동작하려면 **Vercel 프로젝트에도 같은 두 변수**가 올바른 값으로 있어야 한다
(`NEXT_PUBLIC_` 변수는 빌드 시점에 코드에 박힌다). `vercel env ls`로 확인해 없거나 비었으면
세 환경(production·preview·development)에 넣는다 — 값은 표준입력으로 넘겨 비대화형으로:
`printf '%s' '<값>' | vercel env add NEXT_PUBLIC_SUPABASE_URL production` (변수·환경마다 한 번씩).
이미 있는데 값이 비었으면 `vercel env rm <이름> <환경>` 후 다시 add.

`PLAN.md`가 **LLM API 필요**라고 하면, 사용자가 발급받은 LLM API 키를 `.env.local`과 Vercel
프로젝트에 넣는다 — `ANTHROPIC_API_KEY`(또는 `OPENAI_API_KEY`). **`NEXT_PUBLIC_`을 붙이지
않는다** — 브라우저에 노출되면 안 되는 서버 전용 키다.

그다음 `lib/supabase.ts`를 만든다:

```ts
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)
```

## 8. 저장하고 마무리

`MEMORY.md`에 '연결' 항목을 한 개 덧붙인다(append, 덮어쓰지 않는다): 스택·배포 연결·DB를
**어떻게** 붙였나(쓴 명령·순서) / 왜 이 순서로 했나 / **막힌 설정과 그 해결**(예: Vercel
GitHub App, 키 동기화 버그, Supabase 무료 프로젝트 한도, CLI 토큰 저장 실패 등 — 증상·원인·
해결책을 구체적으로). 막힌 게 있었다면 거기가 기록의 핵심이니 자세히 적는다. 형식은 같은
폴더 `memory-log.md`를 따른다. (아직 배포 전이니 '한눈에 보기'의 `라이브:`는 "배포 전"으로 둔다.)

```bash
git add -A && git commit -m "Supabase 연결"
git push
```

`PLAN.md` 진행 체크리스트에서 `Stitch 프로토타입`과 `연결` 항목을 `- [x]`로 바꾼다.

`✅ 연결 완료 — 배포 준비됐습니다. 이제 화면을 만듭니다. 멈춰도 됩니다.` 를 출력한다.
사용자가 계속하면 **바로** 구현 단계(같은 폴더의 `phase-build.md`)로 잇는다 — 빈 화면을 먼저
배포해 확인하는 단계는 없다.
