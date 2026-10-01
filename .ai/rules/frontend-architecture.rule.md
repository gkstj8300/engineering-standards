# 프론트엔드 폴더 구조 — Feature-Sliced Design

FSD(Feature-Sliced Design)를 채택한 프론트엔드 프로젝트의 폴더 구조와 의존 규칙이다. 적용 소스 범위와 경로 별칭은 [frontend.rule.md](frontend.rule.md) 프로필 P2·P4를 따른다.

## 1. 적용 범위

- 단일 앱이면 저장소 간 공유 패키지를 두지 않고, 내부 참조는 경로 별칭(P4)만 쓴다.
- 앱이 여러 개(예: 사용자 앱·관리자 앱)면 앱별 차이를 프로필 P2에 먼저 적은 뒤 적용한다. 협의 없이 구조를 임의로 확장하지 않는다.

## 2. 레이어 계층 — 위에서 아래로만 의존

| 레이어 | 역할 | 예 |
|---|---|---|
| `app/` | 앱 초기화·프로바이더·라우팅 | 전역 스타일, 라우터 설정 |
| `pages/` | 라우트 단위 화면 조합 | 상품 목록 페이지 |
| `widgets/` | 여러 기능을 묶은 큰 UI 블록 | 헤더, 주문 요약 패널 |
| `features/` | 사용자 상호작용 단위 | 로그인, 장바구니 담기, 목록 필터 |
| `entities/` | 비즈니스 도메인 모델 | `user`, `product`, `order` |
| `shared/` | 재사용 UI 키트·lib·API client·config | 버튼, 날짜 유틸, 디자인 토큰 |

프레임워크가 `app/`·`pages/`를 라우팅에 쓰면(예: Next.js) FSD 레이어 이름을 `_app/`·`_pages/`로 바꿔 충돌을 피한다.

### 2.1 의존 규칙

- 상위 레이어에서 하위 레이어로만 import한다(단방향).
- 같은 레이어의 slice끼리 import하지 않고, 순환 의존을 만들지 않는다. FSD가 entities에 허용하는 `@x` 교차 import도 쓰지 않는다.
- `app/`과 `shared/`는 slice 없이 segment로만 나눈다. `shared/`는 다른 레이어를 import하지 않는다.

## 3. 도메인 slice (entities)

- slice 하나는 도메인 하나만 담는다. 이름은 도메인을 나타내는 소문자 kebab-case 명사로 쓴다(예: `user`, `order-item`).
- 프로젝트의 slice 목록과 각 slice가 담는 도메인은 프로필 P17에 적는다.
- 새 도메인이 필요하면 기존 slice에 억지로 넣지 말고 slice를 추가한다.

## 4. segment (slice 내부)

```text
entities/order/
├── ui/       OrderCard.tsx, OrderStatusBadge.tsx
├── model/    types.ts, store.ts, selectors.ts
├── api/      useOrdersQuery.ts, order.dto.ts, mapOrder.ts
├── lib/      formatOrderTotal.ts
├── config/   constants.ts
└── index.ts  Public API(배럴 export)
```

| segment | 역할 |
|---|---|
| `ui/` | 컴포넌트 |
| `model/` | 상태·타입·selector |
| `api/` | 서버 통신(예: 데이터 조회 훅), DTO와 mapper |
| `lib/` | 순수 함수 |
| `config/` | 상수·환경 설정 |

상수와 타입을 어디에 둘지는 [code-organization.rule.md](code-organization.rule.md)를 따른다.

## 5. Public API

- 각 slice는 `index.ts`에서만 export한다. 외부에서는 slice 루트 경로(예: `@/entities/order`)로 import하고, 내부 파일을 직접 import하지 않는다.
- 같은 slice 안에서는 자기 `index.ts`를 거치지 않고 상대 경로로 import한다. `index.ts`를 거치면 순환 import가 생긴다.
- 이 규칙은 린트로 강제하는 것을 권장한다(예: Steiger, eslint-plugin-boundaries).

## 6. 실시간·비동기 데이터 배치

- 실시간 수신(폴링·SSE·WebSocket 등) 로직은 해당 도메인 slice의 `api/`에 두고, 화면(`widgets/`·`pages/`)은 훅만 쓴다. 컴포넌트에서 직접 연결하거나 구독하지 않는다.
- 차트 등 표시 컴포넌트는 데이터 가공을 맡지 않는다. 집계·리샘플링·결측 처리는 slice `lib/`의 순수 함수로 분리한다.
- 여러 도메인을 겹쳐 보여 주는 화면은 `widgets/`에서 조합한다. entities 간 직접 import는 금지한다(§7).

## 7. 안티 패턴 (즉시 거절)

- 같은 레이어 간 import(`features` ↔ `features`, `entities` ↔ `entities` 등)
- 역방향 import(예: `entities` → `features`)
- `shared/`가 상위 레이어를 참조
- slice 내부 파일을 직접 import
- 한 slice에 도메인 2개(예: `entities/user-and-order`)

## 8. 점검 체크리스트 (PR 전)

- [ ] 새 파일이 정확한 레이어와 segment에 있다.
- [ ] 같은 레이어 slice 간 import가 없고, 외부 참조는 `index.ts`를 거친다.
- [ ] `shared/`가 상위 레이어를 참조하지 않고, slice마다 도메인이 하나다.
- [ ] 실시간 구독과 데이터 가공이 컴포넌트가 아닌 `api/`·`lib/`에 있다.

## 9. 원본

- FSD 레이어 — https://feature-sliced.design/docs/reference/layers (확인일 2026-10-01)
- FSD Public API — https://feature-sliced.design/docs/reference/public-api (확인일 2026-10-01)
- FSD와 Next.js — https://feature-sliced.design/docs/guides/tech/with-nextjs (확인일 2026-10-01)
