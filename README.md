# 안녕하세요, 백엔드 개발자 조준연입니다 👋

Java와 Spring을 중심으로 서비스를 개발합니다.  
동시 요청에서도 데이터가 일관되게 반영되는 구조와, 외부 API 지연·실패에 대응하는 처리 흐름에 관심이 있습니다.  
로그와 테스트로 문제를 재현하고, 변경 전후 결과를 비교하며 기존 프로젝트를 꾸준히 개선하고 있습니다.

## Tech Stack

- **Backend** · Java, Spring Boot, Spring MVC, Spring Data JPA, MyBatis, MySQL
- **Infrastructure** · AWS EC2 · S3, Docker
- **Testing** · JUnit 5, Mockito, Testcontainers, k6
- **Also used** · Python, Django REST Framework, JavaScript, Vue 3

## Projects

### [탕탕](https://github.com/GJYeon/TangTang-Monorepo) · 챌린지형 자산관리 서비스

거래내역을 자동 분류하고 소비 관리 챌린지로 연결하는 서비스입니다. 자산현황 화면·API와 거래내역 동기화·분류를 담당했습니다.

- 금융 API 목 서버를 구축하고, 6개 금융 소스의 병렬 수집과 저장 트랜잭션 분리로 동기화 흐름을 개선했습니다.
- 사용자 20명 기준 10회 측정에서 **평균 동기화 시간을 7.25초 → 2.4초**로 줄였습니다.
- 규칙으로 분류하지 못한 거래에 LLM을 적용하고, 후속 개선에서는 공공 상가정보를 검색 근거로 제공하도록 설계했습니다. 어휘·벡터 검색을 비교해 기존 MySQL을 활용하는 방식을 선택했습니다.

`Java` `Spring MVC` `MyBatis` `MySQL` `Vue 3`

### [BillBook](https://github.com/GJYeon/BillBook) · 지역 기반 도서 대여 서비스

책을 등록하고 실시간 채팅으로 대여를 조율하는 서비스입니다. 3인 팀에서 커뮤니티를 제외한 백엔드와 EC2·S3 구성을 담당했습니다.

- 거래글·회원 API, WebSocket·STOMP 채팅, 메시지 저장과 이력 조회를 구현했습니다.
- 2026년 10월 후속 개선에서 조건부 상태 갱신과 DB 잔액 증가 연산을 적용해 **포인트 중복 적립과 갱신 유실**을 보완했습니다. MySQL 통합 테스트에서 같은 결제의 동시 요청 8건 중 한 건만 적립되는 것을 확인했습니다(외부 결제 API는 목 처리).
- S3 호출을 DB 트랜잭션 밖으로 분리하고, DB 저장 실패 시 새로 업로드한 객체를 정리하도록 처리 흐름을 개선했습니다.

`Java` `Spring Boot` `JPA` `MySQL` `WebSocket · STOMP` `AWS EC2 · S3`

### [CookIt](https://github.com/HICC-2025-1-PC-TEAM5/backend) · 냉장고 관리·레시피 추천 서비스

사진·영수증으로 식재료를 등록하고, 보유 재료와 알레르기·취향을 반영해 레시피를 추천하는 서비스입니다.

- 요청마다 외부 API를 호출하던 추천을 **사전 적재한 로컬 DB 조회**로 전환해, 추천 과정의 외부 호출과 반복 저장을 제거했습니다.
- 기록 100건을 가진 회원 조건에서 연관 엔티티를 묶어 조회하도록 개선해 **조회 SQL을 104회 → 5회**로 줄였습니다(로컬 MySQL 측정).
- 화면에서 사용하지 않는 조리 단계를 추천 응답에서 제외해 **응답 본문을 약 67% 줄이고**, 쿼리 수와 응답 계약을 검증하는 회귀 테스트를 추가했습니다.

`Java` `Spring Boot` `JPA` `MySQL` `Docker` `Testcontainers`

## Other Projects

- **[HiccOrder](https://github.com/HiccOrder/HiccOrder-Backend)** · 대학 축제 부스에서 실제 사용한 주문·결제 관리 서비스. Django REST Framework로 주문 API와 주문 상태 관리를 구현했습니다.
- **[Ganadi](https://github.com/20260304-KB7-27/Ganadi)** · 소비 기록과 캐릭터 성장을 결합한 가계부. Vue 3 기반 UI·소비 데이터 시각화를 구현하고 PWA로 배포했습니다.
