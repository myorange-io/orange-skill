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

## Supabase 키가 안 가져와짐

증상: `npx supabase projects list` 또는 `... api-keys`가 실패하거나 값이 빈다.
- **로그인 안 됨** → `npx supabase login` 실행 후 다시 시도한다.
- **프로젝트가 목록에 없음** → `vercel integration add supabase`가 끝까지 됐는지, Supabase
  로그인 계정이 Vercel에 연결된 계정과 같은지 확인한다.
- **최후 수단(대시보드)** → https://supabase.com/dashboard → 프로젝트 → Settings → API에서
  Project URL과 `anon`(또는 publishable) 키를 복사해 `.env.local`에 넣는다.

## Supabase 테이블 생성 실패 (`supabase db query`)

증상: `npx supabase db query --linked`가 오류로 끝남.
- "project not linked" / ref를 못 찾음 → 프로젝트가 link 안 됨. `echo | npx supabase link --project-ref <ref>`
  실행 후 다시 시도한다.
- SQL 문법 오류 → `.sql` 파일 내용을 다시 확인한다.
- 최후 수단: SQL을 Supabase 대시보드 → SQL Editor에 붙여넣어 직접 실행한다.

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

## 포트 3000이 이미 사용 중

증상: `npm run dev`가 "Port 3000 is in use".
- 이전 개발 서버가 떠 있다. 그걸 쓰거나, Next.js가 제안하는 다른 포트(3001 등)를 쓴다.
- 굳이 3000을 비우려면 그 포트의 프로세스를 종료한다: `lsof -ti:3000 | xargs kill`.
