# Decision Curve

A live classroom strategy game in a single HTML file. Teams vote on their phones, a captain locks each team's answer, and every decision moves the team's score curve on the big screen. At the end, each team is compared with a benchmark: what the real company actually did.

It plays like Kahoot, but the result is a line chart that builds up round by round, not a points table.

- **One file, no server, no sign-in.** Open `index.html` from any static host (GitHub Pages, Netlify).
- **Fully editable.** The built-in editor changes the title, rounds, options, scoring, teams and background. You can export the settings as JSON and share them.
- **Phones follow the big screen.** The host broadcasts the game settings, so phones always show the presenter's version.
- **Team decisions stay consistent.** The first player in each team becomes captain, and the captain's lock is the team's answer. With no lock, the team's majority counts. The presenter can override any team.
- **Resilient.** Phones connect to several public relays at once. If a phone can't connect, the presenter can type that team's answer, so the game always runs.

## Quick start

1. Publish `index.html` (see [Hosting](#hosting)).
2. On the presenter's laptop, open the page and choose **Edit game** to set up your rounds, or use the built-in LEGO example.
3. Choose **Host a game**. The big screen shows one QR code and one PIN per team, plus a game code.
4. Players scan their table's QR code and enter their team's PIN.
5. Run the rounds: **Start round → Reveal results → Next round → Show final results**.

## How scoring works

Every team starts at the same score. Each option changes the score in two steps:

```
new score = score + |score| × growth + points
```

- **growth** scales with the current score: 0.5 means +50%, −1 means −100%.
- **points** is a flat amount added after the growth.

The benchmark line follows the real answer of each round. You can also type the real yearly values for a round (one number per year, start year included), and the benchmark will follow them exactly.

At the end, teams are ranked by final score. Two badges are awarded: **Beat [benchmark]** for every team that finished above the benchmark, and **Closest to [benchmark]'s real path** for the team whose curve stayed nearest to it.

The editor's preview draws four teams that always pick A, B, C or D. Use it to check that the right answer wins and the wrong ones lose.

## Settings file

Everything lives in one JSON object. `examples/lego-ceo-challenge.json` is a full example. Main fields:

| Field | What it does |
| --- | --- |
| `title` | Game name on every screen |
| `room` | Optional fixed game code (3–12 letters or digits). Leave empty to get a random code |
| `startYear`, `score.start` | Where every curve starts |
| `score.auto`, `score.min`, `score.max` | The score axis: fit automatically, or a fixed range |
| `benchmark` | Name and colour of the real path |
| `theme` | `bg` background colour, `pattern` (`studs`, `dots`, `grid`, `none`) and `patternColor` |
| `teams[]` | 2–6 teams: `label`, `color`, `text` colour, `pin` (3–6 digits, all different) |
| `rounds[]` | 1–9 rounds: `end` year, `label`, `context`, four `options` (`t` text, `k` growth, `a` points, `o` outcome), `real` answer (0–3), optional `realPath`, `story` |

Settings received over the network are checked before use: text is length-limited and always shown as plain text, colours must be `#rrggbb`, and numbers are range-checked.

## Hosting

Any static host works. With GitHub Pages:

1. Put `index.html` at the root of the repository.
2. Go to **Settings → Pages**, set the source to the `main` branch, root folder.
3. After a minute, the game is live at `https://<user>.github.io/<repo>/`.

## How it connects

There is no backend. Phones and the big screen exchange small messages through public MQTT test relays over secure WebSockets:

- `broker.emqx.io:8084`
- `broker.hivemq.com:8884`
- `mqtt.eclipseprojects.io:443`
- `test.mosquitto.org:8081`

All four are used in parallel, so one relay going down doesn't stop the game. Each game gets its own code, so different games don't mix.

Things to know before using it in a class:

- **Public relays have no guarantee.** They are free test services run by others and can be slow or offline. Do a rehearsal on the venue's Wi-Fi first.
- **Messages are not private.** Anyone who knows a game code can read its messages. The game sends no names or personal data, only team choices, but don't put anything confidential in the settings.
- **Some networks block the relay ports** (8084, 8884, 8081). If phones stay on "Connecting…", try mobile data.
- **Use your own relay if you need to.** Add `?broker=wss://your-relay/mqtt` (comma-separate several) to the URL. Any MQTT broker with WebSockets works.

Tested with 50 players in one room and 100 players over the public relays: every vote arrived and every reveal was correct. See `TESTING.md` for the concurrency tests (everyone joining at once, captain races, last-second votes, network drops).

## Credits and licence

MIT licence; see `LICENSE`. Libraries loaded from CDNs: [MQTT.js](https://github.com/mqttjs/MQTT.js) (MIT) and [QRCode.js](https://github.com/davidshimjs/qrcodejs) (MIT). Fonts: Lilita One and Figtree from Google Fonts (SIL Open Font Licence).

The LEGO example is a teaching case about the LEGO Group's history (1949–2004). Its scores are a stylised index made for the game, not reported figures. LEGO® is a trademark of the LEGO Group, which does not sponsor, authorise or endorse this project.

---

### 中文简介

Decision Curve 是一个单文件的课堂策略游戏：各组用手机投票，队长锁定答案，每一轮的决策都会改变大屏上的分数曲线，最后和真实公司的发展路径对比。首页点 **Edit game** 就能修改题目、选项、计分规则、队伍和背景颜色，设置可以导出成 JSON 分享给别人。把 `index.html` 放到 GitHub Pages 或 Netlify 上就能用，不需要服务器。正式上课前，请先在教室的网络下彩排一次。
