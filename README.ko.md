# wallshuffle

[English](README.md)

macOS 항공사진(Aerial) 배경화면을 **원하는 간격**으로 바꿔주는 도구입니다. 10분마다, 3시간마다 등 마음대로 정할 수 있습니다.

시스템 설정에서는 항공사진 셔플 간격을 "12시간마다"보다 짧게 고를 수 없습니다.
wallshuffle은 작은 백그라운드 타이머를 두고, 내려받은 항공사진 중 하나로 원하는 만큼 자주 바꿔 줍니다.

- **간격 자유** — 60초 이상이면 아무 값 (`every 10m`, `every 1h`, `every 3h` …)
- **시스템 설정에서 고른 셔플을 따라감** — "전체 셔플"이나 "도시 풍경 셔플" 같은 카테고리
- **내려받은 항공사진을 전부 사용** — Apple 셔플이 빼는 하위 폴더(예: 도시 풍경 › 샌프란시스코) 영상도 포함
- **Apple의 숨겨진 "5초마다" 셔플**도 켤 수 있음 (타이머 대신 macOS 내장 셔플을 쓰고 싶을 때)
- **zsh 파일 하나** — sudo, 빌드, 추가 설치 없음 (macOS 기본 zsh, jq, plutil, osascript, launchd만 사용)
- **처음 쓸 때 배경화면 설정을 백업** — `wallshuffle restore`로 원래대로 복구

> Apple과 무관한 비공식 도구입니다. 공개되지 않은 설정 파일을 고치는 방식이라 macOS 업데이트 후 동작이 멈출 수 있습니다.

## 요구 사항

- **macOS 27 이상.** macOS 27.0.1(Apple Silicon)에서 테스트했습니다. 이전 버전은 항공사진을 다른 위치에 저장해서 지원하지 않습니다.
- 배경화면이 **항공사진 셔플**로 설정돼 있어야 합니다 (시스템 설정 › 배경화면).
- 항공사진이 2개 이상 내려받아져 있어야 합니다. 영상이 실제로 디스크에 있는 것만 돌아갑니다.

## 설치

1. 이 저장소에서 [`wallshuffle`](wallshuffle) 파일을 내려받습니다 (또는 저장소를 clone).
2. **시스템 설정 › 배경화면**에서 **전체 셔플**이나 **도시 풍경 셔플** 같은 항공사진 셔플을 고르고, 항공사진을 몇 개 내려받습니다.
3. 터미널에서 원하는 간격으로 한 번 실행합니다.

   ```bash
   zsh ~/Downloads/wallshuffle every 1h
   ```

   스크립트가 `~/Library/Application Support/wallshuffle/`에 복사되고, 타이머가 켜지고, 그 자리에서 배경화면이 한 번 바뀝니다.

### 명령을 짧게 쓰기 (선택)

```bash
echo 'alias wallshuffle="$HOME/Library/Application Support/wallshuffle/wallshuffle"' >> ~/.zshrc
```

터미널을 새로 열면 그다음부터 `wallshuffle every 3h`처럼 짧게 쓸 수 있습니다. 아래 설명은 짧은 형태로 적었습니다.

### 출력 언어

macOS 언어를 따라갑니다. 시스템 언어가 영어인데 한국어로 보고 싶다면 한 번만 실행하세요.

```bash
wallshuffle lang ko
```

## 사용법

| 명령 | 하는 일 |
|---|---|
| `wallshuffle status` | 현재 사진, 타이머, 셔플 범위의 사진 수 보기 |
| `wallshuffle every 3h` | 3시간마다 바꾸기 (`s`/`m`/`h`, 최소 60초). 하루 한 번은 `24h` |
| `wallshuffle next` | 지금 바로 다른 사진으로 바꾸기 |
| `wallshuffle off` | 타이머 끄고 Apple 셔플(12시간마다)로 되돌리기 |
| `wallshuffle native 5s` | 타이머 대신 Apple 내장 셔플 주기 쓰기 (아래 참고) |
| `wallshuffle restore` | 처음 바꾸기 전 배경화면 설정으로 복구 |
| `wallshuffle lang ko\|en\|auto` | 출력 언어 |
| `wallshuffle help` | 도움말 |

간격을 바꾸려면 `every`를 새 값으로 다시 실행하면 됩니다.

## 어떤 사진 중에서 고르나요?

**시스템 설정 › 배경화면에서 마지막으로 고른 셔플**을 따릅니다.

- **전체 셔플** → 내려받은 항공사진 전체
- **도시 풍경 셔플** 등 카테고리 → 그 카테고리에서 내려받은 항공사진

