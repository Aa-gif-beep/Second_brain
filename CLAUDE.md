# CLAUDE.md

이 파일은 이 저장소(`Second_brain/`)에서 작업할 때 Claude Code에게 주는 지침이다.

## 이 파일과 vault의 CLAUDE.md의 관계

`Second_brain/`은 원래 `ObsidianVault/` 안에 있지만, Vault의 Syncthing 동기화 범위에서는 **의도적으로 제외**돼 있다(`.stignore`에 `Second_brain` 한 줄로 제외 — git과 Syncthing이 같은 파일을 동시에 관리하면 `.git` 손상 위험이 있어서). 그래서 이 저장소는 **git push/pull로만 전파**되고, Syncthing 안전망이 없다.

- **연구실 PC·노트북처럼 `Second_brain/`이 Vault 안에 있는 기기**: Claude Code가 cwd 기준 상위 폴더까지 CLAUDE.md를 다 읽으므로, Vault 최상위 CLAUDE.md와 이 파일이 **동시에 로드**된다. 서로 다른 영역을 다루므로 모순 아님 — Vault 쪽은 연구 노트 하네스, 이 파일은 이 프로젝트 전용 규칙.
- **`git clone`만 한 기기(예: 맥미니를 새로 셋업할 때)**: 이 파일만 로드된다. Vault 자체가 로컬에 없을 수 있음.
- 그래서 **Vault 쪽에서만 실행 가능한 지시(예: `.stignore`에 줄 추가하는 것)는 이 파일에 넣지 않는다** — 그런 세션이 이 파일을 못 읽을 수도 있으니까. 그런 지시는 Vault 최상위 CLAUDE.md에 있다.

## 업데이트 하네스 — Second_brain 전용

Vault의 업데이트 하네스(연구 노트 자동 반영)는 여기 적용 안 된다. 대신:

1. **대화 중 이 프로젝트 관련 전략적 결정·진행상황이 나오면, 대화가 끝나기 전에 `README.md`의 해당 섹션(🎯 진짜 목표 / 열린 질문 / 결정 로그 / 진행 현황)에 반영한다.**
2. **세션 시작 시 `git pull origin main`을 먼저 한다** — 다른 기기 세션이 이미 push했을 수 있으니, README를 최신이라고 가정하기 전에 pull부터.
3. **세션 끝나기 전에 commit + push한다** — Syncthing 안전망이 없어서, push 안 하면 다른 기기엔 아무것도 전달 안 됨.
4. **동시 세션 충돌 시**: pull했는데 이미 다른 세션이 같은 섹션을 바꿔놨으면 덮어쓰지 말고 병합한다. 내용이 서로 모순되면 사용자에게 확인한다 (지어내지 말 것 — Vault CLAUDE.md의 절대 규칙과 동일한 원칙).

## 리뷰어(architect/critic) 브리핑용 배경 — ralplan 돌릴 때 재사용

이 프로젝트에 대해 `/oh-my-claudecode:ralplan`으로 계획을 검증할 때, architect/critic 서브에이전트는 매번 컨텍스트 없이 새로 시작한다. 매번 처음부터 설명 안 쓰려면 아래를 브리핑에 포함시킬 것:

- 3대 기기(연구실 PC, 노트북, 맥미니) + Obsidian Vault 기반 second brain 구축 프로젝트. 자세한 배경은 `.omc/specs/deep-interview-workflow-secondbrain.md`.
- **진짜 목표**: 어느 기기에서 Claude Code를 켜도 진행 중인 작업 맥락을 이어받는 것. Syncthing·Telegram·음성 파이프라인은 전부 수단.
- Vault 전체가 Syncthing `sendreceive` 폴더(`obsidian-vault`)이고, `Second_brain/`은 그 안에서 `.stignore`로 제외됨 — git으로만 전파.
- 현재 상태는 이 저장소의 `README.md` "진행 현황"·"결정 로그"·"열린 질문" 섹션이 최신 — 브리핑 작성 전에 먼저 읽을 것.
- 리뷰 프로토콜: Architect와 Critic은 같은 계획 스냅샷을 순차적으로(병렬 아님) 독립적으로 리뷰하고, 서로의 결과를 못 봄. 둘 다 완료된 후에만 Planner가 종합해서 다음 버전을 만듦.
