# 과제 ② — 시스템 설계 & 셋업

**마감** 10/30(금) 23:59 · 같은 저장소, 새 브랜치, PR 하나

중간고사 기간이라 5주가 있다. 한 번에 몰아서 하기보다, 초반에 **가장 불확실한
부분(데이터를 어디서 어떻게 가져올지)** 을 먼저 확인해 두는 걸 추천한다.
거기서 막히면 설계 전체가 바뀌기 때문이다.

---

## 내는 것 세 가지

| | 무엇 | 어디에 |
|---|---|---|
| 1 | 설계 문서 | `docs/02-system-design.md` + `docs/architecture.png` |
| 2 | 셋업된 앱 | 같은 저장소. 내 폰에서 실행되고, CI 통과 |
| 3 | 증명 | `docs/samples/` |

---

# 1. 설계 문서

`docs/02-system-design.md` · **800단어 이하**, 다이어그램 별도.

아래 여섯 섹션을 쓴다. 괄호 안은 IndieGo로 쓴 예시다. 이 정도 길이면 충분하다.

## Overview

내 시스템이 무엇으로 이루어져 있고, 각 부분이 왜 필요한지 몇 줄로.

> (예) 웹사이트 하나, 데이터베이스 하나, 상영시간을 모으는 예약 작업.
> 상영시간은 극장 사이트에 흩어져 있어서 서버가 대신 모아야 한다.

슬라이드의 네 가지 모양(A~D)은 참고용이다. 그중 하나를 고를 필요는 없다.
내 앱에 맞게 설명하면 된다.

## Diagram

쓰는 부분을 박스로 그리고 화살표로 잇는다. **안 쓰는 부분도 그리고 줄을
긋는다** (로그인, 푸시 알림, 파일 저장소 등). 안 쓰기로 한 것도 결정이다.

