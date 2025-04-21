# Dive 프로젝트 포크 및 커스터마이징 가이드

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [기술 스택](#2-기술-스택)
3. [프로젝트 구조](#3-프로젝트-구조)
4. [핵심 컴포넌트 분석](#4-핵심-컴포넌트-분석)
5. [데이터 흐름](#5-데이터-흐름)
6. [포크 및 커스터마이징을 위한 전략](#6-포크-및-커스터마이징을-위한-전략)
7. [고려해야 할 주의사항](#7-고려해야-할-주의사항)
8. [결론](#8-결론)

## 1. 프로젝트 개요

Dive는 함수 호출(Function Calling) 기능을 지원하는 모든 LLM 모델과 원활하게 통합되는 오픈소스 데스크톱 애플리케이션입니다. 이 애플리케이션은 MCP(Model Context Protocol)를 통해 AI 에이전트가 외부 도구를 사용할 수 있도록 지원합니다.

### 주요 기능
- 다양한 LLM 지원 (ChatGPT, Anthropic, Ollama, OpenAI 호환 모델)
- 크로스 플랫폼 지원 (Windows, MacOS, Linux)
- MCP(Model Context Protocol) 지원 (stdio와 SSE 모드 모두 지원)
- 다국어 지원 (영어, 중국어, 일본어, 스페인어 등)
- 고급 API 관리 (여러 API 키 및 모델 전환 지원)
- 커스텀 시스템 프롬프트
- 자동 업데이트 메커니즘

## 2. 기술 스택

### 프론트엔드
- React
- TypeScript
- Sass/SCSS
- Jotai (상태 관리)
- React Router (라우팅)
- i18next (다국어 지원)

### 백엔드
- Node.js
- Express
- LangChain
- SQLite (Better-SQLite3)
- Drizzle ORM

### 데스크톱 애플리케이션
- Electron
- Vite

### AI/LLM 통합
- LangChain
- Model Context Protocol (MCP)
- 다양한 LLM 제공업체 SDK (OpenAI, Anthropic, Google, AWS 등)

## 3. 프로젝트 구조

Dive 프로젝트는 크게 다음과 같은 주요 구성 요소로 나뉩니다:

### 1. Electron 애플리케이션 (데스크톱 UI)
- `/electron/main/` - Electron 메인 프로세스
- `/electron/preload/` - Electron 프리로드 스크립트
- `/src/` - React 프론트엔드 애플리케이션

### 2. API 서비스
- `/services/` - 백엔드 서비스 로직
- `/services/mcpServer/` - MCP 서버 관리
- `/services/models/` - LLM 모델 관리
- `/services/database/` - 데이터베이스 연결 및 스키마

### 3. 도구 및 유틸리티
- `/services/utils/` - 공통 유틸리티 함수
- `/services/prompt/` - 프롬프트 템플릿 관리

## 4. 핵심 컴포넌트 분석

### 1. MCP 서버 관리자 (MCPServerManager)
- MCP 서버와의 연결을 관리하고 도구 호출 처리
- 다양한 전송 방식(stdio, SSE, WebSocket) 지원
- 서버 구성을 동적으로 업데이트할 수 있는 기능 포함

### 2. 모델 관리자 (ModelManager)
- 다양한 LLM 제공업체 및 모델 설정 관리
- LangChain과 통합하여 일관된 인터페이스 제공
- 모델 구성의 로드, 저장 및 갱신 기능

### 3. 쿼리 처리 (processQuery)
- 사용자 쿼리를 AI에 전달하고 스트리밍 응답 처리
- 도구 호출 및 결과 처리를 위한 반복 로직
- 오류 처리 및 중단 로직

### 4. 데이터베이스 관리
- 채팅 및 메시지 저장소
- Drizzle ORM을 사용한 스키마 및 마이그레이션

### 5. 웹 애플리케이션 UI
- 채팅 인터페이스
- 설정 및 구성 UI
- 다국어 지원

## 5. 데이터 흐름

1. 사용자가 메시지를 입력 (`ChatInput.tsx`)
2. 메시지가 서버로 전송 (`Chat/index.tsx`)
3. 서버는 메시지와 컨텍스트를 처리 (`client.ts`)
4. LLM에 메시지 전송 및 응답 스트리밍 (`processQuery.ts`)
5. 함수 호출이 발생하면 적절한 MCP 서버로 라우팅 (`mcpServer/index.ts`)
6. 도구 결과는 LLM에 반환되며 다음 응답에 영향
7. 최종 응답이 사용자에게 표시되고 데이터베이스에 저장

## 6. 포크 및 커스터마이징을 위한 전략

### 1. 브랜치 관리 전략
1. **메인 브랜치**:
   - `upstream/main`: 원본 저장소의 메인 브랜치
   - `origin/main`: 귀하의 포크의 메인 브랜치 (원본과 동기화)
   
2. **개발 브랜치**:
   - `origin/develop`: 모든 커스텀 개발 작업을 위한 브랜치
   - 기능별 브랜치: `feature/custom-feature-name`

### 2. 원본 레포지토리와 동기화 방법
1. 원본 저장소를 업스트림으로 추가:
   ```bash
   git remote add upstream https://github.com/OpenAgentPlatform/Dive.git
   ```

2. 업스트림 변경사항 가져오기:
   ```bash
   git fetch upstream
   ```

3. 업스트림 변경사항을 메인 브랜치에 병합:
   ```bash
   git checkout main
   git merge upstream/main
   git push origin main
   ```

4. 개발 브랜치에 변경사항 리베이스:
   ```bash
   git checkout develop
   git rebase main
   git push origin develop --force  # 주의: force push는 협업 상황에서 주의하여 사용
   ```

### 3. 지속적 통합 파이프라인 설정 (GitHub Actions)
원본 저장소의 업데이트를 자동으로 가져와 포크된 프로젝트에 통합하는 GitHub Actions 워크플로우를 설정할 수 있습니다:

```yaml
name: Sync with upstream

on:
  schedule:
    - cron: '0 0 * * *'  # 매일 실행
  workflow_dispatch:  # 수동 트리거 허용

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          ref: main
          fetch-depth: 0

      - name: Add upstream
        run: |
          git remote add upstream https://github.com/OpenAgentPlatform/Dive.git
          git fetch upstream

      - name: Sync main branch
        run: |
          git checkout main
          git merge upstream/main
          git push origin main

      - name: Sync develop branch
        run: |
          git checkout develop
          git rebase main
          git push origin develop --force
```

### 4. 커스터마이징 영역 식별
Dive 프로젝트를 포크할 때 다음 영역에서 커스터마이징하는 것이 적합합니다:

1. **UI 테마 및 스타일링**:
   - `/src/styles/` 디렉토리의 SCSS 파일 수정
   - 회사 브랜딩에 맞게 색상, 로고, 아이콘 변경

2. **커스텀 MCP 서버 통합**:
   - `/services/mcpServer/` 디렉토리에서 회사별 커스텀 도구 통합
   - 내부 API와 서비스를 위한 전용 MCP 서버 추가

3. **인증 메커니즘**:
   - 회사 SSO 또는 인증 시스템과 통합
   - 사용자 관리 및 액세스 제어 추가

4. **기업용 기능 추가**:
   - 팀 협업 기능
   - 대화 내보내기/가져오기
   - 관리자 기능

5. **모델 제공업체 커스터마이징**:
   - 내부 또는 특정 모델 제공업체 우선 지정
   - 기본 매개변수 및 설정 커스터마이징

### 5. 최소한의 변경으로 커스터마이징하는 전략

1. **훅과 확장 패턴 사용**:
   - 코어 로직을 변경하지 않고 새로운 서비스 레이어 추가
   - 기존 함수를 래핑하고 확장하는 패턴 사용

2. **구성 기반 커스터마이징**:
   - 하드코딩된 변경 대신 구성 파일 활용
   - 환경 변수 및 설정 기반 조건부 로직 사용

3. **플러그인 시스템 개발**:
   - 커스텀 기능을 위한 플러그인 아키텍처 설계
   - 코어 코드베이스를 변경하지 않고 기능 확장

4. **CSS 오버라이드**:
   - 핵심 스타일을 변경하지 않고 오버라이드하는 전용 스타일시트 사용

## 7. 고려해야 할 주의사항

1. **라이선스 준수**: Dive는 오픈 소스 프로젝트이므로 모든 포크와 커스터마이징은 원본 라이선스를 준수해야 합니다.

2. **업데이트 충돌**: 원본 레포지토리의 업데이트가 귀하의 커스터마이징과 충돌할 수 있으므로, 변경사항을 최소화하고 확장 패턴을 사용하는 것이 좋습니다.

3. **보안 고려사항**: API 키 및 자격 증명 관리 과정에서 보안 관행을 준수해야 합니다.

4. **성능 최적화**: 커스텀 기능이 애플리케이션 성능에 영향을 미치지 않도록 해야 합니다.

## 8. 결론

Dive 프로젝트는 다양한 LLM 제공업체를 지원하는 확장 가능한 아키텍처를 가진 잘 구성된 Electron 애플리케이션입니다. MCP(Model Context Protocol)에 대한 기본 지원을 통해 AI 에이전트 기능을 쉽게 확장할 수 있으며, 이는 기업 내부 도구와의 통합에 특히 유용합니다.

포크하여 커스터마이징할 때 제안된 브랜치 관리 전략과 지속적 통합 파이프라인을 구현하면, 원본 프로젝트의 업데이트를 계속 받으면서도 회사별 요구사항에 맞게 확장할 수 있습니다. 최소한의 변경만으로 필요한 기능을 추가하는 접근 방식으로 유지 관리성을 높이고 업데이트 충돌을 최소화할 수 있습니다.
