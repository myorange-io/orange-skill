# Orange Skill

> 아이디어 하나를 **한 번의 Claude Pro 세션** 안에 기획 → 구현 → 배포까지. 바이브코딩 입문자를 위한 경량 Claude Code 플러그인.

바이브코딩 입문자는 늘 같은 벽에 부딪힙니다 — `gstack` 같은 풀 워크플로 도구는 강력하지만,
기획 단계만으로 Claude Pro 사용량을 거의 다 써 버립니다. 정작 만들고 배포할 사용량이 안 남습니다.

**Orange Skill은 그 워크플로를 약 10배 가볍게 다시 설계했습니다.** 텔레메트리·서브에이전트·과한
질문을 걷어내고 — 명령어 하나로 아이디어에서 배포된 웹앱까지, 입문자가 한 세션 안에 *진짜
결과물*을 손에 쥐도록.

- **모델** — Sonnet 4.6
- **스택** — Next.js · Tailwind CSS · shadcn/ui · Supabase · Vercel (전부 무료 플랜)
- **결과물** — 인터넷에 배포된, 공유 가능한 웹앱 URL

## 누구를 위한 것

- **바이브코딩 입문자** — 빈 프롬프트 앞에서 막막한 사람. 단계마다 손을 잡아줍니다.
- **강사** — 한 세션 분량으로 끝나는, 예측 가능한 실습 워크플로가 필요한 분.
- 수강생이 만드는 것: 업무 자동화 · 내부 도구·대시보드 · 신청 페이지 · 데이터 정리·분석 ·
  문서·메일 자동화 · 챗봇/FAQ · 운영 개선 · 발표용 데모 — 전부 Next.js 웹앱으로 표현됩니다.

## 전체 흐름

```
기획  →  Stitch 프로토타입  →  연결  →  구현  →  완성
질문      화면 미리보기        배선     기능     배포·URL
```

- **기획** — 적응형 인터뷰(현재 방식·참고 앱 파악 → 2~3번의 짧은 질문 라운드). 화면·데이터
  모델·설정이 담긴 한 페이지 빌드 스펙 `PLAN.md`를 만듭니다.
