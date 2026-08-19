## Handoff: Phase 1 (Syncthing) — 맥미니용

연구실 PC(Windows, `.omc/handoffs/phase1-lab-pc.md`)와 노트북(Windows, `.omc/handoffs/phase1-laptop.md`)은 이미 Syncthing 설치·페어링 설정 완료. 맥미니는 macOS라 설치 명령만 다르고, 나머지 원칙(특히 `.stignore` 순서·`Second_brain` 제외)은 동일하게 적용.

### 시작 전에

```bash
git clone https://github.com/Aa-gif-beep/Second_brain.git
cd Second_brain
claude  # Claude Code 실행
```

Claude Code에서: `.omc/handoffs/phase1-mac-mini.md` 읽고 이 기기에 Syncthing Phase 1 설정해줘.

**중요 (노트북에서 실제로 겪은 문제)**: 이 git 저장소(`Second_brain`)를 iCloud Drive 같은 다른 클라우드 동기화 폴더 안에 clone하지 말 것. 실제 Obsidian Vault를 동기화할 Syncthing 폴더도 iCloud Drive 밖의 별도 경로로 잡을 것 (예: `~/Sync/ObsidianVault`, iCloud Drive 경로 아님). 두 개의 서로 다른 동기화 메커니즘(iCloud + Syncthing)이 같은 폴더를 동시에 감시하면 충돌 위험.

### 설치 순서

1. **Syncthing 설치**:
   ```bash
   brew install syncthing
   ```

2. **`~/Sync/ObsidianVault/.stignore`부터 먼저 만들 것 (폴더 등록/스캔보다 반드시 먼저)** — `.stignore`는 기기별 로컬 설정이라 절대 동기화되지 않음. 연구실 PC·노트북에 이미 있다고 안심하면 안 되고, 맥미니에서도 직접 만들어야 함:
   ```
   .obsidian/workspace.json
   .obsidian/workspace-mobile.json
   Second_brain
   ```
   - 앞 2줄: UI 충돌 방지
   - `Second_brain`: 자체 git 저장소라 Syncthing이 건드리면 안 됨(git과 Syncthing이 같은 파일을 동시에 관리하면 `.git` 손상 위험). Vault 안에 그대로 두기로 했으므로, 폴더를 스캔하기 *전에* 반드시 제외 설정이 있어야 함.

3. **Syncthing 실행**:
   ```bash
   brew services start syncthing
   ```
   웹 UI: `http://127.0.0.1:8384` (API 키는 `~/Library/Application Support/Syncthing/config.xml`의 `<gui><apikey>`)

4. **자동시작**: `brew services start`가 곧 launchd 등록이라 별도 작업 불필요 (재부팅·재로그인해도 자동 실행됨).

5. **`obsidian-vault` 폴더 등록** (`.stignore` 만든 다음에! **폴더 ID를 반드시 `obsidian-vault`로 통일**):
   ```bash
   syncthing cli config folders add --id=obsidian-vault --label=ObsidianVault --path="$HOME/Sync/ObsidianVault" --type=sendreceive
   ```
   - 확인 방법: `curl -H "X-API-Key: <API 키>" "http://127.0.0.1:8384/rest/db/file?folder=obsidian-vault&file=Second_brain"` 응답의 `local.ignored`가 `true`인지 확인.

6. **연구실 PC + 노트북을 원격 기기로 등록 + 폴더 공유**:
   ```bash
   syncthing cli config devices add --device-id=4FI6A5D-WHHDX3I-BJA4O7F-IHFANDE-HC6D3PG-ZL2GXSY-EBPV4UQ-YALIRAZ --name=LabPC
   syncthing cli config devices add --device-id=NMZQUNM-CGRWQXM-WSGCCIP-VHKVWHL-FI4YRKU-2KIJ5U6-DRZWGUA-IA3I7AX --name=Laptop
   syncthing cli config folders obsidian-vault devices add --device-id=4FI6A5D-WHHDX3I-BJA4O7F-IHFANDE-HC6D3PG-ZL2GXSY-EBPV4UQ-YALIRAZ
   syncthing cli config folders obsidian-vault devices add --device-id=NMZQUNM-CGRWQXM-WSGCCIP-VHKVWHL-FI4YRKU-2KIJ5U6-DRZWGUA-IA3I7AX
   ```

### 맥미니의 Device ID

설치 후 아래로 확인, 연구실 PC와 노트북 양쪽에서 각각 Add Remote Device로 등록 필요 (또는 위 6번처럼 맥미니 쪽에서 먼저 등록해두면, 상대 기기가 pending 요청을 수락하기만 하면 됨):
```bash
syncthing cli show system  # 또는 웹 UI 좌측 상단에 표시됨
```

### 남은 것

- 연구실 PC·노트북 쪽에서 맥미니의 pending 기기 요청 수락 (또는 반대로 맥미니가 먼저 두 기기를 등록해뒀으니 상대가 수락만 하면 됨)
- 3대 전부 켜진 상태로 5분 내 동기화되는지 확인
- 맥미니는 "허브" 역할이라 Phase 3(음성 파이프라인) 진행 시 필요 — 이 문서 이후 단계

### 방화벽

macOS는 첫 실행 시 방화벽 허용 팝업이 뜰 수 있음 — "허용" 선택.
