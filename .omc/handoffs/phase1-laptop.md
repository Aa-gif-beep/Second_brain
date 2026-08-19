## Handoff: Phase 1 (Syncthing) — 노트북 완료 → 연구실 PC 쪽 페어링 수락 필요

- **환경**: 노트북도 WSL2 Ubuntu(호스트명 `hso5807`) + Claude Code, Windows 쪽 Syncthing은 `powershell.exe`/`cmd.exe`/`winget` 인터롭으로 설치·제어. `phase1-lab-pc.md`의 1~7단계를 그대로 재현.

### 중요 결정: Vault 경로를 OneDrive 밖으로 분리

- 이 git 저장소(`Second_brain`)는 `C:\Users\hso28\OneDrive\문서\Sync`(OneDrive 동기화 폴더) 안에 clone됨. 그런데 이 저장소 안에 `.obsidian` 설정과 `무제.md` 같은 실제 vault 콘텐츠가 섞여 커밋되어 있었음.
- OneDrive와 Syncthing이 같은 폴더를 동시에 감시하면 `.obsidian/workspace.json` 등에서 충돌 가능성이 있어, **실제 Syncthing `obsidian-vault` 폴더는 OneDrive 밖의 새 경로로 분리**: `C:\Users\hso28\Sync\ObsidianVault` (빈 상태로 생성, 연구실 PC와 동기화되면 채워짐).
- **주의**: 이 git 저장소(OneDrive 경로)와 Syncthing이 관리하는 `C:\Users\hso28\Sync\ObsidianVault`는 서로 다른 폴더임. 맥미니 등 다음 기기 설정 시에도 같은 원칙 적용 권장 (Syncthing vault 경로를 iCloud/OneDrive 등 다른 클라우드 동기화 폴더와 분리).

### 한 일 (연구실 PC와 동일)

1. `winget install --id Syncthing.Syncthing -e` → v2.1.3 설치
2. `syncthing.exe` 경로: `C:\Users\hso28\AppData\Local\Microsoft\WinGet\Packages\Syncthing.Syncthing_Microsoft.Winget.Source_8wekyb3d8bbwe\syncthing-windows-amd64-v2.1.3\syncthing.exe`
3. `C:\Users\hso28\syncthing_launch.bat` 생성 (`serve --no-browser --no-console`) → `cmd.exe /c start` 로 실행, 정상 구동 확인 (로그: `C:\Users\hso28\syncthing_launch.log`)
4. 자동시작: `C:\Users\hso28\syncthing_launcher.vbs`(콘솔 숨김 실행) + Startup 폴더에 `Syncthing.lnk` 바로가기 등록
5. `obsidian-vault` 폴더 등록: `id=obsidian-vault`, `path=C:\Users\hso28\Sync\ObsidianVault`, `type=sendreceive`
6. `.stignore`에 `.obsidian/workspace.json`, `.obsidian/workspace-mobile.json` 추가
7. 방화벽 인바운드 규칙 자동 생성 확인됨 (`Get-NetFirewallRule -DisplayName '*syncthing*'`)
8. 연구실 PC를 원격 기기로 CLI에서 직접 등록 (`cli config devices add --device-id=<lab PC ID> --name=LabPC`), `obsidian-vault` 폴더도 그 기기와 공유 설정 완료 (`cli config folders obsidian-vault devices add`)

### 노트북의 Device ID (연구실 PC에서 Add Remote Device로 등록 필요)

```
NMZQUNM-CGRWQXM-WSGCCIP-VHKVWHL-FI4YRKU-2KIJ5U6-DRZWGUA-IA3I7AX
```

### 남은 것 (연구실 PC 쪽에서 해야 함 — 노트북에서는 CLI로 더 진행 불가)

1. 연구실 PC Syncthing GUI(`http://127.0.0.1:8384`)에서 **Add Remote Device** → 위 노트북 Device ID 입력하고 수락
2. 연구실 PC가 노트북을 수락하면, `obsidian-vault` 폴더 공유 요청이 노트북 쪽에 뜸 → 노트북에서도 수락 확인 (CLI로 이미 folder-device 링크는 걸어뒀으므로 연결되면 바로 동기화 시작될 것)
3. 두 기기 다 켜진 상태로 5분 내 동기화되는지 파일 하나 수정해서 확인
4. 맥미니 Phase 1 (`brew install syncthing`, 동일하게 vault 경로는 iCloud Drive 밖으로 분리 권장)
