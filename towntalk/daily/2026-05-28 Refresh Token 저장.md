# 2026-05-28 작업 로그

## Today
- Backend: `RefreshTokenService`를 작성해 Redis에 Refresh Token 저장/조회/삭제 흐름을 추가했다.
- Auth: `JwtProvider`에 Refresh Token 생성 기능과 만료 시간 설정을 추가했다.
- Login: 로그인 성공 시 Access Token과 Refresh Token을 함께 생성하고, Refresh Token을 Redis에 저장했다.
- Infra: Docker Redis 컨테이너를 실행하고 `redis-cli`로 Redis 연결 상태를 확인했다.

## Learned
- `StringRedisTemplate`은 Redis에 문자열 기반 key/value를 저장할 때 사용하는 Spring 도구다.
- JWT 로그인 유지는 토큰을 발급하는 것에서 끝나는 게 아니라, 토큰의 역할과 저장 위치를 나누는 구조로 봐야 한다.

## Troubleshooting
- `RefreshTokenService`에서 `StringRedisTemplate`으로 Redis에 Refresh Token을 저장하는 코드 흐름을 정리했다.
  - https://min-soon.tistory.com/69

## Blocker
- Docker 명령어가 실패했을 때 원인이 경로 문제가 아니라 Docker Desktop 엔진 실행 여부일 수 있다는 점을 다시 확인했다.
- Refresh Token을 Redis에 저장하는 단계와 브라우저에 Cookie로 내려주는 단계를 구분하는 데 시간이 걸렸다.

## Next
- 로그인 성공 시 Refresh Token을 HttpOnly Cookie로 내려준다.
- `/api/auth/reissue` API 흐름을 설계한다.
- 로그아웃 시 Redis의 Refresh Token을 삭제하는 API를 준비한다.
