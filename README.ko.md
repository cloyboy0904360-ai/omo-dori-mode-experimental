[English](README.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md) · **한국어**

<p align="center">
  <img src="skills/dori-mode/assets/dori-avatar.png" alt="Dori" width="120">
</p>

# omo-dori-mode-experimental

Dori 모드는 코딩 에이전트 세션 하나를 늘 켜져 있는 메신저 에이전트로 바꿉니다. 사용자는 텔레그램이나 디스코드에서 봇 하나와 이야기합니다. Dori는 일마다 herdr 탭에 에이전트 세션을 따로 띄워 맡기고, 자기가 띄운 세션을 모두 기억하며, 일이 정말 끝났을 때만 닫습니다. PR이 머지되고, 이슈가 닫히고, 버전이 배포된 다음에요.

스킬(`skills/dori-mode/SKILL.md`와 references)과 bun + TypeScript로 만든 작은 CLI `dori`로 구성됩니다. 아직 실험 단계라 거친 부분이 남아 있어요.

## 설치

```sh
curl -fsSL https://raw.githubusercontent.com/sisyphuslabs/omo-dori-mode-experimental/main/install.sh | bash
```

저장소를 `~/.dori/src`에 받고, 스킬을 `~/.agents/skills/dori-mode`에 링크하고, `bun link`로 `dori`를 PATH에 올리고, 예시 설정을 `~/.dori/config.json`에 복사합니다. 에이전트가 다른 곳에서 스킬을 읽는다면 `SKILLS_DIR`를 지정하세요.

그다음 herdr 안에서 에이전트를 열고 "Dori mode"라고 말하면 됩니다.

## 필요한 것

