[English](README.md) · **简体中文** · [日本語](README.ja.md) · [한국어](README.ko.md)

<p align="center">
  <img src="skills/dori-mode/assets/dori-avatar.png" alt="Dori" width="120">
</p>

# omo-dori-mode-experimental

Dori 模式把一个编程智能体会话变成常驻的消息智能体。你只需要在 Telegram 或 Discord 上和一个机器人对话。Dori 把每件事交给 herdr 标签页里单独启动的智能体会话，记住自己启动过的每个会话，只有在工作真正完成后才关闭它:PR 已合并、issue 已关闭、版本已发布。

它由一个技能(`skills/dori-mode/SKILL.md` 和 references)以及一个用 bun + TypeScript 写的小 CLI `dori` 组成。目前还是实验版本，会有不完善的地方。

## 安装

```sh
curl -fsSL https://raw.githubusercontent.com/sisyphuslabs/omo-dori-mode-experimental/main/install.sh | bash
```

它会把仓库克隆到 `~/.dori/src`,把技能链接到 `~/.agents/skills/dori-mode`,用 `bun link` 把 `dori` 放进 PATH,并把示例配置复制到 `~/.dori/config.json`。如果你的智能体从别的目录加载技能，请设置 `SKILLS_DIR`。

然后在 herdr 里打开智能体，说一句 "Dori mode" 就行。

## 依赖

