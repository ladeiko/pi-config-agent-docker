# Pi Docker Wrapper

A shell script wrapper for running [pi-coding-agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) in a Docker container with automatic setup and configuration.

---

## Overview

This script provides a convenient way to run the pi coding agent in an isolated Docker environment. It automatically:

- 🐳 Builds a Docker image with pi-coding-agent installed
- 📁 Creates unique containers per directory
- 🔑 Manages API key retrieval via configurable executables
- 💾 Persists configuration and history across sessions
- 🗂️ Mounts your current working directory for file access
- 🔒 Supports ignore files to prevent sensitive files from being mounted

---

## Prerequisites

- **Docker**: Ensure Docker is installed and running
- **API Key**: At least one API key for a supported provider:
  - Anthropic (Claude)
  - OpenAI (GPT)
  - Google Gemini
  - Groq
  - xAI (Grok)
  - Hugging Face

---

## Installation

### 1. Download the Script

Clone this repository or download the `pi` script:

```bash
git clone <repository-url>
cd <repository-directory>
```

### 2. Make Executable

```bash
chmod +x pi
```

### 3. Add to PATH (Optional)

**Option A: Copy to /usr/local/bin**
```bash
sudo cp pi /usr/local/bin/pi
```

**Option B: Create a symlink**
```bash
sudo ln -s $(pwd)/pi /usr/local/bin/pi
```

---

## Usage

### Basic Usage

Run pi in the current directory:

```bash
./pi
```

Or if added to PATH:

```bash
pi
```

### Rebuild Docker Image

Force rebuild of the Docker image (useful after updates):

```bash
./pi rebuild
```

> **Note:** This stops and removes **all** pi containers and deletes the image so it is rebuilt from scratch.

---

## How It Works

### Container Naming

- Containers are named using an MD5 hash of the current directory path
- Each directory gets its own container instance
- Pattern: `pi-<md5-hash-of-pwd>`

### Docker Image

- **Base Image**: Node.js 22.17.0 Alpine
- **Package**: `@mariozechner/pi-coding-agent` (installed globally)
- **Image Name**: `pi:latest`

### Volume Mounts

| Mount Point | Source | Purpose |
|-------------|--------|---------|
| `/work` | Current directory | Working directory for pi |
| `/root/.pi` | `~/.pi-docker/.pi` | Preserves history, settings, and state |

---

## API Key Management

The script looks for **executable files** in `~/.pi-docker/` for each supported provider. These files are executed, and their output is passed as environment variables to the container.

### Supported Providers

| Executable File Name | Environment Variable | Provider |
|---------------------|---------------------|----------|
| `ANTHROPIC_API_KEY` | `ANTHROPIC_API_KEY` | Anthropic (Claude) |
| `OPENAI_API_KEY` | `OPENAI_API_KEY` | OpenAI (GPT) |
| `GEMINI_API_KEY` | `GEMINI_API_KEY` | Google Gemini |
| `GROQ_API_KEY` | `GROQ_API_KEY` | Groq |
| `XAI_API_KEY` | `XAI_API_KEY` | xAI (Grok) |
| `HF_TOKEN` | `HF_TOKEN` | Hugging Face |

### Setup Examples

#### Using `pass` (Password Store)

```bash
# Anthropic
cat > ~/.pi-docker/ANTHROPIC_API_KEY << 'EOF'
#!/bin/sh
pass anthropic/my-key
EOF
chmod +x ~/.pi-docker/ANTHROPIC_API_KEY

# OpenAI
cat > ~/.pi-docker/OPENAI_API_KEY << 'EOF'
#!/bin/sh
pass openai/my-key
EOF
chmod +x ~/.pi-docker/OPENAI_API_KEY
```

#### Using Plain Text (Less Secure)

```bash
# Anthropic
cat > ~/.pi-docker/ANTHROPIC_API_KEY << 'EOF'
#!/bin/sh
echo "sk-ant-..."
EOF
chmod +x ~/.pi-docker/ANTHROPIC_API_KEY

# OpenAI
cat > ~/.pi-docker/OPENAI_API_KEY << 'EOF'
#!/bin/sh
echo "sk-..."
EOF
chmod +x ~/.pi-docker/OPENAI_API_KEY
```

#### Using macOS Keychain

```bash
# Anthropic
cat > ~/.pi-docker/ANTHROPIC_API_KEY << 'EOF'
#!/bin/sh
security find-generic-password -s anthropic-api-key -w
EOF
chmod +x ~/.pi-docker/ANTHROPIC_API_KEY

# OpenAI
cat > ~/.pi-docker/OPENAI_API_KEY << 'EOF'
#!/bin/sh
security find-generic-password -s openai-api-key -w
EOF
chmod +x ~/.pi-docker/OPENAI_API_KEY
```

