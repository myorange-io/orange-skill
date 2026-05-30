---
name: orange-secure
description: 오렌지 스킬로 만든 Next.js + Supabase 앱의 흔한 보안 구멍을 점검하고 고친다. 배포·공유 전이나 Supabase 보안 경고 메일을 받았을 때 사용. "보안 점검", "오렌지 보안", "Supabase 경고", "RLS 확인" 같은 요청에 사용.
---

# Orange Skill — Secure

오렌지 스킬로 만든 앱의 **흔한 보안 구멍 5가지**를 빠르게 점검·수정한다. 입문자가 공유 가능한
URL을 받았을 때, **그 URL이 진짜로 공유해도 되는지** 확인하는 단계다.

가벼운 검사 — 토큰을 거의 쓰지 않는다. 매 턴 자동 리뷰가 아니라, 사용자가 부를 때만 1회 돈다.

## 1. 사전 점검

현재 폴더가 오렌지 스킬 프로젝트가 맞는지 본다:

- `PLAN.md`·`package.json` 없음 → "오렌지 스킬 프로젝트가 아닌 것 같아요. 프로젝트 폴더에서
  실행하세요." 라고 안내하고 끝낸다.
- `lib/supabase.ts`가 없으면 Supabase 미연결이다. 4번(서버 키)·5번(RLS)은 건너뛴다.

## 2. 5가지 점검 항목

각 항목을 순서대로 본다. **하나씩 결과를 출력하고**, 위험이 있으면 사용자에게 보여준 뒤 "고칠까요?"
한 번만 묻고 진행한다.

### 검사 1 — `.env.local`이 git에 커밋됐는지

```bash
git ls-files | grep -E '^\.env(\.|$)' || echo "OK"
```

`.env.local`·`.env`가 출력되면 **CRITICAL**. 키가 이미 git 히스토리에 들어갔다.

**고치는 법** (사용자가 OK 하면):
1. `git rm --cached .env.local .env 2>/dev/null` 로 추적에서 제외
2. `.gitignore`에 `.env*.local`·`.env` 추가 (이미 있는지 확인)
3. **사용자에게 알린다**: "이미 push 한 적이 있다면 GitHub 히스토리에 키가 남아 있어요.
   Supabase 대시보드 → Settings → API → 'Reset anon key'와 'Reset service_role key'로 키를
   새로 발급하고 `.env.local`·Vercel env에 새 키를 넣으세요. LLM API 키도 마찬가지로 재발급."

### 검사 2 — `NEXT_PUBLIC_*`에 서버 전용 키가 섞였는지

```bash
grep -E '^NEXT_PUBLIC_.*(SERVICE_ROLE|SECRET|PRIVATE)' .env.local .env 2>/dev/null || echo "OK"
```

`NEXT_PUBLIC_SUPABASE_SERVICE_ROLE_KEY`·`NEXT_PUBLIC_ANTHROPIC_API_KEY` 같은 게 잡히면 **CRITICAL**.
`NEXT_PUBLIC_` 접두사는 브라우저 번들에 들어간다 — service_role 키가 노출되면 RLS가 무력화된다.

**고치는 법**:
1. 해당 변수에서 `NEXT_PUBLIC_` 제거 (예: `SUPABASE_SERVICE_ROLE_KEY=...`)
2. 코드에서 이 변수를 쓰는 곳을 grep으로 찾는다:
   ```bash
   grep -rn "NEXT_PUBLIC_SUPABASE_SERVICE_ROLE\|NEXT_PUBLIC_ANTHROPIC\|NEXT_PUBLIC_OPENAI" app/ components/ lib/ 2>/dev/null
   ```
3. 클라이언트 컴포넌트(`'use client'` 있거나 `components/`)에서 쓰고 있으면 **그 호출을 서버
   라우트(`app/api/<이름>/route.ts`)로 옮긴다**. 자세한 패턴은 `phase-build.md`의 LLM API 절 참고.
4. Vercel env(`vercel env ls`)에도 같은 변수가 있으면 사용자에게 대시보드에서 접두사를 고치도록 안내.
5. 키가 한 번이라도 배포·노출됐다면 **반드시 재발급**.

### 검사 3 — 클라이언트 코드에 비밀 키가 하드코딩됐는지

```bash
grep -rE 'sk_live_|sk-ant-|sk-proj-|service_role|eyJ[A-Za-z0-9_-]{20,}' app/ components/ lib/ 2>/dev/null | grep -v 'process.env' || echo "OK"
```

매치가 나오면 사용자에게 보여준다. JWT(`eyJ...`)나 `sk_live_`·`sk-ant-` 같은 키 패턴이 코드에
직접 박혀 있으면 **CRITICAL**. 빌드 결과물에 그대로 들어간다.

**고치는 법**:
1. 해당 값을 `.env.local`로 옮기고 `process.env.<NAME>`으로 참조
2. 노출된 키는 **재발급** (검사 1과 같은 절차)

### 검사 4 — service_role 키가 클라이언트 컴포넌트에서 쓰이는지

