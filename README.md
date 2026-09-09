# CHO JUN YEON 👋

데이터 흐름과 서비스 안정성을 고민하는 Backend Developer

## 💪 Skills

### Languages

![Java](https://img.shields.io/badge/Java-007396?style=for-the-badge&logo=openjdk&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)

### Frontend

![Vue.js](https://img.shields.io/badge/Vue.js-4FC08D?style=for-the-badge&logo=vuedotjs&logoColor=white)

### Backend & Database

![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Spring Data JPA](https://img.shields.io/badge/Spring_Data_JPA-6DB33F?style=for-the-badge&logo=spring&logoColor=white)
![WebSocket](https://img.shields.io/badge/WebSocket-010101?style=for-the-badge&logo=socketdotio&logoColor=white)


### Deployment

![AWS S3](https://img.shields.io/badge/AWS_S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![Amazon EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)


### Tools

![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

## 🚀 Project

### 01. 탕탕 (2026.07.06 - 2026.08.26)

> **프로젝트 설명**<br>
> 새는 돈을 찾아 챌린지로 관리하는 자산관리 서비스입니다. 계좌 연결 후 거래내역을 동기화하면 규칙 기반으로 자동 분류해 미션·챌린지·리포트에 활용할 수 있도록 구현했습니다.
>
> **내 역할**
> - 자산현황 API와 화면을 풀스택으로 구현하고, 거래내역 도메인 담당
> - 마이데이터·CODEF API를 모방한 금융 API 목 서버를 별도 구축
> - 거래내역 동기화 오케스트레이션과 자동 카테고리화 파이프라인 설계·구현
>
> **기술 스택**<br>
> `Java 17` · `Spring MVC 5` · `MyBatis` · `MySQL 8` · `Vue 3`

#### 📌 핵심 구현

- **금융 API 목 서버**: 정상 응답과 계좌 없음·토큰 만료·호출 제한의 4가지 시나리오를 `scenarioKey`로 재현하고, 모든 응답을 `code`, `message`, `data`, `traceId` 형식으로 통일
- **거래내역 동기화**: 은행·예적금·증권·대출·페이머니·카드의 6개 소스를 수집한 뒤, 전체 성공 시 자연키 기반 멱등 저장을 수행
- **트랜잭션 경계 분리**: 외부 HTTP 호출을 트랜잭션 밖에서 처리하고, 수집 완료 후에만 저장 트랜잭션을 열어 DB 커넥션 점유와 부분 저장 방지
- **4단계 자동 카테고리화**: 사용자 수정 이력, 공용 가맹점 매핑, MCC·업종, 키워드 순으로 분류하고 `category_source`로 분류 근거 기록
- **LLM 호출 최적화**: 규칙에서 분류하지 못한 거래만 최대 20건씩 비동기 작업 큐에 넣어 외부 LLM 지연이 동기화 응답에 영향을 주지 않도록 구성

#### 🛠 문제 해결

<details>
<summary><strong>실제 금융 API를 사용할 수 없는 개발·검증 환경</strong></summary>

##### 문제

- 마이데이터 사업자 자격과 CODEF 유료 계약이 없어 실제 금융 데이터를 직접 연동할 수 없었음
- 실제 API는 응답이 매번 달라 기능 검증이 어렵고, 토큰 만료나 호출 제한 같은 실패 상황을 의도적으로 재현할 수 없었음

##### 해결

- 서비스와 분리된 금융 API 목 서버를 구축하고, 사용자별 `scenarioKey`로 정상·계좌 없음·토큰 만료·호출 제한 시나리오를 선택하도록 구현
- 정상 응답은 목 서버의 원천 테이블에서 조립하고, 실패 응답은 고정 JSON으로 반환하도록 구성

##### 결과

- 정상 흐름과 외부 API 실패 시나리오를 반복해서 검증할 수 있게 됨
- 서비스 데이터베이스와 테스트 데이터를 분리해 개발 환경의 데이터 오염 방지

</details>

<details>
<summary><strong>외부 호출 중 데이터베이스 커넥션 점유와 부분 저장 위험</strong></summary>

##### 문제

- 6개 금융 소스를 순차 호출하는 동안 트랜잭션을 유지하면, 사용자 요청이 몰릴 때 데이터베이스 커넥션 풀이 고갈될 수 있었음
- 수집 중 일부 소스가 실패한 상태에서 저장하면 불완전한 거래내역이 미션·챌린지·리포트에 전달될 위험이 있었음

##### 해결

- 외부 HTTP 호출을 트랜잭션 밖에서 모두 완료한 뒤, 전체 성공 시에만 저장 트랜잭션을 열도록 동기화 흐름 분리
- 수집·저장·분류 단계별 실패 이력을 남기고, 실패 이력은 별도 트랜잭션으로 저장

##### 결과

- 외부 응답을 기다리는 동안 데이터베이스 커넥션을 점유하지 않게 됨
- 실패한 동기화 요청은 거래내역을 부분 저장하지 않고 실패 원인을 추적 가능

</details>

<details>
<summary><strong>동시 동기화 요청에서 발생하는 중복 저장 경합</strong></summary>

##### 문제

- 자연키 기준 `update` 후 `insert`하는 저장 방식은 원자적이지 않아, 같은 사용자의 요청이 겹치면 두 요청이 모두 미존재 상태를 확인하고 중복 삽입을 시도할 수 있었음

##### 해결

- 중복 키 예외가 발생한 경우 한 번만 `update`로 되돌려 처리하도록 경합 지점을 제한
- 동기화 결과가 반복 실행에도 같도록 멱등 저장을 적용해 수동 재동기화와 30분 주기 배치가 같은 로직을 사용하도록 구성

##### 결과

- 동시 요청으로 인한 중복 삽입 시도를 예외 처리로 흡수하고 기존 거래내역을 갱신 가능
- 동일한 동기화 작업을 반복해도 일관된 결과를 유지

</details>

<details>
<summary><strong>LLM의 가맹점명 표기 차이 미인식과 오분류 위험</strong></summary>

##### 문제

- 키워드·업종 기반 규칙에서 분류하지 못한 거래를 마지막 단계에서 LLM으로 분류했지만, `CU강남점`은 분류하고 `씨유강남점`은 미분류하는 등 한글 가맹점명과 지점명이 띄어쓰기 없이 들어온 경우 인식률이 낮았음
- 자동 분류에서 오분류는 미분류보다 사용자 신뢰를 더 크게 떨어뜨릴 수 있어, LLM 결과를 그대로 반영하기 어려웠음

##### 해결

- 데이터베이스의 미분류 거래를 분석해 한글 가맹점명과 지점명이 결합된 표기 패턴을 확인하고, 해당 표현을 해석할 수 있도록 LLM 프롬프트를 정교화
- 모델이 분류 결과와 함께 신뢰도를 판단하도록 하고, 기준치에 미달하면 분류 결과가 있어도 미분류로 남기도록 프롬프트에 규칙 추가

##### 결과

- 가맹점명 표기 차이로 발생하던 미분류 사례의 분류 커버리지를 개선
- 신뢰도가 낮은 자동 분류를 차단해 사용자가 잘못된 카테고리를 받아보는 상황을 줄임

</details>

#### 🔗 원본 레포

[![GitHub Repository](https://img.shields.io/badge/탕탕-Repository-181717?style=for-the-badge&logo=github)](https://github.com/KB-TangTang)

---

### 02. BillBook (2025.03 - 2025.11)

> **프로젝트 설명**<br>
> 도서 탐색부터 대여 협의, 포인트 거래, 반납 기한 관리까지 연결한 도서 대여 플랫폼입니다. 소유자와 대여 희망자가 실시간 채팅으로 거래 조건을 협의하고, 서버 검증을 거친 포인트 거래를 진행할 수 있도록 구현했습니다.
>
> **내 역할**
> - 3인 팀 프로젝트에서 회원·프로필 API, 거래 채팅, 포인트 거래 기능 담당
> - EC2 배포와 AWS S3 기반 이미지 저장소 구성 및 협업
> - Spring Boot·JPA·MySQL·STOMP를 활용한 도메인 API와 데이터 모델 구현
>
> **기술 스택**<br>
> `Java 21` · `Spring Boot 3.4` · `JPA` · `MySQL` · `STOMP` · `AWS EC2` · `AWS S3`

#### 📌 핵심 구현

- **실시간 거래 채팅**: 도서·구매자 조합으로 기존 채팅방을 조회하거나 생성해 중복을 방지하고, STOMP로 메시지를 실시간 전달
- **채팅 이력 관리**: 메시지를 데이터베이스에 저장한 뒤 구독자에게 발행하고, 최근 메시지를 30개 단위로 분할 조회하며 채팅방 목록에 마지막 메시지 제공
- **포인트 거래 상태 프로토콜**: UUID로 거래를 식별하고, `PENDING` 상태에서 금액과 결제 상태를 서버가 검증한 경우에만 포인트를 적립해 `SUCCESS`로 전이
- **S3 이미지 저장소**: 도서·프로필·채팅 이미지 업로드, 다운로드, 삭제를 공통 `S3UploadService`로 관리하고 UUID 기반 객체 키로 파일명 충돌 방지

#### 🛠 문제 해결

<details>
<summary><strong>채팅 메시지 저장 중 발생할 수 있는 유실</strong></summary>

##### 문제

- 메시지를 저장한 뒤 응답 DTO를 만드는 과정에서 예외가 발생하면, 각 리포지토리 호출이 개별 커밋되어 일부 데이터만 남거나 클라이언트에 메시지가 전달되지 않을 수 있었음

##### 해결

- `saveAndBuild()`에 `@Transactional`을 적용해 조회·저장·응답 생성을 하나의 트랜잭션으로 묶음
- 데이터베이스 저장이 성공한 이후에만 STOMP 브로커로 메시지를 발행하도록 흐름 구성

##### 결과

- 중간 예외 발생 시 채팅 메시지 관련 변경을 함께 롤백할 수 있게 됨
- 브로커에 전달되는 메시지가 항상 데이터베이스에 저장된 상태를 유지

</details>

<details>
<summary><strong>메시지가 있는 채팅방 삭제 실패</strong></summary>

##### 문제

- 메시지가 없는 채팅방은 삭제되지만, 메시지가 있는 채팅방은 `Message`가 `ChatRoom`을 외래 키로 참조해 제약 조건 위반으로 삭제에 실패

##### 해결

- 자동 `ON DELETE CASCADE` 대신 메시지를 먼저 삭제하고 채팅방을 삭제하는 순서를 명시적으로 제어
- 대화 이력이 거래 근거가 될 수 있음을 고려해 삭제 흐름을 서비스 로직에서 관리

##### 결과

- 메시지 유무와 관계없이 채팅방을 일관되게 삭제 가능
- 연관관계의 삭제 순서와 거래 이력 관리 정책을 코드에서 명확히 표현

</details>

#### 🔗 원본 레포

[![GitHub Repository](https://img.shields.io/badge/BillBook-Repository-181717?style=for-the-badge&logo=github)](https://github.com/BillBook-2025/billbook-backend)

---

### 03. Cookit (2025.06.30 - 2025.08.17)

> **프로젝트 설명**<br>
> 냉장고 속 식재료의 소비기한을 관리하고, 보유 재료와 알레르기·음식 취향을 반영해 레시피를 추천하는 서비스입니다. 영수증 또는 식재료 사진으로 재료를 등록하면 보관 방식과 카테고리에 따라 소비기한을 자동 계산합니다.
>
> **내 역할**
> - 5인 팀 프로젝트에서 OAuth·JWT 인증을 제외한 전 도메인을 단독 설계·구현
> - 냉장고·재료·레시피 추천·사용자 취향 도메인과 OCR/Vision·공공 레시피 API 연동 구현
> - JPA 기반 엔티티 연관관계와 MySQL 데이터베이스 설계
>
> **기술 스택**<br>
> `Java 21` · `Spring Boot 3.4` · `JPA` · `MySQL` · `AWS S3` · `식품안전나라 API` · `CLOVA OCR` · `Google Vision`

#### 📌 핵심 구현

- **이미지 기반 재료 등록**: 영수증 OCR과 식재료 사진 인식 결과를 재료 사전과 매칭해 카테고리를 자동 분류
- **소비기한 자동 계산**: 13개 식재료 카테고리와 실온·냉장·냉동 보관 조건에 따른 기준일로 소비기한 산정
- **맞춤형 레시피 추천**: 식품안전나라 COOKRCP01 API에서 보유 재료 기반 레시피를 조회하고, 알레르기·좋아요·싫어요 정보를 반영해 결과 필터링
- **레시피 데이터 재사용**: 외부 API로 조회한 레시피와 조리 단계를 `Recipe`, `RecipeStep` 엔티티에 저장해 상세 조회·찜·조리 이력에 활용

#### 🛠 문제 해결

<details>
<summary><strong>HEIC 이미지의 OCR 미지원</strong></summary>

##### 문제

- 아이폰 기본 촬영 포맷인 HEIC를 CLOVA OCR API가 지원하지 않아 재료 이미지 인식 요청이 실패할 수 있었음

##### 해결

- 서버에서 ImageMagick으로 HEIC 이미지를 JPG로 변환한 뒤 OCR·Vision API에 전송하도록 처리 경로 분기

##### 결과

- 촬영 기기와 관계없이 이미지 기반 재료 등록 흐름을 유지
- 사용자는 별도 포맷 변환 없이 영수증 또는 식재료 사진을 등록 가능

</details>

<details>
<summary><strong>수량 수정과 재료 삭제의 분리된 요청</strong></summary>

##### 문제

- 재료 수량을 0으로 변경할 때 수정 API와 삭제 API를 별도로 호출하면 클라이언트 처리와 서버 요청이 불필요하게 늘어남

##### 해결

- 수량 수정 로직에서 값이 0인지 확인하고, 하나의 트랜잭션 안에서 해당 재료를 냉장고에서 제거하도록 분기 처리

##### 결과

- 별도 삭제 요청 없이 수량 변경만으로 재료를 정리할 수 있게 됨
- 클라이언트 API 호출 수와 상태 관리 복잡도 감소

</details>

<details>
<summary><strong>외부 레시피 API의 동기 의존</strong></summary>

##### 문제

- 레시피 원본이 외부 API에만 있으면 API 장애 또는 지연 시 상세 조회·찜·조리 이력 기능도 함께 영향을 받음

##### 해결

- 추천 시 조회한 레시피와 조리 단계를 내부 데이터베이스에 저장해 서비스 기능이 외부 API 응답에만 의존하지 않도록 설계

##### 결과

- 저장된 레시피는 외부 API 상태와 무관하게 상세 조회와 사용자 활동 기능에 재사용 가능
- 중복 저장 요청은 409 응답으로 처리해 데이터 일관성 유지

</details>

<details>
<summary><strong>도메인별로 달라질 수 있는 오류 응답</strong></summary>

##### 문제

- 컨트롤러마다 예외 처리가 분산되면 프론트엔드가 오류 원인과 사용자 안내 문구를 일관되게 판단하기 어려움

##### 해결

- `@ControllerAdvice` 기반 전역 예외 처리기를 구성하고, 커스텀 예외를 HTTP 상태 코드로 매핑하는 규약 정의

##### 결과

- 모든 API가 같은 오류 응답 규칙을 따르게 됨
- 프론트엔드는 상태 코드 기준으로 사용자 안내를 일관되게 분기 가능

</details>

#### 🔗 원본 레포

[![GitHub Repository](https://img.shields.io/badge/Cookit-Repository-181717?style=for-the-badge&logo=github)](https://github.com/HICC-2025-1-PC-TEAM5/backend)

---

### 04. HiccOrder (힉오더) (2024.02 - 2024.09)

> **프로젝트 설명**<br>
> 주문이 몰리는 축제 부스에서 QR 접속부터 테이블 주문, 조리 상태 관리, 결제 정산까지 수기 없이 처리하도록 만든 부스 운영 백엔드 서비스입니다. 대학 축제 동아리 부스의 주문·결제 관리에 실제로 사용했습니다.
>
> **내 역할**
> - 9인 팀 프로젝트에서 주문 도메인 API 설계·구현 담당
> - 부스 전체·테이블별 주문 조회, 주문 생성·수정·상태 변경·삭제 API 구현
> - SimpleJWT 기반 refresh 토큰을 활용한 access 토큰 재발급 API 구현
>
> **기술 스택**<br>
> `Python` · `Django REST Framework` · `SimpleJWT` · `SQLite` · `Postman`

#### 📌 핵심 구현

- **테이블 단위 주문 API**: 부스 전체와 테이블별 주문을 조회하고, 주문 생성·수정·삭제 기능을 REST API로 구현
- **주문 상태 흐름 관리**: 주문 완료, 조리 시작, 조리 완료, 결제 완료의 4단계 상태 코드를 정의해 주문 진행 상황을 관리
- **일괄 주문 검증·저장 분리**: 모든 주문 항목의 serializer 유효성과 메뉴 존재 여부를 먼저 검증하고, 전체 통과 시에만 일괄 저장해 부분 저장 방지
- **토큰 재발급**: access 토큰 만료 시 refresh 토큰으로 새 access 토큰을 발급해 인증 상태 유지

#### 🔗 원본 레포

[![GitHub Repository](https://img.shields.io/badge/HiccOrder-Repository-181717?style=for-the-badge&logo=github)](https://github.com/HiccOrder/HiccOrder-Backend)

---

### 05. Ganadi (가나디) (2026.04.03 - 2026.04.14)

> **프로젝트 설명**<br>
> 소비 기록과 캐릭터 성장을 결합한 가계부 서비스입니다.
>
> **내 역할**
> - Vue 3와 Vite를 기반으로 프론트엔드 UI를 구현하고 PWA로 배포
>
> **기술 스택**<br>
> `Vue 3` · `Vite` · `Pinia` · `Vue Router` · `Axios` · `Bootstrap` · `Chart.js` · `PWA`

#### 📌 핵심 구현

- Vue 3와 Vite를 기반으로 반응형 웹 UI 구현
- Pinia와 Vue Router를 활용한 상태 관리 및 SPA 구성
- Axios와 JSON Server를 활용한 REST API 연동 및 CRUD 구현
- Bootstrap과 Chart.js를 활용한 UI 및 소비 데이터 시각화
- Service Worker와 `vite-plugin-pwa`를 적용해 설치 가능한 PWA로 구성 및 배포

#### 🔗 원본 레포

[![GitHub Repository](https://img.shields.io/badge/Ganadi-Repository-181717?style=for-the-badge&logo=github)](https://github.com/20260304-KB7-27/Ganadi)

