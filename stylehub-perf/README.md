# StyleHub B2B — 성능 개선 프로젝트

> 팀 프로젝트로 완성한 StyleHub B2B를 개인적으로 fork해서, k6 부하테스트로 병목을 찾고 개선하는 과정을 기록하는 프로젝트입니다.

## Overview

StyleHub B2B(국내 여성의류 B2B 도매 거래 플랫폼) 팀 프로젝트가 끝난 뒤, 실제 서비스 규모에서는 어떤 병목이 생기는지 직접 확인해보고 싶어서 개인 저장소로 fork했습니다.

기능 구현 중심이었던 팀 프로젝트와 달리, 이번엔 **대량 데이터를 채운 상태에서 실제 API가 얼마나 버티는지 측정하고, 병목을 찾아 개선한 뒤 다시 측정하는 것**에 초점을 두고 진행하고 있습니다.

## Project Status

로컬 개발 환경을 팀 공용 인프라와 완전히 분리하고(Docker MySQL/Redis), Java Faker로 대량 시드 데이터를 구축했습니다. k6로 협의 목록 조회, 주문 생성 두 시나리오의 부하테스트를 완료해 목표 기준치를 통과했고, 주문 생성 코드를 살펴보던 중 재고가 어디서도 차감되지 않아 무제한으로 초과 판매되는 버그를 발견해 원자적 조건부 UPDATE로 수정하고 재검증까지 마쳤습니다. 이어서 재협의 버전 체이닝에서 발생하는 N+1을 재귀 CTE로 해결했고, 멘토 피드백을 반영해 `negotiations` 테이블을 1,600만 건 이상으로 부풀린 뒤 슬로우 쿼리 로그와 EXPLAIN으로 병목을 진단해 인덱스·쿼리 재작성으로 해결했습니다. 이어서 동시 사용자 축을 검증하기 위해 주문 생성 API에 k6로 동시 요청을 단계적으로 늘려가며 부하테스트를 진행, 커넥션 풀·CPU 사용률·BCrypt 인증 로직이 복합적으로 얽힌 병목을 단계별로 분리 검증했습니다. 마지막으로 실제 결제창을 거쳐야만 발급되는 `paymentKey` 때문에 보류돼있던 결제 confirm 시나리오를, 외부 호출 한 줄만 스텁으로 대체하는 방식으로 부하테스트 가능하게 만들고 검증까지 마쳐 우선순위 시나리오 3개를 모두 완료했습니다. 아래는 일자별 작업 내용을 기록합니다.

## Development Log

