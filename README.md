# Claude Code Template

A plug-and-play setup for Claude Code with pre-configured workflows, skills, and behavioral patterns.

---

## Installing Claude Code

### macOS

1. **Open Terminal** (press `Cmd + Space`, type "Terminal", hit Enter)

2. **Install Claude Code:**
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

   If you don't have npm, install Node.js first from [nodejs.org](https://nodejs.org/) or via Homebrew:
   ```bash
   brew install node
   ```

3. **Verify installation:**
   ```bash
   claude --version
   ```

### Windows

1. **Open PowerShell** (press `Win + X`, select "Windows PowerShell" or "Terminal")

2. **Install Claude Code:**
   ```powershell
   npm install -g @anthropic-ai/claude-code
   ```

   If you don't have npm, install Node.js first from [nodejs.org](https://nodejs.org/)

3. **Verify installation:**
   ```powershell
   claude --version
   ```

### First Run

The first time you run `claude`, you'll be prompted to authenticate with your Anthropic account.

```bash
claude
```

Follow the prompts to log in.

---

## Setting Up This Template

Once Claude Code is installed:

1. **Download this repository** (clone or download ZIP)

2. **Open Claude Code in this folder:**
   ```bash
   cd path/to/claude-code-template
   claude
   ```

3. **Tell Claude to set it up:**
   ```
   Set up this template for me
   ```

Claude will ask you a few questions (name, role, timezone) and configure everything automatically.

---

## What's Included

### Behavioral Patterns

The template teaches Claude how to work effectively:

- **Ask before acting** - Clarifies ambiguous requests instead of guessing
- **Save before responding** - Captures decisions and progress to memory
- **Execute, don't narrate** - Does the work instead of explaining what it will do
- **Add value** - Every response provides substance, not just confirmation

### Slash Commands

| Command | What It Does |
|---------|--------------|
| `/day-over` | Create end-of-day summary |
| `/standup` | Generate standup update |
| `/morning-briefing` | Daily overview |
| `/week-review` | Weekly summary |
| `/memory-review` | Review persistent memory |
| `/project-status` | Check project status |

### Skills (17 included)

Claude automatically activates relevant skills based on what you're working on:

- **Development**: React/Next.js, Node.js, databases, testing, Chrome extensions
- **Design**: UI/UX principles, design systems
- **Content**: Technical writing, PRDs, note-taking
- **Automation**: n8n workflows, prompt engineering, shell scripting

---

## Optional Integrations

The template works standalone. These integrations enhance it:

| Integration | What It Adds |
|-------------|--------------|
| Memory Keeper | Remember context across sessions |
| Google Calendar | Calendar-aware briefings |
| Slack | Team communication |
| GitHub | Code-aware commands |

See `mcp-servers/` for setup guides.

---

## Customization

After setup:

- **Change behavior**: Edit `~/.claude/CLAUDE.md`
- **Adjust writing style**: Edit `~/.claude/context/writing-style.md`
- **Add commands**: Create files in `~/.claude/commands/`
- **Add skills**: Create folders in `~/.claude/skills/`

See [PERSONALIZATION.md](PERSONALIZATION.md) for details.

---

## What Gets Installed

```
~/.claude/
├── CLAUDE.md              # Main instruction file
├── context/
│   ├── writing-style.md   # Communication style
│   ├── current-session.md # Session tracking
│   └── tasks/             # Task workflows
├── commands/              # 6 slash commands
└── skills/                # 17 skills
```

---

## Troubleshooting

### "npm: command not found"
Install Node.js from [nodejs.org](https://nodejs.org/) (LTS version recommended)

### "claude: command not found" after install
Try closing and reopening your terminal, or run:
```bash
# macOS/Linux
export PATH="$PATH:$(npm bin -g)"

# Or find where npm installed it
npm bin -g
```

### Permission errors on macOS
```bash
sudo npm install -g @anthropic-ai/claude-code
```

### Permission errors on Windows
Run PowerShell as Administrator (right-click → "Run as administrator")

---

## License

MIT - Use freely, modify as needed.
