# 자주 막히는 곳 — 해결 모음

오류가 났을 때만 읽는다. 증상을 찾아 해결책을 따른다.

## env 변수가 안 읽힘 (`undefined`)

증상: `process.env.NEXT_PUBLIC_SUPABASE_URL`이 `undefined`. Supabase 호출이 실패.
- `.env.local` 파일이 프로젝트 루트에 있는지 확인한다 (이름이 정확히 `.env.local`).
- 클라이언트(브라우저)에서 쓰는 변수는 이름이 `NEXT_PUBLIC_`으로 시작해야 한다.
- env 파일을 바꿨으면 개발 서버를 껐다 켠다 (`npm run dev` 재시작). Next.js는 env를 시작할 때만 읽는다.

## Supabase 쿼리가 빈 결과 / 막힘 (RLS)

증상: 테이블에 데이터가 있는데 앱에서 안 보이거나, insert가 조용히 실패.
- 원인: 테이블에 RLS(Row Level Security)가 켜져 있는데 정책이 없으면 모두 차단된다.
- **해결은 항상 "정책 추가"다 — RLS를 끄지 않는다.** RLS를 끄면 프로젝트 URL만 알면 누구나
  read/edit/delete 할 수 있어 Supabase가 보안 경고 메일을 보낸다.
- 로그인 없는 MVP(공개 데이터): `create policy "anyone can read" on public.<테이블> for select using (true);`
  쓰기도 공개면 `for insert with check (true)`를 추가한다.
- 로그인 없는 MVP(비공개 데이터, 폼 제출·관리자 뷰 등): 정책을 두지 말고 쓰기·읽기는 서버 라우트에서
  `SUPABASE_SERVICE_ROLE_KEY`로 처리한다 (`phase-build.md` 데이터 준비 절 참고).
- 로그인 있는 앱: `auth.uid() = user_id` 정책을 추가한다.

## Supabase 406 오류

증상: Supabase 조회가 406 (Not Acceptable).
- 보통 `.single()` 쿼리가 **0행 또는 여러 행**을 만났을 때 난다 — 조건에 맞는 행이 정확히
  하나인지 확인한다.
- 흔한 원인: **slug·조회 키에 한글이 들어가 URL 인코딩이 꼬임.** slug·식별자는 ASCII만 쓴다
  (한글 이름은 화면 표시용으로만, 식별자는 랜덤 ASCII id).

## Vercel 빌드 실패

증상: `git push` 후 Vercel 배포가 실패.
- Vercel 대시보드의 빌드 로그를 본다. 보통 둘 중 하나다:
  - **env 변수 누락**: 로컬엔 있는데 Vercel 프로젝트에 없는 변수 → `vercel env add <이름>`로 추가.
  - **타입/문법 오류**: 로컬에서 `npm run build`로 재현해 고친다.
- 고친 뒤 다시 `git push`.

## Parsing CSS source code failed

증상: localhost 화면에 'Build Error — Parsing CSS source code failed'.
- 원인: `app/globals.css`가 Tailwind v4 CSS 파서에 안 맞음. 가장 흔한 건 `@import "tailwindcss";`
  **뒤에** 다른 `@import url(...)`(웹폰트 등)를 둔 것 — CSS에서 `@import`는 다른 모든 규칙보다 앞서야 한다.
- 해결: 그 `@import`를 `@import "tailwindcss";` **위로** 옮긴다. 또는 웹폰트는 `app/layout.tsx`에서
  `next/font/google`로 불러온다.
- globals.css에 직접 쓴 커스텀 CSS에 문법 오류가 없는지도 확인한다.
- shadcn가 설정한 `globals.css`에서는 CSS 변수 *값*만 바꾼다 — `@import`·`@theme` 줄의 순서나
  구조는 그대로 둔다.

## gh / vercel 인증 만료

증상: `gh` 또는 `vercel` 명령이 인증 오류.
- GitHub: `gh auth login`을 다시 실행한다.
- Vercel: `vercel login`을 다시 실행한다.

## git push가 자동배포 안 됨

증상: `git push`해도 Vercel에 새 배포가 안 생긴다.
- 원인 대부분: **Vercel GitHub App**이 GitHub 계정에 없거나 이 레포 접근 권한이 없다.
- 해결: https://github.com/apps/vercel 에서 App을 설치하고 해당 레포 접근을 허용한다 (이미
  있으면 Configure에서 레포 추가). 그다음 `vercel git connect --yes`를 다시 실행한다.
