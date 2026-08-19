## Handoff: Phase 1 (Syncthing) — 연구실 PC 완료 → 노트북 진행용

- **환경**: 연구실 PC는 WSL2 Ubuntu(호스트명 `MSCSY`) 안에서 Claude Code 실행 중. Windows 쪽 앱(Syncthing, Obsidian)은 `powershell.exe` / `winget` / `cmd.exe`를 WSL에서 인터롭 호출해 설치·제어함. **노트북도 같은 방식(WSL Ubuntu + Claude Code)으로 작업할 예정** — 아래 절차를 그대로 따르면 됨.

### 연구실 PC에서 한 일 (그대로 재현하면 됨)

1. Syncthing 설치 (Windows 쪽, WSL에서 인터롭으로):
   ```bash
   powershell.exe -NoProfile -Command "winget install --id Syncthing.Syncthing -e --accept-source-agreements --accept-package-agreements"
   ```
2. **주의 (v2 CLI 함정)**: Syncthing v2.x부터 CLI가 서브커맨드 구조로 바뀜. `syncthing.exe -no-browser` 같은 예전 플래그는 `unknown flag` 에러로 즉시 죽는다. 반드시:
   ```
   syncthing.exe serve --no-browser --no-console
   ```
   형태로 실행할 것 (`serve` 서브커맨드 필수).
3. **WSL 인터롭에서 백그라운드 실행 시 주의**: `powershell.exe -Command "Start-Process ..."`나 `cmd.exe /c start ...`로 직접 실행하면 WSL이 호출한 프로세스가 끝나자마자 job object가 죽여버려서 안 죽은 것처럼 보여도 몇 초 뒤 사라짐 (실제 원인은 위 2번 플래그 오류였음 — 플래그만 맞으면 정상적으로 살아있음, 굳이 우회 트릭 필요 없었음). `.bat` 파일 하나 만들어서 `cmd.exe /c start "" "그 bat경로"`로 실행하는 게 제일 안정적:
   ```bat
   @echo off
   "C:\Users\hso28\AppData\Local\Microsoft\WinGet\Packages\Syncthing.Syncthing_Microsoft.Winget.Source_8wekyb3d8bbwe\syncthing-windows-amd64-v2.1.3\syncthing.exe" serve --no-browser --no-console > C:\Users\<사용자명>\syncthing_launch.log 2>&1
   ```
4. 자동시작 등록: winget으로 깐 syncthing.exe 실행 파일 경로를 찾아(`Get-ChildItem -Recurse -Filter syncthing*.exe` on `AppData\Local\Microsoft\WinGet\Packages`), 그걸 실행하는 VBS 런처(콘솔창 안 뜨게 `WScript.Shell.Run ... , 0, False`)를 만들고, 그 VBS에 대한 바로가기(.lnk)를 Windows Startup 폴더(`[Environment]::GetFolderPath('Startup')`)에 넣음. (`schtasks /create`는 이 환경에서 액세스 거부로 실패 — Startup 폴더 방식이 확실히 됨.)
5. **`ObsidianVault/.stignore`부터 먼저 만들 것 (폴더 등록/스캔보다 반드시 먼저)** — `.stignore`는 기기별 로컬 설정이라 절대 동기화되지 않음. 연구실 PC에 이미 있다고 안심하면 안 되고, 노트북에서도 직접 만들어야 함. `Sync\ObsidianVault\.stignore` 파일에 아래 3줄:
   ```
   .obsidian/workspace.json
   .obsidian/workspace-mobile.json
   Second_brain
   ```
   - 앞 2줄: UI 충돌 방지
   - `Second_brain`: 자체 git 저장소라 Syncthing이 건드리면 안 됨(git과 Syncthing이 같은 파일을 동시에 관리하면 `.git` 손상 위험). Vault 안에 그대로 두기로 했으므로, 폴더를 스캔하기 *전에* 반드시 제외 설정이 있어야 함 — 순서가 바뀌면 첫 스캔 때 `Second_brain/.git`이 인덱싱될 수 있음.
6. `obsidian-vault` 폴더 등록 (`.stignore` 만든 다음에! **폴더 ID를 반드시 `obsidian-vault`로 통일** — 다른 기기와 폴더 공유 수락할 때 ID가 같아야 매칭됨):
   ```
   syncthing.exe cli config folders add --id=obsidian-vault --label=ObsidianVault --path="<Sync\ObsidianVault 경로>" --type=sendreceive
   ```
   - 확인 방법: `curl.exe -H "X-API-Key: <GUI API key>" "http://127.0.0.1:8384/rest/db/file?folder=obsidian-vault&file=Second_brain"` 응답의 `local.ignored`가 `true`인지 확인 (연구실 PC에서 이렇게 확인함). API key는 `C:\Users\<사용자명>\AppData\Local\Syncthing\config.xml`의 `<gui><apikey>`.
7. 방화벽: winget 설치 시 인바운드 규칙이 자동 생성됨 (`Get-NetFirewallRule -DisplayName '*syncthing*'`으로 확인). 별도 조치 불필요했음.

### 연구실 PC의 Device ID (노트북에서 Add Remote Device로 등록)

```
4FI6A5D-WHHDX3I-BJA4O7F-IHFANDE-HC6D3PG-ZL2GXSY-EBPV4UQ-YALIRAZ
```

### 노트북에서 할 일

1. 위 1~7단계 그대로 (경로만 노트북 사용자명으로 교체)
2. Syncthing GUI(`http://127.0.0.1:8384`)에서 **Add Remote Device** → 위 연구실 PC Device ID 입력
3. 연구실 PC 쪽에서 노트북을 새 기기로 수락 + `obsidian-vault` 폴더 공유 요청을 보내면, 노트북에서 수락하고 로컬 경로를 `Sync\ObsidianVault`로 지정
4. 두 기기 다 켜진 상태로 5분 내 동기화되는지 파일 하나 수정해서 확인

### 남은 것

- 맥미니 Phase 1 (별도 진행, `brew install syncthing` 방식)
- 3대 전부 페어링 확인되면 Phase 2(Vault 구조 정리)로 이동
