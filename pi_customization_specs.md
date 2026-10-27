# Specifications for pi_customization

This document describes the behavior of the `.pi_customization` script: a zsh wrapper around the
`pi` coding agent (`@earendil-works/pi-coding-agent`, verified against 0.85.1). It is sourced from
`.nilesh` and defines a `pi()` function that adds per-project encrypted provider keys, a session
picker, and terminal recording on top of the real `pi` binary.

## 1. Command Flow

Typing `pi [args...]` runs these steps in order:

1. **Pass-through check** - subcommands and print-and-exit options go straight to `pi` (section 2).
2. **Project files** - locate this folder's files under `~/.pi/wrapper` and migrate any old ones (section 3).
3. **Session menu** - new session (with a name) or pick up a previous one (section 4).
4. **Provider & model** - unlock the provider file and choose a provider and model (section 5).
5. **Session rules** - attach `~/.pi/session_rules.md` to the system prompt (section 6).
6. **Run & record** - run `pi` under `script(1)` and save the terminal log (section 7).

## 2. Pass-Through

These run `command pi "$@"` directly, with no menus, no password prompt, and no recording, and
create nothing under `~/.pi/wrapper`:

-   **Subcommands** (first argument): `install`, `remove`, `uninstall`, `update`, `list`, `config`, `auth`.
-   **Options** (anywhere in the arguments): `--help`/`-h`, `--version`/`-v`, `--list-models`, `--export`.

## 3. Project Files

### 3.1. Layout

All per-project state lives outside the project folder, so keys and logs can never be committed:

```
~/.pi/wrapper/                      (700)
└── projects/
    └── <project key>/              (700)
        ├── providers.enc           (600) encrypted provider keys
        └── sessions/               terminal logs (*.log)
```

