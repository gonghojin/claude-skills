---
name: skill-validate
description: "[Skill] SKILL.md 필수 섹션 검증 — publish 전 품질 게이트."
argument-hint: <skill-name>
---

# skill-validate

스킬의 SKILL.md가 표준 구조를 충족하는지 검증합니다. `/skill-publish` 전에 자동으로 실행되는 품질 게이트입니다.

## Usage

```bash
/skill-validate my-skill
/skill-validate          ← 인자 없으면 전체 스킬 검증
```

**파라미터:**
- `<skill-name>` — 검증할 스킬 이름 (생략 시 `.claude/skills/` 내 전체 스킬 검증)

## Description

SKILL.md의 YAML frontmatter와 markdown body를 검사하여, 필수 필드/섹션이 존재하는지 확인합니다. 검증 결과를 등급별(CRITICAL / WARNING / PASS)로 출력합니다.

---

## Instructions

### Step 1: 대상 결정

1. `{{ARGUMENTS}}`에서 `<skill-name>`을 파싱한다.
2. 이름이 있으면 `.claude/skills/{skill-name}/SKILL.md` 하나만 검증.
3. 이름이 없으면 `.claude/skills/*/SKILL.md` 전체를 순회 검증.
4. 대상 SKILL.md 파일이 존재하지 않으면 에러 출력:
   > "⚠ `.claude/skills/{skill-name}/SKILL.md`를 찾을 수 없습니다."

### Step 2: SKILL.md 읽기 및 파싱

1. Read 도구로 SKILL.md 파일을 읽는다.
2. YAML frontmatter(`---` 사이)를 파싱한다.
3. Markdown body(frontmatter 이후)를 파싱한다.

### Step 3: Frontmatter 검증

다음 필드를 검사한다:

| 필드 | 등급 | 조건 |
|------|------|------|
| `name` | CRITICAL | 존재하고 비어있지 않을 것 |
| `description` | CRITICAL | 존재하고 비어있지 않을 것 |
| `argument-hint` | WARNING | 존재할 것 (없어도 PASS 가능) |

### Step 4: Body 섹션 검증

Markdown body에서 다음 섹션(## 레벨 헤딩)의 존재를 검사한다:

| 섹션 | 등급 | 조건 |
|------|------|------|
| Usage 또는 Instructions | CRITICAL | 둘 중 하나 이상 존재 |
| Description 또는 Purpose | CRITICAL | 둘 중 하나 이상 존재 |
| Steps 또는 Instructions 또는 Workflow | CRITICAL | 하나 이상 존재 (동작 절차) |

**참고:** 섹션 이름 매칭은 대소문자 무시, `##` 또는 `###` 레벨 모두 인정.

### Step 5: 결과 출력

검증 결과를 다음 형식으로 출력한다:

```
🔍 스킬 검증: {skill-name}

Frontmatter:
  ✅ name: "{name 값}"
  ✅ description: "{description 값}"
  ⚠️  argument-hint: 누락 (권장)

Body 섹션:
  ✅ Usage/Instructions: 존재
  ✅ Description/Purpose: 존재
  ✅ Steps/Instructions/Workflow: 존재

결과: ✅ PASS (CRITICAL 0개, WARNING 1개)
```

또는 실패 시:

```
🔍 스킬 검증: {skill-name}

Frontmatter:
  ❌ name: 누락 (필수)
  ✅ description: "{description 값}"

Body 섹션:
  ❌ Usage/Instructions: 누락 (필수)
  ✅ Description/Purpose: 존재

결과: ❌ FAIL (CRITICAL 2개)
  - frontmatter 'name' 필드가 없습니다.
  - 'Usage' 또는 'Instructions' 섹션이 없습니다.
```

### Step 6: 전체 검증 시 요약

인자 없이 전체 검증한 경우, 마지막에 요약을 출력한다:

```
📊 전체 검증 요약: {n}개 스킬 검증 완료

  ✅ PASS: {count}개
  ❌ FAIL: {count}개
  ⚠️  WARNING: {count}개 (PASS이지만 권장 항목 누락)

실패 스킬:
  - {skill-name-1}: CRITICAL 2개 (name 누락, Instructions 누락)
  - {skill-name-2}: CRITICAL 1개 (description 누락)
```

## Examples

**단일 스킬 검증:**
```bash
/skill-validate generate-be
```

**전체 스킬 검증:**
```bash
/skill-validate
```

## Notes

- `/skill-publish` 실행 시 이 검증이 자동으로 선행됩니다. FAIL이면 publish가 차단됩니다.
- 기존 11개 스킬에 대해 모두 PASS가 나와야 검증 기준이 올바른 것입니다.
- 스킬 관리 스킬(skill-create, skill-validate, skill-list, skill-publish, skill-install) 자체도 검증 대상입니다.
