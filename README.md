# codex-universal

`codex-universal` is a reference implementation of the base Docker image available in [OpenAI Codex](http://platform.openai.com/docs/codex).

This repository is intended to help developers cutomize environments in Codex, by providing a similar image that can be pulled and run locally. This is not an identical environment but should help for debugging and development.

For more details on environment setup, see [OpenAI Codex](http://platform.openai.com/docs/codex).

## Usage

The Docker image is available at:

```
docker pull ghcr.io/openai/codex-universal:latest
```

This repository builds the image for both linux/amd64 and linux/arm64. However we only run the linux/amd64 version.
Your installed Docker may support linux/amd64 emulation by passing the `--platform linux/amd64` flag.

The arm64 image differs from the amd64 image in 2 ways:
- OpenJDK 10 is not available on amd64
- The arm64 image skips installing swift because of a current bug with mise

The below script shows how can you approximate the `setup` environment in Codex:

```sh
# See below for environment variable options.
# This script mounts the current directory similar to how it would get cloned in.
docker run --rm -it \
    -e CODEX_ENV_PYTHON_VERSION=3.12 \
    -e CODEX_ENV_NODE_VERSION=20 \
    -e CODEX_ENV_RUST_VERSION=1.87.0 \
    -e CODEX_ENV_GO_VERSION=1.23.8 \
    -e CODEX_ENV_SWIFT_VERSION=6.2 \
    -e CODEX_ENV_RUBY_VERSION=3.4.4 \
    -e CODEX_ENV_PHP_VERSION=8.4 \
    -v $(pwd):/workspace/$(basename $(pwd)) -w /workspace/$(basename $(pwd)) \
    ghcr.io/openai/codex-universal:latest
```

`codex-universal` includes setup scripts that look for `CODEX_ENV_*` environment variables and configures the language version accordingly.

### Configuring language runtimes

The following environment variables can be set to configure runtime installation. Note that a limited subset of versions are supported (indicated in the table below):

| Environment variable       | Description                | Supported versions                               | Additional packages                                                  |
| -------------------------- | -------------------------- | ------------------------------------------------ | -------------------------------------------------------------------- |
| `CODEX_ENV_PYTHON_VERSION` | Python version to install  | `3.10`, `3.11.12`, `3.12`, `3.13`, `3.14.0`        | `pyenv`, `poetry`, `uv`, `ruff`, `black`, `mypy`, `pyright`, `isort` |
| `CODEX_ENV_NODE_VERSION`   | Node.js version to install | `18`, `20`, `22`                                 | `corepack`, `yarn`, `pnpm`, `npm`                                    |
| `CODEX_ENV_RUST_VERSION`   | Rust version to install    | `1.83.0`, `1.84.1`, `1.85.1`, `1.86.0`, `1.87.0`, `1.88.0`, `1.89.0`, `1.90`, `1.91.1`, `1.92.0` |                                                                      |
| `CODEX_ENV_GO_VERSION`     | Go version to install      | `1.22.12`, `1.23.8`, `1.24.3`, `1.25.1`           |                                                                      |
| `CODEX_ENV_SWIFT_VERSION`  | Swift version to install   | `5.10`, `6.1`, `6.2`                              |                                                                      |
| `CODEX_ENV_RUBY_VERSION`   | Ruby version to install  | `3.2.3`, `3.3.8`, `3.4.4`                |                                                                      |
| `CODEX_ENV_PHP_VERSION`   | PHP version to install  | `8.4`, `8.3`, `8.2`                |                                                                      |
| `CODEX_ENV_JAVA_VERSION`   | JDK version to install  | `25`, `24`, `23`, `22`, `21`, `17`, `11`                |                                                                      |

### Disabling languages

To use only the languages you care about, simply **omit or leave empty** the environment variables for languages you don't need. The setup script will skip configuration for any language whose `CODEX_ENV_*` variable is not set or is empty.

For example, to enable only Python and Node.js:

```sh
docker run --rm -it \
    -e CODEX_ENV_PYTHON_VERSION=3.12 \
    -e CODEX_ENV_NODE_VERSION=20 \
    -v $(pwd):/workspace/$(basename $(pwd)) -w /workspace/$(basename $(pwd)) \
    ghcr.io/openai/codex-universal:latest
```

All other languages (Rust, Go, Swift, Ruby, PHP, Java) will remain at their default versions but won't be switched during container startup, saving time and resources.

### Using as a devcontainer

You can use `codex-universal` as a VS Code devcontainer for your project. There are two main approaches:

#### Option 1: Using the pre-built image (Recommended)

This is the fastest and simplest approach. The image is automatically pulled from GitHub Container Registry.

**Step 1:** Create the `.devcontainer` directory in your project root:

```bash
mkdir -p .devcontainer
```

**Step 2:** Create `.devcontainer/devcontainer.json` with your configuration:

```json
{
  "name": "My Dev Container",
  "image": "ghcr.io/openai/codex-universal:latest",
  "containerEnv": {
    "CODEX_ENV_PYTHON_VERSION": "3.12",
    "CODEX_ENV_NODE_VERSION": "20"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "dbaeumer.vscode-eslint"
      ]
    }
  },
  "postCreateCommand": "echo 'Dev container ready!'"
}
```

**Step 3:** Open the project in VS Code and:
- Press `F1` or `Ctrl+Shift+P` (Windows/Linux) / `Cmd+Shift+P` (Mac)
- Type "Dev Containers: Reopen in Container"
- Press Enter

The container will start with Python 3.12 and Node.js 20 configured. All other languages remain at default versions.

#### Option 2: Building from source

Use this approach if you need to customize the Dockerfile or test local changes.

**Step 1:** Create the `.devcontainer` directory:

```bash
mkdir -p .devcontainer
cd .devcontainer
```

**Step 2:** Clone the codex-universal repository (shallow clone to save space):

```bash
git clone https://github.com/openai/codex-universal --depth 1
```

**Step 3:** Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "Custom Dev Container",
  "build": {
    "dockerfile": "./codex-universal/Dockerfile"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "dbaeumer.vscode-eslint"
      ]
    }
  },
  "postCreateCommand": "echo 'Dev container ready!'"
}
```

**Step 4:** Open the project in VS Code and reopen in container (same as Option 1, Step 3).

The Dockerfile will be built locally. This takes longer but allows for customization.

#### Configuring specific languages only

You can enable only the languages you need by setting the corresponding `CODEX_ENV_*` environment variables. Languages without a configured environment variable will remain at their default versions but won't be reconfigured during container startup.

**Example 1: Python-only development**

Perfect for Python projects, data science, or machine learning work.

**Step 1:** Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "Python Dev Container",
  "image": "ghcr.io/openai/codex-universal:latest",
  "containerEnv": {
    "CODEX_ENV_PYTHON_VERSION": "3.12"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-python.black-formatter"
      ]
    }
  },
  "postCreateCommand": "python --version && pip --version"
}
```

