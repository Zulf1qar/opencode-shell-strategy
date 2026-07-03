# Shell Non-Interactive Strategy -- PowerShell 7 / Windows

OpenCode's shell is `pwsh` with no TTY. Any command that waits for input hangs until timeout.
This machine is Windows 11 Pro. There is no bash, no `sudo`, no Unix userland outside WSL.
Unix habits like `grep`, `cat`, `ls`, `find`, `touch` have tool or cmdlet replacements -- see `AGENTS.md`.

## Banned commands (always hang or block)

- Editors: `vim`, `vi`, `nano`, `emacs`, `notepad`, `notepad++` -- never launch an editor or GUI app
- Pagers: `more`, `Out-Host -Paging`, `Get-Help -ShowWindow`, `git log` or `git diff` without `--no-pager`
- Interactive git: `git add -p`, `git rebase -i`, `git commit` without `-m`
- Bare REPLs: `python`, `py`, `node`, nested `pwsh` -- use `-c`, `-e`, `-Command`, or a script file
- Any cmdlet left waiting on a `-Confirm` prompt

## Non-interactive flags -- common cases

NPM: `npm install --yes`, `npm init -y`
Winget: `winget install <id> --silent --accept-source-agreements --accept-package-agreements`
PIP: `py -3.11 -m pip install --no-input <pkg>`
Git commit: `git commit -m "msg"`
Git merge/pull: add `--no-edit`
Git log/diff: `git --no-pager log -n 20`, `git --no-pager diff`
curl: `curl.exe -fsSL <url>` -- bare `curl` is an `Invoke-WebRequest` alias (see `AGENTS.md`)
ssh: `ssh -o BatchMode=yes -o StrictHostKeyChecking=no <host>`
Docker: never `-it`. Compose: `docker compose up -d`
unzip: `Expand-Archive -Path <zip> -DestinationPath <dir> -Force`

Destructive operations (`Remove-Item`, Prisma resets, destructive SQL) are permission-gated.
Never add `-Force` or `-Confirm:$false` to push through a safety prompt -- report and wait.

## Environment variables

Set these in the same command when a tool might prompt (scoped to that session):

```powershell
$env:CI = 'true'
$env:GIT_TERMINAL_PROMPT = '0'
$env:npm_config_yes = 'true'
$env:PIP_NO_INPUT = '1'
```

## WSL escape hatch (non-DB only)

```powershell
wsl.exe -d Ubuntu -e bash -c "apt-get install -y <pkg>"
```

Inside WSL: `apt-get -y` and `DEBIAN_FRONTEND=noninteractive` apply.
Never use WSL for `psql` or Prisma -- see `AGENTS.md` Prisma section.

## When no flag exists

Pipe known input: `'y' | .\installer.ps1` -- only when the prompt is known and expected.
Bound long output: pipe to `Select-Object -First 50` instead of paging.
Multi-line input: write it to a temp file and pass the file path to the command.

## Rule

If a command might prompt and no non-interactive flag exists: stop and report. Do not run it.
