# StyleHub B2B — 성능 개선 프로젝트

> 팀 프로젝트로 완성한 StyleHub B2B를 개인적으로 fork해서, k6 부하테스트로 병목을 찾고 개선하는 과정을 기록하는 프로젝트입니다.

## Overview

StyleHub B2B(국내 여성의류 B2B 도매 거래 플랫폼) 팀 프로젝트가 끝난 뒤, 실제 서비스 규모에서는 어떤 병목이 생기는지 직접 확인해보고 싶어서 개인 저장소로 fork했습니다.

기능 구현 중심이었던 팀 프로젝트와 달리, 이번엔 **대량 데이터를 채운 상태에서 실제 API가 얼마나 버티는지 측정하고, 병목을 찾아 개선한 뒤 다시 측정하는 것**에 초점을 두고 진행하고 있습니다.

## Project Status

로컬 개발 환경을 팀 공용 인프라와 완전히 분리하고(Docker MySQL/Redis), Java Faker로 대량 시드 데이터(상품 1만 건, 주문 2만 건 등)를 만드는 단계까지 마쳤습니다. 현재는 k6를 설치하고 첫 부하테스트 시나리오(로그인 + 협상 목록 조회)를 작성하는 단계입니다. 아래는 진행 중인 작업과 아직 막혀있는 부분을 구분해 기록합니다.

## Development Log

| 날짜 | 작업 | 키워드 | 관련 글 |
|---|---|---|---|
| [2026-07-14](<daily/2026-07-14 로컬 환경 세팅과 보안 수정.md>) | 팀 인프라와 분리된 로컬 개발 환경 재점검, gitignore 시크릿 노출 버그 발견·수정 | Docker, gitignore, Git 브랜치 전략 | |
| [2026-07-16](<daily/2026-07-16 시드 데이터 구축과 k6 준비.md>) | Java Faker 기반 시드 데이터 스크립트 설계·구축, 배치 insert 튜닝, k6 설치 | JPA, Faker, FK 의존성, Batch Insert, k6 | [블로그](https://min-soon.tistory.com/111) |

프로젝트 진행에 따라 계속 추가할 예정입니다.

## 부하테스트 대상 시나리오

| 우선순위 시나리오 | API | 현재 상태 |
|---|---|---|
| 협상 목록 조회 | `GET /api/negotiations` | 즉시 테스트 가능 |
| 주문 생성 | `POST /api/buyer/orders` | 보류 — Cart/Address 시드 데이터 선행 필요 (상품 ID가 아닌 `cartItemIds`, `addressId`를 요구) |
| 결제 confirm | `POST /api/v1/payments/confirm` | 보류 — 실제 토스페이먼츠 `paymentKey`가 필요해 순수 부하테스트로 직접 호출하기 어려움, 접근 방식 검토 중 |

인증은 이메일/비밀번호 로그인 후 발급되는 `accessToken` 쿠키(HttpOnly) 기반이며, `Authorization` 헤더는 사용하지 않습니다. k6는 가상유저(VU) 단위로 쿠키를 자동 유지하므로, 로그인 요청 한 번 이후로는 별도 토큰 관리 없이 인증이 필요한 API를 이어서 호출할 수 있습니다.

## 시드 데이터 설계

부하테스트를 하려면 실제 서비스 규모의 데이터가 필요했지만, 로컬/운영 DB 모두 데이터가 거의 없어서 Java Faker(`datafaker`)로 직접 생성했습니다. DB 외래키(FK) 제약 때문에, 참조당하는 엔티티가 먼저 존재해야 참조하는 엔티티를 만들 수 있어 아래 순서를 지켜야 했습니다.

```
Category ─┐
Company ──┼─→ Brand
          └─→ User (Buyer/Seller) ──→ Product ──→ SourcingRequest ──→ Quote ──→ Negotiation
                                                                              └─→ Contract ──→ Order
```

각 단계를 소규모(5~30개)로 먼저 검증하고, 전체 체인이 정상 동작하는 걸 확인한 뒤에 아래 규모로 키웠습니다.

| 테이블 | 규모 |
|---|---|
| Company | 200 |
| User | 600 |
| Product | 10,000 |
| SourcingRequest | 3,000 |
| Quote | 5,000 |
| Negotiation | 약 1,964 |
| Contract | 약 1,517 |
| Order | 20,000 |
| OrderItem | 약 39,951 |

## Key Design Decisions

### 시드 데이터는 DB 덤프 대신 직접 생성

운영 DB도 실사용자가 적어 덤프해봤자 부하테스트에 의미 있는 규모가 안 됐습니다. Java Faker로 직접 생성하는 대신, 실제 엔티티 연관관계와 FK 제약을 그대로 따르도록 설계해서 실제 서비스 데이터 구조와 최대한 가깝게 만들었습니다.

### 재실행 안전장치와 그로 인한 제약

`CommandLineRunner`는 서버가 켜질 때마다 실행되므로, 재시작마다 데이터가 중복 생성되지 않도록 "이미 데이터가 있으면 건너뛰기" 가드를 두었습니다. 이 가드 때문에 새 단계를 추가할 때마다 이전 데이터를 전부 삭제하고 처음부터 다시 실행해야 하는 제약이 생겼고, 이후로는 "새 단계 추가 → 관련 테이블 삭제(자식 테이블부터) → 재실행 → 확인"을 고정 사이클로 삼았습니다.

### 대량 저장을 위한 배치 insert 튜닝

`saveAll()`을 호출해도 Hibernate는 기본적으로 한 줄씩 INSERT를 보냅니다. 수만 건 단위로 저장할 땐 MySQL 드라이버 옵션(`rewriteBatchedStatements=true`)과 Hibernate 설정(`hibernate.jdbc.batch_size`, `order_inserts`)을 함께 켜야 실제로 여러 insert가 하나의 요청으로 묶여서 나간다는 걸 확인하고 적용했습니다.

## Tech Stack

| 구분 | 기술 |
|---|---|
| **Backend** | Java 17, Spring Boot 3.5, Spring Data JPA |
| **Test Data** | net.datafaker |
| **Load Testing** | k6 |
| **Database** | MySQL(Docker), Redis(Docker) |
