# Pocket Index — 브랜드 등록 방법

1. 무드 이미지를 `images/` 폴더에 올려요.
   - 비율 16:10, 1600×1000px 권장 (최소 1200×750)
   - JPG 또는 WebP, 300KB 이하
   - 파일명은 영문 소문자로 (예: `jisoolee.jpg`)
2. `brands.json`에 브랜드를 한 줄 추가해요. **적힌 순서가 홈 노출 순서예요.**

```json
{ "name": "JISOO LEE", "url": "https://jiisoolee.com/", "instagram": "jiisoolee", "image": "images/jisoolee.jpg" }
```

| 항목 | 설명 |
|---|---|
| name | 브랜드명 (카드·상세 상단에 표시) |
| url | 사이트 주소 (상세페이지에 그대로 띄움 + 새 창 버튼) |
| instagram | 인스타 아이디 (@ 없이). 비우면 인스타 검색으로 연결 |
| image | 무드 이미지 경로 |
| embed | (선택) 사이트가 상세페이지에 안 뜨면 `false` → 무드 이미지로 대신 보여줌 |

3. `staging` 브랜치에 올려서 staging.pocket-index.com에서 확인 → 괜찮으면 `main`에 합치면 실서비스에 반영돼요.

주의: 브랜드명을 바꾸면 그 브랜드를 하트해 둔 사용자의 저장이 풀려요. (이름이 브랜드의 고유 키예요)

---

# 계정 로그인(카카오·Google) 켜는 방법

`index.html` 안의 `SUPABASE_URL`, `SUPABASE_ANON_KEY` 두 값이 비어 있으면 하트 탭의 로그인 안내가 **자동으로 숨겨져요.** 아래를 끝내고 두 값을 채우면 나타나요.

## 1. Supabase 프로젝트 만들기 (무료)
1. supabase.com 가입 → New project (Region: Northeast Asia (Seoul))
2. Project Settings → API 에서 **Project URL**과 **anon public key** 복사 → `index.html`의 두 값에 붙여넣기
3. SQL Editor에서 아래를 그대로 실행

```sql
create table public.user_state (
  user_id uuid primary key references auth.users on delete cascade,
  state jsonb not null default '{}'::jsonb,
  updated_at timestamptz default now()
);
alter table public.user_state enable row level security;
create policy "본인 것만" on public.user_state
  for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

4. Authentication → URL Configuration
   - Site URL: `https://pocket-index.com`
   - Redirect URLs: `https://pocket-index.com/**`, `https://staging.pocket-index.com/**`

## 2. 카카오 로그인
1. developers.kakao.com → 내 애플리케이션 → 애플리케이션 추가
2. 앱 키의 **REST API 키** = Client ID / 보안 메뉴의 **Client Secret** 생성·활성화
3. 카카오 로그인 활성화 → Redirect URI에 `https://<Supabase 프로젝트>.supabase.co/auth/v1/callback`
4. 동의항목: 닉네임·프로필사진 설정. (이메일은 비즈 앱 전환이 필요할 수 있어요)
5. Supabase → Authentication → Providers → Kakao 켜고 키 2개 입력

## 3. Google 로그인
1. console.cloud.google.com → API 및 서비스 → OAuth 동의 화면 설정
2. 사용자 인증 정보 → OAuth 클라이언트 ID (웹 애플리케이션)
   - 승인된 리디렉션 URI: `https://<Supabase 프로젝트>.supabase.co/auth/v1/callback`
3. Supabase → Providers → Google 켜고 Client ID/Secret 입력

참고: Google은 인스타·카톡 앱 안의 브라우저에서 로그인을 막아요. 그래서 그런 브라우저에서는 Google 버튼이 자동으로 숨겨지고 카카오만 보여요.

## 4. Apple 로그인
- Apple Developer Program 가입 필요 (연 $99)
- Services ID 만들고 Return URL에 Supabase callback 주소 입력 → Supabase → Providers → Apple 에 키 입력

## 5. 네이버 로그인
- Supabase에 기본으로 들어 있지 않아서 별도 연결 작업이 필요해요.
- 연결이 끝나면 `index.html`의 `NAVER_PROVIDER` 에 그 이름을 넣으면 네이버 버튼이 나타나요. 비워 두면 버튼이 숨겨져요.
