# Installing MCP for Blender on Blender 4.5 + Claude Desktop

Blender 4.5 is supported — the addon declares a minimum of Blender 3.0 and
`install-addon` discovers versioned addon folders (`4.2`, `4.3`, `4.5`, …),
including the 4.2+ extensions layout.

Run every command below on **your own PC**, in your own terminal.

## 1. One-command install (recommended)

**Windows** (PowerShell):

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://www.mcp-for-blender.com/install.ps1 | iex"
```

**macOS / Linux**:

```bash
curl -LsSf https://www.mcp-for-blender.com/install.sh | sh
```

This installs [uv](https://docs.astral.sh/uv/) if missing, detects the MCP
clients on the machine (Claude Desktop among them), lets you pick which to
configure, then copies the Blender addon into your addons folder and enables
it. It only adds a `blender` entry to existing config and keeps a `.bak` of
every file it changes.

Already have uv? Run `uvx mcp-for-blender setup` instead.

Then **fully quit and reopen Claude Desktop** — on Windows quit it from the
system tray (right-click → Quit), on macOS press <kbd>Cmd</kbd>+<kbd>Q</kbd>.
Closing the window is not enough.

## 2. Manual setup, if the installer doesn't work

### 2a. Install uv

Use uv's **official installer — not `pip install uv`**, which may not create
the `uvx` command.

```powershell
# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# then put uv on the user PATH (restart Claude Desktop afterwards)
$localBin = "$env:USERPROFILE\.local\bin"
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
[Environment]::SetEnvironmentVariable("Path", "$userPath;$localBin", "User")
```

```bash
# macOS
brew install uv

# Linux
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Do not continue until `uvx --version` works in a new terminal.

### 2b. Point Claude Desktop at the server

**Claude Desktop → Settings → Developer → Edit Config**, which opens
`claude_desktop_config.json`:

| OS | Path |
|---|---|
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` (Microsoft Store build: `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\`) |
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

```json
{
    "mcpServers": {
        "blender": {
            "command": "uvx",
            "args": ["mcp-for-blender"]
        }
    }
}
```

If the file already has `mcpServers`, add the `blender` key inside it rather
than replacing the object.

### 2c. Install the Blender addon

```bash
uvx mcp-for-blender install-addon
```

It prints where it wrote `blender_mcp.py` and keeps a `.bak` of anything it
replaced. `uvx mcp-for-blender addon-paths` lists the addon folders it found;
override the destination with `BLENDERMCP_ADDONS_DIR=/path/to/scripts/addons`.

Then in Blender 4.5: **Edit → Preferences → Add-ons**, search
"MCP for Blender", and enable **Interface: MCP for Blender**.

If it isn't listed, install the file by hand — in 4.2+ the button moved into
the dropdown (**∨**) at the top-right of the Add-ons preferences:
**Install from Disk…** → select the copied `blender_mcp.py` (or this repo's
`addon.py`) → enable it.

## 3. Connect

1. Open Blender. The addon starts its server when Blender opens.
2. In the 3D viewport press <kbd>N</kbd> for the sidebar → **MCP for Blender**
   tab.
3. Tick the capabilities you want (Poly Haven, etc.), and click
   **Start MCP Server** if it isn't already running.
4. In Claude Desktop you should now see the tools listed under the
   tools/hammer icon. Ask it to build something.

## Gotchas

- **Run only one MCP client at a time** against Blender — Claude Desktop *or*
  Cursor, not both.
- **Don't run `uvx mcp-for-blender` yourself in a terminal.** Claude Desktop
  launches the server; a second copy fights the first for the port.
- **`spawn uvx ENOENT`** — GUI-launched apps don't inherit your terminal PATH.
  Use the absolute path from `where uvx` / `which uvx` as `"command"`
  (e.g. `C:\Users\<you>\.local\bin\uvx.exe`, `/opt/homebrew/bin/uvx`), or on
  Windows wrap it: `"command": "cmd", "args": ["/c", "uvx", "mcp-for-blender"]`.
- **conda / pyenv / asdf on the machine** — pin the interpreter so uv doesn't
  pick a broken one:

  ```json
  {
      "mcpServers": {
          "blender": {
              "command": "uvx",
              "args": ["--python", "3.11", "mcp-for-blender"],
              "env": { "UV_PYTHON_PREFERENCE": "only-managed" }
          }
      }
  }
  ```

- **A fixed failure keeps replaying** — clear the cache:
  `uv cache clean mcp-for-blender blender-mcp && uvx --refresh mcp-for-blender`
- **Still not connecting** — restart both Blender and Claude Desktop. The first
  request after a cold start sometimes doesn't go through.
- `execute_blender_code` runs arbitrary Python inside Blender. Save your work
  before using it.
