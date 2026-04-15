# AIssue — AI 생성 콘텐츠 신뢰도 검증 서비스

<p align="center">
  <img src="https://img.shields.io/badge/Java-21-007396?style=flat-square&logo=openjdk&logoColor=white"/>
  <img src="https://img.shields.io/badge/Spring_Boot-3.5.13-6DB33F?style=flat-square&logo=springboot&logoColor=white"/>
  <img src="https://img.shields.io/badge/Spring_Cloud-MSA-6DB33F?style=flat-square&logo=spring&logoColor=white"/>
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react&logoColor=black"/>
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white"/>
  <img src="https://img.shields.io/badge/MySQL-8.4-4479A1?style=flat-square&logo=mysql&logoColor=white"/>
  <img src="https://img.shields.io/badge/Redis-7.2-DC382D?style=flat-square&logo=redis&logoColor=white"/>
</p>

<p align="center">
  <b>See The Truth</b> — AI가 만든 콘텐츠인지, 사실인지 확인하세요
</p>

<br/>

## 📌 프로젝트 개요

**AIssue**는 2026년 1월 AI기본법 전면 시행에 따른 사회적 요구에 부응하여, 텍스트 기반의 **AI 생성 여부 탐지**와 **이슈 신뢰도 분석**을 제공하는 서비스입니다.

딥페이크·가짜 뉴스가 SNS를 통해 빠르게 확산되는 환경에서, 일반 시민과 중소 사업자가 콘텐츠의 진위를 손쉽게 검증할 수 있는 실질적인 수단을 제공하는 것을 목표로 합니다.

Spring Cloud 기반 MSA(Micro Service Architecture) 구조로 구현되어, 텍스트 분석·API 연동·신뢰도 점수 산출 3개의 독립 마이크로서비스로 구성됩니다.

<br/>

## 🔍 배경

### 사회적 배경

| 이슈 | 내용 |
|------|------|
| **AI기본법 시행** | 2026년 1월 22일, 세계 최초 전면 시행. AI 생성물 표시 의무화, 위반 시 3천만 원 이하 과태료 |
| **딥페이크·가짜 뉴스** | SNS 통해 허위정보 급속 확산, 딥페이크 관련 피해액 1조 6천억 원 규모 |
| **검증 인프라 부재** | AI 관련 온라인 언급 전년 대비 44% 증가에도 일반인이 활용 가능한 공개 검증 서비스 부재 |

### 기술적 필요성

AI기본법 제31조는 사업자에게 **AI 생성물 표시 의무**를 부과하고 있으나, 이를 실질적으로 뒷받침할 검증 도구가 시장에 부재합니다. AIssue는 이 기술 인프라 공백을 채우기 위한 프로젝트입니다.

<br/>

## 🖥️ 서비스 화면

| 메인페이지 | 콘텐츠 검색 | 신뢰도 분석 결과 |
|:---:|:---:|:---:|
| *See The Truth 랜딩 화면* | *텍스트 입력 / 파일 업로드* | *3단계 분석 플로우 및 점수* |

<br/>

## 🏗️ 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                    Client (React + Vite)                     │
│                     Netlify / Vercel                         │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTPS (Axios)
┌──────────────────────────▼──────────────────────────────────┐
│               Nginx (Reverse Proxy + SSL)                    │
│                    Let's Encrypt                             │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│              API Gateway (Spring Cloud Gateway)              │
│          라우팅 · 요청 크기 제한 · 공통 로깅 필터            │
└────────┬──────────────────┬──────────────────┬──────────────┘
         │                  │                  │
┌────────▼───────┐ ┌────────▼───────┐ ┌────────▼───────┐
│  텍스트 분석   │ │  이슈체크 연동  │ │  신뢰도 점수   │
│   서비스       │ │    서비스       │ │    서비스       │
│ /api/analyze   │ │ /api/issuecheck  │ │  /api/score    │
└────────────────┘ └────────┬───────┘ └────────────────┘
         ↕                  │ RestTemplate           ↕
