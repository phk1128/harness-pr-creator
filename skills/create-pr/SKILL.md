---
name: create-pr
description: "PR 자동 생성 하네스. 브랜치의 변경 사항을 분석하여 Jira 연동, Summary, Changes, Impact, Testing 섹션이 포함된 PR을 자동 생성한다. Automated PR creation harness with Jira integration and standardized templates."
user-invocable: true
---

# PR Creator Harness

현재 브랜치의 커밋/변경 사항을 분석하여 표준 PR을 생성한다.

## Phase 0: 초기 설정 (온보딩)

프로젝트 루트에서 `.claude/settings/pr-creator.json` 파일을 읽는다.

### 설정 파일이 존재하는 경우

설정값을 로드하고 Phase 1로 진행한다.

### 설정 파일이 없는 경우 (첫 실행)

대화형으로 초기 설정을 진행한다:

**Step 1: Jira 연동 여부**

AskUserQuestion:
```
Jira 연동을 사용하시겠습니까?
사용하려면 Jira URL을 입력해주세요. (예: https://your-org.atlassian.net)
사용하지 않으려면 'skip'을 입력해주세요.
```
- URL 입력 시: `jira.enabled = true`, `jira.baseUrl = 입력값`
- 'skip' 입력 시: `jira.enabled = false`

**Step 2: 언어 선택**

AskUserQuestion:
```
PR 요약을 어떤 언어로 작성할까요? (en/ko/ja, 기본값: en)
```

**Step 3: 설정 저장**

수집한 설정을 `.claude/settings/pr-creator.json`에 저장한다:

```json
{
  "jira": { "enabled": true, "baseUrl": "https://your-org.atlassian.net", "ticketPattern": "[A-Z]+-[0-9]+" },
  "pr": { "titleFormat": "{type}({ticket}): {summary}", "titleMaxLength": 70, "baseBranch": "main", "language": "ko" },
  "sections": { "related": true, "summary": true, "changes": true, "impact": true, "testing": true }
}
```

설정 저장 후 "설정이 저장되었습니다. 다음부터는 이 설정이 자동으로 적용됩니다." 안내 후 Phase 1로 진행.

설정 스키마 상세는 `references/config.md` 참조.

## Phase 1: 컨텍스트 수집

아래 명령들을 **병렬**로 실행하여 정보를 수집한다:

```bash
# 현재 브랜치명
git branch --show-current

# base 브랜치 대비 커밋 목록
git log {baseBranch}..HEAD --oneline

# base 브랜치 대비 전체 diff
git diff {baseBranch}...HEAD

# 리모트 추적 상태
git status -sb

# 최근 머지된 PR 스타일 참고
gh pr list --state merged --limit 3 --json title,body
```

`{baseBranch}`는 Phase 0에서 로드한 `pr.baseBranch` 값을 사용한다.

## Phase 2: Jira 티켓번호 추출

`jira.enabled`가 `false`이면 이 Phase를 건너뛴다.

브랜치명에서 `jira.ticketPattern`으로 Jira 티켓번호를 추출한다.

**추출 예시:**
- `feature/CSP-1483-settlement-api` -> `CSP-1483`
- `fix/CGS-234` -> `CGS-234`
- `refactor/excel-sax` -> 티켓 없음

**티켓번호가 없는 경우:**
- AskUserQuestion: "브랜치에서 Jira 티켓번호를 찾을 수 없습니다. 티켓번호를 입력해주세요. (예: CSP-1234, 없으면 'skip')"
- 'skip' 입력 시 Related 섹션의 Jira 항목을 비운다

## Phase 3: PR 제목 생성

**제목 포맷:** `pr.titleFormat` 템플릿을 사용한다.

- `{type}`: 커밋 메시지들의 type을 분석하여 결정 (feat, fix, refactor, chore 등). 혼합된 경우 가장 지배적인 type 사용.
- `{ticket}`: Phase 2에서 추출한 Jira 티켓번호. 없으면 `{ticket}` 부분과 괄호를 함께 제거한다.
  - 예: `{type}({ticket}): {summary}` -> `feat: 변경 요약` (티켓 없을 때)
- `{summary}`: `pr.titleMaxLength` 이내, `pr.language`에 맞는 언어로 핵심 변경을 한 문장으로.

## Phase 4: PR 본문 생성

전체 diff와 커밋 히스토리를 분석하여 아래 템플릿을 채운다. `sections` 설정에서 `false`인 섹션은 생략한다.

```markdown
## Related
- Jira: [{ticket}]({jira.baseUrl}/browse/{ticket})

## Summary
- {주요 변경 내용 요약 — 2~5 bullet}

## Changes
- `{파일/클래스 경로}`: {어떤 기능이 바뀌었는지}

## Impact
- {기존 대비 어떤 영향이 있는지, 리뷰어가 반드시 알아야 할 내용}

## Testing
-
```

**각 섹션 작성 가이드:**

- **Related**: `jira.enabled`가 `true`이고 티켓이 있을 때만 Jira 링크를 생성한다. 형식: `[{ticket}]({jira.baseUrl}/browse/{ticket})`
- **Summary**: 왜 이 변경이 필요한지, 무엇이 바뀌는지를 2~5개 bullet으로. 기술적 세부사항보다 비즈니스/기능 관점 우선. `pr.language` 설정에 맞는 언어로 작성.
- **Changes**: 변경된 파일/클래스 경로를 백틱으로 감싸고, 각각 어떤 변화가 있는지 서술. 관련 파일끼리 그룹핑.
- **Impact**: 기존 동작과 비교해서 달라지는 점, 호환성/성능/보안 영향.
- **Testing**: 비워둔다 (사용자가 직접 채움).

## Phase 5: 사용자 확인 및 PR 생성

1. 생성된 제목과 본문을 사용자에게 **미리보기**로 보여준다
2. AskUserQuestion: "이 내용으로 PR을 생성할까요? (수정할 부분이 있으면 알려주세요)"
3. 사용자 승인 시:
   - 리모트에 push되지 않은 경우 `git push -u origin {branch}` 실행
   - `gh pr create` 실행:
     ```bash
     gh pr create --base {baseBranch} --title "{title}" --body "$(cat <<'EOF'
     {body}
     EOF
     )"
     ```
4. PR URL을 사용자에게 반환

## Phase 6: 후속 안내 (선택)

PR 생성 완료 후:
- 코드 리뷰 스킬이 활성화되어 있으면 `/code-review` 실행 의사 확인
- PR URL 공유

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| base 브랜치 대비 커밋 없음 | "현재 브랜치에 {baseBranch} 대비 변경 사항이 없습니다" 보고 후 종료 |
| gh CLI 미인증 | `gh auth login` 안내 |
| 리모트 push 실패 | 에러 메시지와 함께 수동 해결 안내 |
| PR이 이미 존재 | 기존 PR URL을 보여주고 업데이트 여부 확인 |
| 설정 파일 JSON 파싱 실패 | 기본값으로 폴백, 사용자에게 경고 |

## 주의사항

- PR 제목은 `pr.titleMaxLength` 이내로 유지
- 본문의 Changes 섹션에서 경로는 반드시 백틱으로 감싼다
- `git push --force`는 절대 사용하지 않는다
- PR 생성 전 반드시 사용자 확인을 받는다