- **Stitch 프로토타입** — [Google Stitch](https://stitch.withgoogle.com)에서 화면을 먼저 눈으로
  봅니다. Claude 사용량을 쓰지 않으니 마음껏 다듬으세요.
- **연결** — Next.js 앱을 만들어 GitHub·Vercel·Supabase에 CLI로 한 번에 연결하고 자동배포를
  준비합니다. (빈 화면을 먼저 배포하지 않고 곧바로 구현으로 넘어갑니다.)
- **구현** — 화면을 하나씩 진짜로 만듭니다. 푸시할 때마다 자동 배포·자동 저장되고, 다 만들면
  라이브 URL을 받습니다.

## 빠른 시작

1. **터미널에서** Claude Code 실행 — `/plugin`은 채팅용 데스크톱 앱이 아니라 **Claude Code**에서만 됩니다.
2. 플러그인 설치 (아래 [설치](#설치) 참고).
3. 새 프로젝트 폴더에서 `/model sonnet` → `/orange-start`.
4. 안내를 따라가면 됩니다 — 아이디어를 말하고, Stitch에서 화면을 보고, 배포까지.

## 사전 준비 (수업 전 1회)

| 항목 | 설치 / 설정 |
|------|-------------|
| Node.js 18+ | https://nodejs.org |
| Git | https://git-scm.com |
| GitHub 계정 + `gh` CLI | https://cli.github.com → 설치 후 `gh auth login` |
| Vercel 계정 + `vercel` CLI | `npm install -g vercel` → `vercel login` |
| Vercel GitHub App | https://github.com/apps/vercel 설치 — GitHub→Vercel 자동배포용 (레포 접근 허용) |
| Supabase 계정 | https://supabase.com 가입 (CLI는 연결 단계에서 Homebrew로 자동 설치 — 안내에 따라 `supabase login`만 하면 됩니다) |
| Claude Code (Pro 플랜) | https://claude.com/claude-code |

## 설치

터미널 Claude Code 안에서:

```
/plugin marketplace add myorange-io/orange-skill
/plugin install orange-skill@orange-skill
```

## 이렇게 동작합니다

```
나:     동아리 회비 납부 현황을 관리하는 도구를 만들고 싶어요.
나:     /orange-start

Claude: [지금은 어떻게 하는지·참고 앱을 묻고 — 2~3 라운드로 화면·데이터·앱 이름을 정함]
        [PLAN.md 작성 — 화면별 구성·데이터 모델·로그인/LLM 설정까지 한 페이지로]
        [Stitch 프롬프트 생성] → Stitch에서 화면을 직접 보고 다듬으세요.

나:     (Stitch에서 화면 확정 → '프로젝트 다운로드' zip을 폴더에 넣고 복귀)

Claude: [Next.js 스캐폴드 → GitHub·Vercel·Supabase 연결 (자동배포 준비)]
        [곧바로 화면을 하나씩 구현 — 만들 때마다 git push → 자동 배포]
        ✅ 완성 — 공유 가능한 URL을 받으세요: https://....vercel.app
```

명령어 하나로 아이디어에서 배포까지. 막혔거나 사용량이 부족하면 단계 경계에서 멈추고
`/orange-resume`로 이어가면 됩니다.

## 명령어

| 명령어 | 하는 일 |
|--------|---------|
| `/orange-start` | 기획 → Stitch → 연결 → 구현 전 과정을 단계적으로 안내 |
| `/orange-resume` | 진행 중인 프로젝트의 현황 파악 후 이어가기 |
| `/orange-secure` | 공유 전 흔한 보안 구멍 5가지(.env 커밋·키 노출·RLS 등) 점검·수정 |

## 세션 관리 (Pro 사용량 팁)

- 진행 상태는 프로젝트 폴더 안 `PLAN.md`와 git 기록에 저장됩니다 — 외부 저장소가 없습니다.
- 만드는 **과정**은 프로젝트 폴더 안 `MEMORY.md`에 단계마다 짧게 쌓입니다 — "무엇을·왜·뭐가
  막혔나". 나중에 복기하거나, 수업에서 "이렇게 만들어졌다" 사례로 보여줄 때 쓰세요.
- 단계가 끝날 때마다 자동으로 저장(commit)됩니다. 세션이 끊겨도 잃는 것이 없습니다.
- Stitch 프로토타이핑은 Claude 사용량을 쓰지 않습니다 — 좋은 휴식 지점입니다.
- 사용량이 부족하면 단계 경계에서 멈추고, 다음에 `/orange-resume`로 이어가세요.

## 강사용 노트

- 수강생에게 수업 전 사전 준비를 마치도록 안내하세요. 특히 `/plugin`은 **터미널 Claude Code**에서만
  동작합니다 — 채팅용 데스크톱 앱이 아닙니다.
- `/model sonnet`을 첫 단계로 강조하세요. Opus로 시작하면 사용량이 빨리 소진됩니다.
- LLM API를 쓰는 프로젝트(챗봇 등)는 별도 API 키와 호출 비용이 듭니다 — 나머지 스택은 무료입니다.
- **스킬을 수정했다면** `.claude-plugin/plugin.json`의 `version`을 올린 뒤 푸시하세요. 버전을
  올리지 않으면 수강생이 `/plugin` 업데이트를 해도 변경이 반영되지 않습니다.

## 다음 단계 — 졸업하기

오렌지 스킬은 "첫 앱"까지 데려다 주는 도구입니다. 프로젝트가 커지거나 팀 단위로 운영하기
시작하면, 더 무거운 도구로 갈아탈 때입니다:

- [Claude Code in Large Codebases — 베스트 프랙티스](https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start) — CLAUDE.md 계층화, hooks, skills, plugins, LSP, MCP, subagents까지 7가지 구성요소
- [gstack](https://github.com/gstack-dev/gstack) — 풀 워크플로 바이브코딩 도구 (기획·리뷰·QA·배포까지)

## 라이선스

MIT · v1.8.0
