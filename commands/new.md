# /new

**Description:** Scaffold a new Python project through guided conversation.

**Argument hint:** `[project-name]`

**Allowed tools:** Bash, Read, Write

---

## Instructions

You are helping the user scaffold a new production-ready Python project using `spinup-py`.

### Step 1 — Load user defaults

Check if `~/.spinup-pyrc.toml` exists and read it (falling back to the legacy `~/.create-py-projectrc.toml` if present):

```bash
cat ~/.spinup-pyrc.toml 2>/dev/null || cat ~/.create-py-projectrc.toml 2>/dev/null || echo "(no RC file yet)"
```

Use any stored `author`, `email`, `github_handle`, `python_version` as defaults for the prompts below.

### Step 2 — Collect configuration

Ask the user for each of the following. If a default is available from the RC file, show it and ask them to confirm or change. Gather all values before proceeding (batch the non-obvious ones).

| Field                    | Type                                                                                                                       | Default               |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| `project_name`           | string (kebab-case)                                                                                                        | from argument, or ask |
| `description`            | string                                                                                                                     | —                     |
| `author`                 | string                                                                                                                     | RC default            |
| `email`                  | email                                                                                                                      | RC default            |
| `github_handle`          | string                                                                                                                     | RC default            |
| `python_version`         | one of: 3.12, 3.11, 3.10                                                                                                   | 3.12                  |
| `include_github_actions` | bool                                                                                                                       | true                  |
| `devcontainer`           | bool                                                                                                                       | true                  |
| `include_docs`           | bool (Docusaurus)                                                                                                          | false                 |
| `codecov`                | bool                                                                                                                       | false                 |
| `dockerfile`             | bool                                                                                                                       | false                 |
| `deptry`                 | bool                                                                                                                       | true                  |
| `publish_to_pypi`        | bool                                                                                                                       | false                 |
| `open_source_license`    | one of: MIT license, BSD license, ISC license, Apache Software License 2.0, GNU General Public License v3, Not open source | MIT license           |

Show a summary table of the collected config and confirm with the user before proceeding.

### Step 3 — Scaffold the project

Write the collected values to a JSON file, then run `spinup-py --config` from the directory that should contain the new project. Use JSON `true`/`false` for booleans, and leave out any field the user kept at its default.

```bash
CONFIG="$(mktemp)"
cat > "$CONFIG" << 'EOF'
{
  "project_name": "PROJECT_NAME",
  "description": "DESCRIPTION",
  "author": "AUTHOR",
  "email": "EMAIL",
  "github_handle": "GITHUB_HANDLE",
  "python_version": "PYTHON_VERSION",
  "include_github_actions": true,
  "devcontainer": true,
  "include_docs": false,
  "codecov": false,
  "dockerfile": false,
  "deptry": true,
  "publish_to_pypi": false,
  "open_source_license": "MIT license"
}
EOF
uvx spinup-py --config "$CONFIG"
rm -f "$CONFIG"
```

If `spinup-py` is already on PATH, run `spinup-py --config "$CONFIG"` instead. The file holds the user's name and email, so remove it either way.

This creates `PROJECT_NAME/` in the current directory, makes the initial git commit, and sets up Beads and Serena (each is skipped with a notice if its tool is missing). If the config is invalid, the command prints the reason and exits with code 2. Fix the value and run it again.

### Step 4 — Push to GitHub (optional)

Ask the user: "Push to GitHub? (public / private / skip)"

If public or private:

```bash
cd PROJECT_NAME
gh repo create GITHUB_HANDLE/PROJECT_NAME --VISIBILITY --source=. --remote=origin --push
```

### Step 5 — Summary

Report:

- Full path to the created project
- GitHub URL (if pushed)
- Next steps: `cd PROJECT_NAME && uv sync && uv run pre-commit install`
