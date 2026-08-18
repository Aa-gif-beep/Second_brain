# Deep Interview Spec: 3-Device Workflow Automation + Second Brain (Obsidian)

## Metadata
- Interview ID: wf-sb-2026-0819
- Rounds: 12
- Final Ambiguity Score: 16.7%
- Type: greenfield
- Generated: 2026-08-19
- Threshold: 0.20 (20%)
- Threshold Source: default
- Initial Context Summarized: no
- Status: PASSED

## Clarity Breakdown
| Dimension | Score | Weight | Weighted |
|-----------|-------|--------|----------|
| Goal Clarity | 0.855 | 0.40 | 0.342 |
| Constraint Clarity | 0.835 | 0.30 | 0.2505 |
| Success Criteria | 0.800 | 0.30 | 0.240 |
| **Total Clarity** | | | **0.8325** |
| **Ambiguity** | | | **16.7%** |

## Topology

| Component | Status | Description | Coverage |
|-----------|--------|-------------|----------|
| Multi-Device Workflow | active | 3대 기기(연구실·노트북·맥미니) 간 파일 동기화 및 자동화 파이프라인 | 동기화 대상 확인, LAN 환경 확인, OS 확인 |
| Second Brain (Obsidian) | active | Obsidian 기반 개인 지식관리: 연구노트·할일·독서/아이디어·일정 + AI 컨텍스트 레이어 | 도구 확정, 입력 파이프라인 확인, AI RAG 목표 확인 |

## Goal

**3대 기기(연구실 PC, 노트북, 맥미니)가 항상 동기화된 상태를 유지하며,** 어떤 기기를 켜도 Obsidian Vault·실험 데이터·논문 PDF에 즉시 접근하고 이전 작업을 자연스럽게 이어받을 수 있다. 동시에, **음성 녹음이 자동으로 전사·정리되어 Obsidian에 삽입**되고, Claude 등 AI에게 작업을 지시할 때 **Vault 전체를 컨텍스트로 활용**하여 과거 실험·기록에 기반한 응답을 받을 수 있는 개인 지식 시스템을 구축한다.

## Environment (Constraints)

- **기기 & OS**
  - 연구실 PC: Windows / Ubuntu (dual boot)
  - 노트북: Windows / Ubuntu (dual boot)
  - 맥미니: macOS (메인 처리 허브)
- **네트워크**: 3대 모두 동일 LAN/Wi-Fi → SSH, Syncthing 직접 통신 가능
- **Second Brain 도구**: Obsidian (확정)
- **AI 처리**: 클라우드 API 사용 가능 (OpenAI Whisper API, Claude API)
- **사용자 역할**: 코드 작성 없음, 실험 위주 연구자

## Non-Goals

- 연구실 PC에서 원격으로 코드를 실행하거나 빌드하는 것
- 외부 인터넷에서 기기에 직접 접근 (VPN 터널 등)
- 맞춤형 앱 개발 또는 복잡한 프로그래밍
- 협업/팀 기능 (개인 단독 사용)

## Acceptance Criteria

### Multi-Device Workflow
- [ ] Obsidian Vault가 3대 기기에 실시간(5분 이내) 동기화된다
- [ ] 실험 데이터/결과 파일이 3대 기기에서 접근 가능하다
- [ ] 논문 PDF / 참고자료가 3대 기기에서 접근 가능하다
- [ ] 자동 백업 스케줄(일 1회 이상)이 실행된다
- [ ] 기기 전환 시 이전 작업 맥락을 5분 이내에 파악할 수 있다

### Second Brain
- [ ] 음성 녹음 파일 → Whisper API 전사 → Claude API 구조화 → Obsidian 노트 자동 삽입 파이프라인 동작
- [ ] Daily Note가 자동 생성되며 일정·할 일·오늘 실험 항목이 포함된다
- [ ] Obsidian Tasks 플러그인으로 할 일 관리 가능
- [ ] Claude(또는 MCP)가 Vault 전체를 컨텍스트로 읽고 질문에 답할 수 있다
- [ ] 실험 노트, 읽은 논문, 아이디어, 일정이 각각 분리된 구조로 저장된다

## Proposed Architecture

### 1. 파일 동기화 — Syncthing
- **도구**: Syncthing (오픈소스, 크로스플랫폼, LAN 직접 통신)
- **동기화 폴더**:
  - `~/ObsidianVault/` → 3대 실시간 동기화
  - `~/ExperimentData/` → 3대 단방향 또는 양방향
  - `~/Papers/` → 3대 동기화
- Windows·Ubuntu·macOS 모두 Syncthing 클라이언트 설치
- 맥미니가 항상 켜져 있는 경우 허브 역할 가능

### 2. Obsidian 구조 설계
```
ObsidianVault/
├── Daily/           # 날짜별 일일 노트 (자동 생성)
├── Research/        # 실험 노트, 가설, 결과
├── Papers/          # 논문 요약 및 메모
├── Ideas/           # 아이디어 캡처
├── Tasks/           # 프로젝트별 할 일
└── Archive/         # 완료된 항목
```
- 플러그인: **Templater** (Daily Note 자동 템플릿), **Tasks**, **Dataview**, **Calendar**

### 3. 음성 → Obsidian 파이프라인
```
녹음 (폰/전용 레코더)
    ↓ AirDrop / Syncthing
맥미니 수신 폴더 감시 (Hazel 또는 cron)
    ↓
Whisper API 전사 (Python 스크립트)
    ↓
Claude API 구조화 (제목, 요약, 태그, 날짜 추출)
    ↓
Obsidian Daily Note 또는 Research/ 에 자동 삽입
```

