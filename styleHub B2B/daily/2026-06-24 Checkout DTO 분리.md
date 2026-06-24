# 2026-06-24 작업 로그

> 관련 글: [장바구니 기반 Checkout과 주문 기반 Checkout을 분리한 이유](https://min-soon.tistory.com/94)

## Today

- 주문 목록의 결제 대기 주문에서 `Checkout` 화면으로 다시 진입하는 흐름을 연결했다.
- 기존 Checkout 화면이 `cartItemIds`만 기준으로 동작하면서, 주문 목록에서 이동한 경우 `선택한 장바구니 상품이 없습니다.` 메시지가 표시되는 문제를 확인했다.
- 장바구니 기반 Checkout과 주문 기반 Checkout의 기준 데이터가 다르다는 점을 정리하고, DTO와 조회 흐름을 분리했다.
- 주문 기반 Checkout에서도 결제 화면을 확인할 수 있도록 개발용 임시 결제 완료 버튼을 추가했다.
- 실행 중 발생한 DB, Supabase, Redis 오류 로그를 구분해 각각의 원인을 확인했다.

## Learned

- 같은 화면을 사용하더라도 기준 엔티티가 다르면 API 요청과 응답 DTO를 분리하는 편이 의도가 더 명확하다.
- 장바구니 기반 Checkout은 아직 주문 생성 전이므로 `CartItem`을 기준으로 금액을 계산하고, 주문 기반 Checkout은 이미 생성된 `Order`와 `OrderItem`을 기준으로 결제 정보를 조회한다.
- 하나의 DTO에 `cartItemIds`와 `orderId`를 모두 넣으면 코드 수는 줄어들 수 있지만, 어떤 필드가 언제 필요한지 불명확해지고 null 검증이 복잡해진다.
- 서버 실행 오류를 볼 때는 가장 아래 예외만 보지 않고, DB 연결 실패인지 설정값 누락인지 외부 인프라 연결 실패인지 구분해야 한다.

## Troubleshooting

- 주문 목록에서 `결제하러 가기`를 눌렀을 때 Checkout 화면이 `cartItemIds`를 찾으면서 `선택한 장바구니 상품이 없습니다.` 메시지가 표시됐다. 주문 목록에서 진입한 경우에는 이미 주문이 생성되어 있으므로 `orderId`를 기준으로 조회해야 했다.
- 기존 `CheckoutResponse`를 그대로 재사용할지 고민했지만, 장바구니 기반 응답은 `CartResponse` 중심이고 주문 기반 응답은 `OrderItem` 중심이라 DTO를 분리했다.
- `CartCheckoutRequest`, `CartCheckoutResponse`, `OrderCheckoutResponse`, `OrderCheckoutItemResponse`로 역할을 나누어 장바구니 미리보기와 주문 결제 재진입 흐름을 구분했다.
- 서버 실행 과정에서 `supabase.pdf-bucket` 설정 누락, Redis 미실행, local DB 미실행 문제가 각각 다른 로그로 나타났다. 로그의 원인 위치를 기준으로 설정 문제와 인프라 실행 문제를 분리해서 확인했다.

## Next

- 실제 결제 성공 이후 `PENDING` 주문을 `CONFIRMED` 상태로 변경하는 API 흐름을 결제 담당 파트와 맞춘다.
- 주문 기반 Checkout에서 결제 성공 후 주문 상세 또는 주문 목록으로 이동하는 후속 UX를 정리한다.
- Checkout 관련 DTO와 API 명칭이 장바구니 기준인지 주문 기준인지 더 일관되게 드러나는지 한 번 더 점검한다.