- 급하면 `vercel --prod`로 수동 배포할 수 있지만, 근본 해결은 App 설치다.

## "테이블이 없습니다" / relation does not exist

증상: Supabase 쿼리가 테이블을 못 찾는다.
- 테이블 SQL을 Supabase SQL Editor에서 실제로 실행했는지 확인한다.
- 테이블 이름의 철자가 코드와 정확히 같은지 확인한다 (대소문자 포함).

## Supabase 연동 실패 (`vercel integration add supabase`)

증상: `vercel integration add supabase` 명령이 오류로 끝남.
- 대시보드로 프로젝트를 만든다: Vercel 대시보드 → 프로젝트 → **Integrations**(또는 Storage) →
  Marketplace에서 **Supabase** 추가 → 무료 플랜으로 생성.
- 프로젝트가 생겼으면 키는 평소대로 Supabase CLI로 가져온다 (아래 항목 참고).

## Supabase 무료 프로젝트 한도 초과 (새 프로젝트가 안 만들어짐)

증상: `vercel integration add supabase` 또는 대시보드에서 새 프로젝트 생성이 막힌다 —
"free plan project limit" / "organization has reached its project limit" 류 메시지.
- 원인: Supabase 무료 플랜은 **조직(계정)당 활성 프로젝트 2개**까지다. 이전 실습·다른 앱으로
  이미 2개를 만들었으면 더 못 만든다.
- 해결(권장): https://supabase.com/dashboard 에서 **안 쓰는 기존 프로젝트를 일시정지(pause)
  하거나 삭제**한 뒤 다시 만든다. (삭제 전 데이터가 필요 없는지 확인.)
- 해결(대안): 다른 Supabase 계정으로 로그인해 거기서 프로젝트를 만든다 — 단, 그 경우
  `supabase login`도 그 계정으로 다시 해야 하고 키도 그 계정 프로젝트에서 가져온다.
- **예방**: 수업·실습 전에 안 쓰는 옛 프로젝트를 미리 정리해 한도를 비워 둔다.

## Supabase login이 토큰을 저장 못 함 (계속 미인증)

증상: `supabase login`을 분명히 했는데 `supabase projects list` 등이 계속 "not logged in" /
미인증으로 떨어진다. 로그인 → 명령 실패 → 다시 로그인이 반복된다.
- 원인: CLI가 OS 키체인/설정 경로에 액세스 토큰을 저장하지 못하는 환경 버그가 있다 (권한·
  키체인 접근 거부 등).
- 해결: 브라우저에서 토큰을 직접 발급해 **환경변수로 넘긴다** — 토큰 저장을 우회한다.
  1. https://supabase.com/dashboard/account/tokens 에서 **Generate new token**으로 액세스
     토큰을 만든다 (한 번 생성하면 다시 안 보이니 복사해 둔다).
  2. 그 값을 환경변수로 내보낸 뒤 같은 셸에서 `supabase` 명령을 쓴다:

     ```bash
     export SUPABASE_ACCESS_TOKEN=<발급받은 토큰>
     supabase projects list   # 이제 인증됨
     ```

  - CLI는 `SUPABASE_ACCESS_TOKEN`이 있으면 그 값을 우선 쓰므로 `supabase login` 없이 동작한다.
  - 이 변수는 토큰(비밀)이다 — `.env.local`이나 코드에 넣지 말고 셸 세션에서만 export 한다.

## Supabase 키가 안 가져와짐

증상: `supabase projects list` 또는 `... api-keys`가 실패하거나 값이 빈다.
- **로그인 안 됨** → `supabase login` 실행 후 다시 시도한다.
- **프로젝트가 목록에 없음** → `vercel integration add supabase`가 끝까지 됐는지, Supabase
  로그인 계정이 Vercel에 연결된 계정과 같은지 확인한다.
- **최후 수단(대시보드)** → https://supabase.com/dashboard → 프로젝트 → Settings → API에서
  Project URL과 `anon`(또는 publishable) 키를 복사해 `.env.local`에 넣는다.

## Supabase 테이블 생성 실패 (`supabase db query`)

