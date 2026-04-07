# 사내 스킬 관리 도구 카탈로그

사내 Claude Code 스킬의 라이프사이클(생성 → 검증 → 배포 → 설치)을 관리하는 도구 모음입니다.

## 스킬 목록

| 스킬 | 설명 | 사용법 |
|------|------|--------|
| [skill-create](skill-create/SKILL.md) | 표준 스킬 구조 자동 생성 (스캐폴딩) | `/nkia-skills:skill-create <name>` |
| [skill-validate](skill-validate/SKILL.md) | SKILL.md 필수 섹션 검증 (품질 게이트) | `/nkia-skills:skill-validate [name]` |
| [skill-list](skill-list/SKILL.md) | 로컬/원격 스킬 카탈로그 조회 | `/nkia-skills:skill-list [--remote\|--all]` |
| [skill-publish](skill-publish/SKILL.md) | 사내 Git repo에 스킬 배포 | `/nkia-skills:skill-publish <name>` |
| [skill-install](skill-install/SKILL.md) | 사내 repo에서 스킬 설치 | `/nkia-skills:skill-install <name>` |

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
  remote_url: "https://github.com/gongdel/claude-skills.git"
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
        "url": "https://github.com/gongdel/claude-skills.git"
      }
    }
  }
}
```
