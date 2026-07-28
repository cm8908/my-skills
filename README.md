# my-skills

cm8908의 개인 **플러그인 마켓플레이스**. 커스텀 스킬과 자주 쓰는 서드파티 스킬을
한 레포에 모아두고, 새 서버/머신에서는 **이 레포 하나만 추가**하면 전부 딸려오게 하는 것이 목적.

```text
/plugin marketplace add cm8908/my-skills
/plugin install llm-wiki-meta@my-skills
/plugin install cm8908-custom@my-skills
/plugin install cm8908-thirdparty@my-skills
```

---

## 레포 구조

이 레포는 그 자체로 **하나의 마켓플레이스**(`.claude-plugin/marketplace.json`)이고,
그 안에 여러 **플러그인**을 담는다. 각 플러그인은 여러 **스킬**을 담는다.

```
my-skills/
├── .claude-plugin/
│   └── marketplace.json          # 어떤 플러그인들을 제공하는지 선언
├── plugins/
│   ├── llm-wiki-meta/            # [커스텀·완성품] Karpathy LLM 위키 관리 스킬 9종
│   │   ├── .claude-plugin/plugin.json
│   │   ├── lib/                  # 스킬들이 공유하는 bash 헬퍼
│   │   └── skills/<9개 스킬>/SKILL.md
│   ├── cm8908-custom/            # [커스텀·수집함] 단독 커스텀 스킬을 여기에 drop
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/               # skills/<이름>/SKILL.md 넣으면 자동 인식
│   └── cm8908-thirdparty/        # [서드파티·vendoring] 플러그인이 아닌 외부 스킬 복사본
│       ├── .claude-plugin/plugin.json
│       └── skills/               # 각 스킬 폴더에 SOURCE.md로 출처·커밋 기록
├── LICENSE
└── README.md                     # 이 문서
```

세 플러그인으로 나눈 이유:
- **`llm-wiki-meta`** — 공유 `lib/`를 갖는 하나의 완결된 제품이라 독립 플러그인 유지.
- **`cm8908-custom`** — 앞으로 만들 잡다한 단독 커스텀 스킬의 "집". 폴더만 넣으면 끝.
- **`cm8908-thirdparty`** — 외부에서 가져온(vendoring) 스킬. 출처/라이선스를 커스텀과 분리해 관리.

> 현재 `cm8908-custom`에는 5개(`delegate-or-promote`, `delegate-to-subagents`,
> `karpathy-guidelines`, `msjung-doc-style`, `plain-technical-voice`),
> `cm8908-thirdparty`에는 1개(`plain-english`)가 들어 있다.

---

## 새 서버에 세팅하기 (핵심 시나리오)

```text
# 1. 마켓플레이스 등록 (레포 하나만)
/plugin marketplace add cm8908/my-skills

# 2. 원하는 플러그인 설치 (한 번만)
/plugin install llm-wiki-meta@my-skills
/plugin install cm8908-custom@my-skills
/plugin install cm8908-thirdparty@my-skills
```

Claude Code에는 "한 방에 전부 설치"하는 명령은 없다(플러그인 단위 install). 대신
자주 쓰는 프로젝트라면 그 프로젝트의 `.claude/settings.json`에 아래를 넣어
**설치된 플러그인을 강제로 켜둘** 수 있다:

```json
{
  "enabledPlugins": {
    "llm-wiki-meta@my-skills": true,
    "cm8908-custom@my-skills": true,
    "cm8908-thirdparty@my-skills": true
  }
}
```

---

## 커스텀 스킬 추가하기 — **가장 편한 경로**

핵심: 플러그인은 `skills/` 아래 `SKILL.md`가 있는 모든 하위폴더를 **자동 인식**한다.
매니페스트(`marketplace.json` / `plugin.json`)를 건드릴 필요가 **없다**.

```bash
# 1) cm8908-custom 안에 스킬 폴더 하나 만들기
mkdir -p plugins/cm8908-custom/skills/my-new-skill
$EDITOR plugins/cm8908-custom/skills/my-new-skill/SKILL.md
#   (필요하면 run.sh 등 스크립트/자산을 같은 폴더에)

# 2) 커밋 & 푸시
git add plugins/cm8908-custom/skills/my-new-skill
git commit -m "feat(custom): add my-new-skill"
git push
```

각 서버에서 반영:

```text
/plugin marketplace update my-skills     # 최신 커밋 pull
# cm8908-custom 을 아직 install 안 했다면 최초 1회만:
/plugin install cm8908-custom@my-skills
```

