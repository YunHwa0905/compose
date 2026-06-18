# Docker Compose Full-Stack Study

React · Spring Boot · MySQL · Nginx를 Docker Compose로 단일 명령어에 오케스트레이션하는 풀스택 실습 저장소입니다.

---

## 기술 스택

![Java](https://img.shields.io/badge/Java_17-007396?style=flat&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.3-6DB33F?style=flat&logo=springboot&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat&logo=mysql&logoColor=white)
![JPA](https://img.shields.io/badge/JPA/Hibernate-59666C?style=flat&logo=hibernate&logoColor=white)
![Lombok](https://img.shields.io/badge/Lombok-BC4521?style=flat&logoColor=white)
![React](https://img.shields.io/badge/React_18-61DAFB?style=flat&logo=react&logoColor=black)
![Node.js](https://img.shields.io/badge/Node.js_22-339933?style=flat&logo=nodedotjs&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat&logo=nginx&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=flat&logo=docker&logoColor=white)
![Gradle](https://img.shields.io/badge/Gradle-02303A?style=flat&logo=gradle&logoColor=white)

---

## 프로젝트 구조

```
compose/
├── backend/                    # Spring Boot 3.3 (Java 17)
│   ├── src/main/java/          # REST API — User 엔티티 CRUD
│   ├── src/main/resources/
│   │   ├── application.yml     # DB 접속 설정 (환경변수 주입 지원)
│   │   └── data.sql            # 초기 데이터 시드
│   ├── Dockerfile              # 멀티스테이지 빌드 (Gradle → JDK 17 slim)
│   └── build.gradle
│
├── frontend/                   # React 18 (CRA)
│   ├── src/
│   │   └── App.js              # /api 프록시를 통한 백엔드 fetch 예시
│   ├── Dockerfile              # 멀티스테이지 빌드 (Node 22 → nginx:alpine)
│   └── package.json
│
├── nginx/
│   └── default.conf            # 리버스 프록시 — / → frontend, /api → backend
│
└── docker-compose.yml          # 4개 서비스 오케스트레이션
```

---

## 아키텍처

```
브라우저
  │
  ▼ :80
┌──────────────────┐
│  nginx (proxy)   │  all_dir_network
│  /    → frontend │─────────────────────────────────────┐
│  /api → server   │───────────────────┐                 │
└──────────────────┘                   │                 │
                               ┌───────▼──────┐  ┌──────▼──────┐
                               │   backend    │  │  frontend   │
                               │  :8080 (SB)  │  │  :80 (nginx)│
                               └───────┬──────┘  └─────────────┘
                                       │ JPA / MySQL
                               ┌───────▼──────┐
                               │   database   │
                               │  :3306 (MySQL│
                               └──────────────┘
```

---

## 실행 방법

### 사전 준비

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) 설치 및 실행

### 전체 서비스 기동

```bash
# 기존 컨테이너·이미지 정리 (처음 실행 시 생략 가능)
docker compose down

# 이미지 빌드 + 컨테이너 생성 + 백그라운드 기동
docker compose up -d
```

브라우저에서 `http://localhost` 로 접속합니다.  
백엔드 REST API는 `http://localhost/api/test/home` 으로 확인할 수 있습니다.

### 개별 서비스 로그 확인

```bash
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f nginx
```

### 서비스 종료

```bash
docker compose down
```

> DB 데이터는 `mysql_data` 볼륨에 보존됩니다.  
> 볼륨까지 삭제하려면 `docker compose down -v` 를 사용하세요.

---

## 학습 내용

| 파일 / 폴더 | 주요 학습 내용 |
|---|---|
| `docker-compose.yml` | 멀티 서비스 정의, 커스텀 네트워크·볼륨, `depends_on`, 환경변수 주입 |
| `backend/Dockerfile` | 멀티스테이지 빌드 (빌드 이미지 → 경량 런타임 이미지 분리) |
| `frontend/Dockerfile` | Node.js 빌드 → nginx:alpine 정적 파일 서빙 멀티스테이지 구성 |
| `nginx/default.conf` | `location /api` 리버스 프록시, 컨테이너 도메인명 라우팅 |
| `backend/src/` | Spring Boot REST API, JPA 엔티티, `@RestController`, Lombok |
| `frontend/src/` | React `useEffect` fetch, Nginx 프록시를 통한 CORS-free API 호출 |

---

## 개발 환경

- **Docker Desktop** — [다운로드](https://www.docker.com/products/docker-desktop/)
- **IntelliJ IDEA** — 백엔드 소스 편집 (선택)
- **VS Code** — 프론트엔드 소스 편집 (선택)

### 로컬 DB 직접 접속 (옵션)

`docker compose up -d` 실행 후 아래 정보로 접속합니다.

| 항목 | 값 |
|---|---|
| Host | `127.0.0.1` |
| Port | `3306` |
| Database | `todo` |
| User | `root` |
| Password | `p1234` |

> **백엔드를 IntelliJ에서 직접 실행할 경우**  
> `application.yml`의 `url` 주석을 `127.0.0.1:3306`으로 전환하세요.
