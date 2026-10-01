# 코드 구조 전략 — 상수와 타입 관리 (FSD 정합)

FSD를 쓰는 프론트엔드 프로젝트에서 상수와 타입을 어디에 두는지 정한다. 레이어와 segment 구조는 [frontend-architecture.rule.md](frontend-architecture.rule.md)를 단일 기준으로 삼는다.

## 1. 원칙

상수와 타입은 별도의 전역 `lib/`·`types/` 디렉터리로 분리하지 않고 FSD 레이어와 segment에 귀속시킨다.

## 2. 상수 배치

| 분류 | 위치 | 예시 |
|---|---|---|
| 공통 상수(2개 이상 slice 또는 앱 전역) | `shared/config/constants.ts` | `ROWS_PER_PAGE`, `DATE_FORMAT` |
| 도메인 상수(특정 slice 전용) | `<layer>/<slice>/config/constants.ts`(`<layer>`는 `entities`·`features`·`widgets`·`pages`) | `ORDER_STATUS_LABEL`, `MAX_CART_ITEMS` |
| 파일 전용 상수(한 파일에서만 사용) | 그 파일 상단의 명명 상수 | `ANIMATION_DELAY_MS` |

### 2.1 배치 원칙

1. 한 파일에서만 쓰는 상수는 그 파일 상단에 명명 상수로 둔다. 함께 바뀌는 코드를 한 곳에 두기 위해서다.
2. 같은 slice의 2개 이상 파일에서 쓰면 그 slice의 `config/constants.ts`로 옮긴다.
3. 2개 이상 slice에서 쓰면 `shared/config`로 올린다.
4. `lib/`(순수 함수) 파일에서 다른 파일이 쓰는 상수를 export하지 않는다. 공유할 상수는 `config/`에 둔다.
5. 값의 표시 형식(단위·소수 자릿수·표시 라벨)은 항목별로 한 곳에 정의하고, 화면마다 다시 정의하지 않는다.
6. 상태 값의 라벨·색 토큰·정렬 순서는 한 세트로 묶어 한 곳에 둔다. 따로 흩어지면 한쪽만 바뀌어 조용히 어긋난다.

이유: 중복을 막고, 변경 영향을 파악하기 쉽게 하고, 함께 바뀌는 코드는 가까이 둔다.

### 2.2 네이밍

| 대상 | 규칙 | 예 |
|---|---|---|
| 상수 | `UPPER_SNAKE_CASE` | `ROWS_PER_PAGE` |
| 배열 | 복수형 | `SUPPORTED_LOCALES` |
| 매핑 | `_MAP`·`_LABEL`·`_BADGE` 접미사 | `ORDER_STATUS_LABEL` |
| 선택지 배열 | `_OPTIONS` 접미사 | `SORT_OPTIONS` |
| 임계값 | `_THRESHOLDS` 접미사 | `STOCK_THRESHOLDS` |
| 시간·주기 | 이름에 단위 포함 | `POLLING_INTERVAL_MS`, `SESSION_TTL_HOURS` |

리터럴 타입이 필요하면 `as const`를 붙인다.

### 2.3 허용 예외

- props·state에 의존해 컴포넌트 안에서 만드는 정의(예: 테이블 컬럼 정의 배열)
- 한 번만 쓰는 라벨([frontend.rule.md](frontend.rule.md) R17)
- 테스트용 Mock 데이터

## 3. 타입 배치 (FSD + API 추상화)

| 타입 종류 | 위치 | import 허용 범위 |
|---|---|---|
| 서버 원본(백엔드 JSON 1:1, DTO) | `entities/<slice>/api/*.dto.ts` | 해당 slice의 `api/`, 테스트 |
| UI 비즈니스 모델(안정 계약) | `entities/<slice>/model/types.ts` | 같은 slice, 상위 레이어(Public API 경유) |
| 공통 API 타입(응답 envelope·페이지네이션·오류 코드) | `shared/api/types.ts` | 모든 레이어 |

- 백엔드 변경의 파급을 줄이기 위해 `api/` segment의 mapper로 DTO를 UI 모델로 변환한다. 컴포넌트는 UI 모델(`model/types.ts`)에만 의존한다.
- 상태 코드·카테고리 코드처럼 서버가 문자열로 주는 값은 DTO에서 그대로 받고, mapper에서 UI 모델의 리터럴 유니온으로 좁힌다.
- 매핑되지 않는 값은 임의 기본값으로 바꾸지 않고 그대로 드러낸다(빈 상태·미지원 표기).