| 날짜 | 작업 | 키워드 | 관련 글 |
|---|---|---|---|
| [2026-07-14](<daily/2026-07-14 로컬 환경 세팅과 보안 수정.md>) | 팀 인프라와 분리된 로컬 개발 환경 재점검, gitignore 시크릿 노출 버그 발견·수정 | Docker, gitignore, Git 브랜치 전략 | |
| [2026-07-16](<daily/2026-07-16 시드 데이터 구축과 k6 준비.md>) | Java Faker 기반 시드 데이터 스크립트 설계·구축, 배치 insert 튜닝, k6 설치 | JPA, Faker, FK 의존성, Batch Insert, k6 | [블로그](https://min-soon.tistory.com/111) |
| [2026-07-17](<daily/2026-07-17 k6 쿠키 저장소 버그와 주문 생성 부하테스트.md>) | k6 쿠키 저장소 버그 해결, DataSeeder 도메인별 리팩토링, 주문 생성 부하테스트(콜드 스타트 재확인, 목표 통과) | k6, Cookie, 리팩토링, MOQ, 콜드 스타트 | [블로그](https://min-soon.tistory.com/112) |
| [2026-07-18](<daily/2026-07-18 재고 미차감 버그 발견과 수정.md>) | 주문 생성 시 재고 미차감 버그 발견, k6로 200% 오버셀 재현, 원자적 조건부 UPDATE로 수정 및 재검증 | 동시성, 재고, k6 재현 테스트, 원자적 UPDATE | [블로그](https://min-soon.tistory.com/113) |
| [2026-07-19](<daily/2026-07-19 버전 체이닝 N+1과 대용량 슬로우 쿼리 개선.md>) | 재협의 버전 체이닝 N+1을 재귀 CTE로 해결, `negotiations` 1,600만 건 환경에서 슬로우 쿼리·EXPLAIN으로 병목 진단 후 인덱스·쿼리 재작성(99초→0.23초) | 재귀 CTE, 슬로우 쿼리 로그, EXPLAIN, 인덱스, 대용량 데이터 | [블로그1](https://min-soon.tistory.com/114), [블로그2](https://min-soon.tistory.com/115) |
| [2026-07-20](<daily/2026-07-20 동시 요청 부하테스트와 CPU 병목 진단.md>) | 주문 생성 API 동시 요청 부하테스트(VU 10→50), 커넥션 풀·CPU 사용률·BCrypt 인증 로직을 단계별로 분리 검증해 복합 병목 진단 | 동시성, HikariCP, CPU 프로파일링, BCrypt, N+1 | [블로그](https://min-soon.tistory.com/116) |
| [2026-07-21](<daily/2026-07-21 결제 confirm 부하테스트 — 외부 의존성 스텁으로 우회.md>) | 실제 paymentKey 없이 결제 confirm을 부하테스트하기 위해 외부 호출만 `@Profile` 스텁으로 대체, VU 20 반복 검증으로 우선순위 시나리오 3개 완료 | Spring @Profile, 스텁, k6, 결제 API | [블로그](https://min-soon.tistory.com/117) |

프로젝트 진행에 따라 계속 추가할 예정입니다.

## 부하테스트 대상 시나리오

| 우선순위 시나리오 | API | 현재 상태 |
|---|---|---|
| 협의 목록 조회 | `GET /api/negotiations` | 부하테스트 완료, 목표 통과 (p95 85ms) |
| 주문 생성 | `POST /api/buyer/orders` | 부하테스트 완료, 목표 통과 (p95 235~370ms) |
| 결제 confirm | `POST /api/v1/payments/confirm` | 보류 — 실제 토스페이먼츠 `paymentKey`가 필요해 순수 부하테스트로 직접 호출하기 어려움, 접근 방식 검토 중 |


## 시드 데이터 설계

부하테스트를 하려면 실제 서비스 규모의 데이터가 필요했지만, 로컬/운영 DB 모두 데이터가 거의 없어서 Java Faker(`datafaker`)로 직접 생성했습니다. DB 외래키(FK) 제약 때문에, 참조당하는 엔티티가 먼저 존재해야 참조하는 엔티티를 만들 수 있어 아래 순서를 지켜야 했습니다.

```
Category ─┐
Company ──┼─→ Brand
          └─→ User (Buyer/Seller) ──→ Product ──→ SourcingRequest ──→ Quote ──→ Negotiation
                                                                              └─→ Contract ──→ Order
```

각 단계를 소규모(5~30개)로 먼저 검증하고, 전체 체인이 정상 동작하는 걸 확인한 뒤에 아래 규모로 키웠습니다.

| Company | User | Product | ProductOption | Address | CartItem | SourcingRequest | Quote | Negotiation | Contract | Order | OrderItem |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 200 | 600 | 10,000 | 30,000 | 200 | 608 | 3,000 | 5,000 | 약 1,964 | 약 1,517 | 20,000 | 약 39,951 |

## Tech Stack

| 구분 | 기술 |
|---|---|
| **Backend** | Java 17, Spring Boot 3.5, Spring Data JPA |
| **Test Data** | net.datafaker |
| **Load Testing** | k6 |
| **Database** | MySQL(Docker), Redis(Docker) |
