# TownTalk - 지역 기반 커뮤니티

Spring Security 인증부터 게시글, 댓글, 좋아요, 마이페이지까지 백엔드 핵심 흐름을 직접 설계하고 검증한 개인 MVP 프로젝트입니다.

## Project Focus

- 지역 범위에 따라 동네 게시판과 익명 게시판을 구분했습니다.
- JWT Access Token과 Refresh Token으로 인증 유지 흐름을 구현했습니다.
- Refresh Token은 HttpOnly Cookie와 Redis를 이용해 관리했습니다.
- QueryDSL Projection으로 화면에 필요한 데이터만 조회했습니다.
- 게시글, 댓글, 좋아요 기능에 인증 사용자 기반 권한 검사를 적용했습니다.

## Tech Stack

| 구분 | 기술 |
|---|---|
| Backend | Java 21, Spring Boot, Spring Data JPA |
| Security | Spring Security, JWT |
| Query | QueryDSL |
| Database | PostgreSQL, Supabase |
| Cache | Redis |
| Frontend | React |
| Build & CI | Gradle, GitHub Actions |

## Core Features

- 회원가입, 로그인, 로그아웃 및 토큰 재발급
- 지역 기반 게시글 작성, 조회, 수정, 삭제
- 댓글 작성 및 조회
- 게시글 좋아요 토글과 집계
- 마이페이지 프로필 및 활동 내역 조회
- 인증 사용자와 작성자를 비교한 수정·삭제 권한 검사

## Development Log

| 날짜 | 작업 | 키워드 | 관련 글                                   |
|---|---|---|----------------------------------------|
| [2026-05-25](<daily/2026-05-25 로그인 연동.md>) | React 로그인 API 연동 | Spring Security, React | [블로그](https://min-soon.tistory.com/66) |
| [2026-05-26](<daily/2026-05-26 JWT 인증.md>) | JWT 인증 필터와 사용자 인증 | JWT, SecurityContext | [블로그](https://min-soon.tistory.com/67) |
| [2026-05-27](<daily/2026-05-27 인증 예외와 CI.md>) | 인증 예외 응답과 CI 구성 | 401, GitHub Actions | [블로그](https://min-soon.tistory.com/68) |
| [2026-05-28](<daily/2026-05-28 Refresh Token 저장.md>) | Refresh Token Redis 저장 | Redis, JWT | [블로그](https://min-soon.tistory.com/69) |
| [2026-05-29](<daily/2026-05-29 게시글 작성 연결.md>) | 게시글 작성 API와 화면 연결 | REST API, React | -                                      |
| [2026-05-31](<daily/2026-05-31 글쓰기 오류 추적.md>) | 인증 오류로 보인 DB 오류 추적 | Security, Error Handling | [블로그](https://min-soon.tistory.com/70) |
| [2026-06-01](<daily/2026-06-01 토큰 재발급.md>) | Cookie 기반 Access Token 재발급 | HttpOnly, Reissue | [블로그](https://min-soon.tistory.com/71) |
| [2026-06-02](<daily/2026-06-02 QueryDSL 최신글 조회.md>) | QueryDSL 최신 게시글 조회 | QueryDSL, Projection | [블로그](https://min-soon.tistory.com/72) |
| [2026-06-03](<daily/2026-06-03 게시글 상세와 수정.md>) | 게시글 상세 조회와 수정 | JPA, Authorization | [블로그](https://min-soon.tistory.com/74) |
| [2026-06-04](<daily/2026-06-04 게시글 권한과 삭제.md>) | 작성자 권한 검사와 삭제 처리 | Transaction, Authorization | [블로그](https://min-soon.tistory.com/75) |
| [2026-06-05](<daily/2026-06-05 댓글 API 연동.md>) | 댓글 작성·조회 API 연동 | DTO, Validation | [블로그](https://min-soon.tistory.com/76) |
| [2026-06-06](<daily/2026-06-06 작성자 조회와 권한 검사.md>) | 작성자 조회와 권한 검사 개선 | QueryDSL, Authorization | [블로그](https://min-soon.tistory.com/77) |
| [2026-06-07](<daily/2026-06-07 좋아요와 집계 조회.md>) | 좋아요 토글과 집계 조회 | QueryDSL, Consistency | [블로그](https://min-soon.tistory.com/79) |
| [2026-06-08](<daily/2026-06-08 마이페이지 통계 조회.md>) | 마이페이지 통계 조회 | QueryDSL, Aggregate | [블로그](https://min-soon.tistory.com/80) |
| [2026-06-09](<daily/2026-06-09 마이페이지 활동 조회.md>) | 마이페이지 활동 데이터 조합 | DTO, Service Design | [블로그](https://min-soon.tistory.com/81) |
| [2026-06-10](<daily/2026-06-10 프로필 수정과 변경 감지.md>) | 프로필 수정과 JPA 변경 감지 | Dirty Checking, Validation | [블로그](https://min-soon.tistory.com/82) |
| [2026-06-11](<daily/2026-06-11 검증 메시지 정리.md>) | 검증 메시지와 예외 응답 정리 | Validation, Error Response | -                                      |
전체 작업 기록은 [`daily`](daily) 폴더에서 확인할 수 있습니다.