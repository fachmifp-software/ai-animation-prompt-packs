# Set up ElevenLabs MCP (so Claude can make the voices and sounds)

In these videos Claude also drives **ElevenLabs**: the narrator's voice-over, the music, the sound effects and,
most important, the **Voice Changer** trick that gives a Veo character a clean voice while the lips still match.
Claude talks to ElevenLabs through an MCP server, the same way it talks to Google Flow through
[Flow MCP](SETUP-FLOW-MCP.md). After the one-time setup below you never click around in ElevenLabs: you tell
Claude what you want, and Claude does the rest.

There are two ways to connect it (checked 21 Nov 2026):

| | **Option A: hosted MCP** (easiest) | **Option B: local MCP** (what this channel uses) |
|---|---|---|
| Install | Nothing. Claude connects to `https://api.elevenlabs.io/v1/mcp` | `uv` + the `elevenlabs-mcp` package on your computer |
| Sign-in | Your ElevenLabs account (OAuth). No API key | An API key that you create and paste once |
| Voice-over (text to speech) | Yes | Yes |
| Sound effects, music | Yes (music is not in Claude's directory connector) | Yes |
| **Voice Changer** (speech to speech) | **Not listed in ElevenLabs' docs** | **Yes** |
| Where the audio goes | A download link that expires after about 2 hours | An audio file saved on your computer |
| Status | Current, maintained by ElevenLabs | Archived by ElevenLabs on 20 Aug 2026, no longer updated, but still works |

**Which one?** If you only need a voice-over and a few sound effects, take Option A. If you want the Voice Changer
trick from the Skypulse video (pack 01, section 8), you need Option B, because that is the only one with Voice
Changer and the only one that saves the files where Claude can edit them into the video.

Both spend the credits of your own ElevenLabs account. ElevenLabs says there is a free tier with 10,000 credits a
month; this channel uses the Starter plan.

---

## Option A: hosted MCP (no install)

**Claude Code:** paste this into PowerShell (Windows) or Terminal (Mac):

```bash
claude mcp add --scope user --transport http elevenlabs https://api.elevenlabs.io/v1/mcp
```

Start Claude Code, type `/mcp`, pick **elevenlabs** and sign in with your ElevenLabs account in the browser window
that opens. Choose your workspace, check the permissions (Text to Speech, Flows, ...) and select **Authorize**.

**Claude desktop app / claude.ai:** **Settings** > **Connectors** > **Add custom connector**, name it `ElevenLabs`,
URL `https://api.elevenlabs.io/v1/mcp`, select **Add**, then sign in and **Authorize**.

Generated audio comes back as a link on `elevenlabsusercontent.com` that expires after about two hours, so ask
Claude to download what you want to keep. ElevenLabs also warns: don't ask for a retry while a generation is still
running, because every retry starts (and charges) a new one.

---

## Option B: local MCP (what this channel uses)

**1. Install `uv`** (the tool that runs the server). Skip this if you already have it.

Windows (PowerShell):

```bash
winget install astral-sh.uv
```

Mac:

```bash
brew install uv
```

Then close and reopen PowerShell / Terminal.

**2. Create an API key** at [elevenlabs.io/app/settings/api-keys](https://elevenlabs.io/app/settings/api-keys)
(Developers > API Keys > Create). Two settings on that page broke our videos, so set them now:

- **Permissions.** Switch on everything Claude will use (the labels on the page may differ slightly): **Text to Speech**, **Speech to Speech** (this is the Voice
  Changer), **Sound Effects**, **Music**, **Speech to Text** and read access to **Voices** and **Models**. Our first key
  was missing Speech to Speech, and every Voice Changer call failed with `401 ... missing the permission`.
- **Credit limit.** A key can have its own credit limit, separate from your account balance. Music costs a lot
  (about 12.5 credits per second on our account, so 3 minutes is about 2,250 credits), and twice a music job
  failed with `401 quota_exceeded` while the account still had plenty of credits. Set the limit high enough, or
  raise it before a big job.

Copy the key. ElevenLabs shows it only once.

**3. Connect it to Claude Code** (one line, with your key in place of `YOUR_KEY`):

```bash
claude mcp add --scope user elevenlabs -e ELEVENLABS_API_KEY=YOUR_KEY -- uvx elevenlabs-mcp
```

The key is stored in your Claude Code settings on your own computer. Never paste it into a chat, a video
description or a GitHub file.

**4. Restart Claude Code**, type `/mcp` and check that **elevenlabs** is connected. Then ask:

> List my ElevenLabs voices.

**Claude desktop app instead of Claude Code:** add this to `claude_desktop_config.json` (Settings > Developer >
Edit Config). On Windows, ElevenLabs says you first have to turn on Developer Mode (Help > Enable Developer Mode).

```json
{
  "mcpServers": {
    "ElevenLabs": {
      "command": "uvx",
      "args": ["elevenlabs-mcp"],
      "env": { "ELEVENLABS_API_KEY": "YOUR_KEY" }
    }
  }
}
```

### Good to know (local MCP)

- **Files it can read must be inside one folder.** Tools that read a file (Voice Changer, speech to text, audio
  isolation) only accept files inside the server's base folder, which is your **Desktop** by default. A path
  anywhere else is refused, even a full path. Either let Claude copy the file to the Desktop first, or set the
  base folder to your project, for example:
  `claude mcp add --scope user elevenlabs -e ELEVENLABS_API_KEY=YOUR_KEY -e ELEVENLABS_MCP_BASE_PATH=C:\Users\you\Videos -- uvx elevenlabs-mcp`
- **The output folder must already exist**, or the tool says it is "not writeable". Files get automatic names
  (`tts_...`, `sts_...`, `sfx_...` plus a time stamp), so ask Claude to rename them right away.
- **"spawn uvx ENOENT"** means Claude can't find `uvx`. Find its full path (`where uvx` on Windows, `which uvx` on a
  Mac) and use that path instead of `uvx` in the command.
- **Write "Vay-oh"** when a script says "Veo", or the voice reads it as "V-O".

---

## The Voice Changer trick (why Option B is worth it)

1. Generate the Veo clip with the character saying the line (Veo makes the voice and moves the lips).
2. Claude takes the clip's audio and runs it through ElevenLabs **Voice Changer** with the voice you want.
3. Claude puts the new audio back on the clip.

Voice Changer keeps the exact timing of the original line, so the lips still match. The narrator on this channel
is made the same way. The full prompts are in [pack 01, section 8](01-skypulse-pixar-style/README.md).

---

ElevenLabs MCP (local, archived): https://github.com/elevenlabs/elevenlabs-mcp. Hosted MCP docs:
https://elevenlabs.io/docs/eleven-api/resources/mcp-server. This channel is not affiliated with or endorsed by
ElevenLabs, Anthropic or Google.
