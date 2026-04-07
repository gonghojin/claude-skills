---
name: skill-list
description: "[Skill] 로컬/원격 스킬 카탈로그 조회."
argument-hint: "[--remote | --all]"
---

# skill-list

로컬 프로젝트와 사내 원격 저장소의 스킬 목록을 조회합니다.

## Usage

```bash
/skill-list              ← 로컬 스킬 목록
/skill-list --remote     ← 사내 원격 repo 스킬 목록
/skill-list --all        ← 로컬 + 원격 비교 (설치 여부 표시)
```

**파라미터:**
- `--remote` — 사내 스킬 repo의 원격 목록만 조회
- `--all` — 로컬 + 원격을 모두 출력하고 설치 여부를 표시
- (없음) — 로컬 스킬만 출력 (기본값)

## Description

`.claude/skills/` 디렉토리를 스캔하여 로컬 스킬 목록을 보여주고, `--remote` 옵션으로 사내 GitLab 스킬 repo의 AGENTS.md를 읽어 원격 스킬 목록을 조회합니다.

---

## Instructions

### Step 1: 모드 결정

1. `{{ARGUMENTS}}`를 파싱한다.
2. `--remote`면 원격 모드, `--all`이면 비교 모드, 그 외 로컬 모드.

### Step 2: 로컬 스킬 수집 (로컬 모드 / 비교 모드)

1. Glob 도구로 `.claude/skills/*/SKILL.md` 패턴을 검색한다.
2. 각 SKILL.md의 YAML frontmatter에서 `name`, `description`을 파싱한다.
3. 파싱 실패한 파일은 `(파싱 실패)` 표시.

### Step 3: 원격 스킬 수집 (원격 모드 / 비교 모드)

1. `.claude/skill-registry.yaml` 파일을 읽어 `registry.remote_url`과 `registry.branch`를 가져온다.
2. 설정 파일이 없으면 안내 메시지 출력 후 중단:
   > "⚠ `.claude/skill-registry.yaml`이 없습니다. 원격 스킬 저장소를 설정해주세요."
3. Bash 도구로 사내 repo를 임시 디렉토리에 shallow clone한다:
   ```bash
   tmp_dir=$(mktemp -d)
   git clone --depth 1 --branch {branch} {remote_url} "$tmp_dir" 2>/dev/null
   ```
4. `$tmp_dir/AGENTS.md` 파일을 읽어 스킬 목록을 파싱한다.
   - AGENTS.md가 없으면 `$tmp_dir/skills/*/SKILL.md`를 직접 스캔한다.
5. 임시 디렉토리를 정리한다:
   ```bash
   rm -rf "$tmp_dir"
   ```

### Step 4: 결과 출력

**로컬 모드:**

```
📋 로컬 스킬 목록 ({n}개)

| # | 스킬 이름 | 설명 |
|---|----------|------|
| 1 | extract-api-spec | [Planning] 기획서에서 OpenAPI 3.0 Spec 추출... |
| 2 | generate-be | [BE] OpenAPI Spec 기반 Spring Boot 코드... |
| ... | ... | ... |
```

**원격 모드:**

```
📋 원격 스킬 목록 ({n}개) — {remote_url}

| # | 스킬 이름 | 설명 |
|---|----------|------|
| 1 | extract-api-spec | [Planning] 기획서에서 OpenAPI 3.0 Spec 추출... |
| ... | ... | ... |
```

**비교 모드 (`--all`):**

```
📋 스킬 카탈로그 (로컬: {n}개, 원격: {m}개)

| # | 스킬 이름 | 설명 | 상태 |
|---|----------|------|------|
| 1 | extract-api-spec | [Planning] 기획서에서... | ✅ 설치됨 |
| 2 | generate-be | [BE] OpenAPI Spec... | ✅ 설치됨 |
| 3 | some-remote-skill | [Tool] 원격에만 있는... | 📦 미설치 |
| ... | ... | ... | ... |
```

## Examples

**로컬 목록 보기:**
```bash
/skill-list
```

**원격 저장소 스킬 확인:**
```bash
/skill-list --remote
```

**설치 현황 비교:**
```bash
/skill-list --all
```

## Notes

- 원격 조회 시 네트워크 접근이 필요합니다. 사내 GitLab에 접근 가능해야 합니다.
- `.claude/skill-registry.yaml`에 원격 repo URL이 설정되어 있어야 `--remote`/`--all` 동작합니다.
- 로컬 모드는 설정 파일 없이도 항상 동작합니다.