- [bun](https://bun.sh) 1.3 或更新版本，以及 git
- [herdr](https://herdr.dev):运行各条 lane 的终端复用器
- 能加载技能的编程智能体(按 [OmO](https://github.com/code-yeongyu/oh-my-openagent) 设计，启动命令可以在配置里改)
- 用来确认 PR 合并和 issue 关闭的 `gh`(GitHub CLI),以及确认已发布版本的 `npm`
- 机器人本身用的 [agent-messenger](https://github.com/agent-messenger/agent-messenger)

主机监控在 macOS 上功能完整。在 Linux 上只看负载和磁盘，内存和 swap 显示为未知。

## 给你的 Dori 起名

Dori 第一件事就是问你该怎么称呼它。直接叫 "Dori" 可以，用 Dori 结尾的名字也可以，比如 ShipDori 或 WorkDori,同时运行好几个时更好区分。定下的名字会用在机器人名、消息落款和模式名上，下次只要说一句 "ShipDori mode" 就能重新开启。

## 用 Slack 时

如果你选 Slack,Dori 会再问一个问题，等你回答:

- **用户令牌**:作为工作区里的真实成员行动。需要一个付费席位，费用由你承担。这个成员能看到的它都能读，也能一直显示绿色在线点。
- **机器人令牌**:作为 Slack 应用行动。没有席位费用，但只能看到被邀请进的频道，并且受限于你给应用的权限范围。

## Dori 怎么说话

Dori 会跟着你的说话方式走。你写得简短、随意、用小写，它也这样回。无论用哪种语言，它都不用表情符号。一条回复有好几部分时，它会拆成几条短消息发，而不是一大段;每条准备好就立刻发，不故意停顿。唯一的例外是不断变化的进度：那始终是一条消息，原地修改。

## 配置

所有配置都在 `~/.dori/config.json` 里，每一项都可以省略。通常需要设置的是这些:

| 字段 | 含义 |
|---|---|
| `leadPane` | Dori 自己的 herdr pane(`herdr pane current`)。各条 lane 的汇报会发到这里。 |
| `laneWorkspace` | 新 lane 标签页打开的 herdr workspace |
| `defaultCwd` | lane 的起始目录，也是 lane 创建 worktree 的仓库 |
| `agentCommand` | 启动智能体的命令，写成含 `{model}` 和 `{prompt}` 的 argv 列表 |
| `hooks.threadReply`, `hooks.threadDone` | 你的消息 CLI,写成含 `{thread}` 和 `{text}` 的 argv 列表，用来发布 lane 进度和标记完成 |

其余项(时间、阈值、heavy 槽位数)用默认值就够了。完整表格见 [`references/scripts.md`](skills/dori-mode/references/scripts.md)。环境变量 `DORI_CONFIG`、`DORI_STATE_DIR`、`DORI_LEAD_PANE` 优先于配置文件。

## 初次了解(Onboarding)

第一次设置时,Dori 在读取你的任何东西之前，会先征求同意：能不能了解一下你用哪些工具、在做什么、为什么做，以及你和你的公司是什么样的。只有你同意了，它才会一个一个地查看你的工具。每个工具它都会先说明要用哪个集成、会读取什么，再单独问你。比如:"有一个能读取 Gmail 和日历的 CLI,我想用它帮你留意日程和邮件，可以吗?"你拒绝的工具会被跳过，它也会记住你拒绝过。

整个过程只读不写。它边看边把了解到的内容写进记忆，最后简短地告诉你它了解了什么、还缺什么。完整流程见 [`references/onboarding.md`](skills/dori-mode/references/onboarding.md)。

## 怎么处理一个请求

每条消息怎么处理，由 Dori 自己决定。

- 提问、查状态、查资料，以及几次工具调用就能完成的小改动，直接处理，不开新会话。
- 最终要提交 PR 的代码工作、多步骤的工作、耗时长或可以并行的工作，开一条 lane。如果已经有一条空闲的 lane 负责这个仓库，就交给它，不再新开。
- 内存、磁盘或 pane 数量不够时(`dori can-launch` 显示 HOLD),不开新 lane。工作先排队，并告诉你原因。
- 新的工作开新的线程。后续的跟进回到原来的线程，如果那条 lane 已关闭，就重新打开记录下来的会话。简单的问题在哪里问的就在哪里答。

## 会话登记表

每条 lane 在 `~/.dori/state/lanes/` 下有一个 JSON 文件。它把消息线程对应到 herdr pane,把 pane 对应到智能体自己的会话 id,并记录状态(`working`、`done-claimed`、`verified-done`、`not-done`、`closed`)以及每次变化的历史。

`dori sync` 会把登记表和实际在运行的 pane 对照，告诉你哪里对不上：消失的 pane、变了的会话 id、没有办法证明已完成的 lane。它不会删除任何东西。加上 `--write` 会把找到的会话 id 存下来。

## 5 分钟完成流程

lane 声明自己已完成:

```sh
dori claim-done fix-login --evidence "merged acme/app#412 (a1b2c3d)"
```

Dori 收到 `LANE_DONE_CLAIMED`,lane 会被告知 5 分钟后关闭。这期间你可以提出异议:

```sh
dori object-done fix-login --reason "缺少 changelog 条目"
```

理由会原样发给 lane,lane 修好后再次声明完成。如果没人反对，时间一到 `dori watch` 就会关闭这条 lane。关闭前它会重新实时读取每个 `Done =` 信号；如果 worktree 里还有没推到远端的提交或未提交的改动，它就不关闭，声明会带着原因退回 not-done。重启 watcher 不会让 5 分钟重新计时。

### 怎样才算完成

lane 的 `Done =` 行写的是 watcher 能自己检查的信号:
- PR 已合并
- issue 已关闭
- 包的版本已发布
- 对于不以 PR 结束的工作(本地设置、QA、运行中的服务):退出码为 0 的命令、哈希或 JSON 字段符合预期的文件、返回预期状态码和内容的 URL

```
Done = command ["bun","test"] stdout~" 0 fail"; file qa/report.json json:.passed=true; url http://localhost:3000/health body~"ready"
```

关闭 lane 时,watcher 会自己重新运行这些检查，不经过 shell,也从不凭 lane 自己的说法就算数。无法解析的信号在启动 lane 时就会被拒绝。完整语法见 [`references/sessions.md`](skills/dori-mode/references/sessions.md)。

## 命令

| 命令 | 作用 |
|---|---|
| `dori launch <key> ...` | 在 brief 里写入 lane footer,打开标签页，启动智能体，检查启动错误 |
| `dori adopt <key> --pane ID ...` | 登记一条已经在运行的 lane |
| `dori sync [--write]` | 对照登记表和实际 pane,列出不一致 |
| `dori claim-done` / `object-done` / `close` | 完成流程 |
| `dori watch` | 自动关闭的 watcher,作为常驻监控运行 |
| `dori freshness [--loop MIN]` | 提醒变安静的 lane,再把它最后一条汇报发到线程里 |
| `dori dead-panes [--loop MIN]` | 报告已停止的智能体 pane |
| `dori guard [--loop MIN]` | 负载、内存、磁盘和 pane 数量告警 |
| `dori heavy <label> -- <cmd>` | 只在有空闲槽位且负载低时运行构建或测试 |

发给 pane 的文字总是作为一个参数传入，从不经过 shell 字符串，并且会确认 Enter 真的生效了。

## 实用工具

CLI 还带有 Dori 需要的消息相关功能，也可以作为 `scripts/src/messenger/` 下带类型的模块直接引用。

| 命令 | 作用 |
|---|---|
| `dori send slack\|telegram\|discord --to T --text X [--thread ID] [--edit ID]` | 发送或编辑消息。遇到限流会等待后重发，含有 `$(` 的文字会被拒绝 |
| `dori presence slack\|discord` | 让账号保持在线显示。Discord 机器人通过网关保持;Slack 用户账号通过每分钟发一次信号的网页客户端连接保持 |
| `dori transcribe <file>` | 用 `hooks.transcribe` 的命令把语音消息转成文字 |
| `dori can-launch` | 看看还有没有余量再开一条 lane |
| `dori inbound slack [--loop MIN]` | 不漏掉 Slack 上发给 Dori 的任何消息:Threads 视图里的未读回复、Dori 发过言的线程里的新回复(没 @ 也算)、有未读提及的私信和频道 |

没有对应命令、但模块里提供的功能:
- Telegram:以 "Thinking…" 开头的 `sendMessageDraft` 流式输出、论坛话题、HTML 表格
- Discord:创建线程、改名、归档
- Slack:上传文件
- `typingWhile`:工作进行时一直显示"正在输入"

Dori 在 Slack 上发的每条消息，不管是哪个函数发出的，都会记下所在线程。即使根消息是用原始 API 调用发的，下面没 @ 它的回复也能收到；只监听消息事件的做法会漏掉这种回复。

令牌从 `DORI_SLACK_TOKEN`(用户令牌还需要 `DORI_SLACK_COOKIE`)、`DORI_TELEGRAM_TOKEN` 和 `DORI_DISCORD_TOKEN` 读取。

## 测试

没有 CI,测试在本地运行:

```sh
cd skills/dori-mode/scripts
bun install
bun test           # 用假的 herdr、git、gh 验证行为
bunx tsc --noEmit  # 类型检查
```

测试不会碰真实的 pane、仓库或 GitHub。

## 许可证

MIT

## 从 OmOMeow 迁移

如果你之前用 gist 设置过 OmOMeow 模式，机器人照常能用。改下面四处，它就成了 Dori。

1. **起一个 Dori 名字。** 就叫 "Dori",或者用 Dori 结尾的名字，比如 ShipDori、WorkDori。对智能体说:"从现在起你叫 ShipDori,这是 ShipDori mode。"之后开启模式的说法就从 "OmOMeow mode" 换成 "ShipDori mode"。
2. **改机器人的名字和头像。**
   - Telegram:在 @BotFather 里发送 `/setname`,选中机器人，再发新名字。接着发送 `/setuserpic`,选中机器人，再发新图片。默认的 Dori 头像是 [`skills/dori-mode/assets/dori-avatar.png`](skills/dori-mode/assets/dori-avatar.png),也可以换成你喜欢的图片。机器人的名字和头像只能通过 BotFather 修改。
   - Discord:在 Developer Portal 打开你的应用。在 **Bot** 页面改用户名和图标，在 **General Information** 页面改应用名和图标(可以用同一张默认头像),然后保存。
3. **安装这个仓库。** 运行上面的一行安装命令，再对智能体说 "ShipDori mode",它就会用这个技能和 `dori` CLI,不再用以前粘贴的提示词。
4. **做一次初次了解。** 如果 OmOMeow 时期没做过，说一句"做一下 onboarding"就行。

原有的线程、话题和记忆都会保留。