- [bun](https://bun.sh) 1.3 이상, git
- [herdr](https://herdr.dev): 레인이 돌아가는 터미널 멀티플렉서
- 스킬을 읽는 코딩 에이전트 ([OmO](https://github.com/code-yeongyu/oh-my-openagent) 기준으로 만들었고, 에이전트 실행 명령은 설정으로 바꿀 수 있습니다)
- PR 머지와 이슈 종료 확인용 `gh`(GitHub CLI), 배포 버전 확인용 `npm`
- 봇 자체를 위한 [agent-messenger](https://github.com/agent-messenger/agent-messenger)

호스트 감시는 macOS에서 전부 동작합니다. 리눅스에서는 부하와 디스크만 보고, 메모리와 스왑은 알 수 없음으로 나옵니다.

## Dori 이름 짓기

Dori는 맨 처음에 자기를 뭐라고 부를지 묻습니다. 그냥 "Dori"도 되고, ShipDori나 WorkDori처럼 끝이 Dori로 끝나는 이름도 됩니다. 여러 개를 함께 돌릴 때 구분하기 좋습니다. 정한 이름은 봇 이름과 메시지 끝 서명, 모드 이름에 그대로 쓰입니다. 다음부터는 "ShipDori mode"라고만 하면 다시 켜집니다.

## 슬랙으로 쓸 때

슬랙을 고르면 Dori가 한 가지를 더 묻고 답을 기다립니다.

- **사용자 토큰**: 워크스페이스의 실제 멤버로 움직입니다. 유료 좌석이 하나 필요하고 그 비용은 사용자가 냅니다. 그 멤버가 볼 수 있는 건 모두 읽을 수 있고, 초록색 온라인 표시도 유지할 수 있습니다.
- **봇 토큰**: 슬랙 앱으로 움직입니다. 좌석 비용은 없지만, 초대받은 채널과 앱에 준 권한 범위 안에서만 볼 수 있습니다.

## Dori의 말투

Dori는 사용자의 말투를 따라갑니다. 짧고 편하게, 소문자로 쓰면 그대로 짧고 편하게 답합니다. 어떤 언어로 쓰든 이모지는 쓰지 않습니다. 할 말이 여러 갈래면 긴 글 하나 대신 짧은 메시지 몇 개로 나눠 보내고, 각 메시지는 준비되는 대로 바로 보냅니다. 일부러 뜸을 들이지 않아요. 계속 바뀌는 진행 상황 하나만은 예외라서, 메시지 하나를 그 자리에서 고쳐 갑니다.

## 설정

설정 파일은 `~/.dori/config.json` 하나이고, 어느 항목이든 빼도 됩니다. 보통 정해 두는 항목은 아래와 같아요.

| 항목 | 뜻 |
|---|---|
| `leadPane` | Dori 자신의 herdr pane (`herdr pane current`). 레인 보고가 여기로 옵니다. |
| `laneWorkspace` | 새 레인 탭이 열릴 herdr workspace |
| `defaultCwd` | 레인이 시작하는 디렉터리이자, 레인이 worktree를 만드는 저장소 |
| `agentCommand` | 에이전트 실행 명령. `{model}`, `{prompt}`가 들어간 argv 목록 |
| `hooks.threadReply`, `hooks.threadDone` | 메신저 CLI를 `{thread}`, `{text}`가 들어간 argv 목록으로. 레인 진행 상황을 올리고 완료 표시를 할 때 씁니다. |

나머지(시간, 임계값, heavy 슬롯 수)는 기본값으로 충분합니다. 전체 표는 [`references/scripts.md`](skills/dori-mode/references/scripts.md)에 있습니다. `DORI_CONFIG`, `DORI_STATE_DIR`, `DORI_LEAD_PANE` 환경변수를 주면 파일 값 대신 그 값을 씁니다.

## 온보딩

처음 설정할 때 Dori는 아무것도 읽기 전에 먼저 허락을 구합니다. 어떤 도구를 쓰는지, 무슨 일을 왜 하는지, 사용자와 회사가 어떤 곳인지 알아봐도 되느냐고요. 허락해야만 도구를 하나씩 살펴봅니다. 도구마다 어떤 연동을 쓸지, 그게 무엇을 읽는지 설명하고 따로 묻습니다. 예를 들면 "Gmail과 캘린더를 읽는 CLI로 일정과 메일을 지켜봐도 될까요?" 하는 식입니다. 거절한 도구는 건너뛰고, 거절했다는 사실도 기억합니다.

전 과정은 읽기만 합니다. 알게 된 내용은 그때그때 메모리에 적고, 마지막에 무엇을 알게 됐고 무엇이 아직 비어 있는지 짧게 정리해 줍니다. 자세한 절차는 [`references/onboarding.md`](skills/dori-mode/references/onboarding.md)에 있어요.

## 요청 처리 방식

메시지마다 어떻게 처리할지는 Dori가 스스로 정합니다.

- 질문, 상태 확인, 조회, 몇 번의 도구 호출로 끝나는 작은 수정은 새 세션 없이 바로 처리합니다.
- PR로 끝나는 코드 작업, 여러 단계짜리 일, 오래 걸리거나 나눠서 돌릴 수 있는 일은 레인을 엽니다. 그 저장소를 맡고 있는 쉬는 레인이 있으면 새로 열지 않고 거기에 맡깁니다.
- 메모리, 디스크, pane 수에 여유가 없으면(`dori can-launch`가 HOLD) 새 레인을 열지 않습니다. 일을 대기열에 넣고 이유를 알려 줍니다.
- 새 일은 새 스레드에서 시작합니다. 이어지는 일은 원래 스레드로 돌아가고, 레인이 닫혔으면 예전 세션을 다시 엽니다. 짧은 질문은 물어본 자리에서 답합니다.

## 세션 레지스트리

레인마다 `~/.dori/state/lanes/` 아래에 JSON 파일이 하나씩 생깁니다. 메신저 스레드와 herdr pane, pane과 에이전트 세션 id를 연결하고, 상태(`working`, `done-claimed`, `verified-done`, `not-done`, `closed`)와 그 변화 이력을 남깁니다.

`dori sync`는 이 기록을 실제로 떠 있는 pane과 비교해서 어긋난 곳을 알려 줍니다. 사라진 pane, 바뀐 세션 id, 끝났다는 걸 증명할 방법이 없는 레인 같은 것들입니다. 아무것도 지우지 않습니다. `--write`를 붙이면 찾은 세션 id를 저장합니다.

## 5분 완료 흐름

레인이 끝났다고 알립니다.

```sh
dori claim-done fix-login --evidence "merged acme/app#412 (a1b2c3d)"
```

Dori는 `LANE_DONE_CLAIMED`를 받고, 레인에는 5분 뒤 닫힌다는 메시지가 갑니다. 그 사이에 반대하려면 이렇게 합니다.

```sh
dori object-done fix-login --reason "changelog 항목이 빠졌음"
```

적은 이유가 레인에 그대로 전달되고, 레인은 고친 뒤 다시 완료를 알립니다. 아무도 반대하지 않으면 시간이 지난 뒤 `dori watch`가 레인을 닫습니다. 닫기 전에 `Done =` 신호를 모두 실제로 다시 확인하고, worktree에 원격에 올라가지 않은 커밋이나 커밋하지 않은 변경이 있으면 닫지 않습니다. 이때 완료 요청은 이유와 함께 not-done으로 돌아갑니다. watcher를 다시 켜도 5분 시계는 처음부터 다시 세지 않습니다.

### 무엇을 완료로 보나

레인의 `Done =` 줄에는 watcher가 직접 확인할 수 있는 신호를 적습니다.
- PR 머지
- 이슈 종료
- 패키지 버전 배포
- PR로 끝나지 않는 일(로컬 설정, QA, 띄워 둔 서비스)이라면, 종료 코드 0으로 끝나는 명령, 해시나 JSON 값이 맞는 파일, 기대한 상태 코드와 본문으로 응답하는 URL

```
Done = command ["bun","test"] stdout~" 0 fail"; file qa/report.json json:.passed=true; url http://localhost:3000/health body~"ready"
```

watcher는 레인을 닫을 때 셸 없이 이 확인을 직접 다시 돌리고, 레인이 됐다고 한 말을 그대로 믿지 않습니다. 해석할 수 없는 신호는 레인을 띄울 때 거부합니다. 전체 문법은 [`references/sessions.md`](skills/dori-mode/references/sessions.md)에 있어요.

## 명령

| 명령 | 하는 일 |
|---|---|
| `dori launch <key> ...` | brief에 레인 footer를 쓰고, 탭을 열고, 에이전트를 시작하고, 시작 오류를 확인 |
| `dori adopt <key> --pane ID ...` | 이미 돌고 있는 레인을 등록 |
| `dori sync [--write]` | 레지스트리와 실제 pane 비교, 어긋난 곳 표시 |
| `dori claim-done` / `object-done` / `close` | 완료 흐름 |
| `dori watch` | 자동 닫기 watcher. 지속 모니터로 돌립니다 |
| `dori freshness [--loop MIN]` | 조용해진 레인을 깨우고, 마지막 보고를 스레드에 올림 |
| `dori dead-panes [--loop MIN]` | 멈춘 에이전트 pane 보고 |
| `dori guard [--loop MIN]` | 부하, 메모리, 디스크, pane 수 경고 |
| `dori heavy <label> -- <cmd>` | 슬롯이 비고 부하가 낮을 때만 빌드나 테스트 실행 |

pane에 보내는 글은 항상 인자 하나로 넘기고 셸 문자열을 거치지 않습니다. Enter가 실제로 들어갔는지도 확인합니다.

## 유틸리티

CLI에는 Dori에게 필요한 메신저 기능도 들어 있습니다. `scripts/src/messenger/` 아래 타입이 붙은 모듈로 가져다 쓸 수도 있어요.

| 명령 | 하는 일 |
|---|---|
| `dori send slack\|telegram\|discord --to T --text X [--thread ID] [--edit ID]` | 메시지 보내기와 수정. rate limit은 기다렸다 다시 보내고, `$(`가 들어간 글은 거부합니다 |
| `dori presence slack\|discord` | 계정을 온라인으로 유지. 디스코드 봇은 게이트웨이로, 슬랙 사용자 계정은 1분마다 신호를 보내는 웹 클라이언트 소켓으로 유지합니다 |
| `dori transcribe <file>` | `hooks.transcribe` 명령으로 음성 메시지를 글로 변환 |
| `dori can-launch` | 레인을 하나 더 열 여유가 있는지 확인 |
| `dori inbound slack [--loop MIN]` | 슬랙에서 Dori에게 온 것을 빠짐없이 잡기. Threads 화면의 읽지 않은 답글, Dori가 글을 쓴 스레드의 새 답글(태그가 없어도), 읽지 않은 멘션이 있는 DM과 채널 |

명령은 없지만 모듈로 제공되는 기능도 있습니다.
- 텔레그램: "Thinking…"으로 시작하는 `sendMessageDraft` 스트리밍, 포럼 토픽, HTML 표
- 디스코드: 스레드 만들기, 이름 바꾸기, 보관
- 슬랙: 파일 업로드
- `typingWhile`: 작업이 도는 동안 입력 중 표시를 띄워 둡니다

슬랙에서 Dori가 보내는 메시지는 어떤 함수로 보냈든 그 스레드를 기록해 둡니다. 원시 API 호출로 올린 글 아래에 태그 없이 달린 답글도 놓치지 않는데, 메시지 이벤트만 듣는 방식으로는 이런 답글을 놓칩니다.

토큰은 `DORI_SLACK_TOKEN`(사용자 토큰이면 `DORI_SLACK_COOKIE`도), `DORI_TELEGRAM_TOKEN`, `DORI_DISCORD_TOKEN`에서 읽습니다.

## 테스트

CI는 없습니다. 테스트는 로컬에서 돌립니다.

```sh
cd skills/dori-mode/scripts
bun install
bun test           # 가짜 herdr, git, gh로 동작을 확인하는 테스트
bunx tsc --noEmit  # 타입 검사
```

실제 pane이나 저장소, GitHub는 건드리지 않아요.

## 라이선스

MIT

## OmOMeow에서 옮겨 오기

예전 gist로 OmOMeow 모드를 쓰고 있었다면 봇은 그대로 동작합니다. 네 가지만 바꾸면 Dori가 됩니다.

1. **Dori 이름 정하기.** 그냥 "Dori"나 ShipDori, WorkDori처럼 Dori로 끝나는 이름이면 됩니다. 에이전트에게 "이제부터 네 이름은 ShipDori고, 이건 ShipDori mode야"라고 말하세요. 그다음부터는 "OmOMeow mode" 대신 "ShipDori mode"가 모드를 켜는 말이 됩니다.
2. **봇 이름과 프로필 사진 바꾸기.**
   - 텔레그램: @BotFather에서 `/setname`을 보내고 봇을 고른 뒤 새 이름을 보냅니다. 이어서 `/setuserpic`을 보내고 봇을 고른 뒤 새 이미지를 보냅니다. 기본 Dori 그림은 [`skills/dori-mode/assets/dori-avatar.png`](skills/dori-mode/assets/dori-avatar.png)이고, 원하는 이미지로 바꿔도 됩니다. 봇 이름과 사진은 BotFather에서만 바꿀 수 있습니다.
   - 디스코드: Developer Portal에서 애플리케이션을 엽니다. **Bot** 페이지에서 사용자 이름과 아이콘을, **General Information**에서 앱 이름과 아이콘을 바꾸고(같은 기본 그림을 쓰면 됩니다) 저장합니다.
3. **이 저장소 설치하기.** 위의 한 줄 설치를 실행하고 에이전트에게 "ShipDori mode"라고 말하면, 예전에 붙여 넣던 프롬프트 대신 스킬과 `dori` CLI를 씁니다.
4. **온보딩 하기.** OmOMeow 때 한 적이 없다면 "온보딩 해 줘"라고 하면 됩니다.

기존 스레드, 토픽, 메모리는 그대로 남습니다.
