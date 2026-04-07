---
name: skill-create
description: "[Skill] 표준 스킬 구조(SKILL.md + prompts/ + scripts/ + references/)를 자동 생성하는 스캐폴딩 도구."
argument-hint: <skill-name>
---

# skill-create

새 스킬의 표준 디렉토리 구조와 SKILL.md 보일러플레이트를 자동 생성합니다.

## Usage

```bash
/skill-create my-new-skill
```

**파라미터:**
- `<skill-name>` — 생성할 스킬 이름 (kebab-case 권장)

## Description

이 스킬은 `.claude/skills/{skill-name}/` 디렉토리를 생성하고, 표준 SKILL.md 템플릿과 하위 디렉토리를 자동으로 만들어줍니다.

생성 후 SKILL.md의 내용을 채우면 바로 `/skill-name` 으로 사용할 수 있습니다.

---

## Instructions

### Step 1: 입력 검증

1. `{{ARGUMENTS}}`에서 `<skill-name>`을 파싱한다.
2. 이름이 비어있으면 에러를 출력하고 중단한다:
   > "스킬 이름을 입력해주세요. 사용법: `/skill-create <skill-name>`"
3. `.claude/skills/{skill-name}/` 디렉토리가 이미 존재하면 에러를 출력하고 중단한다:
   > "⚠ `.claude/skills/{skill-name}/`이 이미 존재합니다. 다른 이름을 사용하거나 기존 스킬을 확인해주세요."

### Step 2: 디렉토리 구조 생성

다음 구조를 생성한다:

```
.claude/skills/{skill-name}/
├── SKILL.md          ← 보일러플레이트 (아래 템플릿)
├── prompts/          ← 프롬프트 파일 (필요 시)
│   └── .gitkeep
├── scripts/          ← 스크립트 파일 (필요 시)
│   └── .gitkeep
└── references/       ← 참조 문서 (필요 시)
    └── .gitkeep
```

### Step 3: SKILL.md 템플릿 생성

다음 내용으로 `SKILL.md`를 생성한다. `{skill-name}`을 실제 이름으로 치환:

```markdown
---
name: {skill-name}
description: "[Category] 이 스킬에 대한 한 줄 설명을 작성하세요."
argument-hint: <args>
---

# {skill-name}

이 스킬이 하는 일을 간략히 설명하세요.

## Usage

\```bash
/{skill-name} <args>
\```

**파라미터:**
- `<args>` — 파라미터 설명

## Description

스킬의 목적, 동작 방식, 출력물을 상세히 설명하세요.

## Instructions

### Step 1: (단계 제목)

1. 구체적인 동작을 기술하세요.

### Step 2: (단계 제목)

1. 구체적인 동작을 기술하세요.

## Examples

**기본 사용:**
\```bash
/{skill-name} example-arg
\```

## Notes

- 추가 참고사항이 있으면 기술하세요.
```

### Step 4: 완료 안내

생성 결과를 출력한다:

```
✅ 스킬 '{skill-name}'이 생성되었습니다.

생성된 파일:
  .claude/skills/{skill-name}/SKILL.md
  .claude/skills/{skill-name}/prompts/.gitkeep
  .claude/skills/{skill-name}/scripts/.gitkeep
  .claude/skills/{skill-name}/references/.gitkeep

다음 단계:
  1. SKILL.md의 description, Instructions 등을 작성하세요.
  2. /{skill-name} 으로 실행할 수 있습니다.
  3. 완성 후 /skill-validate {skill-name} 으로 구조를 검증하세요.
  4. /skill-publish {skill-name} 으로 사내 repo에 공유할 수 있습니다.
```
