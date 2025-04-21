# Dive 핵심 컴포넌트 상세 분석

이 문서는 Dive 프로젝트의 핵심 컴포넌트들의 역할, 구조, 동작 원리, 그리고 커스터마이징 방법을 상세히 설명합니다. 각 컴포넌트는 특정 책임을 담당하며, 전체 시스템에서 중요한 역할을 수행합니다.

## 목차

- [MCP 서버 관리자 (MCPServerManager)](#mcp-서버-관리자-mcpservermanager)
- [모델 관리자 (ModelManager)](#모델-관리자-modelmanager)
- [쿼리 처리 (processQuery)](#쿼리-처리-processquery)
- [데이터베이스 관리](#데이터베이스-관리)
- [웹 애플리케이션 UI](#웹-애플리케이션-ui)
- [각 컴포넌트 간 상호작용](#각-컴포넌트-간-상호작용)

## MCP 서버 관리자 (MCPServerManager)

### 개요

MCPServerManager는 Model Context Protocol 서버들과의 연결을 관리하고, 도구 호출을 처리하는 핵심 컴포넌트입니다. 이 컴포넌트는 다양한 전송 방식(stdio, SSE, WebSocket)을 지원하며, 서버 구성을 동적으로 업데이트할 수 있는 기능을 포함합니다.

### 주요 파일
- `/services/mcpServer/index.ts`: 핵심 구현체
- `/services/mcpServer/interface.ts`: 인터페이스 정의
- `/services/connectServer.ts`: 서버 연결 로직

### 클래스 다이어그램

```
┌───────────────────┐
│  MCPServerManager │
├───────────────────┤
│ - servers: Map    │
│ - transports: Map │
│ - toolToServerMap │
│ - availableTools  │
│ - toolInfos       │
├───────────────────┤
│ + getInstance()   │
│ + initialize()    │
│ + connectAllServers()  │
│ + connectSingleServer()│
│ + syncServersWithConfig()│
│ + disconnectAllServers()│
└───────────────────┘
```

### 핵심 메서드

#### `getInstance()`
싱글톤 패턴을 사용하여 MCPServerManager의 인스턴스를 가져옵니다.

#### `initialize()`
서버 관리자를 초기화하고 설정을 로드합니다.

#### `connectAllServers()`
구성 파일에서 정의된 모든 서버에 연결합니다.

#### `connectSingleServer()`
특정 서버에 연결하고 사용 가능한 도구를 로드합니다.

#### `syncServersWithConfig()`
현재 실행 중인 서버를 구성 파일과 동기화합니다.

#### `disconnectSingleServer()`
특정 서버와의 연결을 끊습니다.

#### `disconnectAllServers()`
모든 서버와의 연결을 끊습니다.

### 커스터마이징 방법

1. **커스텀 도구 서버 통합**:
   ```typescript
   // config.json에 커스텀 서버 추가
   {
     "mcpServers": {
       "custom-tool": {
         "command": "npx",
         "args": ["-y", "my-custom-mcp-server"],
         "enabled": true
       }
     }
   }
   ```

2. **서버 연결 확장**:
   `/services/mcpServer/index.ts` 파일을 확장하여 추가 전송 방식이나 인증 메커니즘을 지원할 수 있습니다.

3. **도구 처리 커스터마이징**:
   `/services/utils/toolHandler.ts` 파일에서 도구 변환 및 처리 로직을 확장할 수 있습니다.

## 모델 관리자 (ModelManager)

### 개요

ModelManager는 다양한 LLM 제공업체 및 모델 설정을 관리하는 컴포넌트입니다. LangChain과 통합하여 일관된 인터페이스를 제공하며, 모델 구성의 로드, 저장 및 갱신 기능을 담당합니다.

### 주요 파일
- `/services/models/index.ts`: 핵심 구현체
- `/services/models/interface.ts`: 인터페이스 정의

### 클래스 다이어그램

```
┌───────────────┐
│ ModelManager  │
├───────────────┤
│ - model       │
│ - cleanModel  │
│ - configPath  │
│ - settings    │
│ - enableTools │
├───────────────┤
│ + getInstance()   │
│ + initializeModel() │
│ + getModel()      │
│ + saveModelConfig() │
│ + generateTitle() │
│ + reloadModel()   │
└───────────────┘
```

### 핵심 메서드

#### `getInstance()`
싱글톤 패턴을 사용하여 ModelManager의 인스턴스를 가져옵니다.

#### `initializeModel()`
설정에서 모델을 초기화합니다.

#### `getModelConfig()`
모델 설정을 로드합니다.

#### `saveModelConfig()`
모델 설정을 저장합니다.

#### `generateTitle()`
사용자 입력을 기반으로 대화 제목을 생성합니다.

#### `reloadModel()`
현재 모델을 다시 로드합니다.

### 커스터마이징 방법

1. **새 모델 제공업체 추가**:
   `/services/models/index.ts` 파일에서 LANGCHAIN_SUPPORTED_PROVIDERS 및 MCP_SUPPORTED_PROVIDERS 상수를 확장합니다.

2. **모델 설정 커스터마이징**:
   기본 모델 설정을 변경하거나 기업 특화 모델을 추가할 수 있습니다.
   ```typescript
   // 예: 모델 매니저에 내부 모델 통합 추가
   async initializeModel() {
     // 기존 초기화 로직...
     
     // 내부 모델 통합 추가
     if (modelSettings.modelProvider === "internal") {
       this.model = new InternalModel(modelSettings);
       this.cleanModel = new InternalModel(modelSettings);
       this.currentModelSettings = modelSettings;
       return this.model;
     }
     
     // 기존 로직 계속...
   }
   ```

3. **타이틀 생성 로직 수정**:
   `generateTitle()` 메서드를 수정하여 대화 제목 생성 방식을 커스터마이징할 수 있습니다.

## 쿼리 처리 (processQuery)

### 개요

processQuery 모듈은 사용자 쿼리를 AI에 전달하고 스트리밍 응답을 처리하는 핵심 로직입니다. 도구 호출 및 결과 처리를 위한 반복 로직과 오류 처리 및 중단 메커니즘이 포함되어 있습니다.

### 주요 파일
- `/services/processQuery.ts`: 핵심 구현체

### 함수 다이어그램

```
┌─────────────────────┐
│ handleProcessQuery  │
├─────────────────────┤
│ 1. 초기 설정        │
│ 2. 입력 처리        │
│ 3. LLM 스트리밍     │
│ 4. 도구 호출 처리   │
│ 5. 결과 수집        │
│ 6. 토큰 계산        │
└─────────────────────┘
```

### 핵심 로직

1. **초기 설정**:
   - 중단 컨트롤러(AbortController) 설정
   - 토큰 사용량 추적 초기화

2. **입력 처리**:
   - 텍스트 및 이미지 입력 처리
   - 메시지 형식 변환

3. **LLM 스트리밍**:
   - 모델에 메시지 스트리밍
   - 청크 수집 및 처리

4. **도구 호출 처리**:
   - 도구 호출 감지 및 파싱
   - 적절한 MCP 서버로 도구 호출 라우팅
   - 결과 수집 및 처리

5. **반복 처리**:
   - 도구 결과를 다시 LLM에 전달
   - 최종 응답 생성까지 반복

6. **토큰 사용량 계산**:
   - 입력 및 출력 토큰 추적
   - 총 사용량 계산

### 커스터마이징 방법

1. **도구 호출 커스터마이징**:
   도구 호출 처리 로직을 수정하여 특수 도구 유형이나 내부 시스템과의 통합을 구현할 수 있습니다.

2. **오류 처리 강화**:
   오류 처리 및 재시도 로직을 추가하여 안정성을 개선할 수 있습니다.

3. **응답 후처리**:
   LLM 응답에 대한 후처리 로직을 추가하여 콘텐츠 필터링, 형식 지정 또는 보강을 구현할 수 있습니다.

4. **스트리밍 최적화**:
   스트리밍 처리 로직을 최적화하여 성능을 개선하거나 대용량 응답 처리를 지원할 수 있습니다.

## 데이터베이스 관리

### 개요

데이터베이스 관리 컴포넌트는 채팅 및 메시지 저장소를 담당합니다. Drizzle ORM을 사용한 스키마 및 마이그레이션 관리가 포함되어 있습니다.

### 주요 파일
- `/services/database/index.ts`: 데이터베이스 연결 및 CRUD 작업
- `/services/database/schema.ts`: 데이터베이스 스키마 정의
- `/drizzle/`: 마이그레이션 파일

### 핵심 기능

#### 데이터베이스 초기화
```typescript
// DatabaseMode 열거형
export enum DatabaseMode {
  DIRECT = "direct",
  API = "api",
}

// 데이터베이스 초기화 함수
export async function initDatabase(
  mode: DatabaseMode = DatabaseMode.DIRECT,
  options?: {
    dbPath?: string;
    apiUrl?: string;
  }
) {
  // 데이터베이스 초기화 로직...
}
```

#### 채팅 작업
- `createChat`: 새 채팅 생성
- `getChatWithMessages`: 채팅과 관련 메시지 가져오기
- `updateChatTitle`: 채팅 제목 업데이트
- `deleteChat`: 채팅 삭제

#### 메시지 작업
- `createMessage`: 새 메시지 생성
- `deleteMessagesAfter`: 특정 메시지 이후의 메시지 삭제

### 스키마 설계

```typescript
// 채팅 테이블
export const chats = sqliteTable("chats", {
  id: text("id").primaryKey(),
  title: text("title").notNull(),
  createdAt: text("createdAt").notNull(),
  updatedAt: text("updatedAt").notNull(),
  metadata: text("metadata", { mode: "json" }),
});

// 메시지 테이블
export const messages = sqliteTable("messages", {
  id: text("id").primaryKey(),
  chatId: text("chatId")
    .notNull()
    .references(() => chats.id, { onDelete: "cascade" }),
  role: text("role").notNull(),
  content: text("content").notNull(),
  createdAt: text("createdAt").notNull(),
  files: text("files", { mode: "json" }),
  metadata: text("metadata", { mode: "json" }),
});
```

### 커스터마이징 방법

1. **스키마 확장**:
   `/services/database/schema.ts` 파일을 수정하여 추가 테이블이나 필드를 정의할 수 있습니다.
   ```typescript
   // 예: 사용자 테이블 추가
   export const users = sqliteTable("users", {
     id: text("id").primaryKey(),
     username: text("username").notNull().unique(),
     email: text("email").notNull().unique(),
     createdAt: text("createdAt").notNull(),
     metadata: text("metadata", { mode: "json" }),
   });
   
   // 채팅 테이블에 사용자 참조 추가
   export const chats = sqliteTable("chats", {
     // 기존 필드...
     userId: text("userId").references(() => users.id),
   });
   ```

2. **외부 데이터베이스 통합**:
   DatabaseMode.API를 사용하여 외부 데이터베이스 시스템과 통합할 수 있습니다.

3. **마이그레이션 관리**:
   Drizzle 마이그레이션을 사용하여 스키마 변경을 관리할 수 있습니다.

## 웹 애플리케이션 UI

### 개요

웹 애플리케이션 UI는 React를 기반으로 구축된 프론트엔드 인터페이스입니다. 채팅 인터페이스, 설정 및 구성 UI, 다국어 지원이 포함되어 있습니다.

### 주요 파일
- `/src/views/Chat/index.tsx`: 메인 채팅 인터페이스
- `/src/views/Overlay/Model/index.tsx`: 모델 설정 인터페이스
- `/src/components/`: 재사용 가능한 UI 컴포넌트
- `/src/atoms/`: Jotai 상태 관리
- `/src/styles/`: SCSS 스타일링

### 핵심 컴포넌트

#### 채팅 인터페이스
`/src/views/Chat/index.tsx` 파일은 메인 채팅 인터페이스를 구현합니다.

주요 기능:
- 메시지 목록 표시
- 사용자 입력 처리
- 응답 스트리밍 처리
- 도구 호출 시각화

#### 모델 설정
`/src/views/Overlay/Model/index.tsx` 파일은 LLM 모델 설정 인터페이스를 구현합니다.

주요 기능:
- 모델 선택 및 구성
- API 키 관리
- 매개변수 설정

#### 상태 관리
Jotai 상태 관리 라이브러리를 사용하여 애플리케이션 상태를 관리합니다.

주요 상태 atoms:
- `/src/atoms/chatState.ts`: 채팅 관련 상태
- `/src/atoms/configState.ts`: 구성 관련 상태
- `/src/atoms/modalState.ts`: 모달 표시 상태

### 커스터마이징 방법

1. **UI 테마 커스터마이징**:
   `/src/styles/_variables.scss` 파일에서 색상, 폰트, 간격 등을 변경할 수 있습니다.
   ```scss
   // 색상 변수 커스터마이징
   $primary-color: #your-brand-color;
   $background-color: #your-background-color;
   $text-color: #your-text-color;
   ```

2. **컴포넌트 확장**:
   `/src/components/` 디렉토리에 새 컴포넌트를 추가하여 UI를 확장할 수 있습니다.

3. **새 페이지 추가**:
   `/src/views/` 디렉토리에 새 페이지를 추가하고 `/src/router.tsx` 파일에 라우트를 등록할 수 있습니다.

4. **다국어 지원 확장**:
   `/public/locales/` 디렉토리에 새 언어 파일을 추가하여 다국어 지원을 확장할 수 있습니다.

## 각 컴포넌트 간 상호작용

### 데이터 흐름 다이어그램

```
┌───────────────┐     ┌─────────────────┐     ┌──────────────┐     ┌──────────────┐
│   React UI    │◄────┤  Electron IPC   │◄────┤ MCP 클라이언트│◄────┤ MCP 서버 매니저│
│ (src/views/)  │     │ (electron/ipc/) │     │(services/client)│   │(services/mcpServer)│
└───────┬───────┘     └────────┬────────┘     └───────┬──────┘     └──────┬───────┘
        │                      │                      │                    │
        ▼                      ▼                      ▼                    ▼
┌───────────────┐     ┌─────────────────┐     ┌──────────────┐     ┌──────────────┐
│   Jotai 상태  │     │   서비스 레이어  │     │  모델 매니저 │     │    도구 실행   │
│ (src/atoms/)  │     │ (services/*.ts) │     │(services/models)│   │ (외부 프로세스)│
└───────────────┘     └─────────────────┘     └──────────────┘     └──────────────┘
                              │
                              ▼
                      ┌─────────────────┐
                      │  데이터베이스   │
                      │(services/database)│
                      └─────────────────┘
```

### 주요 상호작용 시나리오

1. **사용자 메시지 처리**:
   - 사용자가 UI(React)에서 메시지 입력
   - 메시지가 IPC를 통해 메인 프로세스로 전송
   - MCP 클라이언트가 메시지 처리
   - processQuery가 LLM에 요청 전송
   - 응답이 스트리밍되어 다시 UI로 전달

2. **도구 호출 처리**:
   - LLM이 도구 호출 생성
   - processQuery가 도구 호출 감지
   - MCPServerManager가 적절한 도구 서버로 요청 라우팅
   - 도구 서버가 요청 처리 및 결과 반환
   - 결과가 다시 LLM에 전달되어 최종 응답 생성

3. **설정 변경**:
   - 사용자가 UI에서 설정 변경
   - 변경사항이 상태 관리를 통해 추적
   - IPC를 통해 설정이 메인 프로세스로 전송
   - 설정이 저장되고 관련 컴포넌트(ModelManager, MCPServerManager 등)가 업데이트