-   **Project key**: the current directory with symlinks resolved, the leading `/` removed, and
    `/`, `\` and `:` replaced by `-`, wrapped in `--...--` (e.g. `--Users-me-work-proj--`). This is
    exactly how pi names its own session folders under `~/.pi/agent/sessions/`, so the wrapper's
    folder and pi's folder for a project have the same name.
-   A project is the directory `pi` is run from; a subfolder is a separate project.
-   **Model catalog**: `.pi_models.conf` is shared by all projects and lives next to the script (in
    this repo). If missing, a default is written.

### 3.2. Migration From Older Versions

Older versions kept `.pi_providers.enc` and `.pi_sessions/` in the project folder. When `pi` or
`pi_show_providers` runs in a folder that still has them:

-   `.pi_providers.enc` is moved to `providers.enc`. The same password keeps working. If a
    `providers.enc` already exists, the old file is left in place with a warning.
-   Every file in `.pi_sessions/` (including hidden temp logs) is moved to `sessions/` without
    overwriting; the folder is removed once empty. Logs whose names collide are left behind with a warning.
-   Each move is reported. Nothing is created when there is nothing to migrate.

## 4. Session Menu

pi owns conversation history: it auto-saves every session to `~/.pi/agent/sessions/<project key>/`
as JSONL, with no prompt on exit. The wrapper only helps choose which session to open.

### 4.1. When It Is Shown

The menu is skipped when the user's arguments already decide the session or run pi
non-interactively: `-c`/`--continue`, `-r`/`--resume`, `--session`, `--session-id`, `--fork`,
`--no-session`, `-n`/`--name`, `-p`/`--print`.

### 4.2. Choices

If this folder has previous pi sessions:

```
Session for this folder:
1) New session
2) Continue last session ("fix the login bug", 2 hours ago)
3) Choose a previous session...
```

-   **New session** - asks `Session name (optional, press Enter to skip)`. A name is passed to pi as
    `--name`, which labels the session in `pi -r` / `/resume` and also names the log file (section 7).
-   **Continue last session** - adds `--continue`. The label is the session's display name (latest
    `session_info` entry) or, if unnamed, its first user message (collapsed to one line, max 50
    characters), plus its age (`just now`, `N min ago`, `N hours ago`, `on YYYY-MM-DD`). The label
    needs `jq`; without it only the age is shown.
-   **Choose a previous session** - adds `--resume`, which opens pi's own picker (search, rename, delete).

If the folder has no previous sessions, the menu is skipped and only the name is asked.
An invalid choice re-shows the menu, up to the retry limit (section 8).

## 5. Provider & Model Selection

### 5.1. Storage and Encryption

-   **Format**: one `provider=key` line per provider.
-   **Encryption**: `openssl enc -aes-256-cbc -pbkdf2`, password-based. The file is created with
    `umask 077`, so it is private from the start.
-   **Valid entries**: the provider name must be letters, digits, `-` or `_`; the key must be
    non-empty printable text (backslashes are kept as typed). Entries are validated when added, so
    anything saved can always be read back.
-   **Wrong-password detection**: openssl's exit status alone is not enough - about 1 in 256 wrong
    passwords still decrypt to garbage. Decryption is accepted only if every non-empty line is a
    valid entry.

### 5.2. Password Prompt

The password is never shown while typing and is always entered twice. Mismatches re-ask, up to
the retry limit. This applies to creating the file, unlocking it, and `pi_show_providers`.

### 5.3. Workflows

-   **First run** (no provider file): offer to create one (`[Y/n]`), set a password, then add the
    first provider. Invalid input re-asks up to the retry limit; if it never succeeds, no file is
    written (an empty file would look like a bad password forever).
-   **Standard run**: unlock the file and show:
    -   A numbered list of saved providers.
    -   `[Add a new provider]` - on success the key is saved and that provider is used for this
        session immediately (no re-run). On blank/invalid input the menu is shown again, up to the
        retry limit. Adding a name that already exists replaces its key in place (menu order is kept).
    -   `[Continue without a provider]` - pi uses its own defaults. Any non-numeric or
        out-of-range choice also continues without a provider.
-   **Model selection**: models for the chosen provider come from that provider's `[section]` in
    `.pi_models.conf` (`model_id="Display Name"`). An invalid choice aborts.

### 5.4. Credential Injection

-   The wrapper adds `--provider <name> --model <id>` to the pi command line.
-   The API key is exported in the environment variable pi reads for that provider. Most follow
    `PROVIDER_API_KEY` (upper-cased, `-` becomes `_`); exceptions from pi's provider table include
    `google` → `GEMINI_API_KEY`, `google-vertex` → `GOOGLE_CLOUD_API_KEY`,
    `azure-openai-responses` → `AZURE_OPENAI_API_KEY`, `huggingface` → `HF_TOKEN`.

### 5.5. `pi_show_providers [--reveal]`

Prints the current project's providers after asking for the password. Keys are masked
(`sk-a...7890`; keys of 8 characters or fewer show as `****`) unless `--reveal` is given.

## 6. Session Rules

-   `~/.pi/session_rules.md` is passed as `--append-system-prompt` on every recorded session, and a
    status line is printed: `Session rules: ~/.pi/session_rules.md (20 lines)`.
-   If the file is missing, empty, or unreadable, a warning is printed and the flag is not passed.
    (pi treats a non-existent path as literal text, so passing it anyway would append the path
    itself to the system prompt.)
-   Passing this flag stops pi from auto-loading `~/.pi/agent/APPEND_SYSTEM.md`.

## 7. Session Recording

-   Every session that is not passed through runs under `script -q`, recording the terminal to a
    temporary file in `sessions/`. These logs are for human reading only; they are not fed back into pi.
-   **Naming**, decided when the session ends:
    1.  The session name, if one was given (`--name`/`-n` or the session menu).
    2.  Otherwise a timestamp of when the session ended (`2026-10-27_14-30-00.log`). Continued and
        resumed sessions get a new timestamped log; older logs are never appended to.
-   **Sanitization**: anything outside `a-z A-Z 0-9 . _` becomes `-`, repeated dashes are merged,
    and leading/trailing dashes are dropped. A name that sanitizes to nothing falls back to the timestamp.
-   **No overwrites**: if the log name is taken, `-2`, `-3`, ... is appended.
-   **Interrupted sessions** (e.g. Ctrl+C before the log is named): the partial log is kept as
    `<timestamp>_INCOMPLETE.log`. Closing the terminal kills the shell before this can run, which
    can leave a hidden `.session_inprogress_<pid>.tmp` behind.
-   `--name`/`-n` without a value is dropped with a warning. pi's other flags pass through untouched.

## 8. Configuration

| Setting | Default | Purpose |
|---|---|---|
| `PI_MAX_RETRIES` | `3` | Attempts for re-asked prompts: password confirmation, adding a provider, the session menu. Read when the script is sourced. |
| `~/.pi/session_rules.md` | - | Rules appended to the system prompt (section 6). |
| `.pi_models.conf` | default written if missing | Models offered per provider (section 5.3). |

## 9. Function Map

| Function | Responsibility |
|---|---|
| `pi()` | Orchestrator: pass-through, then steps 2-6 of section 1. |
| `_pi_project_key()` / `_pi_project_files()` | Project folder name; file locations and migration. |
| `_pi_session_menu()` / `_pi_session_label()` | Session menu; describing the last pi session. |
| `_pi_handle_providers()` | Provider workflows and credential injection. |
| `_pi_get_password()` / `_pi_unlock()` | Hidden, confirmed password entry; decrypting the provider file. |
| `_pi_encrypt_providers()` / `_pi_decrypt_providers()` / `_pi_valid_entry()` | Encryption and validation. |
| `_pi_add_new_provider()` | Adding or replacing a provider key. |
| `_pi_menu()` / `_pi_model_menu()` | Generic numbered menu; model menu from `.pi_models.conf`. |
| `_pi_provider_env_var()` | Provider → API-key environment variable. |
| `_pi_parse_args()` | Reads `--name`/`-n` for the log file name, keeping it for pi. |
| `_pi_run_new_session()` / `_pi_log_path()` / `_pi_sanitize_filename()` | Recording and log naming. |
| `pi_show_providers()` | Public helper (section 5.5). |

## 10. Superseded Designs

-   **Wrapper-level `--resume` (removed)**: the first version listed the wrapper's logs, passed the
    chosen log's full text to pi as the opening prompt, and appended to it with `script -a`. It
    was removed because it skipped provider selection, broke on logs over ~1 MB (the
    command-line length limit), and fed pi a screen recording full of escape codes rather than a
    conversation. pi's native sessions (`-c`, `-r`, `--session`, `--fork`) replace it, now
    offered through the session menu (section 4).
-   **`--api-key` injection (never used)**: credentials are passed through environment variables
    (section 5.4) instead.
-   **In-project storage (migrated)**: `.pi_providers.enc` and `.pi_sessions/` in the project
    folder were replaced by `~/.pi/wrapper` (section 3).

## 11. Known Gaps

-   **Password visible to other processes**: `openssl -pass pass:<password>` puts the password on
    the command line while openssl runs, where `ps` can see it.
-   **API key outlives the session**: the exported key stays in the interactive shell after `pi`
    exits, instead of being held only for the session.
-   **Debug output**: `DEBUG: Running command -> ...` is printed before every session.
-   **Startup message**: `Loading Pi customization functions...` is printed in every new shell.
-   **Moved projects**: per-project folders are keyed by path, so renaming or moving a project
    starts a fresh provider file and session list (pi's own history has the same limitation).
