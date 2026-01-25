# MCP 서버 설정 가이드

iOS 개발을 위한 MCP 서버 설정 방법입니다.

---

## 개요

MCP (Model Context Protocol)는 Claude Code가 외부 도구와 통신하는 프로토콜입니다.
iOS 개발에 유용한 MCP 서버들을 설정하는 방법을 안내합니다.

---

## 기본 설정

### 1. 설정 파일 위치

```bash
# 글로벌 설정 (모든 프로젝트에 적용)
~/.claude.json

# 프로젝트별 설정
/path/to/project/.claude/settings.json
```

### 2. 기본 설정 예시

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxx"
      }
    }
  }
}
```

---

## iOS 개발 권장 MCP 서버

### 1. GitHub (필수)

PR 관리, 이슈 트래킹, 코드 리뷰에 필수입니다.

```json
{
  "github": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxx"
    }
  }
}
```

**토큰 생성:**
1. GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token (classic)
3. 권한: `repo`, `workflow`, `read:org`

### 2. Memory (권장)

세션 간 컨텍스트 유지에 유용합니다.

```json
{
  "memory": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-memory"]
  }
}
```

**용도:**
- 프로젝트 컨텍스트 저장
- 이전 세션 결정 사항 기억
- 반복 작업 최소화

### 3. Context7 (권장)

Swift, SwiftUI, 서드파티 라이브러리 문서 조회에 유용합니다.

```json
{
  "context7": {
    "command": "npx",
    "args": ["-y", "@context7/mcp-server"]
  }
}
```

**용도:**
- Alamofire, Kingfisher 등 라이브러리 문서
- SwiftUI API 레퍼런스
- iOS 버전별 API 가용성 확인

### 4. Sequential Thinking (선택)

복잡한 아키텍처 결정에 도움됩니다.

```json
{
  "sequential-thinking": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
  }
}
```

**용도:**
- 아키텍처 설계 시 단계별 추론
- 복잡한 디버깅
- 트레이드오프 분석

### 5. Filesystem (선택)

특정 디렉토리 접근 권한 부여가 필요할 때 사용합니다.

```json
{
  "filesystem": {
    "command": "npx",
    "args": [
      "-y",
      "@modelcontextprotocol/server-filesystem",
      "/Users/username/Projects/MyiOSApp"
    ]
  }
}
```

---

## 설정 팁

### 컨텍스트 윈도우 관리

MCP 서버가 많으면 컨텍스트 윈도우가 줄어듭니다.

```json
{
  "mcpServers": {
    // 필수만 활성화
    "github": { ... },
    "memory": { ... }
  },
  "disabledMcpServers": [
    // 필요할 때만 활성화
    "filesystem",
    "sequential-thinking"
  ]
}
```

### 환경 변수 사용

토큰을 직접 파일에 넣지 않고 환경 변수 사용:

```json
{
  "github": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-github"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
    }
  }
}
```

```bash
# ~/.zshrc 또는 ~/.bashrc
export GITHUB_TOKEN="ghp_xxxxxxxxxxxx"
```

---

## iOS 특화 설정 예시

### 전체 설정 파일

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@context7/mcp-server"]
    }
  },
  "permissions": {
    "allow": [
      "Bash(xcodebuild:*)",
      "Bash(swift:*)",
      "Bash(git:*)",
      "Bash(gh:*)"
    ]
  }
}
```

---

## 문제 해결

### MCP 서버 연결 실패

```bash
# npx 캐시 정리
npx clear-npx-cache

# Node.js 버전 확인 (18+ 필요)
node --version

# MCP 서버 직접 실행 테스트
npx -y @modelcontextprotocol/server-github
```

### 토큰 권한 오류

```
Error: Bad credentials
```

→ GitHub 토큰 권한 확인 (repo, workflow 필요)

### 컨텍스트 부족

```
Warning: Context window reduced
```

→ 불필요한 MCP 서버 비활성화

---

## 관련 문서

- [everything-claude-code MCP 가이드](https://github.com/anthropics/everything-claude-code)
- [MCP 공식 문서](https://modelcontextprotocol.io)
- [ios-mcp-servers.json](../../mcp-configs/ios-mcp-servers.json)
