# Academy LMS Spring Boot Migration

> 기존 Spring MVC 기반 LMS의 기능을 유지하면서 Spring Boot, Spring Security와 JPA 구조로 점진적으로 전환한 개인 마이그레이션 프로젝트입니다.

## Overview

첫 팀 프로젝트로 개발했던 Academy LMS를 다시 살펴보며, 제가 담당한 회원·관리자 영역을 중심으로 마이그레이션했습니다. 새로운 기술을 적용하는 것보다 **내부 구조가 바뀌어도 기존 기능이 유지되는지 검증하는 것**을 가장 중요한 기준으로 삼았습니다.

[코드 저장소](https://github.com/minjae55/academy-lms-boot) · [상세 마이그레이션 로드맵](https://github.com/minjae55/academy-lms-boot/blob/master/docs/migration-roadmap.md)

## Project Status

Spring Boot 기반 전환과 회원·관리자 기능 테스트, Spring Security, 회원 영역의 JPA·QueryDSL 전환 및 GitHub Actions CI 적용을 완료했습니다. 강의, 수강 신청, 게시판과 댓글 등 다른 팀원이 담당했던 영역은 기존 MyBatis 구조를 유지했습니다.

## Migration Records

별도의 날짜별 개발일지 대신, 각 단계에서 고민한 내용과 선택 이유를 기술 블로그에 기록했습니다.

| 단계 | 주요 내용 | 기록 |
|---|---|---|
| 시작 | Spring MVC 프로젝트를 Spring Boot·Gradle 기반으로 전환 | [마이그레이션 시작](https://min-soon.tistory.com/52) |
| 테스트 안전망 | 기존 회원·관리자 기능의 정상 및 예외 동작 고정 | [테스트 코드 작성](https://min-soon.tistory.com/64) |
| 인증·인가 | 인터셉터 중심 구조를 Spring Security로 전환 | [Spring Security 전환](https://min-soon.tistory.com/73) |
| 데이터 접근 | 회원 영역 MyBatis를 JPA·QueryDSL로 점진 전환 | [JPA 점진 전환](https://min-soon.tistory.com/78) |
| 자동 검증 | GitHub Actions에서 테스트와 JaCoCo 반복 실행 | [GitHub Actions CI 적용](https://min-soon.tistory.com/89) |

전체 글은 [Migration 카테고리](https://min-soon.tistory.com/category/Migration)에서 확인할 수 있습니다.

## Migration Strategy

```text
Spring Boot 기반 전환
→ 기존 회원·관리자 동작을 테스트로 고정
→ Spring Security 전환 후 재검증
→ JPA·QueryDSL 전환 후 재검증
→ GitHub Actions CI로 반복 검증 자동화
```

핵심 동작을 테스트로 먼저 고정하고, 구조를 전환하는 과정에서 관련 테스트를 수정·보강했습니다. 전체 도메인을 한 번에 바꾸지 않고 담당 영역부터 전환해 변경 범위를 통제했습니다.

## Migration Result

| 영역 | 변경 내용 | 상태 |
|---|---|---|
| 프로젝트 기반 | Spring MVC → Spring Boot, Maven → Gradle, `javax` → `jakarta` | 완료 |
| 테스트 | 회원·관리자 Controller/Service 테스트 작성 | 완료 |
| 인증·인가 | 인터셉터 중심 처리 → Spring Security | 완료 |
| 데이터 접근 | 회원 영역 MyBatis → JPA·QueryDSL | 부분 완료 |
| 요청 보호 | JSP form과 fetch 요청에 CSRF 적용 | 완료 |
| 자동 검증 | GitHub Actions에서 테스트와 JaCoCo 실행 | 완료 |

## Key Decisions

- 브라우저 수동 확인에만 의존하지 않고 기존 정상·예외 동작을 테스트로 고정했습니다.
- 서비스가 Repository 인터페이스에 의존하도록 구성해 MyBatis와 JPA 구현을 점진적으로 교체했습니다.
- 인증·인가 책임은 Spring Security로 옮기고, 수강 신청 기간처럼 도메인에 가까운 검사는 기존 인터셉터로 유지했습니다.
- 다중 인스턴스 운영과 배포가 목표가 아니어서 Redis Session과 CD는 이번 범위에서 제외했습니다.

## Tech Stack

| 영역 | 기술 |
|---|---|
| Backend | Java 17, Spring Boot 3.5, Spring MVC |
| View | JSP, JSTL, SiteMesh |
| Security | Spring Security, CSRF |
| Persistence | Spring Data JPA, QueryDSL, MyBatis |
| Database | PostgreSQL, H2 for test |
| Test | JUnit 5, Mockito, MockMvc, Spring Security Test |
| CI | GitHub Actions, JaCoCo |
