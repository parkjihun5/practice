# 🏦 계좌 관리 및 이체 시스템 (Transfer System)
> **Java Swing과 MySQL을 활용한 4인 협업 가상 뱅킹 미니 프로젝트** > **개발 기간**: 2026.05.18 ~ 2026.05.20

<div align="center">
  <img src="https://img.shields.io/badge/Java-007396?style=for-the-badge&logo=openjdk&logoColor=white">
  <img src="https://img.shields.io/badge/JDBC-007396?style=for-the-badge&logo=java&logoColor=white">
  <img src="https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white">
  <img src="https://img.shields.io/badge/Lombok-BC0C34?style=for-the-badge&logo=lombok&logoColor=white">
  <img src="https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white">
</div>

---

## 1. 프로젝트 소개
본 프로젝트는 사용자가 안전하게 회원 계정을 생성하고, 본인 명의의 가상 계좌를 개설하여 팀원(타 회원)들 간에 실시간 송금 및 거래 내역을 조회할 수 있는 **데스크톱 가상 뱅킹 애플리케이션**입니다.  
효율적인 협업을 위해 `Model - UI(View) - DAO` 구조로 역할을 분담하여 개발을 진행했습니다.

---

## 2. 데이터베이스 설계 (ERD)
시스템의 무결성과 확장성을 고려하여 설계된 테이블 구조입니다. 회원 테이블(`user`)과 계좌 테이블(`account`), 이체 이력 테이블(`transfer`)이 유기적으로 매핑되어 있습니다.

<div align="center">
  <img src="./images/erd_diagram.png" width="80%" alt="Database ERD">
</div>

---

## 3. 담당 역할 및 핵심 구현 기능 (My Role)
저는 시스템의 보안 관문인 **회원 관리 시스템(인증/인가 및 보안 유틸리티)과 사용자 친화적 UI 엔지니어링**을 전담하여 구현했습니다.

* **보안성 높은 회원 인증 체계 구축**
  - Java의 `MessageDigest`를 활용한 **SHA-256 단방향 암호화** 유틸리티(`PasswordUtil`)를 직접 구현하여, 사용자 비밀번호를 DB 저장 시점부터 철저하게 암호화 처리했습니다.
* **소프트 딜리트(Soft Delete) 메커니즘을 통한 탈퇴 처리**
  - 데이터를 물리적으로 삭제하지 않고 `status` 컬럼을 `ACTIVE` / `DEACTIVE` 상태로 관리하여 데이터 무결성을 유지했습니다.
  - 로그인 쿼리(`SELECT`)에 `status = 'ACTIVE'` 조건을 바인딩하여, **탈퇴 처리된 계정은 비밀번호가 일치하더라도 로그인이 절대 불가능하도록 백엔드 방어 로직**을 구축했습니다.
* **안전한 DB 커넥션 및 자원 관리**
  - 싱글톤으로 설계된 `DBConnection` 객체의 커넥션이 자동으로 닫히는 사이드 이펙트를 차단하기 위해, `UserDAO` 내에서 `try-with-resources` 문법을 정밀하게 제어(PreparedStatement와 ResultSet만 자동 해제)했습니다.
* **사용자 경험(UX) 중심의 커텀 UI 구현**
  - Swing의 기본 기능을 확장한 **`HintTextField`** 커스텀 클래스를 개발하여, 이메일/생년월일/전화번호 입력창에 웹 표준 스타일의 **반투명 플레이스홀더 힌트 텍스트** 효과를 적용했습니다.

---

## 4. 전체 서비스 흐름 및 실행 화면 (User Flow)

### 🔓 1단계: 인증 및 계정 관리 (Auth)
- 사용자는 `LoginFrame`을 통해 진입하며, 회원가입 시 웹 표준 플레이스홀더가 적용된 `RegisterFrame`으로 부드럽게 화면이 전환됩니다. 중복 아이디 검증 및 생년월일 날짜 포맷팅 검증(`LocalDate.parse`)이 선행됩니다.

| 로그인 화면 (`LoginFrame`) | 회원가입 화면 (`RegisterFrame`) |
| :---: | :---: |
| <img src="./images/01_login.png" width="350"> | <img src="./images/02_register.png" width="350"> |

<br>

### 🏦 2단계: 메인 대시보드 및 계좌 개설 (Dashboard)
- 인증 성공 시 진입하는 메인 대시보드입니다. 회원이 보유한 실시간 계좌 잔액 총합과 목록이 요약되어 나타나며, 즉시 가상 계좌를 추가 개설할 수 있습니다.

| 메인 대시보드 (`MainFrame`) | 계좌 개설 팝업 (`AccountDialog`) |
| :---: | :---: |
| <img src="./images/03_main.png" width="350"> | <img src="./images/04_create_account.png" width="350"> |

<br>

### 💸 3단계: 가상 뱅킹 및 송금 이력 (Banking)
- 상대방의 계좌번호와 이체 금액을 입력하여 안전하게 송금을 실행합니다. 이체 완료 즉시 출금/입금 처리가 원자적(Atomic)으로 수행되며 거래 내역 테이블에 이력이 실시간 적재됩니다.

| 송금/이체 실행 화면 | 실시간 거래 내역 조회 |
| :---: | :---: |
| <img src="./images/05_transfer.png" width="350"> | <img src="./images/06_history.png" width="350"> |

<br>

### ⚙️ 4단계: 마이페이지 및 회원 탈퇴 (My Page)
- 개인 정보를 수정하거나, 더 이상 서비스를 이용하지 않을 경우 회원 탈퇴를 진행합니다. 탈퇴 즉시 DB의 계정 상태가 `DEACTIVE`로 갱신되며 안전하게 세션이 파기됩니다.

| 마이페이지 정보 관리 | 회원 탈퇴 확인 다이얼로그 |
| :---: | :---: |
| <img src="./images/07_mypage.png" width="350"> | <img src="./images/08_withdraw.png" width="350"> |

---
