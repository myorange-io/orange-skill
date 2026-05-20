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
- 로그인 없는 MVP: Supabase 대시보드 → 테이블 → RLS를 끈다 (또는 켜고 anon에 read/write 정책 추가).
- 로그인 있는 앱: `auth.uid() = user_id` 정책을 추가한다.

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

## gh / vercel 인증 만료

증상: `gh` 또는 `vercel` 명령이 인증 오류.
- GitHub: `gh auth login`을 다시 실행한다.
- Vercel: `vercel login`을 다시 실행한다.

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

## 포트 3000이 이미 사용 중

증상: `npm run dev`가 "Port 3000 is in use".
- 이전 개발 서버가 떠 있다. 그걸 쓰거나, Next.js가 제안하는 다른 포트(3001 등)를 쓴다.
- 굳이 3000을 비우려면 그 포트의 프로세스를 종료한다: `lsof -ti:3000 | xargs kill`.
