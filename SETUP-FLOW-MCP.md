# Set up Flow MCP (so Claude can drive Google Flow)

Every video and prompt pack in this repo was made the same way: **Claude Code** writes the prompts and drives
**Google Flow** (Nano Banana Pro images, Veo 3.1 videos) through **[Flow MCP](https://github.com/GTAI-1/flow-mcp)**.
Flow MCP is a free, open-source tool by GTAI-1. I didn't make it; I use it in every video.

It runs on your own computer, opens its own Chrome window signed in to your Google account, and clicks Flow's
real buttons for Claude. So it spends your normal Flow credits, not the pay-per-use Veo API.

This page covers setting it up on Windows (the channel runs it on Windows 11) or Mac, **plus a fix for a bug
in the current version** (checked 21 Nov 2026).

---

## What you need

| | |
|---|---|
| **Claude Code** | Needs a paid Claude plan (Pro or higher) |
| **A Google account** | The videos here use the **Google AI Pro** plan: Nano Banana Pro images cost 0 credits, a Veo 3.1 Fast clip (8 s) costs 20 credits. Flow MCP's own guide says a free account gets 50 Flow credits a day |
| **Windows 10/11 or a Mac** | It can't run in the cloud or on a phone. It drives a real Chrome window, so keep the computer awake while it generates |
| **Node.js 22+, Google Chrome, Git** | Option 1 below installs them for you |
| **FFmpeg** *(optional)* | Only needed for Flow MCP's own joining, music and narration tools |

---

## Option 1: let Claude Code set it up (easiest)

**1.** Open Claude Code in any folder and paste this:

> Set up Flow MCP from https://github.com/GTAI-1/flow-mcp for me. Install anything it needs that I don't have
> yet (Node.js 22 or newer, Google Chrome, Git, FFmpeg). Clone it into my home folder, not into Desktop,
> Documents, Downloads or a OneDrive folder. Run npm install. Then open src/flow.ts: if it finds Flow's Agent
> button with `name: "Agent", exact: true`, replace every one of those with `name: /^(Flow )?Agent$/`.
> Then run npm run build, connect it to Claude Code with `claude mcp add --scope user flow -- node <full path to flow-mcp>/dist/index.js`,
> and finally run npm run login.

It asks before each step. Say yes. (The `src/flow.ts` part is the bug fix explained further down.)

**2.** Two things only you can do:

- Type your computer password if an installer asks for it. Claude never types it.
- **Sign in to Google** in the Chrome window that `npm run login` opens. Then open a Flow project (or create
  one) and leave that window open. This Chrome is separate from your everyday one, and the tool never sees
  your Google password.

**3.** Quit Claude Code and start it again, so it loads the new tool. Then ask:

> Check my Flow session.

You should get your plan and credit balance back. Done.

---

## Option 2: set it up by hand

Paste these into **PowerShell** (Windows) or **Terminal** (Mac), one at a time.

**1. Install the helpers** (skip any you already have)

Windows:

```bash
winget install OpenJS.NodeJS.LTS
winget install Google.Chrome
winget install Git.Git
winget install Gyan.FFmpeg
```

Then close and reopen PowerShell.

Mac (with [Homebrew](https://brew.sh)):

```bash
brew install node git ffmpeg
```

**2. Get Flow MCP and build it**

```bash
cd ~
git clone https://github.com/GTAI-1/flow-mcp.git
cd flow-mcp
npm install
```

Keep it in your home folder (`C:\Users\<you>\flow-mcp` or `~/flow-mcp`). Not in Desktop, Documents or
Downloads, and not in a folder OneDrive syncs.

**3. Apply the "Flow Agent" fix** (see [Known issue](#known-issue-every-generation-fails-on-the-agent-button) below), then build:

```bash
npm run build
```

**4. Sign in to Flow, once**

```bash
npm run login
```

Sign in to Google in the Chrome window that opens, open (or create) a Flow project, and leave it open.

**5. Connect it to Claude Code**

Windows:

```bash
claude mcp add --scope user flow -- node "C:\Users\<you>\flow-mcp\dist\index.js"
```

Mac:

```bash
claude mcp add --scope user flow -- node ~/flow-mcp/dist/index.js
```

`--scope user` makes it work in every folder. Restart Claude Code, type `/mcp` and check that **flow** is
connected, then ask: *"Check my Flow session."*

---

## Known issue: every generation fails on the "Agent" button

**What you see:** every image or video job fails with an error like

```
waiting for getByRole('button', { name: 'Agent', exact: true })
```

**Why:** Google renamed the **Agent** button in Flow to **Flow Agent**. The current Flow MCP (checked 21 Nov 2026)
still looks for the old name, so it never finds the button.

**Fix:** in your `flow-mcp` folder, open `src/flow.ts` and replace every

```ts
name: "Agent", exact: true
```

with

```ts
name: /^(Flow )?Agent$/
```

There are two of them. The new version matches both the old and the new button name. Then run
`npm run build`, and in Claude Code type `/mcp` → **flow** → **Reconnect** (or restart Claude Code). The Flow
Chrome window reopens on Flow's home page, so open your project again.

Or let Claude do it. Paste into Claude Code:

> In my flow-mcp folder, in src/flow.ts, replace every `name: "Agent", exact: true` with
> `name: /^(Flow )?Agent$/`, then run npm run build.

**Updating Flow MCP later:** your edit can block `git pull`. Undo it first, then update, then check whether the
new version still needs the fix:

```bash
git checkout -- src/flow.ts
git pull
npm install
npm run build
```

---

## Tips from making these videos

- **Keyframes first.** Nano Banana Pro images cost 0 credits on AI Pro. Have Claude make and check every first
  frame before a single video credit is spent, then use that keyframe as the first frame of the Veo clip.
- **Set a credit limit per clip.** Flow MCP reads Flow's live price before it generates and skips a clip that
  costs more than your limit. Tell Claude something like *"max 20 credits per clip"*.
- **Look at every clip after it downloads.** Flow gives tiles automatic titles and reuses them. Once, a job
  reported done but downloaded an older clip with the same title, and some jobs sat out the full timeout
  before giving up. The real clip was still in Flow. Ask Claude to download it again by its title: that's free.
  Never generate it again and pay twice.
- **Characters:** attach the character's portrait as a **file** in the reference images. Pointing at a saved
  Flow character (`asset:<name>`) made the download step fail in our first video.
- **"We noticed some unusual activity"** on a failed tile is Google's bot check. You're not charged. Stop,
  wait, and run one job at a time. Don't keep retrying.
- **"Audio generation failed"** on a Veo clip: Flow refunds the credits. Just try again.

---

Flow MCP: https://github.com/GTAI-1/flow-mcp (its own `GETTING-STARTED.md` covers more apps: the Claude
desktop app, Codex, Antigravity). This channel is not affiliated with or endorsed by GTAI-1, Anthropic or Google.
