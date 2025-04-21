# Dive 프로젝트 포크 및 커스터마이징 전략

이 문서는 Dive 프로젝트를 포크하고 커스터마이징하기 위한 효과적인 전략과 방법을 제공합니다. 기업 환경에서 Dive를 포크하여 내부 도구로 활용하고자 할 때 고려해야 할 브랜치 관리, 코드 구성, 업스트림 통합에 대한 상세한 지침을 포함합니다.

## 목차

- [포크 설정 및 초기화](#포크-설정-및-초기화)
- [브랜치 관리 전략](#브랜치-관리-전략)
- [지속적 통합 파이프라인](#지속적-통합-파이프라인)
- [코드 구성 및 모듈화](#코드-구성-및-모듈화)
- [익스텐션 패턴](#익스텐션-패턴)
- [테스트 전략](#테스트-전략)
- [릴리스 관리](#릴리스-관리)

## 포크 설정 및 초기화

### 1. GitHub에서 포크 생성

1. GitHub에서 [Dive 저장소](https://github.com/OpenAgentPlatform/Dive)로 이동합니다.
2. 우측 상단의 "Fork" 버튼을 클릭합니다.
3. 필요한 경우 조직을 선택하고 포크를 생성합니다.

### 2. 로컬 개발 환경 설정

```bash
# 포크한 저장소 클론
git clone https://github.com/YOUR_ORG/Dive.git
cd Dive

# 원본 저장소를 upstream 원격으로 추가
git remote add upstream https://github.com/OpenAgentPlatform/Dive.git

# 원격 저장소 확인
git remote -v
```

### 3. 개발 브랜치 생성

```bash
# 메인 브랜치 최신 상태로 유지
git fetch upstream
git checkout main
git merge upstream/main

# 개발 브랜치 생성
git checkout -b develop
git push -u origin develop
```

### 4. 초기 빌드 및 테스트

```bash
# 의존성 설치
npm install

# 개발 서버 실행
npm run dev

# 전체 빌드 테스트
npm run build
```

## 브랜치 관리 전략

Dive 프로젝트를 포크할 때는 다음과 같은 브랜치 관리 전략을 권장합니다:

### 1. 주요 브랜치

- **main**: 원본 Dive 프로젝트와 동기화된 브랜치
- **develop**: 모든 커스텀 개발 작업의 기본 브랜치
- **feature/[feature-name]**: 개별 기능 개발 브랜치
- **release/[version]**: 릴리스 준비 브랜치
- **hotfix/[fix-name]**: 긴급 수정 브랜치

### 2. 브랜치 목적 및 수명

| 브랜치 유형 | 목적 | 소스 브랜치 | 대상 브랜치 | 수명 |
|------------|------|------------|------------|------|
| main | 원본 동기화 | upstream/main | - | 영구 |
| develop | 내부 개발 | main | - | 영구 |
| feature/* | 기능 개발 | develop | develop | 임시 |
| release/* | 릴리스 준비 | develop | develop, main | 임시 |
| hotfix/* | 긴급 수정 | main | develop, main | 임시 |

### 3. 브랜치 워크플로우

#### 기능 개발 워크플로우

```bash
# develop 브랜치에서 시작
git checkout develop
git pull

# 기능 브랜치 생성
git checkout -b feature/custom-tool

# 개발 작업...

# 기능 브랜치 업데이트
git add .
git commit -m "Add custom tool integration"
git push -u origin feature/custom-tool

# 개발 완료 후 develop에 병합 (PR 또는 직접 병합)
git checkout develop
git merge feature/custom-tool
git push origin develop
```

#### 업스트림 동기화 워크플로우

```bash
# 업스트림 변경사항 가져오기
git fetch upstream

# main 브랜치 동기화
git checkout main
git merge upstream/main
git push origin main

# develop 브랜치 리베이스
git checkout develop
git rebase main
git push origin develop --force  # 주의: 협업 시 조심해서 사용
```

## 지속적 통합 파이프라인

원본 Dive 저장소와의 지속적인 동기화와 커스텀 코드의 통합을 위한 GitHub Actions 워크플로우를 설정하는 것이 좋습니다.

### 1. 업스트림 동기화 워크플로우

`.github/workflows/sync-upstream.yml` 파일 생성:

```yaml
name: Sync with upstream

on:
  schedule:
    - cron: '0 0 * * *'  # 매일 UTC 00:00에 실행
  workflow_dispatch:  # 수동 트리거 허용

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          ref: main
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Add upstream
        run: |
          git remote add upstream https://github.com/OpenAgentPlatform/Dive.git
          git fetch upstream

      - name: Sync main branch
        run: |
          git checkout main
          git merge upstream/main
          git push origin main

      - name: Create pull request for develop
        uses: peter-evans/create-pull-request@v5
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          branch: sync/upstream-to-develop
          base: develop
          title: "Sync: Upstream changes to develop"
          body: "This PR syncs changes from upstream/main to develop branch."
          commit-message: "Sync: Merge upstream changes into develop"
```

### 2. 충돌 감지 및 알림 워크플로우

`.github/workflows/conflict-check.yml` 파일 생성:

```yaml
name: Check for merge conflicts

on:
  push:
    branches: [develop]
  schedule:
    - cron: '0 12 * * *'  # 매일 UTC 12:00에 실행

jobs:
  check-conflicts:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Check conflicts with upstream
        run: |
          git remote add upstream https://github.com/OpenAgentPlatform/Dive.git
          git fetch upstream
          
          if git merge-tree $(git merge-base develop upstream/main) develop upstream/main | grep -e '<<<<<<<' -e '>>>>>>>' -e '======='; then
            echo "CONFLICTS=true" >> $GITHUB_ENV
          else
            echo "CONFLICTS=false" >> $GITHUB_ENV
          fi
          
      - name: Send notification on conflicts
        if: env.CONFLICTS == 'true'
        uses: slackapi/slack-github-action@v1
        with:
          channel-id: 'dev-alerts'
          slack-message: "⚠️ Potential merge conflicts detected between develop and upstream/main!"
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

### 3. 테스트 및 빌드 워크플로우

`.github/workflows/test-build.yml` 파일 생성:

```yaml
name: Test and Build

on:
  push:
    branches: [develop, main, 'feature/**']
  pull_request:
    branches: [develop, main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm test
        
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build
        run: npm run build
```

## 코드 구성 및 모듈화

원본 코드베이스를 수정하는 것보다 모듈화된 구조로 커스텀 코드를 분리하는 것이 좋습니다. 이렇게 하면 업스트림 변경 사항과의 충돌을 최소화할 수 있습니다.

### 1. 디렉토리 구조

다음과 같은 구조로 커스텀 코드를 관리하는 것을 권장합니다:

```
/
├── custom/                   # 모든 커스텀 코드의 루트 디렉토리
│   ├── models/               # 커스텀 모델 통합
│   ├── tools/                # 커스텀 MCP 도구
│   ├── themes/               # 커스텀 UI 테마
│   ├── components/           # 커스텀 UI 컴포넌트
│   └── extensions/           # 코어 기능 확장
├── // 원본 Dive 디렉토리 구조...
```

### 2. 진입점 확장

원본 코드를 직접 수정하는 대신, 진입점을 확장하는 방식으로 커스텀 코드를 통합하세요:

```typescript
// custom/index.ts
import { registerCustomModels } from './models';
import { registerCustomTools } from './tools';
import { applyCustomTheme } from './themes';
import { extendComponents } from './components';

export function initializeCustomizations() {
  registerCustomModels();
  registerCustomTools();
  applyCustomTheme();
  extendComponents();
}
```

그리고 적절한 위치에서 이 초기화 함수를 호출합니다:

```typescript
// src/main.tsx 수정
import { initializeCustomizations } from '../custom';

// 기존 초기화 코드...
initializeCustomizations();
// 기존 코드 계속...
```

## 익스텐션 패턴

원본 코드를 직접 수정하는 대신 확장 패턴을 사용하면 업스트림 변경사항과의 충돌을 최소화할 수 있습니다.

### 1. 래퍼 패턴

원본 클래스를 확장하여 추가 기능을 구현합니다:

```typescript
// custom/models/CustomModelManager.ts
import { ModelManager } from '../../services/models';

export class CustomModelManager extends ModelManager {
  // 싱글톤 확장
  private static customInstance: CustomModelManager;
  
  public static getCustomInstance(configPath?: string): CustomModelManager {
    if (!CustomModelManager.customInstance) {
      // 기존 인스턴스를 사용하여 초기화
      const originalInstance = ModelManager.getInstance(configPath);
      CustomModelManager.customInstance = new CustomModelManager(originalInstance, configPath);
    }
    return CustomModelManager.customInstance;
  }
  
  private originalManager: ModelManager;
  
  private constructor(originalManager: ModelManager, configPath?: string) {
    super(configPath);
    this.originalManager = originalManager;
  }
  
  // 기존 메서드 오버라이드 및 확장
  async initializeModel() {
    // 기존 초기화 수행
    const model = await this.originalManager.initializeModel();
    
    // 추가 초기화 로직...
    
    return model;
  }
  
  // 추가 기능
  async initializeCustomModel() {
    // 커스텀 모델 초기화...
  }
}

// 사용 예시
export function registerCustomModels() {
  // 원본 ModelManager 대신 CustomModelManager 사용
  const customModelManager = CustomModelManager.getCustomInstance();
  
  // 추가 초기화...
}
```

### 2. 플러그인 패턴

플러그인 시스템을 구현하여 코어 코드 변경 없이 기능을 확장합니다:

```typescript
// custom/plugins/PluginManager.ts
export interface Plugin {
  id: string;
  name: string;
  description: string;
  initialize: () => Promise<void>;
  shutdown: () => Promise<void>;
}

export class PluginManager {
  private static instance: PluginManager;
  private plugins: Map<string, Plugin> = new Map();
  
  public static getInstance(): PluginManager {
    if (!PluginManager.instance) {
      PluginManager.instance = new PluginManager();
    }
    return PluginManager.instance;
  }
  
  public register(plugin: Plugin): void {
    this.plugins.set(plugin.id, plugin);
  }
  
  public async initializeAll(): Promise<void> {
    for (const plugin of this.plugins.values()) {
      await plugin.initialize();
    }
  }
  
  public async shutdownAll(): Promise<void> {
    for (const plugin of this.plugins.values()) {
      await plugin.shutdown();
    }
  }
}

// 플러그인 구현 예시
export class CustomToolPlugin implements Plugin {
  id = 'custom-tools';
  name = 'Custom Tools Plugin';
  description = 'Adds custom tools to the application';
  
  async initialize(): Promise<void> {
    // 도구 등록 로직...
  }
  
  async shutdown(): Promise<void> {
    // 정리 로직...
  }
}
```

### 3. 설정 기반 확장

하드코딩된 변경 대신 설정 파일을 사용하여 동작을 변경합니다:

```typescript
// custom/config/customConfig.ts
export interface CustomConfig {
  customModels: {
    [key: string]: {
      name: string;
      baseURL: string;
      apiKey: string;
      enabled: boolean;
    }
  };
  customTools: {
    [key: string]: {
      command: string;
      args: string[];
      enabled: boolean;
    }
  };
  customTheme: {
    primaryColor: string;
    secondaryColor: string;
    fontFamily: string;
  };
}

export const defaultCustomConfig: CustomConfig = {
  customModels: {},
  customTools: {},
  customTheme: {
    primaryColor: '#1a73e8',
    secondaryColor: '#ea4335',
    fontFamily: 'Roboto, sans-serif',
  },
};

// 구성 로드 및 병합 함수
export async function loadCustomConfig(): Promise<CustomConfig> {
  try {
    // 외부 구성 파일 로드 로직...
    return { ...defaultCustomConfig, ...externalConfig };
  } catch (error) {
    console.error('Failed to load custom config:', error);
    return defaultCustomConfig;
  }
}
```

## 테스트 전략

커스텀 코드에 대한 효과적인 테스트 전략을 구현하여 원본 기능을 유지하면서 새로운 기능의 안정성을 보장해야 합니다.

### 1. 테스트 구조

```
/
├── custom/
│   └── __tests__/             # 커스텀 코드 테스트
│       ├── models/
│       ├── tools/
│       ├── integration/       # 통합 테스트
│       └── e2e/               # 엔드-투-엔드 테스트
```

### 2. 단위 테스트 예시

```typescript
// custom/__tests__/models/CustomModelManager.test.ts
import { CustomModelManager } from '../../models/CustomModelManager';

describe('CustomModelManager', () => {
  let manager: CustomModelManager;
  
  beforeEach(() => {
    // 테스트 설정...
    manager = CustomModelManager.getCustomInstance();
  });
  
  test('should initialize custom models', async () => {
    // 테스트 구현...
  });
  
  test('should override original behavior correctly', async () => {
    // 테스트 구현...
  });
});
```

### 3. 통합 테스트 예시

```typescript
// custom/__tests__/integration/CustomToolsIntegration.test.ts
import { PluginManager } from '../../plugins/PluginManager';
import { CustomToolPlugin } from '../../plugins/CustomToolPlugin';
import { MCPServerManager } from '../../../services/mcpServer';

describe('Custom Tools Integration', () => {
  beforeEach(() => {
    // 테스트 설정...
  });
  
  test('should register custom tools with MCP server manager', async () => {
    // 플러그인 매니저 설정
    const pluginManager = PluginManager.getInstance();
    pluginManager.register(new CustomToolPlugin());
    await pluginManager.initializeAll();
    
    // MCP 서버 매니저 확인
    const mcpServerManager = MCPServerManager.getInstance();
    const tools = mcpServerManager.getToolInfos();
    
    // 커스텀 도구가 등록되었는지 확인
    expect(tools.some(tool => tool.name === 'custom-tool')).toBe(true);
  });
});
```

## 릴리스 관리

커스텀 버전의 Dive를 효율적으로 릴리스하기 위한 전략을 구현하세요.

### 1. 버전 관리

원본 Dive 버전과 커스텀 버전을 명확히 구분하는 버전 체계를 사용하세요:

```
원본 Dive: 0.7.9
커스텀 버전: 0.7.9-custom.1
```

`package.json` 파일 예시:

```json
{
  "name": "custom-dive",
  "private": true,
  "version": "0.7.9-custom.1",
  "description": "Customized version of Dive AI Agent",
  "main": "dist-electron/main/index.js",
  // 나머지 구성...
}
```

### 2. 릴리스 체크리스트

새 버전을 릴리스하기 전에 다음 체크리스트를 수행하세요:

1. 최신 업스트림 변경사항 통합
   ```bash
   git checkout main
   git fetch upstream
   git merge upstream/main
   git push origin main
   
   git checkout develop
   git rebase main
   git push origin develop --force
   ```

2. 테스트 실행 및 확인
   ```bash
   npm test
   ```

3. 빌드 및 패키징 테스트
   ```bash
   npm run build
   npm run package:darwin-dmg  # 또는 다른 플랫폼용 패키지
   ```

4. 버전 업데이트
   ```bash
   # package.json 버전 업데이트
   ```

5. 릴리스 노트 작성
   ```bash
   # CHANGELOG.md 업데이트
   ```

6. 릴리스 브랜치 생성 및 배포
   ```bash
   git checkout -b release/0.7.9-custom.1
   git add .
   git commit -m "Release version 0.7.9-custom.1"
   git tag v0.7.9-custom.1
   git push origin release/0.7.9-custom.1
   git push origin v0.7.9-custom.1
   ```

7. 최종 릴리스 빌드 및 배포
   ```bash
   # 필요한 플랫폼용 패키지 생성
   npm run package:windows
   npm run package:darwin-dmg
   npm run package:linux
   
   # 릴리스 배포 (내부 배포 시스템 또는 GitHub Releases)
   ```

### 3. 자동화된 릴리스 워크플로우

GitHub Actions를 사용하여 릴리스 프로세스를 자동화할 수 있습니다:

```yaml
# .github/workflows/release.yml
name: Release Custom Dive

on:
  push:
    tags:
      - 'v*'

jobs:
  create-release:
    runs-on: ubuntu-latest
    steps:
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false
          
  build-windows:
    needs: create-release
    runs-on: windows-latest
    steps:
      # Windows 빌드 단계...
  
  build-macos:
    needs: create-release
    runs-on: macos-latest
    steps:
      # macOS 빌드 단계...
      
  build-linux:
    needs: create-release
    runs-on: ubuntu-latest
    steps:
      # Linux 빌드 단계...
```