범위를 바꾸려면 설정 앱에서 다른 셔플을 고르면 다음 교체부터 반영됩니다 (바로 보려면 `wallshuffle next`).
새로 내려받은 항공사진도 다음 교체부터 자동으로 후보에 들어갑니다.
같은 사진이 연달아 나오지 않고, Mac·다이내믹(라이트/다크) 배경은 제외합니다.

## Apple 내장 셔플 주기 (`native`)

타이머 대신 macOS 자체 셔플을 쓰고 싶을 때:

| 값 | 주기 |
|---|---|
| `5s` | 5초마다 — 설정 앱 메뉴에 없는 숨김 옵션 (Apple 내부 기능 플래그로 가려져 있음) |
| `continuous` | 계속 — 재생 중인 영상이 끝날 때마다 |
| `12h` `1d` `2d` `1w` `1mo` | 12시간 / 매일 / 격일 / 매주 / 매달 |

```bash
wallshuffle native 5s
```

`native`를 쓰면 타이머는 꺼집니다. Apple 내장 셔플은 하위 폴더 영상을 빼고 돌립니다.

## 알아두면 좋은 점

- **타이머가 켜져 있으면 설정 앱에는 "셔플" 대신 사진 한 장이 선택된 것처럼 보입니다.** 한 번에 한 장씩 지정하는 방식이라 정상입니다.
- **설정 앱에서 항공사진 한 장을 직접 고르면** 다시 셔플을 고를 때까지 타이머가 건드리지 않습니다.
- 항공사진이 아닌 배경화면이면 타이머는 아무것도 하지 않습니다.
- 바꿀 때마다 `WallpaperAgent`를 재시작하므로 화면이 잠깐 깜빡일 수 있습니다.
- 잠자기 중에는 바뀌지 않고 깨어난 뒤 바뀝니다. 간격은 `every`를 실행한 시점부터 계산합니다 (매 정각 아님).

## 동작 방식

macOS는 배경화면 설정을 `~/Library/Application Support/com.apple.wallpaper/Store/Index.plist`에 저장합니다.
wallshuffle은 정해진 간격마다:

1. Apple 항공사진 목록(`aerials/manifest/entries.json`)에서 셔플 범위에 속하고 영상이 내려받아진 것을 고르고,
2. 그중 하나를 무작위로 뽑아 (지금 것은 제외),
3. `Index.plist`에 그 ID를 쓰고 `WallpaperAgent`를 재시작한 뒤, 제대로 적용됐는지 확인합니다.

타이머는 사용자 LaunchAgent(`launchd`)입니다.

### 파일 위치

| 위치 | 내용 |
|---|---|
| `~/Library/Application Support/wallshuffle/wallshuffle` | 타이머가 실행하는 스크립트 |
| `~/Library/Application Support/wallshuffle/state` | 셔플 범위, 마지막으로 고른 사진, 언어 |
| `~/Library/Application Support/wallshuffle/error.log` | 타이머 실행 중 오류 기록 |
| `~/Library/LaunchAgents/local.wallshuffle.plist` | 타이머 |
| `~/Library/Application Support/com.apple.wallpaper/Store/Index.plist.wallshuffle-backup` | 원래 배경화면 설정 백업 (`restore`가 사용) |

## 삭제

```bash
wallshuffle off
```
```bash
rm -rf "$HOME/Library/Application Support/wallshuffle"
```

`off`가 타이머를 지우고 Apple 셔플로 되돌립니다. 두 번째 명령은 스크립트를 지웁니다.
백업 파일 `Index.plist.wallshuffle-backup`은 남겨도 되고 지워도 됩니다. 별칭을 만들었다면 `~/.zshrc`에서 `alias wallshuffle=…` 줄도 지우세요.

## 비슷한 프로젝트

- [apple-aerial-controller](https://github.com/JoeyJia02/apple-aerial-controller) — 재생목록, 시간별 교체, 바탕화면 영상 재생을 지원하는 메뉴 막대 앱 (Swift, 직접 빌드)
- [aerial-rotate](https://github.com/TylerJamesNz/aerial-rotate) — 항공사진을 1개만 남기고 매일 교체 (sudo 필요)
- [macos-wallpaper-rotator](https://github.com/Icosahedral-Dice/macos-wallpaper-rotator) — 일출·일몰에 맞춰 교체 (Python)
- [tahoe-aerial-shuffle](https://github.com/adele-with-a-b/tahoe-aerial-shuffle) — 1분마다 바뀌는 잠금화면 항공사진 화면보호기

## 라이선스

[MIT](LICENSE). Apple 항공사진 영상은 이 프로젝트에 포함되지 않으며 각 권리자에게 있습니다.