`lib/supabase.ts`가 있으면 그 내용을 본다. `createClient`에 `SERVICE_ROLE`이 들어가 있으면서
파일 상단에 `'use client'`가 없어도, `app/` 하위 클라이언트 컴포넌트(`'use client'` 선언 + 이 모듈
import)에서 쓰이고 있는지 확인:

```bash
grep -rln "from.*lib/supabase" app/ components/ 2>/dev/null | xargs grep -l "'use client'" 2>/dev/null || echo "OK"
```

클라이언트 컴포넌트에서 import 하고 있고 그 모듈이 service_role을 쓴다면 **CRITICAL**. 클라이언트용
Supabase 클라이언트(`lib/supabase.ts` — anon 키)와 서버용(`lib/supabase-admin.ts` — service_role)을
분리한다.

**고치는 법**: `lib/supabase-admin.ts`를 새로 만들고 service_role 클라이언트는 거기에 둔다. 서버
라우트에서만 import 한다. 자세한 분리 패턴은 `phase-build.md` 참고.

### 검사 5 — Supabase 테이블에 RLS가 켜져 있는지

`supabase/` 폴더의 `.sql` 파일들을 본다:

```bash
ls supabase/*.sql 2>/dev/null
```

각 SQL 파일에서 `create table public.<이름>` 다음에 `alter table public.<이름> enable row level security`가
있는지 확인. 빠진 테이블이 있으면 **CRITICAL** (Supabase가 보낸 경고 메일의 원인).

**고치는 법**:
1. 빠진 테이블마다 SQL 파일에 다음을 추가:
   ```sql
   alter table public.<테이블> enable row level security;
   ```
2. `PLAN.md` `## 설정`의 **로그인** 항목에 따라 정책을 추가한다. 정책 SQL 템플릿은
   `phase-build.md` "데이터 준비" 절에 시나리오별로 정리돼 있다 — 거기서 골라 붙인다.
3. 새 SQL을 Supabase에 적용:
   ```bash
   supabase db query --linked --file supabase/<파일>.sql
   ```
4. **이미 RLS 없이 배포된 상태라면 라이브 DB에도 같은 SQL을 적용해야 한다** — Supabase 대시보드
   SQL Editor에 붙여넣고 실행.

## 3. 최종 보고

5가지 검사를 다 끝낸 뒤 한 줄 요약을 출력한다:

- 전부 OK: `✅ 보안 점검 통과 — 공유해도 안전합니다.`
- 수정함: `✅ 보안 문제 N건을 수정했습니다. 라이브 URL을 다시 한 번 확인해 보세요.`
- 사용자가 수정을 거부함: `⚠️  CRITICAL N건이 남아 있습니다. 공유 전에 꼭 고치세요.`

`MEMORY.md`가 있으면 '보안 점검' 항목을 **한 개 덧붙인다**(append, 덮어쓰지 않는다):
점검한 항목 / 결과 / 수정한 것과 **어떻게 고쳤는지** / 남은 위험과 **그렇게 판단한 근거**를
구체적으로 적는다 (나중에 같은 점검을 다시 할 때 기준이 된다). 날짜는 `date +%Y-%m-%d`로 얻는다.
형식은 `../orange-start/references/memory-log.md` 참고. 예:

```markdown
### [YYYY-MM-DD] 보안 점검
- **점검 항목**: .env git 커밋 여부 · NEXT_PUBLIC 키 노출 · 하드코딩 비밀 키 ·
  service_role 클라이언트 사용 · RLS 활성화
- **결과**: CRITICAL 1건 발견·수정. `.env.local`이 git에 커밋돼 있었다 →
  `git rm --cached .env.local` 후 `.gitignore`에 `.env*` 추가, 커밋.
- **남은 위험**: 한 번 커밋돼 노출됐던 anon 키는 git 히스토리에 남아 있다 →
  Supabase 대시보드에서 키 재발급이 안전하다고 사용자에게 안내함.
- **배운 것**: anon 키는 공개돼도 RLS가 막아 주지만, 히스토리에 남는 건 별개 문제다.
```

수정 사항이 있었으면 마지막에 커밋을 권한다 (`MEMORY.md`도 함께 담는다):
```bash
git add -A && git commit -m "보안: 점검 결과 반영" && git push
```

## 원칙

- **한 번에 한 검사**, 결과를 보여주고 사용자가 OK 한 뒤 다음으로. 자동으로 5건을 한꺼번에 고치지
  않는다 — 입문자는 무엇이 바뀌었는지 알아야 한다.
- **재발급이 필요한 상황은 명확히 말한다.** 키가 git이나 브라우저에 한 번이라도 노출됐다면 코드만
  고치는 건 충분하지 않다.
- **공식 보안 도구를 대체하지 않는다.** 더 깊은 검사가 필요하면 README의 "졸업하기" 섹션에서
  Anthropic 공식 `security-guidance` 플러그인이나 CI 스캐너를 안내한다.