- `docs/architecture.png`로 저장하고 문서에 `![](architecture.png)`로 넣는다.
- 도구: [Excalidraw](https://excalidraw.com) 또는 종이에 그려서 사진.

## Core flow

핵심 흐름 하나를 단계별로. 각 단계를 **어느 부분이** 처리하는지 적는다.

> (예) 주소를 입력한다 → 앱이 action을 부른다 → action이 카카오 장소 검색을
> 호출한다 → 결과를 화면에 보여준다

## Data

테이블(또는 저장할 데이터)과 각 테이블의 필드. 그리고 **두 행이 같은
것인지 무엇으로 판단하는지** (키).

> (예) `screenings`: cinemaId, movieId, startsAt, screen
> 키: 극장 + 시작 시각 + 상영관. 한 상영관은 한 번에 한 편만 튼다.

데이터가 폰에만 있으면 AsyncStorage / SQLite / SecureStore 중 무엇에 무엇을
저장하는지 적는다.

## Data sources

데이터를 가져오는 곳마다:

- **방법:** 공식 API · 공공데이터 · 크롤링 · 폰 기능(권한)
- **실제로 한 번 불러본 결과:** 무엇이 돌아왔나 (3번 증명과 연결)
- **한계:** 호출 제한, 비용, 로그인 필요, 차단, 권한 심사
- **언제 가져오나:** 한 번 · 정해진 주기로 · 사용자가 요청할 때
- **막히면:** 대안

> (예) CGV: 크롤링. AWS에서 보내면 모든 요청이 403 → 프록시를 거친다.
> 하루 여섯 번 가져온다.

## Decisions

중요한 결정 2~3개. 각각 몇 줄씩:

```
### 이동 시간은 미리 계산해 둔다
선택지   카드마다 경로 API 호출 · 직선거리 · 미리 계산해 두고 찾아보기
고른 것  미리 계산. 영화 카드마다 이동 시간이 있어서 카드마다 호출은 안 맞는다.
대가     만드는 데 오래 걸린다.
```

**안 고른 선택지와 그 이유를 꼭 적는다.** 그게 있어야 왜 이 선택이 내 상황에
맞는지 보인다.

---

# 2. 셋업

## ① Expo 앱 만들기

저장소 안에 `mobile/` 폴더로 만든다. `README.md`와 `docs/`는 그대로 두고,
앱 코드는 전부 `mobile/` 안에 들어간다.

```bash
npx create-expo-app@latest mobile
cd mobile
```

"Skip initializing a new git repository?"라고 물으면 **Enter** (이미 저장소 안이라 새로 만들 필요 없다).

이후 명령은 전부 `mobile/` 안에서 실행한다.

## ② 내 폰에서 실행

```bash
npx expo start
```

폰에 **Expo Go**를 설치하고 QR 코드를 찍는다.

앱이 Expo Go에 없는 네이티브 기능을 쓰면 (다른 앱 위에 그리기, 알림 읽기,
위젯, HealthKit 등) **development build**가 필요하다:
[Expo — Development builds](https://docs.expo.dev/develop/development-builds/introduction/)

## ③ Convex 연결 (백엔드가 필요하면)

```bash
npm install convex
npx convex dev
```

GitHub로 로그인하고 프로젝트를 만들면 `convex/` 폴더가 생긴다.
[Convex — React Native quickstart](https://docs.convex.dev/quickstart/react-native)를 따라
앱에 연결한다.

- `mobile/convex/_generated/`는 **커밋한다.** 이게 없으면 타입 체크가 실패한다.
- `.env`, `.env.local`은 커밋하지 않는다 (템플릿의 `.gitignore`가 이미 막아 준다).
- API 키는 앱에 넣지 않는다. `EXPO_PUBLIC_`으로 시작하는 값은 앱 안에 그대로
  보인다. 대신 Convex에 넣고 action에서 쓴다:

```bash
npx convex env set LLM_KEY 여기에_키
```

```ts
// mobile/convex/ai.ts 안의 action에서
process.env.LLM_KEY
```

## ④ CI 켜기

저장소 루트에 `.github/workflows/ci.yml`:

```yaml
name: CI
on: pull_request
jobs:
  typecheck:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: mobile
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          cache-dependency-path: mobile/package-lock.json
      - run: npm ci
      - run: echo '/// <reference types="expo/types" />' > expo-env.d.ts
      - run: npx tsc --noEmit
```

PR을 열면 타입 체크가 자동으로 돈다. 초록색이 되면 된다.

가운데 `expo-env.d.ts` 줄은 지우지 않는다. 이 파일은 `npx expo start`를 할 때
생기고 `.gitignore`에 들어 있어서, CI에서는 직접 만들어 줘야 타입 체크가 통과한다.

---

# 3. 증명 — 가장 불확실한 부분

내 앱에서 **가장 불확실한 데이터**를 하나 골라서 실제로 불러본다.

1. Convex action이나 작은 스크립트로 **진짜로 호출한다.**
2. 돌아온 응답을 `docs/samples/`에 저장한다. (예: `docs/samples/kakao-transit.json`)
3. 설계 문서의 Data sources에서 링크한다.

폰 권한이 핵심인 앱이라면 (알림 읽기, 다른 앱 위에 그리기 등) development
build에서 **그 권한이 실제로 동작하는 화면 녹화**가 증명이다.

여기서 안 된다는 걸 알게 되면 그것도 좋은 결과다. 무엇이 막혔고 대신 무엇을
할지 Data sources에 적는다.

---

# 제출

1. 새 브랜치: `git checkout -b 02-system-design`
2. 커밋, 푸시
3. 내 저장소의 `main`으로 PR을 연다
4. 리뷰어로 **@parkchaehyun**을 요청한다
5. 피드백은 PR 리뷰 코멘트로 간다. 승인되면 직접 머지한다.

## 내기 전에 확인

- [ ] 설계 문서에 여섯 섹션이 있다 (Overview · Diagram · Core flow · Data · Data sources · Decisions)
- [ ] 다이어그램에 안 쓰는 부분도 그려져 있고 줄이 그어져 있다
- [ ] 데이터마다 **무엇이 두 행을 같게 만드는지** 적혀 있다
- [ ] 데이터 출처마다 **실제로 한 번 불러본 결과**가 있다
- [ ] 결정마다 **안 고른 선택지**가 있다
- [ ] 앱이 내 폰에서 실행된다
- [ ] CI가 초록색이다
- [ ] `docs/samples/`에 증명이 있다
- [ ] API 키가 커밋되지 않았다

---

# AI와 함께 할 때

- **바로 코드부터 시키지 않는다.** 먼저 설계 문서를 같이 쓰고, 그다음에 코드.
- 설계 초안이 나오면 반대를 요청한다:

```
아래는 내 앱의 시스템 설계다. 너는 이걸 통과시키지 않으려는 리뷰어다.
1. 가장 먼저 막힐 데이터 출처는 어디이고, 왜인가
2. 앱 안에 들어가면 안 되는 비밀 값이 있나
3. 더 단순하게 만들 수 있는 부분이 있나 (빼도 되는 부분)
4. 같은 데이터가 두 번 저장될 수 있는 경로가 있나
동의하지 말고 문제만 짚어줘.

[설계 붙여넣기]
```

- 에이전트가 스스로 확인할 수 있게 한다: `npx tsc --noEmit`이 통과하는지
  보라고 시키면 된다.

---

# 막히면

| 막힌 지점 | 읽을 것 |
|---|---|
| 프론트엔드·백엔드가 뭔지 | [Software Engineering for Vibe Coders](https://technically.dev/learning-tracks/software-engineering-for-vibe-coders), technically.dev |
| Convex를 처음 쓴다 | [Convex tutorial](https://docs.convex.dev/tutorial/) — 한 시간이면 된다 |
| Expo + Convex 연결 | [React Native quickstart](https://docs.convex.dev/quickstart/react-native) |
| 폰에 데이터 저장 | [Expo — Store data](https://docs.expo.dev/develop/user-interface/store-data/) |
| 비밀 키를 어디에 | [Expo — Environment variables](https://docs.expo.dev/guides/environment-variables/) · [Convex — Environment variables](https://docs.convex.dev/production/environment-variables) |
| 정해진 주기로 가져오기 | [Convex — Cron jobs](https://docs.convex.dev/scheduling/cron-jobs) |
| 선택지를 어떻게 비교하나 | [Design Docs at Google](https://www.industrialempathy.com/posts/design-docs-at-google/), Malte Ubl |
| 결정을 어떻게 적나 | [MADR](https://adr.github.io/madr/) |
| 새 기술을 써도 되나 | [Choose Boring Technology](https://mcfunley.com/choose-boring-technology), Dan McKinley |
