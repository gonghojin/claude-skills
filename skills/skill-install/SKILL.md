---
name: skill-install
description: "[Skill] 사내 Git repo에서 스킬을 가져와 로컬 프로젝트에 설치."
argument-hint: <skill-name>
---

# skill-install

사내 GitLab 스킬 저장소에서 스킬을 가져와 현재 프로젝트의 `.claude/skills/`에 설치합니다.

## Usage

```bash
/skill-install generate-be
```

**파라미터:**
- `<skill-name>` — 설치할 스킬 이름 (사내 repo의 `skills/{skill-name}/`에 존재해야 함)

## Description

사내 스킬 저장소에서 지정된 스킬을 현재 프로젝트로 복사합니다. 설치 후 바로 `/{skill-name}`으로 사용할 수 있습니다.

어떤 스킬이 있는지는 `/skill-list --remote`로 확인할 수 있습니다.

---

## Instructions

### Step 1: 입력 검증

1. `{{ARGUMENTS}}`에서 `<skill-name>`을 파싱한다.
2. 이름이 비어있으면 에러 출력:
   > "스킬 이름을 입력해주세요. 사용법: `/skill-install <skill-name>`"
   > "사용 가능한 스킬 목록: `/skill-list --remote`"

### Step 2: 설정 파일 읽기

1. `.claude/skill-registry.yaml`을 Read 도구로 읽는다.
2. `registry.remote_url`과 `registry.branch`를 파싱한다.
3. 설정 파일이 없으면 에러 출력:
   > "⚠ `.claude/skill-registry.yaml`이 없습니다. 원격 스킬 저장소를 설정해주세요."
   > ```yaml
   > registry:
   >   remote_url: "https://your-gitlab.com/group/claude-skills.git"
   >   branch: main
   > ```

### Step 3: 사내 repo clone 및 스킬 확인

```bash
# 1. 임시 디렉토리 생성 및 shallow clone
tmp_dir=$(mktemp -d)
git clone --depth 1 --branch {branch} {remote_url} "$tmp_dir"
```

1. `$tmp_dir/skills/{skill-name}/SKILL.md`가 존재하는지 확인.
2. 존재하지 않으면 임시 디렉토리 정리 후 에러 출력:
   > "⚠ 원격 저장소에 '{skill-name}' 스킬이 없습니다."
   > "사용 가능한 스킬: `/skill-list --remote`"

### Step 4: 로컬 충돌 확인

1. `.claude/skills/{skill-name}/`이 이미 존재하면 사용자에게 확인:
   > "⚠ 로컬에 '{skill-name}' 스킬이 이미 존재합니다. 덮어쓰시겠습니까?"
2. AskUserQuestion 도구로 확인:
   - "덮어쓰기" → 기존 디렉토리를 삭제 후 복사
   - "취소" → 임시 디렉토리 정리 후 중단

### Step 5: 스킬 복사

```bash
# 기존 디렉토리가 있고 덮어쓰기 선택 시
rm -rf .claude/skills/{skill-name}

# 스킬 파일 복사
cp -r "$tmp_dir/skills/{skill-name}" .claude/skills/{skill-name}
```

### Step 6: 정리 및 완료 안내

```bash
rm -rf "$tmp_dir"
```

1. 설치된 스킬의 SKILL.md에서 `name`, `description`, Usage 섹션을 읽는다.
2. 완료 메시지 출력:

```
✅ '{skill-name}' 스킬이 설치되었습니다.

  설치 경로: .claude/skills/{skill-name}/
  설명: {description}

사용법:
  /{skill-name} {argument-hint}

스킬 목록 확인:
  /skill-list
```

## Examples

**스킬 설치:**
```bash
/skill-install generate-be
```

**원격에 없는 스킬:**
```
⚠ 원격 저장소에 'non-existent-skill' 스킬이 없습니다.
사용 가능한 스킬: /skill-list --remote
```

**로컬 충돌:**
```
⚠ 로컬에 'generate-be' 스킬이 이미 존재합니다. 덮어쓰시겠습니까?
  [덮어쓰기] [취소]
```

## Notes

- 사내 GitLab에 대한 Git 인증(SSH key 또는 HTTPS token)이 설정되어 있어야 합니다.
- `.claude/skill-registry.yaml`에 원격 repo URL이 설정되어 있어야 합니다.
- 설치된 스킬은 바로 `/{skill-name}`으로 사용 가능합니다 (Claude Code가 자동 디스커버리).
- 어떤 스킬이 있는지 먼저 `/skill-list --remote`로 확인하세요.
