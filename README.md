# Second Brain — Workflow Automation Setup

3대 기기(연구실 PC · 노트북 · 맥미니) + Obsidian 기반 Second Brain 구축 프로젝트.

## 컨텍스트 파일

| 파일 | 내용 |
|------|------|
| `.omc/specs/deep-interview-workflow-secondbrain.md` | Deep Interview 결과 — 전체 요구사항 |
| `.omc/plans/workflow-secondbrain-plan.md` | 5단계 실행 계획 |
| `.omc/handoffs/team-plan.md` | 주요 결정사항 요약 |

## 연구실 컴퓨터에서 이어받는 법

```bash
git clone git@github.com:Aa-gif-beep/Second_brain.git
cd Second_brain
claude  # Claude Code 실행
```

Claude Code에서:
> `.omc/specs/deep-interview-workflow-secondbrain.md` 와 `.omc/plans/workflow-secondbrain-plan.md` 읽고, Phase 1 Tailscale + Synology Drive 설정부터 도와줘.

## 진행 현황

- [x] Deep Interview 완료 (모호함 16.7%)
- [x] 실행 계획 수립
- [ ] Phase 1: Tailscale + Synology Drive 연결
- [ ] Phase 2: Obsidian 구조 설정
- [ ] Phase 3: 음성 → Obsidian 파이프라인
- [ ] Phase 4: Claude RAG 연결
- [ ] Phase 5: 자동화 스케줄러
