---
name: skill-publish
description: "[Skill] 사내 Git repo에 스킬을 배포 — validate 자동 선행, AGENTS.md 카탈로그 갱신."
argument-hint: <skill-name>
---

# skill-publish

로컬 스킬을 사내 GitLab 스킬 저장소에 배포합니다. 배포 전 `/skill-validate`를 자동으로 실행하여 품질을 보장합니다.

## Usage

```bash
/skill-publish generate-be
```

**파라미터:**
- `<skill-name>` — 배포할 스킬 이름 (`.claude/skills/{skill-name}/` 에 존재해야 함)

## Description

이 스킬은 로컬 프로젝트의 `.claude/skills/{skill-name}/` 전체를 사내 GitLab 스킬 전용 repo에 push합니다. 다른 프로젝트에서 `/skill-install`로 설치할 수 있도록 합니다.

배포 전 자동으로 `/skill-validate`를 실행하여 SKILL.md 구조 검증을 통과해야만 배포가 진행됩니다.

---

## Instructions

### Step 1: 입력 검증

1. `{{ARGUMENTS}}`에서 `<skill-name>`을 파싱한다.
2. 이름이 비어있으면 에러 출력:
   > "스킬 이름을 입력해주세요. 사용법: `/skill-publish <skill-name>`"
3. `.claude/skills/{skill-name}/SKILL.md`가 존재하지 않으면 에러 출력:
   > "⚠ `.claude/skills/{skill-name}/`을 찾을 수 없습니다."

### Step 2: 사전 검증 (품질 게이트)

1. `/skill-validate {skill-name}`의 검증 로직을 실행한다.
   - SKILL.md를 읽어 frontmatter 필수 필드(name, description)와 body 필수 섹션(Usage/Instructions, Description/Purpose, Steps/Instructions/Workflow)을 검증.
2. **FAIL이면 publish를 중단**하고 검증 결과를 출력한다:
   > "❌ 검증 실패. 아래 항목을 수정한 후 다시 시도해주세요."
3. PASS면 계속 진행.

### Step 3: 설정 파일 읽기

1. `.claude/skill-registry.yaml`을 Read 도구로 읽는다.
2. `registry.remote_url`과 `registry.branch`를 파싱한다.
3. 설정 파일이 없거나 `remote_url`이 비어있으면 에러 출력:
   > "⚠ `.claude/skill-registry.yaml`이 없거나 `registry.remote_url`이 설정되지 않았습니다."
   > "다음 형식으로 설정해주세요:"
   > ```yaml
   > registry:
   >   remote_url: "https://your-gitlab.com/group/claude-skills.git"
   >   branch: main
   > ```

### Step 4: 사내 repo clone 및 스킬 복사

다음 Bash 명령을 순차 실행한다:

```bash
# 1. 임시 디렉토리 생성 및 shallow clone
tmp_dir=$(mktemp -d)
git clone --depth 1 --branch {branch} {remote_url} "$tmp_dir"

# 2. 스킬 디렉토리 복사
mkdir -p "$tmp_dir/skills/{skill-name}"
cp -r .claude/skills/{skill-name}/* "$tmp_dir/skills/{skill-name}/"
```

### Step 5: 원격에 동일 스킬 존재 시 확인

1. `$tmp_dir/skills/{skill-name}/SKILL.md`가 clone 시점에 이미 존재했으면, 사용자에게 확인:
   > "⚠ 원격 저장소에 '{skill-name}' 스킬이 이미 존재합니다. 덮어쓰시겠습니까?"
2. AskUserQuestion 도구로 확인:
   - "덮어쓰기" → 계속 진행
   - "취소" → 임시 디렉토리 정리 후 중단

### Step 6: AGENTS.md 카탈로그 갱신

1. `$tmp_dir/AGENTS.md` 파일을 읽는다 (없으면 새로 생성).
2. 해당 스킬의 항목을 추가하거나 갱신한다.
3. AGENTS.md 형식:

```markdown
# 사내 스킬 카탈로그

| 스킬 이름 | 설명 | 배포일 | 배포자 |
|----------|------|--------|--------|
| generate-be | [BE] OpenAPI Spec 기반 Spring Boot 코드 자동 생성 | 2026-04-07 | gongdel |
| extract-api-spec | [Planning] 기획서에서 OpenAPI 3.0 Spec 추출 | 2026-04-07 | gongdel |
```

4. 배포일은 오늘 날짜, 배포자는 `git config user.name` 값을 사용.
5. 이미 같은 이름의 행이 있으면 해당 행을 갱신 (새 행 추가 X).

### Step 7: Commit 및 Push

```bash
cd "$tmp_dir"
git add skills/{skill-name}/ AGENTS.md
git commit -m "publish: {skill-name} by {git-user}"
git push origin {branch}
```

### Step 8: 정리 및 완료 안내

```bash
rm -rf "$tmp_dir"
```

완료 메시지 출력:

```
✅ '{skill-name}' 스킬이 사내 저장소에 배포되었습니다.

  저장소: {remote_url}
  경로: skills/{skill-name}/
  카탈로그: AGENTS.md 갱신 완료

다른 프로젝트에서 설치:
  /skill-install {skill-name}
```

## Examples

**스킬 배포:**
```bash
/skill-publish generate-be
```

**검증 실패 시:**
```
❌ 검증 실패 — publish 중단

  ❌ frontmatter 'name' 필드가 없습니다.
  ❌ 'Instructions' 섹션이 없습니다.

수정 후 다시 실행해주세요: /skill-publish generate-be
```

## Notes

- 배포 전 `/skill-validate`가 자동 실행됩니다. FAIL이면 배포가 차단됩니다.
- 사내 GitLab에 대한 Git 인증(SSH key 또는 HTTPS token)이 설정되어 있어야 합니다.
- `.claude/skill-registry.yaml`에 원격 repo URL이 설정되어 있어야 합니다.
- 원격에 동일 스킬이 있으면 사용자 확인 후 덮어씁니다.
