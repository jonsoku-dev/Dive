# Dive 프로젝트 구조 상세 분석

이 문서는 Dive 프로젝트의 전체 디렉토리 구조와 주요 파일들의 역할을 자세히 설명합니다. 각 디렉토리의 목적과 커스터마이징 시 고려해야 할 부분에 중점을 두고 있습니다.

## 목차

- [최상위 디렉토리 구조](#최상위-디렉토리-구조)
- [전체 구조 다이어그램](#전체-구조-다이어그램)
- [디렉토리별 상세 분석](#디렉토리별-상세-분석)
  - [electron 디렉토리](#electron-디렉토리)
  - [services 디렉토리](#services-디렉토리)
  - [src 디렉토리](#src-디렉토리)
  - [public 디렉토리](#public-디렉토리)
  - [기타 설정 파일들](#기타-설정-파일들)
- [커스터마이징을 위한 주요 파일 및 디렉토리](#커스터마이징을-위한-주요-파일-및-디렉토리)

## 최상위 디렉토리 구조

```
/
├── build/                  # 빌드 리소스 (아이콘 등)
├── dist-electron/          # 빌드된 Electron 앱 (gitignore)
├── docs/                   # 문서 파일
├── drizzle/                # 데이터베이스 마이그레이션
├── electron/               # Electron 애플리케이션 소스 코드
├── node_modules/           # NPM 의존성 (gitignore)
├── patches/                # 패키지 패치 파일
├── prebuilt/               # 사전 빌드된 리소스
├── public/                 # 정적 파일 (아이콘, 번역 등)
├── resources/              # 빌드 리소스
├── scripts/                # 빌드 및 설치 스크립트
├── services/               # 백엔드 서비스 코드
├── src/                    # React 프론트엔드 코드
├── .dockerignore           # Docker 빌드 제외 파일
├── .env.example            # 환경 변수 예제
├── .gitignore              # Git 제외 파일
├── BUILD.md                # 빌드 지침
├── LICENSE                 # 라이선스 파일
├── README.md               # 프로젝트 설명
├── drizzle.config.ts       # Drizzle ORM 설정
├── electron-builder.json   # Electron 빌더 설정
├── eslint.config.js        # ESLint 설정
├── index.html              # 메인 HTML 파일
├── package-lock.json       # NPM 의존성 잠금 파일
├── package.json            # 프로젝트 메타데이터 및 의존성
├── tsconfig.app.json       # TypeScript 설정 (애플리케이션)
├── tsconfig.json           # TypeScript 기본 설정
├── tsconfig.node.json      # TypeScript 설정 (Node.js)
├── vite.config.electron.ts # Vite 설정 (Electron)
├── vite.config.service.ts  # Vite 설정 (서비스)
├── vite.config.ts          # Vite 기본 설정
└── vitest.config.ts        # Vitest 테스트 설정
```

## 전체 구조 다이어그램

```
Dive
│
├── 프론트엔드 (src/)
│   ├── 컴포넌트 (components/)
│   ├── 페이지 뷰 (views/)
│   ├── 상태 관리 (atoms/)
│   └── 스타일 (styles/)
│
├── 백엔드 서비스 (services/)
│   ├── MCP 서버 관리 (mcpServer/)
│   ├── 모델 관리 (models/)
│   ├── 데이터베이스 (database/)
│   ├── 라우트 (routes/)
│   └── 유틸리티 (utils/)
│
└── Electron 코어 (electron/)
    ├── 메인 프로세스 (main/)
    ├── 프리로드 스크립트 (preload/)
    └── 설정 (config/)
```

## 디렉토리별 상세 분석

### electron 디렉토리

Electron 애플리케이션의 코어를 담당하는 부분입니다.

```
electron/
├── config/                # 환경 설정
│   ├── index.ts          # 설정 통합
│   └── keymap.ts         # 키보드 단축키 매핑
├── main/                 # 메인 프로세스
│   ├── index.ts          # 진입점
│   ├── service.ts        # 서비스 초기화
│   ├── state.ts          # 애플리케이션 상태
│   ├── store.ts          # 설정 저장
│   ├── tray.ts           # 시스템 트레이
│   ├── update.ts         # 자동 업데이트
│   ├── util.ts           # 유틸리티 함수
│   ├── ipc/              # IPC 통신
│   └── platform/         # 플랫폼별 기능
└── preload/              # 프리로드 스크립트
    └── index.ts          # 프리로드 진입점
```

**주요 파일**:
- `electron/main/index.ts`: Electron 애플리케이션의 주 진입점
- `electron/main/service.ts`: MCP 클라이언트 및 서비스 초기화
- `electron/main/ipc/`: 렌더러 프로세스와의 통신 처리

**커스터마이징 포인트**:
- 시스템 통합 및 네이티브 기능은 `electron/main/` 디렉토리에서 확장
- 플랫폼별 특정 기능은 `electron/main/platform/` 디렉토리에 추가
- 주요 설정 및 단축키는 `electron/config/` 디렉토리에서 조정

### services 디렉토리

백엔드 서비스 로직, 모델 관리, 데이터베이스 상호작용을 담당합니다.

```
services/
├── database/             # 데이터베이스 관리
│   ├── index.ts          # DB 연결 및 쿼리
│   └── schema.ts         # 스키마 정의
├── mcpServer/            # MCP 서버 관리
│   ├── index.ts          # MCP 서버 매니저
│   └── interface.ts      # 인터페이스 정의
├── models/               # LLM 모델 관리
│   ├── index.ts          # 모델 매니저
│   └── interface.ts      # 인터페이스 정의
├── prompt/               # 프롬프트 관리
│   ├── index.ts          # 프롬프트 매니저
│   └── system.ts         # 시스템 프롬프트
├── routes/               # API 라우트
│   ├── chat.ts           # 채팅 API
│   ├── config.ts         # 설정 API
│   ├── tools.ts          # 도구 API
│   └── ...               # 기타 API
├── syscmd/               # 시스템 명령
│   └── index.ts          # 시스템 명령 관리
├── utils/                # 유틸리티
│   ├── fileHandler.ts    # 파일 처리
│   ├── image.ts          # 이미지 처리
│   ├── logger.ts         # 로깅
│   ├── modelHandler.ts   # 모델 처리
│   ├── processHistory.ts # 히스토리 처리
│   ├── toolHandler.ts    # 도구 처리
│   └── types.ts          # 타입 정의
├── client.ts             # MCP 클라이언트
├── connectServer.ts      # 서버 연결
├── index.ts              # 모듈 내보내기
├── main.ts               # 서비스 진입점
├── processQuery.ts       # 쿼리 처리
└── webServer.ts          # 웹 서버
```

**주요 파일**:
- `services/main.ts`: 백엔드 서비스의 진입점
- `services/client.ts`: MCP 클라이언트 구현
- `services/processQuery.ts`: LLM 쿼리 처리 및 응답 스트리밍
- `services/mcpServer/index.ts`: MCP 서버 연결 및 관리
- `services/models/index.ts`: LLM 모델 로딩 및 관리

**커스터마이징 포인트**:
- 커스텀 MCP 도구는 `services/mcpServer/` 확장
- 내부 모델 및 API 통합은 `services/models/` 확장
- 새로운 API 엔드포인트는 `services/routes/` 추가
- 데이터 스키마 확장은 `services/database/schema.ts` 수정

### src 디렉토리

React 프론트엔드 애플리케이션을 담당합니다.

```
src/
├── assets/               # 정적 에셋
├── atoms/                # Jotai 상태
│   ├── chatState.ts      # 채팅 상태
│   ├── configState.ts    # 설정 상태
│   ├── modalState.ts     # 모달 상태
│   └── ...               # 기타 상태
├── components/           # 재사용 컴포넌트
│   ├── Header.tsx        # 헤더
│   ├── ModelSelect.tsx   # 모델 선택
│   ├── ToolPanel.tsx     # 도구 패널
│   └── ...               # 기타 컴포넌트
├── hooks/                # 커스텀 훅
├── styles/               # SCSS 스타일
│   ├── components/       # 컴포넌트 스타일
│   ├── overlay/          # 오버레이 스타일
│   ├── pages/            # 페이지 스타일
│   └── ...               # 기타 스타일
├── views/                # 페이지 뷰
│   ├── Chat/             # 채팅 화면
│   ├── Overlay/          # 오버레이 화면
│   ├── Setup/            # 설정 화면
│   └── ...               # 기타 화면
├── App.tsx               # 루트 컴포넌트
├── constants.ts          # 상수
├── i18n.ts               # 다국어 설정
├── main.tsx              # React 진입점
├── router.tsx            # 라우팅 설정
└── updater.tsx           # 업데이트 관리
```

**주요 파일**:
- `src/main.tsx`: React 애플리케이션 진입점
- `src/App.tsx`: 루트 컴포넌트
- `src/router.tsx`: 라우팅 설정
- `src/views/Chat/index.tsx`: 메인 채팅 인터페이스
- `src/views/Overlay/Model/index.tsx`: 모델 설정 인터페이스

**커스터마이징 포인트**:
- UI 컴포넌트는 `src/components/` 확장
- 상태 관리는 `src/atoms/` 확장
- 스타일링은 `src/styles/` 수정
- 새로운 페이지는 `src/views/` 추가

### public 디렉토리

정적 파일 및 번역 리소스를 담당합니다.

```
public/
├── image/                # 이미지 리소스
│   ├── model_*.svg       # 모델 아이콘
│   └── ...               # 기타 이미지
├── linux/                # Linux 특정 파일
├── locales/              # 다국어 리소스
│   ├── en/               # 영어
│   ├── es/               # 스페인어
│   ├── ja/               # 일본어
│   ├── zh-CN/            # 중국어(간체)
│   └── zh-TW/            # 중국어(번체)
├── icon.ico              # 윈도우 아이콘
├── icon.png              # 기본 아이콘
└── vite.svg              # Vite 로고
```

**주요 파일**:
- `public/locales/*/translation.json`: 각 언어별 번역 문자열
- `public/image/`: 애플리케이션에서 사용하는 아이콘 및 이미지

**커스터마이징 포인트**:
- `public/image/`: 브랜드 이미지 및 아이콘 교체
- `public/locales/`: 다국어 리소스 추가 및 수정

### 기타 설정 파일들

```
├── drizzle.config.ts       # Drizzle ORM 설정
├── electron-builder.json   # Electron 빌더 설정
├── eslint.config.js        # ESLint 설정
├── package.json            # 프로젝트 메타데이터 및 의존성
├── tsconfig.*.json         # TypeScript 설정
└── vite.config.*.ts        # Vite 빌드 설정
```

**주요 파일**:
- `package.json`: 프로젝트 의존성 및 스크립트
- `electron-builder.json`: 애플리케이션 패키징 설정
- `vite.config.*.ts`: 빌드 구성

**커스터마이징 포인트**:
- `package.json`: 의존성 추가 및 스크립트 수정
- `electron-builder.json`: 배포 설정 변경
- `vite.config.*.ts`: 빌드 프로세스 최적화

## 커스터마이징을 위한 주요 파일 및 디렉토리

### UI 커스터마이징
- 테마 및 브랜딩: `/src/styles/_variables.scss`
- 컴포넌트 스타일: `/src/styles/components/`
- 아이콘 및 이미지: `/public/image/`
- 헤더 및 네비게이션: `/src/components/Header.tsx`

### 기능 커스터마이징
- 모델 통합: `/services/models/index.ts`
- MCP 도구: `/services/mcpServer/index.ts`
- API 엔드포인트: `/services/routes/`
- 프롬프트 템플릿: `/services/prompt/system.ts`

### 데이터 커스터마이징
- 데이터 스키마: `/services/database/schema.ts`
- 마이그레이션: `/drizzle/`
- 데이터 처리: `/services/database/index.ts`

### 시스템 통합 커스터마이징
- Electron 통합: `/electron/main/`
- IPC 통신: `/electron/main/ipc/`
- 플랫폼별 기능: `/electron/main/platform/`

### 국제화 및 현지화
- 번역 파일: `/public/locales/`
- i18n 설정: `/src/i18n.ts`