### 4. AI 컨텍스트 레이어 (RAG)
- **Claude MCP + Obsidian**: MCP 서버가 Vault를 읽어 Claude에 컨텍스트 제공
  - 현재 Claude Code에서 직접 Vault 파일 참조 가능
- **대안**: Claude Projects에 Vault 주요 파일 업로드 후 Project 기반 대화
- **목표**: "2주 전 실험 결과가 어땠어?" 같은 질문에 Claude가 Vault를 검색해 답함

### 5. 자동화 스케줄러
- **맥미니**: launchd 또는 cron으로 일일 백업, 로그 정리, 파이프라인 실행
- **Windows**: Task Scheduler로 Syncthing 자동 시작, 백업 스크립트
- **Ubuntu**: systemd timer 또는 cron

## Key Tools Summary

| 역할 | 도구 | 이유 |
|------|------|------|
| 파일 동기화 | Syncthing | 크로스플랫폼, LAN 직통, 무료 |
| Second Brain | Obsidian | 사용자 확정, 로컬 Markdown |
| 음성 전사 | OpenAI Whisper API | 높은 정확도, 한국어 지원 |
| 노트 구조화 | Claude API | 사용자가 이미 사용 중 |
| AI 컨텍스트 | Claude MCP / Projects | Vault RAG |
| 자동화 트리거 | launchd (Mac), cron (Ubuntu), Task Scheduler (Win) | OS 기본 제공 |
| 파일 감시 | Hazel (Mac) 또는 Python watchdog | 녹음 파일 감지 자동 실행 |

## Assumptions Exposed & Resolved

| Assumption | Challenge | Resolution |
|------------|-----------|------------|
| "task 자동 실행 = 코드 실행" | 코드를 안 짠다고 밝힘 | task = 백업/정리/파이프라인 실행 |
| "외부에서도 접근 필요" | 모두 같은 LAN이면? | 3대 동일 LAN → 클라우드 중계 불필요 |
| "Second Brain은 Notion일 것" | 선호 도구 확인 | Obsidian 확정 |
| "AI 처리는 로컬만" | 프라이버시 우려 확인 | 클라우드 API 허용 |

## Interview Transcript
<details>
<summary>전체 Q&A (12라운드)</summary>

### Round 0
**Q:** Topology 확인 — Multi-Device Workflow + Second Brain 2개로 맞나요?
**A:** 맞아요 (2개 모두)

### Round 1
**Q:** 3대 기기 사이에서 자동화하고 싶은 작업이 구체적으로 뭐어요?
**A:** 자동으로 task를 실행할 수 있도록 하고 데이터 공유 등 실시간으로 가능하게

### Round 2
**Q:** Second Brain으로 주로 어떤 종류의 정보를 저장하고 싶어요?
**A:** 일정/시간 관리, 읽은 것/아이디어 정리, 할 일 관리, 연구/논문 노트 (4가지 모두)

### Round 3
**Q:** "task를 자동으로 실행한다"에서, 실제로 어떤 종류의 task를 말하는 거예요?
**A:** 정보와 할 일 정리, 로그 정리, 백업/동기화

### Round 4
**Q:** 3대 기기의 OS가 어떻게 돼요?
**A:** 연구실: Windows/Ubuntu, 노트북: Windows/Ubuntu, 맥미니: macOS

### Round 5
**Q:** Second Brain 도구로 어떤 걸 선호하세요?
**A:** Obsidian

### Round 6
**Q:** 3대 기기들이 네트워크적으로 어떻게 연결되나요?
**A:** 3대 모두 같은 Wi-Fi/LAN에

### Round 7
**Q:** 완성 후 매일 아침 어떤 일이 자동으로 일어나야 성공인가요?
**A:** Vault가 동기화되어 있으며, 작업을 자연스럽게 이어받을 수 있으며, 할 일과 한 일들이 자동으로 Obsidian으로 채워지는 것. 그리고 기록들로 미래 Task에 Context를 제공할 수 있는 것.

### Round 8
**Q:** Obsidian에 자동으로 들어와야 할 정보의 입력 소스는 어디인가요?
**A:** 코드 작업 없음. 실험 위주 연구. 스스로 입력하거나 녹음 기반으로 진행 예정.

### Round 9
**Q:** '녹음 기반 입력'의 시나리오는?
**A:** 실험 중 녹음 → 전사 → Obsidian / 일상 메모 → 정리 → Obsidian (둘 다)

### Round 10
**Q:** 3대 기기 사이에 주로 동기화하고 싶은 파일/데이터가 뭐어요?
**A:** Obsidian Vault 전체, 실험 데이터/결과 파일, 논문 PDF/참고자료

### Round 11
**Q:** "Obsidian 기록으로 미래 Task에 컨텍스트를 제공한다"는 게 어떤 모습이어야 해요?
**A:** AI에게 Task를 시켰을 때 나의 기존 Vault 전체를 살펴보고 Context를 파악해서 일을 진행해달라는 의미

### Round 12
**Q:** 실험 노트/녹음을 AI로 처리할 때, 클라우드 API를 써도 관찮은가요?
**A:** 클라우드 사용 관찮음 (OpenAI Whisper API, Claude API 등)

</details>
