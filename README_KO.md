# harness-pr-creator

**PR 자동 생성 하네스** - Claude Code 플러그인

[English](README.md) | **한국어**

브랜치의 커밋과 diff를 분석하여 Jira 연동, 표준 템플릿이 적용된 PR을 자동 생성하는 스킬 기반 하네스.

## 주요 기능

- **Jira 연동** - 브랜치명에서 티켓 번호를 자동 추출하여 PR에 링크
- **커밋 분석** - 커밋 타입(feat, fix, refactor...)을 분석하여 PR 제목 결정
- **표준 템플릿** - Related, Summary, Changes, Impact, Testing 섹션으로 구성된 PR 생성
- **설정 가능** - Jira URL, 제목 포맷, base 브랜치, 언어, 섹션 표시 여부 커스터마이징
- **사용자 확인** - PR 생성 전 항상 미리보기 제공
- **다국어 지원** - 영어, 한국어, 일본어 요약 지원

## 설치

### 방법 1: CLI (권장)

```bash
# 1. 마켓플레이스 등록
/plugin marketplace add phk1128/harness-pr-creator

# 2. 플러그인 설치
/plugin install harness-pr-creator@phk1128-harness-pr-creator
```

### 방법 2: 수동 설정

Claude Code 설정(`~/.claude/settings.json`)에 추가:

```json
{
  "enabledPlugins": {
    "harness-pr-creator@phk1128-harness-pr-creator": true
  },
  "extraKnownMarketplaces": {
    "phk1128-harness-pr-creator": {
      "source": {
        "source": "github",
        "repo": "phk1128/harness-pr-creator"
      }
    }
  }
}
```

## 사용법

```
/create-pr              # 브랜치명에서 티켓 자동 추출
/create-pr CSP-1234     # Jira 티켓 직접 지정
```

스킬이 자동으로:

1. 프로젝트 설정을 로드 (없으면 기본값 사용)
2. 브랜치 정보, 커밋, diff 수집
3. 브랜치명에서 Jira 티켓 추출
4. PR 제목과 본문 생성
5. 미리보기 후 확인 요청
6. Push 및 PR 생성

## 설정

프로젝트 루트에 `.claude/settings/pr-creator.json` 생성:

```json
{
  "jira": {
    "enabled": true,
    "baseUrl": "https://your-org.atlassian.net",
    "ticketPattern": "[A-Z]+-[0-9]+"
  },
  "pr": {
    "titleFormat": "{type}({ticket}): {summary}",
    "titleMaxLength": 70,
    "baseBranch": "main",
    "language": "ko"
  },
  "sections": {
    "related": true,
    "summary": true,
    "changes": true,
    "impact": true,
    "testing": true
  }
}
```

### 설정 필드

| 필드 | 기본값 | 설명 |
|------|--------|------|
| `jira.enabled` | `true` | Jira 연동 활성화 |
| `jira.baseUrl` | `""` | Jira URL. 비어있으면 첫 실행 시 입력 요청 |
| `jira.ticketPattern` | `[A-Z]+-[0-9]+` | 브랜치명에서 티켓 추출 정규식 |
| `pr.titleFormat` | `{type}({ticket}): {summary}` | 제목 템플릿 |
| `pr.titleMaxLength` | `70` | 최대 제목 길이 |
| `pr.baseBranch` | `main` | 비교 대상 브랜치 |
| `pr.language` | `en` | 요약 언어 (`en`, `ko`, `ja`) |
| `sections.*` | `true` | 개별 섹션 표시 여부 |

### 설정 파일이 없다면?

설정 파일이 없으면 기본값을 사용하고, Jira URL 같은 필수 값은 대화형으로 입력받습니다.

## 사용 예시

### Jira 연동 시

브랜치: `feature/CSP-1483-settlement-api`

```
feat(CSP-1483): 정산 API 엔드포인트 추가

## Related
- Jira: [CSP-1483](https://your-org.atlassian.net/browse/CSP-1483)

## Summary
- 정산 계산 및 대사 API 추가
- ...
```

### Jira 없이

브랜치: `refactor/excel-sax`

```
refactor: 엑셀 파서 SAX 스트리밍 전환

## Summary
- DOM 기반 파싱을 SAX로 전환하여 메모리 효율 확보
- ...
```

## 요구사항

- [Claude Code](https://claude.ai/code) CLI
- [GitHub CLI](https://cli.github.com/) (`gh`) - 인증 완료 상태
- 리모트가 설정된 Git 저장소

## 라이선스

[MIT](LICENSE)