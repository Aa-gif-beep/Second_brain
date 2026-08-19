# 실행 계획: 3-Device Workflow Automation + Second Brain

Source: `.omc/specs/deep-interview-workflow-secondbrain.md`
Created: 2026-08-19
Status: pending approval

---

## 환경 요약

| 기기 | OS | 역할 |
|------|-----|------|
| 연구실 PC | Windows / Ubuntu dual-boot | 실험 데이터 생성 |
| 노트북 | Windows / Ubuntu dual-boot | 이동 작업 |
| 맥미니 | macOS | 허브 · AI 처리 · 스케줄러 |
| 아이폰 | iOS | 녹음 캡처 |

네트워크: 3대 모두 동일 LAN

---

## Phase 1 — 파일 동기화 (Syncthing 기반)

**목표:** Obsidian Vault·실험 데이터·논문 PDF가 3대 기기에서 5분 이내 동기화

### 1-1. Syncthing 설치 (각 기기)

**맥미니 (macOS)**
```bash
brew install syncthing
brew services start syncthing
# 웹 UI: http://127.0.0.1:8384
```

**연구실 PC / 노트북 (Ubuntu)**
```bash
sudo apt install syncthing
sudo systemctl enable syncthing@$USER
sudo systemctl start syncthing@$USER
```

**연구실 PC / 노트북 (Windows)**
- syncthing.net 에서 설치 파일 다운로드
- Windows Startup 폴더에 등록 (`shell:startup`)

### 1-2. 동기화 폴더 설정 (맥미니 기준)

```
~/Sync/
├── ObsidianVault/          # 3대 ↔ 양방향
├── Papers/                 # 3대 ↔ 양방향
├── ExperimentData/         # 연구실 PC → 맥미니·노트북 단방향
└── IncomingRecordings/     # 아이폰 → 맥미니 단방향
```

**Syncthing 설정 원칙:**
- `ObsidianVault`, `Papers`: Ignore Patterns에 `.obsidian/workspace.json` 추가 (UI 충돌 방지)
- `ExperimentData`: Send Only (연구실 PC) / Receive Only (나머지)

### 1-3. 아이폰 → 맥미니 연결

옵션 A (권장, 무료): **iCloud Drive**
- 아이폰 Voice Memos → Files 앱 → iCloud Drive/Recordings/ 저장
- 맥미니: iCloud Drive 자동 다운로드 활성화

옵션 B: **Möbius Sync** (iOS Syncthing 앱, 유료 약 $6)
- `IncomingRecordings/` 폴더를 Syncthing으로 동기화

---

## Phase 2 — Obsidian 구조 설계

**목표:** 연구·할일·독서·일정이 체계적으로 정리되는 Vault

### 2-1. Vault 폴더 구조 (확정, 2026-08-19 — 아래 제안 대신 이 구조 채택)

원안(00-Inbox/10-Daily/20-Research 번호 체계)은 기각. 기존에 쓰던 한글 구조를 유지하면서 **최상위 폴더명만 영문 전환**하는 쪽으로 확정:

```
ObsidianVault/
├── Research/               # 구 연구/ — 진행 중인 연구 프로젝트
│   ├── ASC(Anode Supported Cell) 제작/   # 하위 폴더명은 한글 유지 (의도적 결정)
│   ├── COMSOL Trench/
│   ├── In-situ Raman/
│   ├── SOEC Station/
│   ├── SOFC ship/
│   └── 셀 아카이빙/
├── Research-Plan/          # 구 연구계획/ — 학위·진로 문서
├── Admin/                  # 구 행정/ — 행정 서류
├── _System/
│   ├── Daily/              # 구 _system/데일리/ — 날짜별 데일리 노트
│   └── _templates/
├── _Archive/                # 구 _보관/ — 참고자료 아카이브
├── Second_brain/           # 이 저장소. Syncthing에서 폴더째 제외됨(.stignore)
├── 🏠 Home.md               # 대시보드, 시작점
└── 📅 일정.md
```

**결정 이유**: 하위 폴더까지 전부 영문화하는 건 추가 작업 대비 이득이 없다고 판단 — 최상위 구조만 정리되면 탐색·자동화(Syncthing, 스크립트)엔 충분하고, 하위 폴더는 이미 익숙한 이름이라 그대로 두는 게 실용적.

### 2-2. 필수 플러그인 — 현재 미설치 확인됨 (2026-08-19)

`ObsidianVault/.obsidian/community-plugins.json` 확인 결과 **`table-editor-obsidian` 하나만 설치**되어 있고, 아래 5개는 전부 미설치. Obsidian 플러그인 설치는 GUI 조작이 필요해 Claude가 대신 할 수 없음 — 사용자가 직접 설치해야 함.

