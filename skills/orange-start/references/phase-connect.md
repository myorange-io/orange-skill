# 연결 단계 — GitHub · Vercel · Supabase

목표: 빈 Next.js 앱을 만들어 **인터넷에 배포**하고 **데이터베이스까지 연결**한다. 이 단계가
끝나면 라이브 URL이 생기고, 이후 코드를 푸시할 때마다 자동 배포된다.

> 왜 지금 배포하나: Supabase를 Vercel을 통해 연결하려면 Vercel 프로젝트가 먼저 있어야 한다.
> 그래서 구현 전에 배포 환경을 모두 잡아둔다. 덤으로, 사용자는 몇 분 만에 라이브 URL을 갖는다.

## 1. Stitch 디자인 정리

Stitch 산출물을 `design/` 폴더로 정리한다:
- 프로젝트 폴더에 Stitch **zip 파일이 있으면** `unzip`으로 압축을 푼다. 푼 내용물을 `design/`
  폴더 안으로 옮기고 (한 폴더로 감싸여 나오면 그 안의 내용을 `design/`로), 정리가 끝나면 zip은 지운다.
- 이미 `design/`에 파일이 있으면 (사용자가 직접 풀어 넣었거나 스크린샷) 그대로 쓴다.
- `design/`에 정리된 내용을 한 줄로 알린다 (예: "Stitch 화면 5개를 design/에 정리했어요").
- zip도 `design/`도 없으면: "Stitch 프로토타입을 아직 안 만드셨나요?" 물어본다. 만들 거면
  phase-plan의 Stitch 안내로 잠깐 돌아간다. 건너뛸 거면 그대로 진행한다 (구현 단계에서
  화면을 글 설명만으로 만든다).

### 디자인 토큰 다듬기

`PLAN.md`의 `## 디자인`에는 이미 기본 토큰이 채워져 있다 (기획 단계에서). 이게 디자인의
바닥선이다. 여기서 할 일은 그 기본값을 Stitch 디자인으로 **다듬는** 것이다:

- `design/`에 Stitch 산출물이 **있고 기본값보다 나으면**: Stitch의 실제 값으로 `## 디자인`을
  덮어쓴다. 색은 **Stitch의 HTML/CSS 소스에서 실제 값을 딴다** — 스크린샷뿐이면(zip 없음)
  눈대중으로 추출하지 말고 기본값을 유지한다. 강조색은 하나로 추린다.
- `design/`이 **없거나** Stitch 결과가 제네릭·어설프면: `## 디자인` 기본값을 **그대로 둔다.**
  기본값 자체가 깔끔한 디자인 시스템이다 — 억지로 바꾸지 않는다.

최종 `## 디자인`을 사용자에게 한 줄로 요약해 알린다.

## 2. 사전 점검

다음을 실행해 도구가 준비됐는지 확인한다:

| 명령어 | 실패하면 |
|--------|----------|
| `node -v` | Node.js 미설치 → https://nodejs.org 설치 안내 후 중단 |
| `git --version` | Git 미설치 → https://git-scm.com 설치 안내 후 중단 |
| `gh auth status` | GitHub 미인증 → `gh auth login` 실행 안내 후 중단 |
| `vercel whoami` | Vercel 미인증 → `vercel login` 실행 안내 후 중단 |
| `npx supabase projects list` | Supabase 미인증 → `npx supabase login` 실행 안내 후 중단 |

하나라도 실패하면 거기서 멈추고, 사용자가 해결한 뒤 다시 부르도록 안내한다.

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

### 첫 화면 = 브랜드 플레이스홀더

`app/page.tsx`를 기본 Next.js 화면 대신 앱 이름이 보이는 간단한 랜딩으로 바꾼다. 곧 첫
배포에서 이 화면이 라이브 URL로 뜬다 — 입문자가 자기 앱을 처음 만나는 순간이니 제네릭한
Next.js 기본 화면을 보여주지 않는다.

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

## 5. Vercel 배포 (GitHub 자동배포)

목표: `git push` → Vercel 자동 빌드·배포. 동작하려면 **Vercel GitHub App**이 필요하다.

1. Vercel 프로젝트에 연결한다 (프로젝트 이름 = slug): `vercel link --yes --project <slug>`
2. **Vercel GitHub App 확인** — 자동배포의 필수 조건이다.
   `gh api /user/installations --jq '.installations[].app_slug' 2>/dev/null`의 출력에 `vercel`이
   있으면 OK. 없거나 확인이 안 되면 사용자에게 https://github.com/apps/vercel 에서 설치하도록
   안내한다 (브라우저, 1회 — 레포 접근 권한을 허용). 끝나면 이어간다.
3. GitHub 레포를 Vercel에 연결한다: `vercel git connect --yes`
4. 첫 배포로 라이브 URL을 받는다: `vercel --prod`

라이브 URL을 사용자에게 보여주고 **"이제 `git push`하면 자동 배포된다"**고 알린다. 나중에
push해도 배포가 안 되면 같은 폴더 `troubleshooting.md`의 'git push가 자동배포 안 됨'을 본다.

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
키 확보용으로 쓰지 않는다 (사전 준비에서 `npx supabase login`을 해 둔 상태여야 한다):

```bash
npx supabase projects list --output json
```

- 출력에서 **방금 만든 프로젝트**(가장 최근 생성, 이름은 보통 `<slug>-db`)의 `reference_id`를 찾는다.
- 그 ref로 API 키를 가져온다: `npx supabase projects api-keys --project-ref <ref> --output json`
- 프로젝트 **URL**은 `https://<ref>.supabase.co` 다.
- **anon 키**는 출력의 `publishable`(또는 `anon`) 키를 쓴다 — 브라우저에서 안전하게 쓰는 공개
  키다. `secret`·`service_role` 키는 절대 쓰지 않는다.
- 로그인 오류가 나면 사용자에게 `npx supabase login` 실행을 안내한 뒤 다시 시도한다.

이어서 **프로젝트를 link** 한다 (구현 단계의 테이블 생성에 쓴다):

```bash
echo | npx supabase link --project-ref <ref>
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

```bash
git add -A && git commit -m "Supabase 연결"
git push
```

`PLAN.md` 진행 체크리스트에서 `Stitch 프로토타입`과 `연결` 항목을 `- [x]`로 바꾼다.

`✅ 연결 완료 — 앱이 인터넷에 떴습니다: <라이브 URL>. 멈춰도 됩니다.` 를 출력한다.
사용자가 계속하면 구현 단계(같은 폴더의 `phase-build.md`)로 잇는다.
