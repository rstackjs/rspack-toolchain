# Rspack Toolchain

A collection of reusable GitHub Actions for building and distributing Rspack native bindings across multiple platforms.

## ⚙️ Configuration

The default values in this toolchain are determined based on [@rstackjs/rspack-binding-template](https://github.com/rstackjs/rspack-binding-template), including default package.json paths, build commands, and other configurations.

You can also customize the input configurations for each action to adapt to your own rspack custom binding repository. Please refer to the detailed documentation of each action for specific input parameters.

## 📦 Actions

### 🔽 Download Rspack Binding

**Location**: [`download-rspack-binding/action.yml`](./download-rspack-binding/action.yml)

Downloads rspack binding artifacts from GitHub Actions workflow runs. Supports downloading either specific target bindings or all available bindings.

> **Note**: This action downloads artifacts with the `bindings-*` naming pattern because the build reusable workflow uploads compiled native bindings using this format (e.g., `bindings-x86_64-apple-darwin`, `bindings-aarch64-unknown-linux-gnu`).

#### Inputs

| Name     | Description                                                                                                      | Required | Default     |
| -------- | ---------------------------------------------------------------------------------------------------------------- | -------- | ----------- |
| `target` | Specific target to download (e.g., `x86_64-apple-darwin`). If not provided, downloads all `bindings-*` artifacts | No       | -           |
| `path`   | Destination path for downloaded artifacts                                                                        | No       | `artifacts` |

#### Usage

```yaml
# Download all binding artifacts
- uses: rstackjs/rspack-toolchain/download-rspack-binding@v1
  with:
    path: artifacts

# Download specific target binding
- uses: rstackjs/rspack-toolchain/download-rspack-binding@v1
  with:
    target: x86_64-apple-darwin
    path: artifacts
```

### 📋 Get NAPI Info

**Location**: [`get-napi-info/action.yml`](./get-napi-info/action.yml)

Extracts NAPI targets from `package.json` and generates a build matrix for cross-platform native module compilation. This action automatically configures the appropriate host runners and build commands for each target platform.

> **⚠️ Important**: If your `package.json` contains `napi.targets` that are not defined in the supported targets list below, the action will fail with an error. Please submit a pull request to add support for additional compilation targets if needed.

#### Inputs

| Name                 | Description                                                                                                                                                                                                                                                                                                                                                                                            | Required | Default                       |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | ----------------------------- |
| `package-json-path`  | Path to binding package.json                                                                                                                                                                                                                                                                                                                                                                           | No       | `crates/binding/package.json` |
| `napi-build-command` | Command to call napi build. Use this option as an alias to build napi binding package. The directory of the package.json file is the working directory of the command. You can pass `--release` to build optimized release binaries ([see all options](https://napi.rs/docs/cli/build#options)). **Note**: Different package managers have different syntax for passing arguments (see examples below) | No       | `pnpm build`                  |

#### Outputs

| Name                | Description                                           |
| ------------------- | ----------------------------------------------------- |
| `matrix`            | Generated build matrix for napi targets (JSON format) |
| `binding-directory` | Path to directory containing `*.node` files           |
| `targets`           | List of napi targets (JSON array)                     |

#### Package Manager Command Syntax

The `napi-build-command` parameter works differently with various package managers because the action automatically appends `--target <target>` to your command.

**Important**: It's assumed that commands like `pnpm build`, `npm run build`, and `yarn build` are configured in your `package.json` scripts to execute `napi build`. For example:

```json
{
  "scripts": {
    "build": "napi build --platform"
  }
}
```

Here are the correct syntax patterns for each package manager:

##### pnpm

With pnpm, you can pass arguments directly to the script:

```yaml
- uses: rstackjs/rspack-toolchain/get-napi-info@v1
  with:
    napi-build-command: pnpm build
    # Will generate: pnpm build --target x86_64-apple-darwin
```

For release builds:

```yaml
napi-build-command: pnpm build --release
# Will generate: pnpm build --release --target x86_64-apple-darwin
```

##### npm

With npm, you need to use the `--` separator when your script needs additional arguments:

```yaml
- uses: rstackjs/rspack-toolchain/get-napi-info@v1
  with:
    napi-build-command: npm run build --
    # Will generate: npm run build -- --target x86_64-apple-darwin
```

For release builds:

```yaml
napi-build-command: npm run build -- --release
# Will generate: npm run build -- --release --target x86_64-apple-darwin
```

##### yarn

With yarn, similar to npm, you need the `--` separator:

```yaml
- uses: rstackjs/rspack-toolchain/get-napi-info@v1
  with:
    napi-build-command: yarn build --
    # Will generate: yarn build -- --target x86_64-apple-darwin
```

For release builds:

```yaml
napi-build-command: yarn build -- --release
# Will generate: yarn build -- --release --target x86_64-apple-darwin
```

> **⚠️ Important**: The action automatically appends `--target <target>` to your command for each target in the build matrix. Make sure your command syntax is compatible with this behavior for your chosen package manager.

#### Supported Targets

The action supports the following target platforms:

| Target                          | Host Runner      | Notes               |
| ------------------------------- | ---------------- | ------------------- |
| `x86_64-apple-darwin`           | `macos-latest`   | macOS Intel         |
| `aarch64-apple-darwin`          | `macos-latest`   | macOS Apple Silicon |
| `x86_64-pc-windows-msvc`        | `windows-latest` | Windows 64-bit      |
| `i686-pc-windows-msvc`          | `windows-latest` | Windows 32-bit      |
| `aarch64-pc-windows-msvc`       | `windows-latest` | Windows ARM64       |
| `x86_64-unknown-linux-gnu`      | `ubuntu-22.04`   | Linux x64 (GNU)     |
| `x86_64-unknown-linux-musl`     | `ubuntu-22.04`   | Linux x64 (musl)    |
| `aarch64-unknown-linux-gnu`     | `ubuntu-22.04`   | Linux ARM64 (GNU)   |
| `aarch64-unknown-linux-musl`    | `ubuntu-22.04`   | Linux ARM64 (musl)  |
| `armv7-unknown-linux-gnueabihf` | `ubuntu-22.04`   | Linux ARMv7         |
| `aarch64-linux-android`         | `ubuntu-22.04`   | Android ARM64       |
| `armv7-linux-androideabi`       | `ubuntu-22.04`   | Android ARMv7       |
| `x86_64-unknown-freebsd`        | `ubuntu-22.04`   | FreeBSD x64         |
| `aarch64-unknown-freebsd`       | `ubuntu-22.04`   | FreeBSD ARM64¹      |
| `riscv64gc-unknown-linux-gnu`   | `ubuntu-22.04`   | RISC-V 64-bit (GNU) |
| `riscv64gc-unknown-linux-musl`  | `ubuntu-22.04`   | RISC-V 64-bit (musl)|

¹ Uses `-Zbuild-std=core,std,alloc,proc_macro,panic_abort` due to [rust-lang/rust#80685](https://github.com/rust-lang/rust/issues/80685)

#### Usage

```yaml
jobs:
  get-info:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.napi.outputs.matrix }}
      targets: ${{ steps.napi.outputs.targets }}
      binding-directory: ${{ steps.napi.outputs.binding-directory }}
    steps:
      - uses: actions/checkout@v4
      - id: napi
        uses: rstackjs/rspack-toolchain/get-napi-info@v1
        with:
          package-json-path: packages/binding/package.json
          napi-build-command: pnpm build

  build:
    needs: get-info
    strategy:
      matrix: ${{ fromJSON(needs.get-info.outputs.matrix) }}
    runs-on: ${{ matrix.settings.host }}
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: ${{ matrix.settings.build }}
```

## 🔄 Reusable Workflows

### 🏗️ Build Workflow

**Location**: [`.github/workflows/build.yml`](./.github/workflows/build.yml)

A reusable workflow that automatically builds native bindings for all platforms specified in your `package.json`. This workflow:

1. Extracts NAPI targets from your `package.json`
2. Generates a build matrix for cross-platform compilation
3. Sets up the appropriate toolchain for each target platform
4. Builds native bindings in parallel across different runners
5. Uploads artifacts with the `bindings-*` naming pattern

#### Inputs

| Name                 | Description                                                                                                                                                            | Required | Default                       |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ----------------------------- |
| `package-json-path`  | Path to binding `package.json`. Same as `--package-json-path` in `@napi-rs/cli build`                                                                                  | No       | `crates/binding/package.json` |
| `napi-build-command` | Command to call napi build. Use this option as an alias to build napi binding package. The directory of the package.json file is the working directory of the command. | No       | `pnpm build`                  |

#### Usage

```yaml
jobs:
  build:
    name: Build
    uses: rstackjs/rspack-toolchain/.github/workflows/build.yml@v1
    with:
      package-json-path: crates/binding/package.json
      # pnpm example (arguments passed directly)
      napi-build-command: pnpm build --release

      # npm example (requires -- separator)
      # napi-build-command: npm run build -- --release

      # yarn example (requires -- separator)
      # napi-build-command: yarn build -- --release
```

## 🔧 Package.json Configuration

For the `get-napi-info` action to work properly, your `package.json` should include both a `napi.targets` array and a `build` script that executes `napi build`:

```json
{
  "scripts": {
    "build": "napi build --platform"
  },
  "napi": {
    "targets": [
      "x86_64-apple-darwin",
      "aarch64-apple-darwin",
      "x86_64-pc-windows-msvc",
      "x86_64-unknown-linux-gnu"
    ]
  }
}
```

The `build` script should execute the `napi build` command, which allows the action to pass additional arguments like `--target` and `--release` through your package manager.

## 🚀 Complete Workflow Example

Here's a complete workflow example showing how to use the reusable build workflow with the actions for a release process:

[release.yml](https://github.com/rstackjs/rspack-binding-template/blob/main/.github/workflows/release.yml)

## 📝 License

This project is licensed under the MIT License.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