이미 `cm8908-custom`이 설치돼 있으면 **재설치 불필요** — `marketplace update`만 하면
새 스킬이 슬래시 메뉴에 뜬다.

### SKILL.md 최소 형식

```markdown
---
name: my-new-skill
description: <이 스킬을 언제 써야 하는지를 트리거 문구 위주로. Claude는 로드 여부를 결정할 때 이 description만 본다.>
---

# my-new-skill

## Contract
<무엇을 하는지 한두 문장>

## How To Run
<결정적 작업이 있으면 스크립트로 빼고 `bash "$SKILL_DIR/run.sh"` 처럼 호출>
```

- `name`은 kebab-case, 폴더명과 맞추는 걸 권장.
- 결정적(deterministic) 로직은 `SKILL.md`에 인라인 bash로 쓰지 말고 `run.sh` 같은
  스크립트로 빼면 호출이 빠르고 권한 프롬프트 표면이 작아진다 (llm-wiki-meta가 이 패턴).
- 스킬 하나 만드는 게 처음이면 `/write-a-skill` 스킬을 쓰면 골격을 잡아준다.

> 도메인이 뚜렷하게 다른 스킬 묶음(예: `k8s-*`)이 여러 개 쌓이면, `cm8908-custom`에
> 다 넣지 말고 새 플러그인 `plugins/<도메인>/`으로 분리하고 `marketplace.json`에
> 항목을 하나 추가하면 된다. (아래 "새 플러그인 추가" 참고)

---

## 서드파티 스킬 추가하기 — 두 가지 경로

먼저 판단: **업스트림이 어떤 형태로 배포되나?**

| 업스트림 형태 | 권장 경로 |
| --- | --- |
| 그 자체가 플러그인/마켓플레이스 (`plugin.json` 또는 `marketplace.json` 있음) | **A. 참조(reference)** |
| 모노레포 하위폴더에 플러그인/`skills/`가 들어 있음 | **A. 참조(git-subdir)** |
| 그냥 `SKILL.md` 낱개 파일들 (플러그인 아님) | **B. vendoring(복사)** |
| 오프라인/완전 자급자족이 최우선, 업스트림 변화 신경 안 씀 | **B. vendoring** |

> ⚠️ **git submodule은 쓰지 말 것.** `/plugin marketplace add`가 clone할 때
> `--recurse-submodules`를 하는지 문서화돼 있지 않아, 서브모듈 내용이 서버에서
> 비어버릴 수 있다. 참조(A) 또는 vendoring(B)만 사용한다.

### 경로 A — 참조 (업데이트 자동, 복사 안 함)

루트 `.claude-plugin/marketplace.json`의 `plugins` 배열에 항목을 추가한다.
`source`가 원격을 가리키면, 사용자가 install할 때 Claude Code가 업스트림을 알아서 당겨온다.

전체 레포가 플러그인인 경우 (`github`):

```jsonc
{
  "name": "some-tool",
  "source": { "source": "github", "repo": "owner/some-tool", "ref": "main" },
  "description": "…"
}
```

모노레포의 하위폴더인 경우 (`git-subdir` — `path` 지원). 이 레포에 실제로 등록된
AKB 예시:

```jsonc
{
  "name": "akb-wiki",
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/dnotitia/akb.git",
    "path": "plugins/claude/akb-wiki",
    "ref": "main"
  },
  "description": "[third-party: dnotitia/akb] …"
}
```

- **재현성**을 원하면 `ref`(브랜치/태그) 대신 `sha`(40자 커밋)로 **핀 고정**한다.
  서버마다 같은 버전이 보장된다.
- 항목 추가 후 `marketplace.json`이 여전히 valid JSON인지 확인할 것 —
  source 하나라도 못 풀면 마켓플레이스 전체 로드가 실패한다.
- 반영: `git push` → 각 서버에서 `/plugin marketplace update my-skills` →
  `/plugin install <name>@my-skills`.

#### 현재 참조 중인 서드파티 (경로 A)

