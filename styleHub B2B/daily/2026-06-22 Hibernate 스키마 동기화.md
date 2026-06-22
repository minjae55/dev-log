# 2026-06-22 작업 로그

> 관련 글: [Hibernate ddl-auto: update인데 왜 기존 DB 컬럼은 삭제되지 않았을까?](https://min-soon.tistory.com/92)

## Today

- 장바구니에서 선택한 상품을 판매 회사별로 묶어 `Order`와 `OrderItem`을 생성하는 주문 저장 흐름을 구현했다.
- Checkout에서 실제 주문 생성 API를 호출하고, 생성된 판매사별 주문번호와 전체 주문 금액을 확인할 수 있는 개발용 결과 모달을 추가했다.
- 법인카드와 무통장입금 선택에 따라 토스페이먼츠 테스트 결제창을 호출하도록 프론트를 연결했다.
- 로컬 DB의 `orders` 테이블에서 기존 `seller_id` 외래키와 인덱스를 제거하고, `seller_company_id` 중심의 엔티티 구조를 반영했다.

## Learned

- Hibernate는 JPA 엔티티를 해석해 SQL을 생성하는 ORM 구현체이며, `ddl-auto: update`가 엔티티와 실제 DB 스키마를 완전히 동기화해 주는 것은 아니다.
- `update`는 새로운 컬럼을 추가할 수 있지만 데이터 손실 위험이 있는 기존 컬럼, 외래키, 인덱스 삭제는 자동으로 처리하지 않을 수 있다.
- 엔티티를 변경한 뒤에는 Hibernate가 생성한 SQL뿐 아니라 `SHOW CREATE TABLE`로 실제 테이블 구조를 함께 확인해야 한다.

## Troubleshooting

- 주문 저장 시 `Field 'seller_id' doesn't have a default value`가 발생했다. 엔티티에서는 `seller_id`를 제거했지만 실제 DB에는 `NOT NULL` 컬럼과 외래키가 남아 있는 것이 원인이었다.
- `SHOW CREATE TABLE orders`로 외래키와 관련 인덱스를 확인한 뒤, 외래키와 인덱스를 먼저 제거하고 마지막에 `seller_id` 컬럼을 삭제했다.
- 주문 생성 요청의 `addressId`가 `null`로 전달되어 `findById(null)` 예외가 발생했다. Network Payload와 `OrderService`의 최초 애플리케이션 코드 라인을 기준으로 추적하고, Controller에 `@RequestBody`가 적용되었는지 확인했다.

## Next

- 생성된 주문과 주문 상품이 판매 회사별로 정확히 분리되는지 여러 회사의 장바구니 데이터로 검증한다.
- 주문 생성 성공 후 사용한 장바구니 상품을 언제, 어떤 조건으로 제거할지 트랜잭션 흐름을 정리한다.
- 결제 담당 팀원에게 토스 결제 성공·실패 처리와 여러 주문을 하나의 결제에 연결하기 위한 인터페이스를 공유한다.
