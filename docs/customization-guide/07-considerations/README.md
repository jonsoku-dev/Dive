# Dive 프로젝트 커스터마이징 시 고려사항

이 문서는 Dive 프로젝트를 포크하고 커스터마이징할 때 고려해야 할 중요한 사항들을 상세히 설명합니다. 이러한 고려사항을 염두에 두면 원활한 개발 경험과 안정적인 커스텀 애플리케이션을 보장할 수 있습니다.

## 목차

- [라이선스 준수](#라이선스-준수)
- [업데이트 충돌 관리](#업데이트-충돌-관리)
- [보안 고려사항](#보안-고려사항)
- [성능 최적화](#성능-최적화)
- [배포 및 업데이트 관리](#배포-및-업데이트-관리)
- [디버깅 및 로깅](#디버깅-및-로깅)
- [호환성 보장](#호환성-보장)

## 라이선스 준수

Dive는 오픈 소스 프로젝트이므로 포크 및 커스터마이징 시 원본 라이선스를 준수해야 합니다.

### 라이선스 분석

Dive는 MIT 라이선스로 배포되며, 이 라이선스는 다음과 같은 특징이 있습니다:

- 상업적 사용 허용
- 수정 허용
- 배포 허용
- 사적 사용 허용
- 저작권 표시 및 라이선스 사본 포함 필요

### 라이선스 준수 체크리스트

1. **저작권 고지 유지**:
   - 원본 Dive 프로젝트의 저작권 고지를 제거하거나 수정하지 마세요.
   - LICENSE 파일을 유지하세요.

2. **라이선스 정보 표시**:
   - 포크한 코드베이스의 README.md 파일에 원본 프로젝트와 라이선스 정보를 표시하세요.
   ```markdown
   이 프로젝트는 [OpenAgentPlatform/Dive](https://github.com/OpenAgentPlatform/Dive)의 포크이며, MIT 라이선스에 따라 배포됩니다.
   ```

3. **의존성 라이선스 검토**:
   - 새로 추가하는 의존성들의 라이선스가 기존 라이선스와 호환되는지 확인하세요.
   - 라이선스 충돌이 발생할 수 있는 의존성은 피하세요.

4. **파생 작업 라이선스**:
   - 커스터마이징된 버전을 배포할 때 라이선스 조건을 명확히 하세요.
   - 내부 사용만을 위한 경우에도 라이선스 정보는 유지하는 것이 좋습니다.

### 상용 활용 시 고려사항

기업 내부적으로 활용하거나 상업적 제품으로 발전시킬 경우 다음 사항을 고려하세요:

- 법률 전문가의 검토: 대규모 상업적 활용 시 법률 전문가의 검토를 받는 것이 좋습니다.
- 기여 활동 고려: 가능하다면 개선 사항을 원본 프로젝트에 기여하는 것을 고려하세요.

## 업데이트 충돌 관리

원본 Dive 저장소와 동기화하는 과정에서 발생할 수 있는 충돌을 효과적으로 관리하는 전략이 필요합니다.

### 잠재적 충돌 영역

다음 영역에서 충돌이 자주 발생할 수 있습니다:

1. **코어 로직 수정**: 직접적인 코어 로직 수정은 업스트림 변경과 충돌할 가능성이 매우 높습니다.
2. **의존성 버전**: package.json 파일의 의존성 버전 업데이트는 충돌을 일으킬 수 있습니다.
3. **API 변경**: 내부 API나 인터페이스가 변경되면 커스텀 코드가 손상될 수 있습니다.
4. **스타일 및 UI 변경**: 스타일 관련 파일은 자주 변경되며 충돌이 발생할 수 있습니다.

### 충돌 최소화 전략

1. **추상화 계층 추가**:
   - 원본 코드를 직접 수정하는 대신 추상화 계층을 통해 확장하세요.
   ```typescript
   // 직접 수정 (피해야 함)
   // services/models/index.ts를 직접 수정
   
   // 추상화 계층 추가 (권장)
   // custom/models/customModelAdapter.ts 생성
   import { ModelManager } from '../../services/models';
   
   export class CustomModelAdapter {
     private modelManager: ModelManager;
     
     constructor() {
       this.modelManager = ModelManager.getInstance();
     }
     
     // 기존 기능을 확장하는 메서드
     async initializeWithCustomOptions(options) {
       // 기존 모델 초기화 후 커스텀 로직 적용
       await this.modelManager.initializeModel();
       // 커스텀 로직...
     }
   }
   ```

2. **설정 기반 구현**:
   - 하드코딩 대신 설정 파일을 통한 동작 변경을 구현하세요.
   ```typescript
   // custom/config/index.ts
   export interface CustomConfig {
     enabledFeatures: {
       customModelProvider: boolean;
       enhancedUI: boolean;
       enterpriseAuth: boolean;
     };
     // 기타 설정...
   }
   
   // 설정에 기반한 기능 활성화
   if (config.enabledFeatures.customModelProvider) {
     // 커스텀 모델 제공자 초기화
   }
   ```

3. **모듈화된 확장**:
   - 기능을 자체 포함된 모듈로 구현하여 핵심 코드와의 의존성을 최소화하세요.
   ```typescript
   // custom/modules/enterpriseAuth/index.ts
   export function initializeEnterpriseAuth() {
     // 독립적인 인증 모듈 구현
   }
   
   // 필요한 지점에서만 통합
   import { initializeEnterpriseAuth } from './custom/modules/enterpriseAuth';
   
   // 적절한 위치에서 초기화
   initializeEnterpriseAuth();
   ```

4. **변경 추적 자동화**:
   - 업스트림 변경 사항을 자동으로 모니터링하고 잠재적 충돌을 식별하는 스크립트를 구현하세요.
   ```bash
   #!/bin/bash
   # upstream-monitor.sh
   
   git fetch upstream
   
   # 핵심 파일 변경 확인
   CORE_CHANGES=$(git diff --name-only upstream/main..main | grep -E 'services/|src/atoms/')
   
   if [ ! -z "$CORE_CHANGES" ]; then
     echo "⚠️ Potential conflicts in core files:"
     echo "$CORE_CHANGES"
     # 알림 발송...
   fi
   ```

### 충돌 해결 가이드라인

충돌이 발생했을 때 다음 단계에 따라 해결하세요:

1. **변경 내용 분석**:
   - 업스트림 변경 사항의 목적과 범위를 이해하세요.
   - 이 변경이 커스텀 코드에 미치는 영향을 평가하세요.

2. **우선순위 결정**:
   - 보안 업데이트나 중요 버그 수정은 높은 우선순위로 처리하세요.
   - 기능 업데이트는 호환성 검토 후 통합 여부를 결정하세요.

3. **단계적 병합**:
   - 대규모 변경은 작은 단위로 나누어 병합하세요.
   - 각 단계마다 테스트를 실행하여 안정성을 확인하세요.

4. **커스텀 코드 리팩토링**:
   - 충돌이 계속 발생하는 영역의 코드를 더 모듈화된 방식으로 리팩토링하세요.
   - 필요하다면 아키텍처를 수정하여 장기적으로 충돌을 줄이세요.

## 보안 고려사항

Dive 애플리케이션의 커스터마이징 과정에서 보안을 강화하고 기업 환경에 적합하게 만드는 것이 중요합니다.

### API 키 및 자격 증명 관리

1. **안전한 저장**:
   - API 키를 하드코딩하지 마세요.
   - 환경 변수나 안전한 자격 증명 관리 시스템을 사용하세요.
   ```typescript
   // 권장 방식
   const apiKey = process.env.CUSTOM_API_KEY;
   
   // 기업 환경을 위한 보안 강화
   import { getSecretFromVault } from './custom/security/vault';
   
   async function getApiKey() {
     return await getSecretFromVault('custom_model_api_key');
   }
   ```

2. **키 순환 메커니즘**:
   - API 키 정기 교체를 지원하는 메커니즘을 구현하세요.
   - 최소 권한 원칙을 적용하여 각 키의 권한을 제한하세요.

3. **암호화**:
   - 디스크에 저장된 자격 증명은 항상 암호화하세요.
   ```typescript
   import { encrypt, decrypt } from './custom/security/crypto';
   
   // 자격 증명 암호화 저장
   async function saveCredentials(credentials) {
     const encrypted = await encrypt(JSON.stringify(credentials));
     // 암호화된 데이터 저장...
   }
   
   // 자격 증명 복호화 및 사용
   async function getCredentials() {
     // 암호화된 데이터 로드...
     return JSON.parse(await decrypt(encrypted));
   }
   ```

### 데이터 보안

1. **로컬 데이터 보호**:
   - 로컬 SQLite 데이터베이스 암호화를 구현하세요.
   - 민감한 대화 내용에 대한 암호화 옵션을 제공하세요.
   ```typescript
   // custom/database/secureDatabase.ts
   import { initDatabase } from '../../services/database';
   
   export async function initSecureDatabase(encryptionKey: string) {
     // 암호화된 데이터베이스 초기화
     // SQLCipher 또는 유사 솔루션 사용
   }
   ```

2. **대화 콘텐츠 보호**:
   - 민감한 정보가 포함된 대화에 대한 필터링 및 마스킹을 구현하세요.
   - 필요한 경우 대화 기록의 보존 기간을 제한하는 정책을 구현하세요.

3. **모델 응답 검증**:
   - 모델 응답에서 민감한 정보 유출을 방지하는 필터를 구현하세요.
   - 데이터 유출 가능성이 있는 응답을 탐지하는 분석 시스템을 구축하세요.

### 네트워크 보안

1. **API 통신 보안**:
   - 모든 외부 API 통신에 TLS/SSL을 사용하세요.
   - 인증서 검증을 적절히 구현하세요.
   - 프록시 설정을 지원하여 기업 네트워크 환경에 적응할 수 있게 하세요.
   ```typescript
   // custom/network/secureClient.ts
   import axios from 'axios';
   import https from 'https';
   
   export function createSecureClient(config) {
     return axios.create({
       // 기본 설정
       ...config,
       // TLS 설정
       httpsAgent: new https.Agent({
         rejectUnauthorized: true,  // 잘못된 인증서 거부
         ca: config.customCA,       // 기업 CA 지원
       }),
       // 프록시 설정
       proxy: config.proxySettings,
     });
   }
   ```

2. **네트워크 격리**:
   - 민감한 내부 도구는 외부 네트워크 접근을 제한하는 메커니즘을 구현하세요.
   - 필요한 경우 내부 전용 MCP 서버에 대한 격리된 통신 채널을 구현하세요.

3. **통신 모니터링**:
   - 비정상적인 API 요청이나 데이터 유출 패턴을 탐지하는 모니터링 시스템을 구현하세요.
   - 이상 활동 감지 시 알림 메커니즘을 구축하세요.

### 인증 및 권한 관리

1. **SSO 통합**:
   - 기업 환경을 위한 SSO(Single Sign-On) 인증을 구현하세요.
   ```typescript
   // custom/auth/ssoAuth.ts
   import { OAuth2Client } from 'google-auth-library';
   
   export async function initializeSSOAuth() {
     // SSO 인증 초기화 및 토큰 검증
   }
   
   export async function validateSSOToken(token) {
     // 토큰 검증 및 사용자 정보 추출
   }
   ```

2. **역할 기반 접근 제어**:
   - 사용자 역할에 따른 기능 및 데이터 접근 제한을 구현하세요.
   - 관리자 역할을 위한 추가 기능 및 제어 옵션을 제공하세요.

3. **감사 로깅**:
   - 보안 관련 이벤트에 대한 감사 로그를 구현하세요.
   - 중요한 작업 및 접근에 대한 기록을 유지하세요.

## 성능 최적화

커스터마이징된 Dive 애플리케이션의 성능을 최적화하여 사용자 경험을 향상시키세요.

### 메모리 관리

1. **메모리 누수 방지**:
   - 이벤트 리스너 및 구독을 적절히 해제하세요.
   - 대규모 객체의 생명주기를 관리하세요.
   ```typescript
   // 메모리 관리 예시
   class CustomComponent {
     private listeners = [];
     
     public initialize() {
       const listener = () => { /* ... */ };
       document.addEventListener('event', listener);
       this.listeners.push({ element: document, event: 'event', listener });
     }
     
     public dispose() {
       // 모든 리스너 해제
       this.listeners.forEach(({ element, event, listener }) => {
         element.removeEventListener(event, listener);
       });
       this.listeners = [];
     }
   }
   ```

2. **대화 기록 페이징**:
   - 긴 대화 기록을 효율적으로 관리하기 위한 페이징 메커니즘을 구현하세요.
   - 메모리에 로드하는 메시지 수를 제한하세요.

3. **이미지 최적화**:
   - 대용량 이미지 파일을 효율적으로 처리하는 메커니즘을 구현하세요.
   - 필요한 경우 이미지 리사이징 및 압축을 수행하세요.

### 응답성 개선

1. **비동기 작업 최적화**:
   - 무거운 작업은 별도 스레드나 워커로 분리하세요.
   - UI 렌더링을 차단하지 않도록 작업을 최적화하세요.
   ```typescript
   // 별도 스레드에서 무거운 작업 실행
   import { Worker } from 'worker_threads';
   
   export function runHeavyTaskInWorker(data) {
     return new Promise((resolve, reject) => {
       const worker = new Worker('./custom/workers/heavyTask.js');
       
       worker.on('message', resolve);
       worker.on('error', reject);
       worker.on('exit', (code) => {
         if (code !== 0) {
           reject(new Error(`Worker stopped with exit code ${code}`));
         }
       });
       
       worker.postMessage(data);
     });
   }
   ```

2. **프로그레시브 UI**:
   - 로딩 및 처리 상태를 명확히 표시하세요.
   - 긴 응답을 점진적으로 렌더링하여 사용자 경험을 개선하세요.

3. **캐싱 전략**:
   - 자주 사용되는 데이터 및 응답에 대한 캐싱을 구현하세요.
   - 모델 설정 및 MCP 서버 구성과 같은 정적 데이터를 효율적으로 캐싱하세요.
   ```typescript
   // 간단한 캐싱 메커니즘
   class SimpleCache {
     private cache = new Map();
     private ttl: number;
     
     constructor(ttlMs = 3600000) { // 기본 1시간 TTL
       this.ttl = ttlMs;
     }
     
     async get(key: string, fetchFn: () => Promise<any>) {
       const cachedItem = this.cache.get(key);
       
       if (cachedItem && Date.now() - cachedItem.timestamp < this.ttl) {
         return cachedItem.data;
       }
       
       // 캐시 미스 또는 만료
       const data = await fetchFn();
       this.cache.set(key, {
         data,
         timestamp: Date.now()
       });
       
       return data;
     }
     
     clear() {
       this.cache.clear();
     }
   }
   ```

### 네트워크 최적화

1. **요청 배칭**:
   - 여러 개의 작은 요청을 하나의 배치로 결합하세요.
   - 서버 부하 및 네트워크 오버헤드를 줄이세요.

2. **압축**:
   - 데이터 교환을 위한 압축을 구현하세요.
   - 대용량 응답에 대한 효율적인 처리를 구현하세요.

3. **연결 관리**:
   - 장기 실행 연결을 효율적으로 관리하세요.
   - 연결 재시도 및 장애 조치 메커니즘을 구현하세요.

### 도구 성능 최적화

1. **도구 호출 최적화**:
   - 무거운 MCP 도구 작업을 효율적으로 실행하세요.
   - 필요한 경우 도구 실행 결과를 캐싱하세요.

2. **병렬 처리**:
   - 독립적인 도구 호출을 병렬로 처리하세요.
   - 결과 집계 및 오류 처리를 효율적으로 구현하세요.
   ```typescript
   // 병렬 도구 호출
   async function executeParallelToolCalls(toolCalls) {
     const results = await Promise.allSettled(
       toolCalls.map(async (call) => {
         try {
           return await executeToolCall(call);
         } catch (error) {
           // 오류 처리...
           throw error;
         }
       })
     );
     
     // 결과 처리 및 집계
     return results.map((result, index) => {
       if (result.status === 'fulfilled') {
         return {
           tool_call_id: toolCalls[index].id,
           status: 'success',
           result: result.value
         };
       } else {
         return {
           tool_call_id: toolCalls[index].id,
           status: 'error',
           error: result.reason.message
         };
       }
     });
   }
   ```

## 배포 및 업데이트 관리

커스터마이징된 Dive 애플리케이션의 배포 및 업데이트 프로세스를 효율적으로 관리하세요.

### 배포 자동화

1. **CI/CD 파이프라인**:
   - 지속적 통합 및 배포 파이프라인을 구축하세요.
   - 자동화된 테스트, 빌드 및 배포 프로세스를 구현하세요.
   ```yaml
   # .github/workflows/deploy.yml 예시
   name: Build and Deploy
   
   on:
     push:
       tags:
         - 'v*'
   
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - name: Setup Node.js
           uses: actions/setup-node@v3
           with:
             node-version: '18'
         - name: Install dependencies
           run: npm ci
         - name: Build
           run: npm run build
         # 추가 배포 단계...
   ```

2. **환경별 구성**:
   - 개발, 테스트, 프로덕션 환경에 대한 별도 구성을 관리하세요.
   - 환경 변수 및 설정 파일을 통해 환경별 구성을 분리하세요.
   ```typescript
   // custom/config/environment.ts
   export enum Environment {
     DEVELOPMENT = 'development',
     TESTING = 'testing',
     PRODUCTION = 'production',
   }
   
   export function loadEnvironmentConfig(env: Environment) {
     switch (env) {
       case Environment.DEVELOPMENT:
         return { /* 개발 환경 설정 */ };
       case Environment.TESTING:
         return { /* 테스트 환경 설정 */ };
       case Environment.PRODUCTION:
         return { /* 프로덕션 환경 설정 */ };
     }
   }
   ```

3. **배포 아티팩트 관리**:
   - 빌드 아티팩트를 효율적으로 관리하고 버전 관리하세요.
   - 롤백을 위한 이전 버전 아티팩트를 보존하세요.

### 업데이트 메커니즘

1. **자동 업데이트**:
   - Electron 애플리케이션의 자동 업데이트 기능을 커스터마이징하세요.
   - 업데이트 알림 및 설치 프로세스를 사용자 친화적으로 구현하세요.
   ```typescript
   // custom/update/customUpdater.ts
   import { autoUpdater } from 'electron-updater';
   
   export function setupCustomUpdater(config) {
     // 기업 환경을 위한 자동 업데이트 구성
     autoUpdater.setFeedURL({
       provider: 'generic',
       url: config.updateServerUrl,
       // 기타 구성...
     });
     
     // 사용자 정의 이벤트 핸들러
     autoUpdater.on('update-available', (info) => {
       // 커스텀 알림 표시
     });
     
     // 업데이트 확인 시작
     autoUpdater.checkForUpdates();
   }
   ```

2. **점진적 롤아웃**:
   - 대규모 업데이트의 경우 점진적 롤아웃 전략을 구현하세요.
   - 사용자 그룹별로 업데이트를 단계적으로 배포하세요.

3. **롤백 메커니즘**:
   - 문제 발생 시 빠른 롤백을 위한 메커니즘을 구현하세요.
   - 업데이트 전 상태를 백업하고 복원하는 기능을 구현하세요.

### 배포 구성 관리

1. **패키징 최적화**:
   - 각 플랫폼(Windows, macOS, Linux)에 대한 최적화된 패키징 구성을 유지하세요.
   - 불필요한 파일을 제외하여 패키지 크기를 최적화하세요.
   ```json
   // electron-builder.json 최적화 예시
   {
     "appId": "com.yourcompany.dive",
     "productName": "Custom Dive",
     "directories": {
       "output": "releases"
     },
     "files": [
       "dist/**/*",
       "dist-electron/**/*",
       "!node_modules/**/*"
     ],
     "extraResources": [
       {
         "from": "custom/resources",
         "to": "resources"
       }
     ],
     "win": {
       "target": ["nsis"],
       "icon": "build/icon.ico"
     },
     "mac": {
       "target": ["dmg"],
       "icon": "build/icon.icns"
     },
     "linux": {
       "target": ["AppImage"],
       "icon": "build/icon.png"
     }
   }
   ```

2. **구성 템플릿**:
   - 다양한 배포 환경에 대한 구성 템플릿을 유지하세요.
   - 빌드 시 적절한 템플릿을 선택하여 적용하세요.

3. **자산 관리**:
   - 이미지, 아이콘 및 기타 정적 자산을 효율적으로 관리하세요.
   - 기업 브랜딩을 적용한 자산으로 교체하세요.

## 디버깅 및 로깅

효과적인 디버깅 및 로깅 전략은 개발 및 유지보수 과정에서 중요합니다.

### 로깅 개선

1. **구조화된 로깅**:
   - JSON 형식과 같은 구조화된 로그 형식을 사용하세요.
   - 추적성을 위해 각 로그 항목에 상관 ID를 포함하세요.
   ```typescript
   // custom/logging/structuredLogger.ts
   import { createLogger, format, transports } from 'winston';
   
   export const logger = createLogger({
     format: format.combine(
       format.timestamp(),
       format.json()
     ),
     defaultMeta: { service: 'custom-dive' },
     transports: [
       new transports.Console(),
       new transports.File({ filename: 'custom-dive.log' })
     ]
   });
   
   export function logWithCorrelation(level, message, correlationId, meta = {}) {
     logger.log(level, message, {
       correlationId,
       ...meta
     });
   }
   ```

2. **로그 수준 관리**:
   - 환경에 따라 적절한 로그 수준을 구성하세요.
   - 개발 환경에서는 더 상세한 로그를, 프로덕션 환경에서는 중요 로그만 캡처하세요.

3. **로그 저장 및 분석**:
   - 로그를 중앙 위치에 저장하고 분석하는 메커니즘을 구현하세요.
   - 기업 로그 시스템과의 통합을 고려하세요.

### 오류 처리

1. **전역 오류 처리**:
   - 처리되지 않은 예외를 캡처하고 로깅하는 메커니즘을 구현하세요.
   - 사용자 친화적인 오류 메시지를 표시하세요.
   ```typescript
   // custom/error/globalErrorHandler.ts
   import { app, dialog } from 'electron';
   import { logger } from '../logging/structuredLogger';
   
   export function setupGlobalErrorHandling() {
     // 처리되지 않은 예외 처리
     process.on('uncaughtException', (error) => {
       logger.error('Uncaught exception', { error: error.toString(), stack: error.stack });
       
       dialog.showErrorBox(
         'An error occurred',
         'The application encountered an unexpected error. Details have been logged.'
       );
     });
     
     // 처리되지 않은 거부 처리
     process.on('unhandledRejection', (reason) => {
       logger.error('Unhandled rejection', { reason });
     });
     
     // 렌더러 프로세스 충돌 처리
     app.on('render-process-gone', (event, webContents, details) => {
       logger.error('Renderer process crashed', { reason: details.reason });
       // 복구 시도...
     });
   }
   ```

2. **오류 분류 및 보고**:
   - 오류를 분류하고 우선순위를 지정하는 시스템을 구현하세요.
   - 중요한 오류에 대한 알림 메커니즘을 구축하세요.

3. **사용자 피드백**:
   - 오류 발생 시 사용자 피드백을 수집하는 메커니즘을 구현하세요.
   - 피드백을 통해 문제를 분석하고 개선하세요.

### 디버깅 도구

1. **원격 디버깅**:
   - 프로덕션 환경에서 문제를 진단하기 위한 원격 디버깅 기능을 구현하세요.
   - 보안을 고려하여 접근을 제한하세요.

2. **진단 유틸리티**:
   - 시스템 상태 및 구성을 진단하는 내장 유틸리티를 구현하세요.
   - 문제 해결을 위한 자가 진단 도구를 제공하세요.
   ```typescript
   // custom/diagnostics/systemCheck.ts
   export async function runSystemDiagnostics() {
     const results = {
       system: {
         platform: process.platform,
         arch: process.arch,
         nodeVersion: process.version,
         electronVersion: process.versions.electron,
       },
       network: await checkNetworkConnectivity(),
       database: await checkDatabaseConnection(),
       modelProviders: await checkModelProviderConnectivity(),
       mcpServers: await checkMCPServerStatus(),
     };
     
     return results;
   }
   ```

3. **테스트 모드**:
   - 특정 시나리오나 구성을 테스트하기 위한 특수 모드를 구현하세요.
   - 문제 재현 및 디버깅을 위한 도구를 제공하세요.

## 호환성 보장

커스터마이징된 Dive가 다양한 환경과 호환되도록 보장하세요.

### 플랫폼 호환성

1. **크로스 플랫폼 고려**:
   - Windows, macOS 및 Linux에서의 일관된 동작을 보장하세요.
   - 플랫폼별 차이점을 처리하는 추상화 계층을 구현하세요.
   ```typescript
   // custom/platform/index.ts
   export enum Platform {
     WINDOWS,
     MACOS,
     LINUX,
   }
   
   export function getCurrentPlatform(): Platform {
     switch (process.platform) {
       case 'win32': return Platform.WINDOWS;
       case 'darwin': return Platform.MACOS;
       default: return Platform.LINUX;
     }
   }
   
   export function getPlatformSpecificPath(basePath: string): string {
     switch (getCurrentPlatform()) {
       case Platform.WINDOWS:
         return `${basePath}\\windows`;
       case Platform.MACOS:
         return `${basePath}/macos`;
       case Platform.LINUX:
         return `${basePath}/linux`;
     }
   }
   ```

2. **하드웨어 호환성**:
   - 다양한 하드웨어 구성(CPU 아키텍처, 메모리 등)에서의 호환성을 테스트하세요.
   - 저사양 환경을 위한 경량 모드를 고려하세요.

3. **접근성**:
   - 다양한 접근성 요구사항을 충족하도록 UI를 최적화하세요.
   - 스크린 리더 호환성 및 키보드 네비게이션을 개선하세요.

### 모델 및 도구 호환성

1. **다양한 LLM 제공업체**:
   - 다양한 LLM 제공업체와의 호환성을 유지하세요.
   - 제공업체별 특수 기능을 적절히 처리하세요.
   ```typescript
   // custom/models/providerAdapter.ts
   export interface ModelProviderAdapter {
     initialize(config: any): Promise<void>;
     generateCompletion(prompt: string, options?: any): Promise<string>;
     streamCompletion(prompt: string, callback: (chunk: string) => void, options?: any): Promise<void>;
     // 기타 공통 인터페이스...
   }
   
   export class OpenAIAdapter implements ModelProviderAdapter {
     // OpenAI 구현...
   }
   
   export class AnthropicAdapter implements ModelProviderAdapter {
     // Anthropic 구현...
   }
   
   export class CustomModelAdapter implements ModelProviderAdapter {
     // 커스텀 모델 구현...
   }
   ```

2. **도구 버전 호환성**:
   - 다양한 MCP 도구 버전과의 호환성을 관리하세요.
   - 버전 불일치를 감지하고 해결하는 메커니즘을 구현하세요.

3. **확장성**:
   - 새로운 모델 및 도구를 쉽게 통합할 수 있는 확장 아키텍처를 구현하세요.
   - 플러그인 시스템을 통한 모듈식 확장을 고려하세요.

### 네트워크 환경 호환성

1. **오프라인 지원**:
   - 제한된 네트워크 환경에서의 작동을 개선하세요.
   - 오프라인 모드 및 로컬 모델 지원을 고려하세요.
   ```typescript
   // custom/network/connectivityManager.ts
   export enum ConnectivityStatus {
     ONLINE,
     LIMITED,
     OFFLINE,
   }
   
   export class ConnectivityManager {
     private status: ConnectivityStatus = ConnectivityStatus.ONLINE;
     private listeners: ((status: ConnectivityStatus) => void)[] = [];
     
     constructor() {
       this.monitorConnectivity();
     }
     
     private monitorConnectivity() {
       // 연결 상태 모니터링 구현
       window.addEventListener('online', () => this.updateStatus(ConnectivityStatus.ONLINE));
       window.addEventListener('offline', () => this.updateStatus(ConnectivityStatus.OFFLINE));
       
       // 주기적으로 연결 품질 확인
       setInterval(async () => {
         try {
           // 연결 품질 테스트
           const status = await this.checkConnectivity();
           this.updateStatus(status);
         } catch (error) {
           this.updateStatus(ConnectivityStatus.OFFLINE);
         }
       }, 30000);
     }
     
     private updateStatus(status: ConnectivityStatus) {
       if (this.status !== status) {
         this.status = status;
         this.notifyListeners();
       }
     }
     
     private async checkConnectivity(): Promise<ConnectivityStatus> {
       // 연결 품질 테스트 구현
       try {
         const start = Date.now();
         await fetch('https://www.google.com/generate_204', { mode: 'no-cors', cache: 'no-store' });
         const latency = Date.now() - start;
         
         return latency < 500 ? ConnectivityStatus.ONLINE : ConnectivityStatus.LIMITED;
       } catch {
         return ConnectivityStatus.OFFLINE;
       }
     }
     
     public addListener(listener: (status: ConnectivityStatus) => void) {
       this.listeners.push(listener);
     }
     
     public removeListener(listener: (status: ConnectivityStatus) => void) {
       this.listeners = this.listeners.filter(l => l !== listener);
     }
     
     private notifyListeners() {
       for (const listener of this.listeners) {
         listener(this.status);
       }
     }
     
     public getStatus(): ConnectivityStatus {
       return this.status;
     }
   }
   ```

2. **프록시 지원**:
   - 기업 프록시 환경에서의 작동을 보장하세요.
   - 프록시 설정을 구성하는 인터페이스를 제공하세요.

3. **네트워크 제한 처리**:
   - 네트워크 제한(방화벽, 대역폭 제한 등)을 처리하는 메커니즘을 구현하세요.
   - 제한된 환경에서의 성능 저하를 최소화하세요.