| 플러그인 | 역할 | 상태 |
|----------|------|------|
| **Templater** | Daily Note 자동 템플릿 적용 | ⬜ 미설치 |
| **Tasks** | 할 일 관리, 마감일, 필터 | ⬜ 미설치 |
| **Dataview** | 동적 쿼리 (오늘 할 일 모아보기 등) | ⬜ 미설치 |
| **Calendar** | 날짜 기반 탐색 | ⬜ 미설치 |
| **Periodic Notes** | Daily/Weekly/Monthly Note 자동화 | ⬜ 미설치 |

설치 방법: Obsidian 설정 → Community plugins → Browse → 위 5개 검색·설치·활성화.

### 2-3. Daily Note 템플릿 예시 (`Templates/Daily.md`)

```markdown
# {{date:YYYY-MM-DD}} Daily Note

## 오늘 일정
<%* 
// 캘린더에서 오늘 일정 자동 삽입 (Phase 4에서 구현)
%>

## 오늘 할 일
```tasks
due today
not done
```

## 오늘 한 일

## 실험 기록

## 메모
```

---

## Phase 3 — 음성 → Obsidian 파이프라인

**목표:** 녹음 파일이 맥미니에 도착하면 자동으로 전사·구조화하여 Obsidian에 삽입

**⚠️ 보류 중 (2026-08-19) — 실행 전 확인 필요한 것들:**
1. 맥미니가 아직 없음(Phase 1 미완료) — 온라인 되기 전엔 애초에 실행 불가
2. OpenAI/Anthropic API 키 아직 미발급 — 비용 발생 항목이라 발급 전 확인 필요 (인터뷰에서 원칙적 동의는 받음)
3. 아래 스크립트의 `model="claude-opus-4-8"`은 **존재하지 않는 모델 ID** — 실제 구현 시 그 시점의 최신 모델 ID로 교체할 것
4. 아래 스크립트의 `folder_map`은 Phase 2 확정 전(00-Inbox/20-Research 등 옛 구조 기준)에 작성됨 — 실제 구현 시 확정된 구조(`Research/`, `Research-Plan/`, `Admin/`, `_System/`, `_Archive/`)에 맞게 다시 설계해야 함, 지금 그대로 실행하면 존재하지 않는 폴더에 쓰게 됨

### 3-1. 처리 스크립트 (`~/scripts/process_recording.py`)

```python
import openai, anthropic, sys, os
from datetime import datetime
from pathlib import Path

VAULT = Path.home() / "Sync/ObsidianVault"
OPENAI_API_KEY = os.environ["OPENAI_API_KEY"]
ANTHROPIC_API_KEY = os.environ["ANTHROPIC_API_KEY"]

def transcribe(audio_path: str) -> str:
    client = openai.OpenAI(api_key=OPENAI_API_KEY)
    with open(audio_path, "rb") as f:
        result = client.audio.transcriptions.create(
            model="whisper-1", file=f, language="ko"
        )
    return result.text

def structure_note(transcript: str, filename: str) -> dict:
    client = anthropic.Anthropic(api_key=ANTHROPIC_API_KEY)
    msg = client.messages.create(
        model="claude-opus-4-8",
        max_tokens=2048,
        messages=[{
            "role": "user",
            "content": f"""다음 녹음 전사본을 Obsidian 노트로 구조화해줘.
파일명: {filename}
전사본:
{transcript}

출력 형식 (JSON):
{{
  "title": "...",
  "type": "research | daily_memo | experiment | idea",
  "date": "YYYY-MM-DD",
  "summary": "한 줄 요약",
  "tags": ["tag1", "tag2"],
  "content": "마크다운 형식의 구조화된 내용"
}}"""
        }]
    )
    import json
    return json.loads(msg.content[0].text)

def save_to_obsidian(note: dict):
    date = note["date"]
    folder_map = {
        "research": "20-Research/Experiments",
        "experiment": "20-Research/Experiments",
        "daily_memo": "00-Inbox",
        "idea": "40-Ideas"
    }
    folder = VAULT / folder_map.get(note["type"], "00-Inbox")
    folder.mkdir(parents=True, exist_ok=True)
    
    tags_str = " ".join([f"#{t}" for t in note["tags"]])
    content = f"""---
date: {date}
tags: {note["tags"]}
type: {note["type"]}
source: voice-recording
---

# {note["title"]}

> {note["summary"]}

{note["content"]}
"""
    filepath = folder / f"{date}-{note['title'][:30]}.md"
    filepath.write_text(content, encoding="utf-8")
    print(f"저장됨: {filepath}")

if __name__ == "__main__":
    audio_path = sys.argv[1]
    print(f"처리 중: {audio_path}")
    transcript = transcribe(audio_path)
    note = structure_note(transcript, os.path.basename(audio_path))
    save_to_obsidian(note)
    os.rename(audio_path, audio_path + ".processed")
```

### 3-2. 자동 감시 설정 (맥미니 launchd)