┌────────────────────────────────────────────────────────────┐
│           Eureka Server (Service Registry)                  │
│       서비스 동적 등록 · 탐색 · 헬스 체크                   │
└────────────────────────────────────────────────────────────┘
         ↕                  ↕
┌──────────────┐   ┌──────────────────────────────────────────┐
│    MySQL     │   │  External APIs                           │
│  (DB per     │   │  - 빅카인즈(BigKinds) API (언론진흥재단) │
│   Service)   │   │  - 한국언론진흥재단 뉴스빅데이터 API     │
└──────────────┘   └──────────────────────────────────────────┘
         ↕
┌──────────────┐
│    Redis     │
│  Fallback    │
│  Cache + TTL │
└──────────────┘
```

<br/>

## ⚙️ 핵심 마이크로서비스

### ① 텍스트 분석 서비스 (`analyze-service`)

입력된 텍스트를 4가지 언어 통계 지표로 분석하여 AI 생성 확률(0~100%)을 산출합니다.

| 분석 지표 | 측정 방법 | AI 생성 특징 |
|-----------|-----------|-------------|
| 문장 길이 분포 | 전체 문장의 평균·표준편차 계산 | 문장 길이가 지나치게 균일한 경향 |
| 어휘 다양성 (TTR) | 고유 단어 수 ÷ 전체 단어 수 | 높은 TTR, 어휘 반복이 적음 |
| 접속사·전환어 빈도 | '또한', '따라서', '결론적으로' 등 빈도 측정 | 구조적 전환어를 과도하게 사용 |
| 문장 구조 반복도 | 주어-서술어 패턴 반복 비율 | 동일 문장 패턴이 반복되는 경향 |

- **기술**: Spring Boot + 통계 분석
- **엔드포인트**: `POST /api/analyze`
- **외부 의존 없음**: 자체 로직만으로 독립 동작

<br/>

### ② 이슈체크 연동 서비스 (`issuecheck-service`)

입력 텍스트에서 핵심 키워드를 추출 후 공공 데이터와 연동하여 관련 보도 및 출처 신뢰도를 조회합니다.

```
입력 텍스트
    → 형태소 분석 기반 핵심 키워드 추출
    → 빅카인즈(BigKinds) API 조회 (언론진흥재단)
        - 관련 보도 목록 수집
        - 언론사 출처 신뢰도 참조
    → Circuit Breaker: API 장애 시 Fallback (Redis 캐시 반환)
```

- **기술**: Spring Boot + RestTemplate + Resilience4j
- **엔드포인트**: `POST /api/Issuecheck`
- **Circuit Breaker**: 실패율 50% 초과 시 OPEN → Fallback 반환

<br/>

### ③ 신뢰도 점수 서비스 (`score-service`)

텍스트 분석과 팩트체크 두 서비스의 결과를 FeignClient로 집계하여 최종 신뢰도 점수(0~100)를 산출합니다.

**가중치 기준**

| 구성 요소 | 가중치 | 설명 |
|-----------|--------|------|
| AI 생성 확률 (역산) | 40% | AI 생성 확률이 높을수록 신뢰도 감점 |
| 이슈 출처 신뢰도 | 40% | 인용 사이트 신뢰도 기반 점수 반영 |
| 관련 보도 수 | 20% | 관련 신뢰 보도가 많을수록 가점 |

**신뢰도 등급**

| 점수 | 등급 | 의미 |
|------|------|------|
| 70 ~ 100 | 🟢 신뢰 | 사실일 가능성이 높고 AI 생성 흔적 미약 |
| 40 ~ 69 | 🟡 주의 | 일부 허위 가능성 또는 AI 생성 흔적 존재 |
| 0 ~ 39 | 🔴 위험 | 허위정보 가능성 높거나 AI 생성 강하게 의심 |

<br/>

## 🔒 Circuit Breaker 동작 흐름

```
외부 API 호출 시작
       │
   ┌───▼───┐    실패율 < 50%     ┌──────────┐
   │CLOSED │ ─────────────────▶ │  정상 응답 │
   └───┬───┘                    └──────────┘
       │ 실패율 ≥ 50% (최근 10회)
   ┌───▼───┐                    ┌──────────────────────┐
   │ OPEN  │ ─────────────────▶ │ Fallback 즉시 반환    │
   └───┬───┘                    │ (Redis 캐시 데이터)   │
       │ 60초 경과               └──────────────────────┘
   ┌───▼─────┐  성공            ┌──────────┐
   │HALF-OPEN│ ───────────────▶ │  CLOSED  │
   └─────────┘                  └──────────┘
       │ 실패
   ┌───▼───┐
   │ OPEN  │
   └───────┘