증상: `supabase db query --linked`가 오류로 끝남.
- "project not linked" / ref를 못 찾음 → 프로젝트가 link 안 됨. `echo | supabase link --project-ref <ref>`
  실행 후 다시 시도한다.
- SQL 문법 오류 → `.sql` 파일 내용을 다시 확인한다.
- 최후 수단: SQL을 Supabase 대시보드 → SQL Editor에 붙여넣어 직접 실행한다.

## npm install이 깨짐 / 빌드가 "module not found" (optional deps 버그)

증상: `npm install`은 끝났는데 `npm run dev`/`build`가 네이티브 모듈을 못 찾는다. 메시지에
`@next/swc-darwin-arm64`, `lightningcss.darwin-arm64.node`, `@rollup/rollup-*` 같은
**플랫폼 전용 바이너리**가 없다고 나온다.
- 원인: npm의 알려진 optional dependencies 버그다 — `package-lock.json`이 있거나 캐시가
  꼬이면 현재 CPU(Apple Silicon = arm64)에 맞는 바이너리를 건너뛰고 설치한다.
- 해결: `node_modules`와 lock 파일을 지우고 **완전히 다시 설치**한다.

  ```bash
  rm -rf node_modules package-lock.json
  npm install
  ```

- 그래도 같으면 npm 캐시를 비우고 다시: `npm cache clean --force && npm install`.

## create-next-app이 멈춤 ("directory contains files")

증상: scaffold가 "기존 파일과 충돌" 오류.
- `phase-connect.md`의 방법대로 **임시 폴더에서 만들어 복사**한다 (`mktemp -d` 사용).
  현재 폴더에 바로 `create-next-app .`을 하지 않는다.

## shadcn/ui 설치 실패

증상: `npx shadcn@latest init` 또는 `add`가 오류로 끝남.
- `create-next-app`이 끝나고 `npm install`까지 된 폴더에서 실행해야 한다.
- 프롬프트에서 막히면 기본값(New York 스타일, neutral 베이스)을 고른다. 비대화형 플래그는
  `npx shadcn@latest init --help`로 확인한다.
- Tailwind/React 버전 경고가 떠도 대체로 끝까지 진행된다 — `components.json`이 생겼는지로
  성공을 확인한다.
- 정 안 되면 shadcn 없이 진행한다 — 화면을 Tailwind 유틸리티 클래스로 직접 만들되,
  `PLAN.md` `## 디자인` 토큰을 더 엄격히 따라 일관성을 지킨다.

## shadcn 컴포넌트가 빌드 깨짐 / `asChild`가 안 먹음 (비기본 스타일)

증상: shadcn 설치는 됐는데 ① `Cannot find module '@base-ui/react'`(또는 `@radix-ui/*`) 같은
모듈 없음 오류가 나거나, ② `<Button asChild>`로 `Link`를 감쌌는데 스타일이 안 먹거나 깨진다.
- 원인: **기본(New York / neutral) 대신 다른 스타일**(예: base-nova)을 골랐을 때다. 그런
  스타일은 `@base-ui/react` 등 **추가 패키지**를 따로 깔아야 하고, 일부는 `asChild` prop을
  지원하지 않는다.
- **권장 해결**: 이 워크플로는 **기본 스타일(New York, neutral 베이스)**을 쓰도록 돼 있다.
  비기본 스타일에서 막혔으면 기본으로 다시 init 하는 게 가장 깔끔하다.
- 그 스타일을 유지해야 한다면:
  - **누락 패키지**: 오류에 나온 패키지를 그대로 설치한다 — 예: `npm install @base-ui/react`.
  - **`asChild` 미지원**: `asChild`로 감싸지 말고, **링크에 버튼 클래스를 직접 준다.**
    예: `<Link href="..." className="inline-flex items-center justify-center rounded-md bg-primary px-4 py-2 text-primary-foreground">…</Link>`.
    (Stitch 디자인의 버튼 스타일을 토큰 클래스로 옮기면 일관성이 유지된다.)

## 포트 3000이 이미 사용 중

증상: `npm run dev`가 "Port 3000 is in use".
- 이전 개발 서버가 떠 있다. 그걸 쓰거나, Next.js가 제안하는 다른 포트(3001 등)를 쓴다.
- 굳이 3000을 비우려면 그 포트의 프로세스를 종료한다: `lsof -ti:3000 | xargs kill`.
