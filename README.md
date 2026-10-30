# 유기화학 메커니즘 스튜디오

원자와 결합을 직접 그리고 전자쌍 화살표로 유기화학 반응 메커니즘을 표현한 뒤,
단계별 애니메이션으로 다시 재생하며 공부하는 웹 앱입니다.

**사이트:** https://organic-mechanism-studio.vercel.app

## 주요 기능

- **반응 그리기** — 원소 12개(C H O N S P F Cl Br I B Si)와 직접 입력한 약어(Me, Ph, OTs …)를 놓고, 결합(단일·이중·삼중), 전자쌍 화살표, 형식전하로 메커니즘을 표현합니다.
  - 골격 구조식 규칙: 탄소는 꼭짓점으로만, 이종원자는 `OH`, `NH₂`처럼 수소 개수를 자동으로 붙여 표시합니다.
  - 형식전하에 맞춰 수소 개수가 바뀝니다 (NH₄⁺, BH₄⁻, 카보양이온 등).
  - **고정 길이·고정 각도**: 결합을 45px·30° 단위로 이어 그려서 지그재그 사슬이나 육각 고리를 깔끔하게 그릴 수 있습니다.
- **단계 관리** — 반응을 여러 단계로 나누고, 단계마다 설명을 붙입니다. 새 단계는 이전 단계의 구조를 복제해서 시작합니다.
- **학습 모드** — 반응을 골라 단계별로 넘겨보거나 자동재생합니다. 화살표가 곡선을 따라 그려지는 애니메이션으로 전자의 이동을 보여줍니다. SN2·SN1 예시 반응이 들어 있습니다.
- **로그인과 저장** — 이메일로 가입하면 만든 반응을 계정에 저장하고, 자기 반응만 볼 수 있습니다.

## 사용 방법

1. 홈에서 **반응 만들기**를 누릅니다.
2. 원소 버튼을 클릭하거나 캔버스로 끌어서 원자를 놓습니다. **결합 도구**로 원자를 고른 뒤 빈 곳을 클릭하면 탄소가 이어집니다.
3. **전자쌍 화살표** 도구로 전자가 출발하는 원자·결합과 도착하는 원자·결합을 차례로 클릭합니다.
4. **+ 단계 추가**로 다음 단계를 만들고 구조를 고칩니다.
5. 반응 이름을 쓰고 **저장**합니다(로그인 필요).
6. 홈에서 **학습 모드**를 열어 반응을 고르고 재생합니다.

## 단축키

| 화면 | 키 | 동작 |
|---|---|---|
| 에디터 | `1` ~ `5` | 도구 바꾸기 (선택·이동 / 결합 / 삭제 / 화살표 / 형식전하) |
| 에디터 | `C` `H` `O` `N` `S` `P` `F` `I` `B` | 원소 놓기. 원자 위에 마우스를 올리고 누르면 그 원자를 바꿈 |
| 에디터 | `Shift` + `C` / `B` / `S` | Cl / Br / Si |
| 에디터 | `Delete` | 마우스를 올린(또는 선택한) 원자·결합 지우기 |
| 에디터 | `Esc` | 그리던 결합·화살표 멈추기 |
| 에디터 | `Ctrl` + `S` | 저장 |
| 에디터 | `Alt` + 클릭·드래그 | 길이·각도를 맞추지 않고 자유롭게 |
| 학습 모드 | `←` `→` | 이전·다음 단계 |
| 학습 모드 | `Space` | 자동재생 / 멈춤 |

## 기술 스택

- **프론트엔드**: HTML + CSS + 순수 JavaScript, 그래픽은 SVG. 빌드 도구 없이 `index.html` 파일 하나로 동작합니다.
- **백엔드**: [Supabase](https://supabase.com) — Postgres DB(반응 저장), Auth(이메일 로그인), RLS(계정별 접근 제한). 클라이언트 라이브러리는 CDN에서 불러옵니다.
- **배포**: [Vercel](https://vercel.com) — 이 저장소의 `main` 브랜치에 push하면 자동으로 다시 배포됩니다.

## 파일 구조

| 파일 | 내용 |
|---|---|
| `index.html` | 앱 전체 (HTML·CSS·JavaScript) |
| `PLAN.md` | 기능 계획과 진행 상황, 데이터 모델 |
| `README.md` | 이 문서 |

## 직접 실행하기

`index.html`을 브라우저로 열면 바로 실행됩니다. 에디터와 예시 반응은 인터넷 없이도 동작하고, 로그인·저장에는 인터넷 연결이 필요합니다.

## Supabase를 새로 설정할 때

다른 Supabase 프로젝트에 연결하려면:

1. Supabase에서 프로젝트를 만들고 **SQL Editor**에서 아래를 실행합니다.

   ```sql
   create table public.reactions (
     id uuid primary key default gen_random_uuid(),
     user_id uuid not null default auth.uid() references auth.users (id) on delete cascade,
     name text not null default '',
     steps jsonb not null default '[]'::jsonb,
     created_at timestamptz not null default now(),
     updated_at timestamptz not null default now()
   );
   create index reactions_user_id_idx on public.reactions (user_id);

   alter table public.reactions enable row level security;
   revoke all on public.reactions from anon;
   grant select, insert, update on public.reactions to authenticated;

   create policy "own rows: read"   on public.reactions for select to authenticated
     using ((select auth.uid()) = user_id);
   create policy "own rows: insert" on public.reactions for insert to authenticated
     with check ((select auth.uid()) = user_id);
   create policy "own rows: update" on public.reactions for update to authenticated
     using ((select auth.uid()) = user_id) with check ((select auth.uid()) = user_id);
   ```

2. **Authentication → Sign In / Providers → Email**에서 **Confirm email**을 끕니다. 무료 요금제의 기본 메일은 프로젝트 팀원에게만 발송되기 때문입니다.
3. **Authentication → URL Configuration**의 **Site URL**을 배포 주소로 바꿉니다.
4. `index.html`의 `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`를 새 프로젝트 값(Project Settings → API Keys)으로 바꿉니다. publishable 키는 공개돼도 되는 키이고, 실제 접근 제한은 위의 RLS 규칙이 담당합니다.