```

<br/>

## 🛠️ 기술 스택

### Backend / MSA

| 구분 | 기술 |
|------|------|
| 언어 | Java |
| 프레임워크 | Spring Boot |
| 서비스 등록·탐색 | Spring Cloud Eureka |
| API 게이트웨이 | Spring Cloud Gateway |
| 서비스 간 통신 | Spring Cloud OpenFeign |
| 부하 분산 | Spring Cloud LoadBalancer |
| 장애 처리 | Resilience4j |
| ORM | Spring Data JPA |
| 빌드 | Gradle |

### Database / Cache

| 구분 | 기술 | 용도 |
|------|------|------|
| RDBMS | MySQL 8.x | 서비스별 독립 DB (DB per Service) |
| Cache | Redis 7.x | Fallback 캐시 + 검증 이력 TTL 관리 |
| Test DB | H2 | 단위·통합 테스트용 인메모리 DB |

### Frontend

| 구분 | 기술 |
|------|------|
| 프레임워크 | React 18 + Vite |
| 스타일링 | Tailwind CSS |
| 상태 관리 | Zustand |
| API 통신 | Axios |
| 배포 | Netlify |

### Infra / DevOps

| 구분 | 기술 |
|------|------|
| 클라우드 | AWS EC2 + AWS S3 |
| 컨테이너 | Docker + Docker Compose |
| 이미지 저장소 | Docker Hub |
| CI/CD | GitHub Actions |
| 리버스 프록시 | Nginx + Let's Encrypt |

<br/>

## 📋 주요 기능

- **AI 생성 텍스트 탐지**: 입력 텍스트의 AI 생성 여부를 분석하여 탐지 확률 점수(0~100%) 반환
- **뉴스 연관 검색**: 빅카인즈 API를 통해 관련 이슈 목록 및 언론사 출처 신뢰도 조회
- **신뢰도 점수 산출**: AI 탐지·뉴스 출처·보도량을 종합한 최종 신뢰도 점수 계산
- **검증 이력 조회**: 이전에 검증한 콘텐츠 이력 저장 및 동일 콘텐츠 재요청 시 캐시 반환
- **Fallback 처리**: 외부 API 장애 시 Circuit Breaker가 자동 전환되어 서비스 중단 없이 AI 탐지 결과만 단독 제공

<br/>

## 📁 프로젝트 구조

```
aissue/
├── eureka-server/                  # 서비스 등록·탐색 서버
│   └── src/main/java/
│       └── EurekaServerApplication.java
│
├── api-gateway/                    # API 게이트웨이
│   └── src/main/resources/
│       └── application.yml         # 라우팅 규칙 정의
│
├── analyze-service/                # 텍스트 분석 마이크로서비스
│   └── src/main/java/
│       ├── controller/
│       ├── service/
│       │   └── TextAnalyzeService.java
│       └── dto/
│
├── issuecheck-service/             # 이슈체크 연동 마이크로서비스
│   └── src/main/java/
│       ├── controller/
│       ├── service/
│       │   └── IssueCheckService.java
│       ├── client/
│       │   └── BigKindsApiClient.java
│       └── fallback/
│           └── IssueCheckFallback.java
│
├── score-service/                  # 신뢰도 점수 산출 마이크로서비스
│   └── src/main/java/
│       ├── controller/
│       ├── service/
│       │   └── ScoreService.java
│       └── client/
│           ├── AnalyzeServiceClient.java
│           └── IssueCheckServiceClient.java
│
├── frontend/                       # React + Vite 프론트엔드
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   └── store/                  # Zustand 상태 관리
│   └── package.json
│
├── docker-compose.yml              # 로컬 전체 스택 통합 실행
└── .github/
    └── workflows/
        └── ci-cd.yml               # GitHub Actions CI/CD