---

## File Ignoring

The wrapper prevents sensitive files and directories from being visible inside the container by bind-mounting `/dev/null` over files and an empty directory over directories.

### Default Ignored Items

The following are automatically ignored if they exist in the working directory:

**Files & Directories:**
- `.env`
- `.deploy`
- `.secret`
- `.git`
- `.build`
- `.vscode`
- `.gitmodules`
- `.pi`
- `.DS_Store`

**Ignore Config Files:**
- `.aiignore`
- `.aiexclude`
- `.copilotignore`
- `.gitignore`
- `.geminiignore`
- `.codeiumignore`
- `.cursorignore`

### Custom Ignore Patterns

You can specify additional file patterns to ignore using any of the config files listed above.

**Config File Locations:**

1. **Local**: In the current working directory (applies to current project only)
2. **Global**: In `~/.pi-docker/` (applies to all projects)

**Format:**
- One glob pattern per line
- Lines starting with `#` are comments
- Blank lines are ignored

**Example `.aiignore`:**

```gitignore
# Ignore secret configs
*.secret
credentials.json

# Ignore build artifacts
dist/
build/
*.log
```

### Project Tree Display

If `tree` is installed on the host, the script displays a filtered project tree before launching the container, showing only files that will be visible inside the container.

---

## Directory Structure

```
~/.pi-docker/
├── ANTHROPIC_API_KEY          # (Optional) Executable that outputs Anthropic API key
├── OPENAI_API_KEY             # (Optional) Executable that outputs OpenAI API key
├── GEMINI_API_KEY             # (Optional) Executable that outputs Google Gemini API key
├── GROQ_API_KEY               # (Optional) Executable that outputs Groq API key
├── XAI_API_KEY                # (Optional) Executable that outputs xAI API key
├── HF_TOKEN                   # (Optional) Executable that outputs Hugging Face token
├── .aiignore                  # (Optional) Global ignore patterns
├── .aiexclude                 # (Optional) Global ignore patterns
├── .copilotignore             # (Optional) Global ignore patterns
├── .gitignore                 # (Optional) Global ignore patterns
├── .geminiignore              # (Optional) Global ignore patterns
├── .codeiumignore             # (Optional) Global ignore patterns
├── .cursorignore              # (Optional) Global ignore patterns
└── .pi/                       # Shared pi configuration
    ├── history                # Command history
    └── config                 # User settings
```

---

## Troubleshooting

### Container Already Running

The script automatically stops and removes existing containers with the same name before starting a new one. No manual intervention needed.

### Image Not Building

**Check:**
- Docker is running
- You have internet connectivity to pull the base Node.js image
- You have sufficient disk space

**Solution:**
```bash
./pi rebuild
```

### API Key Issues

**Verify your API key executable works:**

```bash
sh -c ~/.pi-docker/ANTHROPIC_API_KEY
sh -c ~/.pi-docker/OPENAI_API_KEY
# ... etc for other providers
```

**Expected output:** Your API key should be printed to stdout.

### Permission Denied

**Make sure the script and API key files are executable:**

```bash
chmod +x pi
chmod +x ~/.pi-docker/ANTHROPIC_API_KEY
chmod +x ~/.pi-docker/OPENAI_API_KEY
# ... etc for other providers
```

### Files Not Visible in Container

Check if the files are being ignored. The script prints which files and directories are being ignored during startup.

---

## Important Notes

- ⚠️ Containers are ephemeral (removed after exit with `--rm` flag)
- 📂 All directories share the same pi configuration (`~/.pi-docker/.pi`)
- 🧹 The `rebuild` command cleans up **all** pi containers, not just the current directory's
- 🌳 If `tree` is available on the host, a filtered project tree is displayed before launch
- 🔒 Ignored files are completely hidden from the container (not just excluded from processing)

---

## License

This wrapper script is available under the MIT License — see [LICENSE](LICENSE).

For pi-coding-agent licensing, see the [official repository](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent).

---

## Related Links

- 🤖 [Pi Coding Agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent)
- 🐳 [Docker](https://www.docker.com/)
- 🔐 [Pass - Password Store](https://www.passwordstore.org/)

---

## Authors

- **Siarhei Ladzeika** - [sergey.ladeiko@gmail.com](mailto:sergey.ladeiko@gmail.com)