**Step 2:** Open in VS Code and reopen in container.

**Step 3:** Verify Python is configured:

```bash
python --version  # Should show Python 3.12.x
poetry --version  # Poetry is pre-installed
uv --version      # uv is pre-installed
```

**Example 2: Rust and Go development**

Ideal for systems programming or cloud-native development.

**Step 1:** Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "Rust and Go Dev Container",
  "image": "ghcr.io/openai/codex-universal:latest",
  "containerEnv": {
    "CODEX_ENV_RUST_VERSION": "1.87.0",
    "CODEX_ENV_GO_VERSION": "1.23.8"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "rust-lang.rust-analyzer",
        "golang.go"
      ]
    }
  },
  "postCreateCommand": "rustc --version && go version"
}
```

**Step 2:** Open in VS Code and reopen in container.

**Step 3:** Verify both languages are configured:

```bash
rustc --version  # Should show rustc 1.87.0
cargo --version  # Cargo is included with Rust
go version       # Should show go version go1.23.8
```

**Example 3: Full-stack web development (Node.js + Python + PHP)**

For web applications with multiple backend languages.

**Step 1:** Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "Full-Stack Web Dev Container",
  "image": "ghcr.io/openai/codex-universal:latest",
  "containerEnv": {
    "CODEX_ENV_NODE_VERSION": "20",
    "CODEX_ENV_PYTHON_VERSION": "3.12",
    "CODEX_ENV_PHP_VERSION": "8.4"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "ms-python.python",
        "bmewburn.vscode-intelephense-client"
      ]
    }
  },
  "postCreateCommand": "node --version && python --version && php --version"
}
```

**Step 2:** Open in VS Code and reopen in container.

**Step 3:** Verify all languages:

```bash
node --version    # Should show v20.x.x
npm --version     # npm, yarn, pnpm are pre-installed
python --version  # Should show Python 3.12.x
php --version     # Should show PHP 8.4.x
```

## What's included

In addition to the packages specified in the table above, the following packages are also installed:

- `bun`: 1.2.10
- `bazelisk` / `bazel`
- `erlang`: 27.1.2
- `elixir`: 1.18.3

See [Dockerfile](Dockerfile) for the full details of installed packages.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