```

<br/>

## 🚀 서비스 관련

### 서비스별 포트

| 서비스 | 포트 |
|--------|------|
| API Gateway | 8080 |
| Eureka Server | 8761 |
| 텍스트 분석 서비스 | 8081 |
| 신뢰도 점수 서비스 | 8082 |
| API 연동 서비스 | 8083 |
| MySQL | 3306 |
| Redis | 6379 |

### Eureka Dashboard

서비스 기동 후 `http://localhost:8761` 에서 등록된 서비스 인스턴스를 확인할 수 있습니다.

<br/>

## 📡 API 명세

> 전체 API 명세는 각 서비스 기동 후 Swagger UI에서 확인 가능합니다.
> - 텍스트 분석: `http://localhost:8081/swagger-ui.html`
> - 팩트체크: `http://localhost:8082/swagger-ui.html`
> - 신뢰도 점수: `http://localhost:8083/swagger-ui.html`

### 주요 엔드포인트

#### `POST /api/score` — 신뢰도 점수 산출 (통합 분석)

```json
// Request
{
  "text": "분석할 텍스트 내용을 입력합니다."
}

// Response
{
  "score": 42,
  "grade": "주의",
  "aiDetectRate": 78.5,
  "relatedNews": [
    {
      "title": "관련 기사 제목",
      "source": "언론사명",
      "publishedAt": "2026-04-10",
      "url": "https://..."
    }
  ],
  "fallback": false
}
```

#### `POST /api/analyze` — AI 생성 여부 탐지

```json
// Request
{
  "text": "분석할 텍스트"
}

// Response
{
  "aiDetectRate": 78.5,
  "details": {
    "sentenceLengthScore": 82.0,
    "ttrScore": 75.0,
    "conjunctionScore": 88.0,
    "structureRepeatScore": 69.0
  }
}
```

#### `POST /api/issuecheck` — 이슈 출처 신뢰도 조회

```json
// Request
{
  "text": "키워드를 추출할 텍스트"
}

// Response
{
  "keywords": ["인공지능", "규제"],
  "relatedNews": [...],
  "sourceReliabilityScore": 68.0,
  "fallback": false
}
```

<br/>

## 🔄 CI/CD 파이프라인

```
Push to main
    └─▶ GitHub Actions
            ├─▶ Java 테스트 (JUnit 5)
            ├─▶ Docker 이미지 빌드
            ├─▶ Docker Hub Push
            └─▶ AWS EC2 배포 (docker-compose pull & up)
```

<br/>

## 📅 개발 일정

| 기간 | 구분 | 세부 내용 |
|------|------|-----------|
| 1주차 | 환경 구성 | 멀티 모듈 프로젝트, Eureka Server 기동 확인 |
| 2주차 | 공공 API 연동 | 빅카인즈 API 수집 및 파싱 구현 |
| 3주차 | 텍스트 분석 서비스 | AI 탐지 로직 및 조회 API · UI 개발 |
| 4주차 | 신뢰도 점수 서비스 | FeignClient 통신 구현 및 UI 연동 |
| 5~6주차 | API G/W + Circuit Breaker | 라우팅, Fallback 로직, UI 마무리 |
| 7~9주차 | 고도화 + 통합 테스트 | 장애 시나리오 테스트 |
| 10주차 | 문서화 + 마무리 | API 명세, 발표 자료 |

> **수행 기간**: 2026. 4. 10. ~ 2026. 6. 16.

<br/>

## 👥 팀원

| 이름 | 역할 | GitHub |
|------|------|--------|
| 윤희준 | 아키텍처 설계, Eureka / API Gateway / 텍스트 분석 서비스 / Circuit Breaker / 팩트체크 연동 / 신뢰도 점수 서비스 | [@uni-j-uni](https://github.com/uni-j-uni) |
| 장근호 | UI/UX 설계 / 디자인 | [@EpoHopE](https://github.com/EpoHopE) |
