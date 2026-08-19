# Second Brain — Workflow Automation Setup

3대 기기(연구실 PC · 노트북 · 맥미니) + Obsidian 기반 Second Brain 구축 프로젝트.

## 컨텍스트 파일

| 파일 | 내용 |
|------|------|
| `.omc/specs/deep-interview-workflow-secondbrain.md` | Deep Interview 결과 — 전체 요구사항 |
| `.omc/plans/workflow-secondbrain-plan.md` | 5단계 실행 계획 |
| `.omc/handoffs/team-plan.md` | 주요 결정사항 요약 |
| `.omc/handoffs/phase1-lab-pc.md` | 연구실 PC Phase 1(Syncthing) 완료 기록 — 노트북도 동일 절차 |
| `.omc/handoffs/phase1-laptop.md` | 노트북 Phase 1 완료 기록 — OneDrive 경로 분리 이슈 포함 |

## 다른 기기(노트북 등)에서 이어받는 법

노트북도 연구실 PC와 동일하게 **WSL Ubuntu + Claude Code**로 작업할 예정.

```bash
git clone https://github.com/Aa-gif-beep/Second_brain.git
cd Second_brain
claude  # Claude Code 실행
```

Claude Code에서:
> `.omc/handoffs/phase1-lab-pc.md` 읽고, 연구실 PC와 같은 방식으로 이 기기에 Syncthing Phase 1 설정해줘. (플랜 원본은 `.omc/plans/workflow-secondbrain-plan.md`)

(참고: 최종 확정 계획(`workflow-secondbrain-plan.md`)은 Tailscale/Synology 대신 **Syncthing**을 채택했다 — 유료 도구·VPN 터널 배제, 3대 동일 LAN 직통 동기화. 이 파일의 이전 버전은 구버전 문구였음.)

## 진행 현황

- [x] Deep Interview 완료 (모호함 16.7%)
- [x] 실행 계획 수립
- [x] Phase 1 — 연구실 PC: Syncthing v2.1.3 설치·구동·자동시작 등록, `obsidian-vault` 폴더 추가 (2026-08-19)
- [x] Phase 1 — 노트북: Syncthing v2.1.3 설치·구동·자동시작 등록, `obsidian-vault` 폴더 추가, 연구실 PC를 원격 기기로 등록·폴더 공유 설정 (2026-08-19, `.omc/handoffs/phase1-laptop.md` 참조)
- [ ] Phase 1 — 맥미니: Syncthing 설치 필요
- [x] Phase 1 — 연구실 PC ↔ 노트북: 양쪽 다 CLI로 기기 등록 + 폴더 공유 설정 완료 (2026-08-19). 실시간 연결은 아직 안 됨 — 두 기기가 동시에 켜져 같은 네트워크에 있어야 붙음. 다음에 둘 다 켜지면 자동 연결·동기화 시작될 것 (수동 개입 불필요)
- [ ] Phase 1 — 연구실 PC ↔ 노트북 실시간 동기화 확인 (5분 내 반영 여부, 파일 하나 수정해서 테스트)
- [ ] Phase 2: Obsidian 구조 설정
- [ ] Phase 3: 음성 → Obsidian 파이프라인
- [ ] Phase 4: Claude RAG 연결
- [ ] Phase 5: 자동화 스케줄러
