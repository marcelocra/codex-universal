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

You can use `codex-universal` to power a VS Code devcontainer in your project. Here's a minimal setup:

1. Create a `.devcontainer` directory in your project:

```bash
mkdir -p .devcontainer
cd .devcontainer
```

2. Clone this repository (shallow clone to save space):

```bash
git clone https://github.com/openai/codex-universal --depth 1
```

3. Create a `devcontainer.json` file:

```bash
cat <<'EOF' > devcontainer.json
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
EOF
```

Alternatively, you can reference the pre-built image directly without cloning:

```json
{
  "name": "Codex Universal Dev Container",
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

This approach is faster and doesn't require cloning the repository, but requires internet access to pull the image.

## What's included

In addition to the packages specified in the table above, the following packages are also installed:

- `bun`: 1.2.10
- `bazelisk` / `bazel`
- `erlang`: 27.1.2
- `elixir`: 1.18.3

See [Dockerfile](Dockerfile) for the full details of installed packages.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
