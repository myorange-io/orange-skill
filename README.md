# Orange Skill

> 경량 바이브코딩 스킬 팩 — 아이디어 하나를 한 번의 Claude Pro 세션 안에 기획 → 구현 → 배포까지.

Claude Code용 플러그인입니다. 바이브코딩 입문자가 막힘없이 **아이디어를 진짜 웹앱으로** 만들고
인터넷에 배포하도록 안내합니다.

## 왜 만들었나

gstack 같은 풀 워크플로 도구는 강력하지만, 기획 단계만으로 Claude Pro 사용량을 거의 다 씁니다.
Orange Skill은 같은 흐름을 **약 10배 가볍게** 다시 설계해, 입문자가 한 세션 안에 결과물을
손에 쥐도록 합니다.

- **모델**: Sonnet 4.6 권장
- **스택**: Next.js · Tailwind CSS · Supabase · Vercel (모두 무료 플랜)
- **결과물**: 인터넷에 배포된, 공유 가능한 웹앱 URL

## 전체 흐름

```
기획  →  Stitch 프로토타입  →  연결  →  구현  →  완성
(질문)    (화면 미리보기)      (배포)   (기능)   (URL)
```

- **기획** — 2~3번의 짧은 질문으로 `PLAN.md`(한 페이지 기획서)를 만듭니다.
- **Stitch 프로토타입** — [Google Stitch](https://stitch.withgoogle.com)에서 화면을 먼저
  눈으로 봅니다. Claude 사용량을 쓰지 않으므로 마음껏 다듬으세요.
- **연결** — Next.js 앱을 만들어 GitHub·Vercel·Supabase에 한 번에 연결합니다.
- **구현** — 화면을 하나씩 진짜로 만듭니다. 만들 때마다 자동 저장·자동 배포됩니다.

## 사전 준비 (수업 전 1회)

| 항목 | 설치 / 설정 |
|------|-------------|
| Node.js 18+ | https://nodejs.org |
| Git | https://git-scm.com |
| GitHub 계정 + `gh` CLI | https://cli.github.com → 설치 후 `gh auth login` |
| Vercel 계정 + `vercel` CLI | `npm install -g vercel` → `vercel login` |
| Vercel GitHub App | https://github.com/apps/vercel 설치 — GitHub→Vercel 자동배포용 (레포 접근 허용) |
| Supabase 계정 | https://supabase.com 가입 → 터미널에서 `npx supabase login` |
| Claude Code (Pro 플랜) | https://claude.com/claude-code |

## 설치

Claude Code 안에서:

```
/plugin marketplace add myorange-io/orange-skill
/plugin install orange-skill@orange-skill
```

## 사용법

1. 새 프로젝트 폴더를 만들고 그 안에서 Claude Code를 엽니다.
2. 모델을 Sonnet으로 설정합니다: `/model sonnet`
3. 시작합니다: `/orange-skill:start`
4. 안내를 따라가면 됩니다.

세션이 끊겼거나 나중에 이어서 할 때:

```
/orange-skill:resume
```

`PLAN.md`와 git 기록을 읽어 "어디까지 했고 다음은 무엇"인지 알려줍니다.

## 명령어

| 명령어 | 하는 일 |
|--------|---------|
| `/orange-skill:start` | 기획부터 배포까지 전체 흐름을 단계적으로 안내 |
| `/orange-skill:resume` | 진행 중인 프로젝트의 현황 파악 후 이어가기 |

## 세션 관리 (Pro 사용량 팁)

- 진행 상태는 프로젝트 폴더 안 `PLAN.md`와 git 기록에 저장됩니다. 외부 저장소가 없습니다.
- 단계가 끝날 때마다 자동으로 저장(commit)됩니다. 세션이 끊겨도 잃는 것이 없습니다.
- Stitch 프로토타이핑은 Claude 사용량을 쓰지 않습니다 — 좋은 휴식 지점입니다.
- 사용량이 부족하면 단계 경계에서 멈추고, 다음에 `/orange-skill:resume`로 이어가세요.

## 강사용 노트

- 수강생에게 수업 전 사전 준비를 마치도록 안내하세요.
- `/model sonnet`을 첫 단계로 강조하세요. Opus로 시작하면 사용량이 빨리 소진됩니다.
- 수강생이 만드는 것들(업무 자동화, 내부 도구·대시보드, 신청 페이지, 데이터 정리,
  문서·메일 자동화, 챗봇·FAQ, 운영 개선, 발표용 데모)은 모두 Next.js 웹앱으로 표현됩니다.
- 기획 단계의 질문은 2~3 라운드로 제한됩니다. 깊이를 조정하려면
  `skills/start/references/phase-plan.md`를 수정하세요.

## 라이선스

MIT