[`dnotitia/akb`](https://github.com/dnotitia/akb) skillpack의 세 플러그인을
`git-subdir`로 참조한다 (각 플러그인은 `plugins/claude/<name>/`에서 self-contained):

| 플러그인 | 설명 |
| --- | --- |
| `akb-wiki` | AKB 볼트 문서 ingest/query. 공유 `akb` MCP 서버 탑재 — **먼저 설치**. |
| `akb-sessions` | 코딩 세션을 구조화 노트(TIL/task/idea/decision)로 볼트에 기록. |
| `akb-claude-code` | 세션 라이프사이클 훅 — 세션 시작 시 메모리 주입, compaction 전 스냅샷, 종료 시 recap. |

```text
/plugin marketplace update my-skills
/plugin install akb-wiki@my-skills        # 먼저 (MCP 서버 포함)
/plugin install akb-sessions@my-skills
/plugin install akb-claude-code@my-skills
```

설치 시 `AKB_MCP_URL`(끝이 `/mcp/`)과 `AKB_PAT`를 물어본다. `dnotitia/akb`는 public
이라 참조 fetch에는 별도 인증이 필요 없다(참조는 install 시점에만 해석됨).

> **기존 `akb-skillpack` 정리(선택):** 이미 `dnotitia/akb`를 직접 마켓플레이스로
> 추가해 `akb-*@akb-skillpack`을 설치한 상태라면, 같은 스킬/MCP가 중복 로드된다.
> "레포 하나로 통일"하려면 옛것을 내린다:
> ```text
> /plugin uninstall akb-wiki@akb-skillpack
> /plugin uninstall akb-sessions@akb-skillpack
> /plugin uninstall akb-claude-code@akb-skillpack
> /plugin marketplace remove akb-skillpack
> ```
> 반대로 dnotitia 공식 마켓플레이스를 직접 쓰는 게 편하면 그대로 둬도 된다 —
> 그 경우 my-skills의 akb-* 참조는 install하지 않으면 그만이다.

[`mattpocock/skills`](https://github.com/mattpocock/skills)의 `mattpocock-skills`
플러그인(약 22개 스킬: grilling, to-spec/to-tickets/triage, tdd, code-review,
domain-modeling, codebase-design, implement, prototype, research, handoff 등, MIT)을
레포 루트 플러그인이라 `github` 소스로 참조한다.

```text
/plugin marketplace update my-skills
/plugin install mattpocock-skills@my-skills   # /mattpocock-skills:<name> 으로 노출
```

> **중복 주의:** matt pocock 스킬을 `setup-matt-pocock-skills`나 `~/.claude/skills`
> 심볼릭 링크로 이미 쓰고 있다면, 플러그인 설치 시 같은 스킬이 두 벌 뜬다. "레포
> 하나로 통일"하려면 기존 링크/사본(예: `~/.claude/skills/{tdd,grill-me,handoff,…}`)을
> 정리하고 플러그인 쪽만 남기는 걸 권장한다. 플러그인은 `/mattpocock-skills:` 네임스페이스로
> 뜨므로 이름이 완전히 겹치지는 않지만, 내용이 동일해 혼선을 준다.

[`epoko77-ai/im-not-ai`](https://github.com/epoko77-ai/im-not-ai)의 `humanize-korean`
플러그인(AI가 쓴 한글을 사람 글처럼 윤문 — `humanize`(빠른 monolith)·`humanize-korean`(엄격
5단계 파이프라인)·`humanize-redo` 3종 스킬 + 12개 서브에이전트, 10대 카테고리 40+ 번역투/AI 티
탐지·재작성, MIT)을 레포 루트 플러그인이라 `github` 소스로 참조한다. 업스트림 `plugin.json`이
`skills`를 `./.claude/skills/`로 이미 선언하므로 별도 skills override는 불필요.

[`Imbad0202/academic-research-skills`](https://github.com/Imbad0202/academic-research-skills)의
`academic-research-skills` 플러그인(학술 연구 파이프라인: research → write → review → revise →
finalize, 4종 스킬 `academic-paper`/`academic-paper-reviewer`/`academic-pipeline`/`deep-research`,
27 modes, 39-agent 앙상블 + claim-faithfulness·citation-verification 게이트, **CC-BY-NC-4.0
비상업**)도 `github` 소스로 참조한다.

> ⚠️ **skills override 필요.** 이 업스트림은 스킬을 레포 **최상위**(`academic-paper/` 등)에 두면서
> 루트 `plugin.json`에는 `skills` 필드가 **없다**(자기네 `marketplace.json` 항목에만 skills를 나열).
> `github` 소스는 대상 레포의 `plugin.json`을 읽으므로, 그대로 두면 스킬이 **0개** 로드된다.
> 그래서 우리 `marketplace.json` 항목에 `"skills": ["./academic-paper", …]`를 직접 넣어 경로를
> 알려준다(업스트림이 자기 마켓플레이스에서 쓰는 것과 동일한 필드라 스키마상 유효).

[`blader/humanizer`](https://github.com/blader/humanizer)의 `humanizer` 플러그인(영문 산문에서
AI 티 제거 — Wikipedia "Signs of AI writing"(WikiProject AI Cleanup) 기반 20여 개 패턴군 +
before/after 예시, 사용자 글 샘플로 voice calibration, 무미건조해지지 않게 하는 PERSONALITY
AND SOUL 패스, MIT)을 레포 루트가 곧 플러그인이라 `github` 소스로 참조한다.
`humanize-korean`(한글)의 영어판 짝이다.

[`conorbronsdon/avoid-ai-writing`](https://github.com/conorbronsdon/avoid-ai-writing)의
`avoid-ai-writing` 플러그인(영문 AI-ism 감사·재작성 — detect/rewrite/edit 3모드, voice 프로파일
5종, `--context linkedin|blog|docs|…` 프리셋, iterate-to-convergence, 결정론적 JS 패턴
디텍터 동봉, MIT)은 모노레포 하위폴더(`plugins/avoid-ai-writing`)라 `git-subdir`로 참조한다.

> **`humanizer` vs `avoid-ai-writing` vs `plain-english`:** 셋 다 "AI 티 제거"로 겹친다.
> 골라 쓰는 기준 — 넓고 예시 풍부한 기본값은 `humanizer`, 파일 in-place 수정·감사만·채널별
> 톤 프리셋이 필요하면 `avoid-ai-writing`, AI 티가 아니라 **문장 자체를 조이고 싶으면**
> (수동태·추상명사 주어·라틴계 군더더기·죽은 은유 = Orwell/Gowers 계열) vendoring한
> `plain-english`. 셋을 동시에 켜두면 트리거 문구가 서로 겹치니, 상시로는 하나만 켜는 걸 권한다.

```text
/plugin marketplace update my-skills
/plugin install humanize-korean@my-skills            # /humanize-korean:<name> 으로 노출
/plugin install humanizer@my-skills                  # /humanizer:humanizer
/plugin install avoid-ai-writing@my-skills           # /avoid-ai-writing:avoid-ai-writing
/plugin install academic-research-skills@my-skills   # /academic-research-skills:<name> 으로 노출
```

### 경로 B — vendoring (자급자족, 수동 동기화)

업스트림 `SKILL.md`(+필요한 스크립트/자산)를 `cm8908-thirdparty/skills/<이름>/`에 복사하고,
같은 폴더에 `SOURCE.md`로 출처를 남긴다.

```bash
mkdir -p plugins/cm8908-thirdparty/skills/cool-skill
cp -R <upstream>/cool-skill/. plugins/cm8908-thirdparty/skills/cool-skill/
$EDITOR plugins/cm8908-thirdparty/skills/cool-skill/SOURCE.md
git add plugins/cm8908-thirdparty/skills/cool-skill
git commit -m "vendor(thirdparty): add cool-skill @<sha7>"
git push
```

`SOURCE.md` 템플릿:

```markdown
- upstream: https://github.com/<owner>/<repo>
- path: <레포 내 경로>
- commit: <복사한 시점의 40자 SHA>
- license: <SPDX id> (필요하면 upstream LICENSE도 같이 복사)
- vendored: <YYYY-MM-DD>
- notes: <로컬 수정 사항 있으면>
```

- 장점: 이 레포 하나로 완전 자급자족(오프라인 OK), 업스트림이 사라져도 안전.
- 단점: 업데이트가 수동 — 새 커밋에서 다시 복사하고 `SOURCE.md`의 SHA만 갱신.
- 라이선스 준수: 대부분의 스킬 라이선스는 원저작권/라이선스 고지 유지를 요구하므로
  `SOURCE.md`(및 필요 시 LICENSE 사본)를 반드시 남긴다.

**A vs B 요약:** 최신 유지가 중요하고 업스트림이 플러그인 형태다 → **A**.
완전 자급자족·오프라인·업스트림 소멸 대비가 중요하거나 낱개 SKILL.md다 → **B**.

#### 현재 vendoring한 서드파티 (경로 B)

| 스킬 | 위치 | 업스트림 | 라이선스 |
| --- | --- | --- | --- |
| `karpathy-guidelines` | `cm8908-custom/skills/` (행동 지침 계열이라 커스텀 쪽) | `multica-ai/andrej-karpathy-skills` | MIT |
| `plain-english` | `cm8908-thirdparty/skills/` | `b1rdmania/claude-plain-english-skill` | ⚠️ **없음** |

> ⚠️ **`plain-english`은 업스트림에 라이선스가 없다.** LICENSE 파일도, README의 SPDX
> 표기도 없다 — 즉 기본 저작권만 남아 있어 엄밀히는 재배포 근거가 없다. 개인용으로만
> 두고, 이 레포 밖으로 재배포하지 않는다. 저자가 비호환 라이선스를 명시하면 삭제한다.
> 낱개 `SKILL.md` + `REFERENCE.md` 두 파일뿐이라(`.claude-plugin/` 없음) 참조는 불가,
> vendoring이 유일한 경로였다.

---

## 새 플러그인(도메인 묶음) 추가하기

스킬이 많아져 도메인별로 나누고 싶을 때:

```bash
mkdir -p plugins/<domain>/.claude-plugin plugins/<domain>/skills
$EDITOR plugins/<domain>/.claude-plugin/plugin.json    # name, description 필수
```

그리고 루트 `marketplace.json`의 `plugins`에 `{ "name": "<domain>", "source": "./plugins/<domain>", "description": "…" }` 한 줄 추가 → 커밋/푸시 → `/plugin marketplace update my-skills`.

`plugin.json` 필드: `name`·`description`은 **필수**, 나머지(`version`, `author`,
`homepage`, `repository`, `license`, `keywords`)는 선택.

---

## 업데이트 / 동기화

| 상황 | 명령 |
| --- | --- |
| 새 커밋을 서버에 반영 | `/plugin marketplace update my-skills` |
| 새 플러그인을 처음 켜기 | `/plugin install <plugin>@my-skills` |
| 플러그인 끄기 | `/plugin uninstall <plugin>@my-skills` |
| 마켓플레이스 제거 | `/plugin marketplace remove my-skills` |

이미 install된 플러그인에 **스킬만 추가**했다면 `marketplace update`만으로 반영된다
(재설치 불필요, 자동 인식 덕분).

---

## 기존 `cm8908-llm-wiki` 마켓플레이스에서 이전 (정리)

`llm-wiki-meta`는 원래 별도 마켓플레이스(`cm8908/llm-wiki-meta`)에서 설치돼 있었다.
`my-skills`로 옮겼으니 **중복(같은 스킬 2벌)** 을 피하려면 옛것을 내린다:

```text
/plugin uninstall llm-wiki-meta@cm8908-llm-wiki
/plugin marketplace remove cm8908-llm-wiki
/plugin marketplace add cm8908/my-skills
/plugin install llm-wiki-meta@my-skills
```

옛 레포 `cm8908/llm-wiki-meta`는 GitHub에서 archive 하거나 README에 "moved to
cm8908/my-skills" 안내만 남겨도 된다(선택).

---

## 권한 프롬프트 줄이기

스킬의 결정적 작업은 install 경로의 bash 스크립트로 실행된다. 매번 뜨는 Bash 권한
프롬프트를 없애려면 `~/.claude/settings.json`에 한 번 allowlist 해두면 된다:

```json
{
  "permissions": {
    "allow": [
      "Bash(bash ~/.claude/plugins/**/skills/*/*.sh:*)",
      "Bash(bash ~/.claude/plugins/**/lib/*.sh:*)"
    ]
  }
}
```

`/fewer-permission-prompts` 스킬이 자동으로 해줄 수도 있다.

---

## `source` 형식 치트시트

`marketplace.json`의 `plugins[].source`가 받는 형식:

```jsonc
"./plugins/foo"                                              // 이 레포 안의 로컬 경로
{ "source": "github", "repo": "owner/repo", "ref": "main" } // GitHub 전체 레포 (sha 핀 가능)
{ "source": "url", "url": "https://gitlab.com/x/y.git" }    // 임의 git URL
{ "source": "git-subdir", "url": "…", "path": "sub/dir" }   // 모노레포 하위폴더
{ "source": "npm", "package": "@org/name", "version": "…" } // npm 패키지
```

(`github`/`url`에는 `path` 필드가 없다 — 하위폴더는 반드시 `git-subdir` 사용.)

---

## License

MIT. `LICENSE` 참고. vendoring한 서드파티 스킬은 각 `SOURCE.md`에 명시된 원 라이선스를 따른다.
