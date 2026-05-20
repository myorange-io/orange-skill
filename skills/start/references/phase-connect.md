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

## 2. 사전 점검

다음을 실행해 도구가 준비됐는지 확인한다:

| 명령어 | 실패하면 |
|--------|----------|
| `node -v` | Node.js 미설치 → https://nodejs.org 설치 안내 후 중단 |
| `git --version` | Git 미설치 → https://git-scm.com 설치 안내 후 중단 |
| `gh auth status` | GitHub 미인증 → `gh auth login` 실행 안내 후 중단 |
| `vercel whoami` | Vercel 미인증 → `vercel login` 실행 안내 후 중단 |

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

## 4. GitHub 레포 만들기

현재 폴더 이름을 소문자와 하이픈만 남기고 다듬어 레포 이름으로 쓴다.

```bash
git init
git add -A && git commit -m "기획 + Next.js 스캐폴드"
gh repo create <레포이름> --source=. --public --push
```

(비공개로 하려면 `--public` 대신 `--private`.)

## 5. Vercel 배포

```bash
vercel link --yes          # 폴더를 Vercel 프로젝트에 연결
vercel git connect         # GitHub 레포 연결 → 이후 git push마다 자동 배포
vercel --prod              # 첫 배포 → 라이브 URL 출력
```

출력된 라이브 URL을 사용자에게 보여준다. (`vercel git connect`가 실패해도 치명적이지 않다 —
구현 단계에서 `vercel --prod`로 직접 배포하면 된다.)

## 6. Supabase 연결 (Vercel Marketplace)

Vercel CLI로 Supabase를 프로비저닝한다:

```bash
vercel integration add supabase --name <레포이름>-db \
  --environment production preview development
```

- 무료(Hobby/Free) 플랜을 고른다. 플랜·리전 선택을 물으면 무료 플랜과 가까운 리전을 고른다.
- 플래그가 불확실하면 `vercel integration add --help`로 확인한다.
- **이 명령이 실패하면** 같은 폴더의 `troubleshooting.md`에서 'Supabase 연동 실패' 항목을 따라
  Vercel 대시보드로 연결한다.

연결되면 Supabase 환경변수가 Vercel 프로젝트에 자동 주입된다. 이를 로컬로 가져온다:

```bash
vercel env pull .env.local
```

## 7. Supabase 클라이언트 코드

`.env.local`을 열어 Supabase **URL**과 **anon key** 변수명을 확인한다. Next.js 클라이언트에서
쓰려면 `NEXT_PUBLIC_` 접두사가 필요하다. 해당 접두사 변수가 없으면:

- `.env.local`에 `NEXT_PUBLIC_SUPABASE_URL`·`NEXT_PUBLIC_SUPABASE_ANON_KEY`를 추가한다.
- Vercel 프로젝트에도 같은 변수를 추가한다: `vercel env add NEXT_PUBLIC_SUPABASE_URL` 등.

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

`PLAN.md`의 진행 체크리스트에서 `- [ ] 연결` 항목을 `- [x]`로 바꾼다.

`✅ 연결 완료 — 앱이 인터넷에 떴습니다: <라이브 URL>. 멈춰도 됩니다.` 를 출력한다.
사용자가 계속하면 구현 단계(같은 폴더의 `phase-build.md`)로 잇는다.
