# AI 개발 규칙 세트

Claude Code와 Codex로 개발하는 프로젝트에 공통으로 적용하는 규칙 모음이다. 특정 프로젝트에 묶이지 않으며, 프로젝트에 복사한 뒤 지정된 곳에만 프로젝트 값을 채운다.

## 1. 규칙 목록

| 파일 | 적용 대상 | 프로젝트 값을 채우는 곳 |
|---|---|---|
| [document-template.rule.md](rules/document-template.rule.md) | 모든 Markdown 문서 | - |
| [unclear.rule.md](rules/unclear.rule.md) | 모든 지시 | 명확화 기록 위치(§4.1, 선택) |
| [code-comments.rule.md](rules/code-comments.rule.md) | 모든 소스 코드 | - |
| [test-guide.rule.md](rules/test-guide.rule.md) | 모든 코드의 구현·테스트 | §2 도구 표와 커버리지 측정 대상 |
| [frontend.rule.md](rules/frontend.rule.md) | 프론트엔드 소스 | §6 프로필(P1~P17) |
| [frontend-architecture.rule.md](rules/frontend-architecture.rule.md) | FSD를 쓰는 프론트엔드 | frontend.rule.md P2·P4·P17 |
| [code-organization.rule.md](rules/code-organization.rule.md) | FSD를 쓰는 프론트엔드 | - |
| [figma-implementation.rule.md](rules/figma-implementation.rule.md) | Figma 기반 UI 구현 | frontend.rule.md P7·P8·P16 |

## 2. 채택 절차

1. `.ai/` 폴더를 프로젝트 루트에 복사한다. 규칙끼리 상대 경로로 링크하므로 `rules/` 안의 파일 이름과 위치를 바꾸지 않는다.
2. 쓰지 않는 규칙은 지워도 된다. 지우기 전에 다른 규칙과 이 문서에서 그 파일을 가리키는 링크를 정리한다.
3. §1 표의 "프로젝트 값을 채우는 곳"만 채운다. 정해지지 않은 값은 `[TBD]`로 둔다.
4. §3에 따라 에이전트가 규칙을 읽게 연결한다.

## 3. 에이전트 연결

규칙 전체는 약 720줄, 46 KB다. 본문을 지침 파일에 붙여 넣지 말고 경로 목록만 적어, 에이전트가 작업에 해당하는 규칙을 그때 읽게 한다.

- Codex: 루트 `AGENTS.md`에 아래 예시처럼 규칙 경로를 적는다. Codex 문서에는 `AGENTS.md` 안에서 다른 파일을 불러오는 문법이 없고, 모든 `AGENTS.md`의 합산 크기가 `project_doc_max_bytes`(기본 32 KiB)에 이르면 그 뒤의 파일은 더하지 않는다.
- Claude Code: `CLAUDE.md`가 없으면 `AGENTS.md`를 직접 읽는다(v2.1.277 이상). `CLAUDE.md`가 있으면 그 안에 `@AGENTS.md`를 넣어 같은 목록을 쓴다.
- Claude Code에서 `@`로 규칙 파일 자체를 불러오면 세션 시작 시 전부 로드되어 컨텍스트를 차지한다. 특정 경로에서만 필요한 규칙은 `.claude/rules/`에 두고 `paths` frontmatter로 범위를 좁힐 수 있다.

`AGENTS.md` 예시:

```markdown
## 프로젝트 규칙

작업 전에 해당하는 규칙 파일을 읽고 따른다.

- Markdown 문서 작성·수정: `.ai/rules/document-template.rule.md`
- 모호한 지시: `.ai/rules/unclear.rule.md`
- 코드 주석: `.ai/rules/code-comments.rule.md`
- 구현과 테스트: `.ai/rules/test-guide.rule.md`
- 프론트엔드 코드: `.ai/rules/frontend.rule.md` (함께 쓰는 규칙은 그 문서 §1)
```

## 4. 원본

- Claude Code 메모리(CLAUDE.md, `@` import, `.claude/rules/`, AGENTS.md 읽기) — https://code.claude.com/docs/en/memory (확인일 2026-10-01)
- Codex AGENTS.md(파일 탐색, `project_doc_max_bytes`) — https://learn.chatgpt.com/docs/agent-configuration/agents-md (확인일 2026-10-01)
