# 2026-06-23 작업 로그

> 관련 글: [ApiResponse 적용 후 응답 구조가 꼬인 이유](https://min-soon.tistory.com/93)

## Today

- 팀원이 적용한 `ApiResponse` 공통 응답 포맷과 프론트 공통 axios 처리 흐름을 확인했다.
- 장바구니와 Checkout 화면에서 공통 axios가 반환한 값을 다시 `response.data`로 꺼내면서 발생하던 `undefined` 문제를 수정했다.
- 장바구니 화면에서도 Checkout과 동일하게 선택 상품 기준 배송비와 무료 배송 조건을 표시하도록 정리했다.
- 바이어 대시보드의 주문 관리 링크가 `/buyer/orders`로 이동하지 않던 문제를 확인하고, 바이어 주문 목록 라우트를 연결했다.

## Learned

- 공통 axios가 `ApiResponse.data`를 이미 꺼내 반환하는 구조라면, 컴포넌트에서는 응답 객체가 아니라 실제 데이터가 넘어온다고 이해해야 한다.
- 팀 프로젝트에서는 내가 작성하지 않은 공통 코드라도, 어느 계층에서 응답을 가공하는지 먼저 확인해야 한다.
- 장바구니의 배송비 계산은 사용자 안내용 예상 금액이고, Checkout과 주문 생성 단계의 서버 계산은 결제 기준 확정 금액으로 구분해야 한다.
- 프론트 라우팅은 링크만 추가한다고 동작하는 것이 아니라, 해당 URL을 처리할 Route 등록까지 함께 확인해야 한다.

## Troubleshooting

- Checkout 배송지 조회에서 `Cannot read properties of undefined` 오류가 발생했다. 백엔드 응답 실패가 아니라, 공통 axios가 이미 `ApiResponse.data`를 반환하는데 컴포넌트에서 다시 `.data`를 접근한 것이 원인이었다.
- 장바구니 조회와 Checkout 조회 코드를 공통 axios 반환 방식에 맞춰 실제 데이터를 바로 사용하도록 수정했다.
- 대시보드의 주문 관리 링크는 `/buyer/orders?status=DELIVERED`를 바라보고 있었지만, 라우터에는 `/buyer/orders` 목록 경로가 없어 이동이 되지 않았다. `buyer` 하위 라우트에 주문 목록 컴포넌트를 연결해 해결했다.

## Next

- 주문 목록 조회 API를 실제 백엔드 데이터와 연결할지, 기존 목업 화면을 먼저 정리할지 결정한다.
- 주문 상세 화면에서 `orders`와 `order_items` 스냅샷 데이터를 어떤 DTO로 내려줄지 설계한다.
- 장바구니와 Checkout에서 같은 금액 기준을 사용하고 있는지, 서버 계산값과 프론트 표시값의 역할을 계속 분리해 검증한다.
