# OpenCode-Finish-Sound-Plugin
OpenCode Finish Sound Plugin plays a song when its in idle mode.
cat << 'EOF' > README.md
# OpenCode Finish Sound Plugin (Windows)

A simple OpenCode plugin for Windows that plays a sound when a session finishes
(when OpenCode becomes idle).

Useful if you want an audible notification after long runs or background tasks.

---

## Features

- Plays a sound when an OpenCode session finishes
- Uses the session.idle event (most reliable signal)
- Windows-only (PowerShell-based)
- Very small and easy to customize

---

## Requirements

- Windows
- OpenCode (tested with v1.1.x)
- PowerShell available in PATH (default on Windows)

---

## Installation

1. Create the plugins directory (if needed)

Command (Windows CMD):

    mkdir %USERPROFILE%\.config\opencode\plugins

Some OpenCode installs also load plugins from:

    C:\Users\<USERNAME>\.opencode\plugins

---

2. Create the plugin file

Create this file:

    finish-sound.ts

Location:

    %USERPROFILE%\.config\opencode\plugins\finish-sound.ts

---

3. Plugin code (beep on finish)

Paste the following code into finish-sound.ts:

    import type { Plugin } from "@opencode-ai/plugin";
    import { spawn } from "child_process";

    function beep() {
      spawn(
        "powershell.exe",
        ["-NoProfile", "-Command", "[console]::beep(1200,200)"],
        { windowsHide: true, stdio: "ignore" }
      );
    }

    export const FinishSound: Plugin = async () => {
      return {
        event: async ({ event }) => {
          if (event.type === "session.idle") {
            beep();
            return;
          }

          if (event.type === "session.status") {
            const props: any = (event as any).properties;
            if (props?.status?.type === "idle") {
              beep();
            }
          }
        },
      };
    };

---

## Optional: Play a WAV file instead of a beep

You can replace the beep with a WAV file using PowerShell SoundPlayer.

Example helper function:

    function playWav(path: string) {
      spawn(
        "powershell.exe",
        [
          "-NoProfile",
          "-Command",
          "(New-Object Media.SoundPlayer '" + path.replace(/'/g, \"''\") + "').PlaySync()"
        ],
        { windowsHide: true, stdio: "ignore" }
      );
    }

Then call:

    playWav("C:\\Windows\\Media\\notify.wav");

WAV files are recommended for reliability on Windows.

---

## How It Works

- OpenCode emits multiple events during a session
- When execution finishes, it emits:
  - session.idle (most reliable)
  - or session.status with status.type = idle
- This plugin listens for those events and triggers a sound

---

## Debugging Tips

- Restart OpenCode after adding or changing plugins
- Test PowerShell sound manually:

    powershell -NoProfile -Command "[console]::beep(1000,300)"

- Make sure the plugin file is in a loaded plugins directory

---

## License

MIT — do whatever you want 🙂
EOF
