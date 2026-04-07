# 사내 Claude Code 스킬 카탈로그

사내 Claude Code 스킬 플러그인입니다. 스킬 관리 도구와 일상 업무 자동화 스킬을 제공합니다.

## 스킬 관리 도구

| 스킬 | 설명 | 사용법 |
|------|------|--------|
| [skill-create](skill-create/SKILL.md) | 표준 스킬 구조 자동 생성 (스캐폴딩) | `/nkia-skills:skill-create <name>` |
| [skill-validate](skill-validate/SKILL.md) | SKILL.md 필수 섹션 검증 (품질 게이트) | `/nkia-skills:skill-validate [name]` |
| [skill-list](skill-list/SKILL.md) | 로컬/원격 스킬 카탈로그 조회 | `/nkia-skills:skill-list [--remote\|--all]` |
| [skill-publish](skill-publish/SKILL.md) | 사내 Git repo에 스킬 배포 | `/nkia-skills:skill-publish <name>` |
| [skill-install](skill-install/SKILL.md) | 사내 repo에서 스킬 설치 | `/nkia-skills:skill-install <name>` |

## 일상 업무 자동화

PIMS + git 기반 범용 스킬. 어떤 프로젝트에서든 사용 가능합니다.

| 스킬 | 설명 | 사용법 |
|------|------|--------|
| [dev-start](dev-start/SKILL.md) | 개발 시작 대시보드 (리마인더 + 이슈 + 어제 작업) | `/nkia-skills:dev-start` |
| [log-work](log-work/SKILL.md) | PIMS 작업 시간 기록 + 진행률 갱신 | `/nkia-skills:log-work [#ticket] <hours>` |
| [weekly-report](weekly-report/SKILL.md) | 주간 업무 보고서 자동 생성/제출 | `/nkia-skills:weekly-report [--dry-run]` |
| [sprint-status](sprint-status/SKILL.md) | 스프린트/마일스톤 현황 대시보드 + 일괄 갱신 | `/nkia-skills:sprint-status [milestone]` |
| [task-review](task-review/SKILL.md) | git log + AI 대화 기반 진행사항 보고서 생성 | `/nkia-skills:task-review [날짜]` |

## 워크플로우

```
/nkia-skills:skill-create my-skill     ← 표준 구조 생성
  ↓ (SKILL.md 작성)
/nkia-skills:skill-validate my-skill   ← 구조 검증
  ↓ (PASS)
/nkia-skills:skill-publish my-skill    ← 사내 repo에 배포
  ↓
/nkia-skills:skill-install my-skill    ← 다른 프로젝트에서 설치
```

## 설정

`.claude/skill-registry.yaml`에 사내 스킬 repo URL을 설정해야 합니다:

```yaml
registry:
  remote_url: "https://github.com/gonghojin/claude-skills.git"
  branch: main
```

## 설치

`~/.claude/settings.json`에 추가:

```json
{
  "extraKnownMarketplaces": {
    "nkia-skills": {
      "source": {
        "source": "git",
        "url": "https://github.com/gonghojin/claude-skills.git"
      }
    }
  }
}
```