`~/Library/LaunchAgents/com.user.recording-watcher.plist`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.user.recording-watcher</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/bin/python3</string>
    <string>/Users/YOUR_USERNAME/scripts/watch_recordings.py</string>
  </array>
  <key>WatchPaths</key>
  <array>
    <string>/Users/YOUR_USERNAME/Sync/IncomingRecordings</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>OPENAI_API_KEY</key>
    <string>YOUR_KEY</string>
    <key>ANTHROPIC_API_KEY</key>
    <string>YOUR_KEY</string>
  </dict>
  <key>RunAtLoad</key>
  <true/>
</dict>
</plist>
```

활성화:
```bash
launchctl load ~/Library/LaunchAgents/com.user.recording-watcher.plist
```

---

## Phase 4 — AI 컨텍스트 레이어 (Claude + Obsidian RAG)

**목표:** Claude에게 작업 지시 시 Vault 전체를 컨텍스트로 활용

### 방법 A (권장): Claude Projects 활용

1. claude.ai → Projects → "My Research Brain" 프로젝트 생성
2. Vault 주요 폴더의 .md 파일들을 Project Knowledge에 업로드
3. 주 1회 업데이트 스크립트:

```bash
# ~/scripts/update_claude_project.sh
# Vault에서 최근 수정된 노트를 ZIP으로 묶어 수동 업로드 안내
find ~/Sync/ObsidianVault -name "*.md" -newer /tmp/last_upload \
  -not -path "*/.obsidian/*" | head -50
```

### 방법 B (고급): MCP + Obsidian

Claude Desktop에 파일시스템 MCP 서버 설정 (경로 수정됨, 원래 `~/.claude/claude_desktop_config.json`으로 잘못 적혀있었음):
```json
// macOS: ~/Library/Application Support/Claude/claude_desktop_config.json
// Windows: %APPDATA%\Claude\claude_desktop_config.json
{
  "mcpServers": {
    "obsidian-vault": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem",
               "/Users/YOUR_USERNAME/Sync/ObsidianVault"]
    }
  }
}
```

→ Claude Desktop에서 "Vault를 읽어서 2주 전 실험 결과 요약해줘" 바로 실행 가능

---

## Phase 5 — 자동화 스케줄러

### 맥미니 (launchd)

| 작업 | 주기 |
|------|------|
| 녹음 파일 처리 | 파일 감지 즉시 |
| Daily Note 생성 | 매일 오전 7시 |
| 실험 데이터 백업 | 매일 자정 |
| Vault 오래된 파일 Archive | 매주 일요일 |

### 연구실 PC / 노트북 (Windows Task Scheduler)

```powershell
# Syncthing 자동 시작 (로그인 시)
schtasks /create /tn "Syncthing" /tr "C:\syncthing\syncthing.exe" /sc onlogon
```

### 연구실 PC / 노트북 (Ubuntu cron)

```bash
crontab -e
# Syncthing 크래시 시 재시작
*/5 * * * * systemctl is-active --quiet syncthing@$USER || systemctl restart syncthing@$USER
```

---

## 실행 순서 (권장)

```
Week 1: Phase 1 (Syncthing 설치 + 동기화 확인)
         ↓ 완료 확인: 3대에서 파일 변경 시 5분 내 반영
Week 2: Phase 2 (Obsidian 구조 + 플러그인 설치)
         ↓ 완료 확인: Daily Note 자동 생성
Week 3: Phase 3 (음성 파이프라인 테스트)
         ↓ 완료 확인: 테스트 녹음 → Obsidian 노트 자동 삽입
Week 4: Phase 4 (Claude Projects or MCP 설정)
         ↓ 완료 확인: Vault 질문 → Claude 정확한 답변
Week 5: Phase 5 (스케줄러 전체 연결 + 안정화)
```

---

## 수용 기준 (Acceptance Criteria)

- [ ] 3대 기기에서 Obsidian Vault 변경사항이 5분 내 동기화
- [ ] 논문 PDF / 실험 데이터 3대에서 접근 가능
- [ ] 아이폰 녹음 → 맥미니 자동 도착 (10분 이내)
- [ ] 녹음 → Whisper 전사 → Claude 구조화 → Obsidian 삽입 파이프라인 작동
- [ ] Daily Note가 매일 자동 생성 (할 일 포함)
- [ ] Claude에게 Vault 기반 질문 시 정확한 과거 기록 참조

---

## 사용할 API / 서비스

| 서비스 | 용도 | 비용 예상 |
|--------|------|-----------|
| OpenAI Whisper API | 녹음 전사 | $0.006/분 |
| Anthropic Claude API | 노트 구조화 | ~$0.01/녹음 |
| Syncthing | 파일 동기화 | 무료 |
| Obsidian | Second Brain | 무료 (개인) |
| Claude Projects / Desktop | RAG | Claude 구독 포함 |
