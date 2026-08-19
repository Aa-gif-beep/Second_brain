# Second Brain — Workflow Automation Setup

3대 기기(연구실 PC · 노트북 · 맥미니) + Obsidian 기반 Second Brain 구축 프로젝트.

## 🎯 진짜 목표

Syncthing 동기화·Telegram 체크인·음성 파이프라인은 전부 수단이다. **진짜 목표는: 어느 기기에서 Claude Code를 켜도, 지금 하고 있는 작업의 맥락을 최소한의 마찰로 이어받을 수 있는 것.**

- Vault(데이터)가 각 기기에 최신 상태로 있어야 함 → Phase 1 (Syncthing)
- Claude Code가 세션 시작 시 "지금 뭐가 진행 중인지" 재구성할 수 있는 문서가 있어야 함 → Vault의 `🏠 Home.md`(연구), 이 저장소의 `README.md`(이 프로젝트 자체)
- 그 문서가 항상 최신이어야 함 → 대화 끝나기 전에 반영하는 습관/규칙 (`CLAUDE.md` 참조)

## 열린 질문

- Telegram 일일 체크인 기능 — 응답이 실제로 어디에 기록될지는 **맥미니 Phase 1 완료 후**로 결정 보류 (맥미니가 받아서 직접 vault에 쓰고, Syncthing이 나머지 기기에 전파하는 구조가 유력)

## 결정 로그

| 날짜 | 결정 | 이유 |
|---|---|---|
| 2026-08-19 | 맥미니 = 자동화 허브 (연구실 PC 아님) | 전력 소비 적고 상시 가동 가능. "데이터 소스가 연구실 PC"는 허브 결정 근거가 아님 — Syncthing이 어차피 데이터를 복제해주므로, 진짜 기준은 "누가 24시간 안 꺼지고 리스너를 돌릴 수 있냐"임. 연구실 PC는 공용 기기라 꺼지거나 재부팅될 수 있음 |
| 2026-08-19 | Second_brain을 Vault 밖으로 옮기지 않고, 그대로 두되 Syncthing에서 폴더째 제외 | 사용자가 Vault 안에 있는 걸 선호함(컨텍스트 연속성 목적) — git과 Syncthing 이중 관리 위험은 `.stignore` 전체 제외로 해결 |
| (원안) | Obsidian Sync(유료) 대신 Syncthing 채택 | 무료, 크로스플랫폼, LAN 직통 |
| (원안) | iCloud 단독 대신 Syncthing으로 3대 통일 | 플랫폼 상관없이 하나의 동기화 메커니즘으로 통일 |

(원안 두 개는 `.omc/handoffs/team-plan.md`에서 이관 — 그 파일은 이제 역사적 기록)

## 컨텍스트 파일

| 파일 | 내용 |
|------|------|
| `CLAUDE.md` | 이 저장소 작업 규칙 — 업데이트 하네스, git pull/push 규율, ralplan 리뷰어 브리핑용 배경 |
| `.omc/specs/deep-interview-workflow-secondbrain.md` | Deep Interview 결과 — 전체 요구사항 |
| `.omc/plans/workflow-secondbrain-plan.md` | 5단계 실행 계획 |
| `.omc/handoffs/team-plan.md` | (역사적 기록) 초기 결정사항 — 최신은 위 결정 로그 참고 |
| `.omc/handoffs/phase1-lab-pc.md` | 연구실 PC Phase 1(Syncthing) 완료 기록 — 노트북도 동일 절차 |
| `.omc/handoffs/phase1-laptop.md` | 노트북 Phase 1 완료 기록 — OneDrive 경로 분리 이슈 포함 |
| `.omc/handoffs/phase1-mac-mini.md` | 맥미니 Phase 1 안내 (미실행) — brew/launchd 버전 |

## 다른 기기(노트북 등)에서 이어받는 법

노트북도 연구실 PC와 동일하게 **WSL Ubuntu + Claude Code**로 작업할 예정.

```bash
git clone https://github.com/Aa-gif-beep/Second_brain.git
cd Second_brain
claude  # Claude Code 실행
```

Claude Code에서:
> `CLAUDE.md`와 `.omc/handoffs/phase1-lab-pc.md` 읽고, 연구실 PC와 같은 방식으로 이 기기에 Syncthing Phase 1 설정해줘. (플랜 원본은 `.omc/plans/workflow-secondbrain-plan.md`)

**세션 시작하자마자 `git pull origin main`부터** — 다른 기기 세션이 이미 push했을 수 있음 (`CLAUDE.md` 규칙).

(참고: 최종 확정 계획(`workflow-secondbrain-plan.md`)은 Tailscale/Synology 대신 **Syncthing**을 채택했다 — 유료 도구·VPN 터널 배제, 3대 동일 LAN 직통 동기화. 이 파일의 이전 버전은 구버전 문구였음.)

## 진행 현황

- [x] Deep Interview 완료 (모호함 16.7%)
- [x] 실행 계획 수립
- [x] Phase 1 — 연구실 PC: Syncthing v2.1.3 설치·구동·자동시작 등록, `obsidian-vault` 폴더 추가 (2026-08-19)
- [x] Phase 1 — 노트북: Syncthing v2.1.3 설치·구동·자동시작 등록, `obsidian-vault` 폴더 추가, 연구실 PC를 원격 기기로 등록·폴더 공유 설정 (2026-08-19, `.omc/handoffs/phase1-laptop.md` 참조)
- [ ] Phase 1 — 맥미니: Syncthing 설치 필요 (`.omc/handoffs/phase1-mac-mini.md` 안내 작성 완료, 실행은 미완료)
- [x] Phase 1 — 연구실 PC ↔ 노트북: 양쪽 다 CLI로 기기 등록 + 폴더 공유 설정 완료 (2026-08-19). 실시간 연결은 아직 안 됨 — 두 기기가 동시에 켜져 같은 네트워크에 있어야 붙음. 다음에 둘 다 켜지면 자동 연결·동기화 시작될 것 (수동 개입 불필요)
- [ ] Phase 1 — 연구실 PC ↔ 노트북 실시간 동기화 확인 (5분 내 반영 여부, 파일 하나 수정해서 테스트)
- [x] Phase 2: Obsidian 구조 설정 — 최상위만 영문, 하위 폴더는 한글 유지로 확정 (2026-08-19). 플러그인 5개 미설치 확인, 🏠 Home.md에 체크리스트 추가함 (사용자가 직접 설치 필요)
- [ ] Phase 3: 음성 → Obsidian 파이프라인 — 실행 보류(맥미니·API 키 필요), 알려진 스크립트 버그 3개 문서에 주석 처리함 (2026-08-19)
- [ ] Phase 4: Claude RAG 연결
- [ ] Phase 5: 자동화 스케줄러
