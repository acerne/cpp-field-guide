[← Tools](README.md)

# clang-tidy

> Static analysis tool that enforces coding guidelines, catches common bugs, and can automatically apply fixes.

[Documentation](https://clang.llvm.org/extra/clang-tidy/)

## Setup

clang-tidy requires a compilation database to know include paths and compiler flags. Generate one from CMake:

```cmake
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

This produces `build/compile_commands.json`.

### Linux

```bash
sudo apt install clang-tidy
```

#### Manual installation

Manual installation enables us to install a specific version. Latest versions might not be available in the `apt` repository.

**Step 1**: download the release from [LLVM GitHub releases](https://github.com/llvm/llvm-project/releases).
```bash
# Example: 22.1.5 Linux x68_64
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-22.1.5/LLVM-22.1.5-Linux-X64.tar.xz
```
This will download the full clang toolbox, but we will only take what we need.

**Step 2**: extract the archive
```bash
tar -xf LLVM-22.1.5-Linux-X64.tar.xz 
cd LLVM-22.1.5-Linux-X64.tar.xz 
```

**Step 3**: uninstall the old version (if installed)
```bash
sudo apt remove clang-tidy
```

**Step 4**: install the binary
```bash
sudo cp bin/clang-tidy /usr/local/bin/
```

**Step 5**: verify
```bash
clang-tidy --version
```

### Windows

Windows has no single canonical source; pick the option that fits your workflow:

- **Package manager:** `winget install LLVM.LLVM` (or `choco install llvm` / `scoop install llvm`) — adds `clang-tidy.exe` and `run-clang-tidy` to PATH.
- **LLVM installer / binaries:** Pre-built Windows installer or archive of all binaries from [LLVM GitHub releases](https://github.com/llvm/llvm-project/releases); no package manager required.
- **Visual Studio:** Ships its own copy under `VC\Tools\Llvm\x64\bin\clang-tidy.exe` but does not add it to PATH; recent versions expose it through the IDE's code analysis integration.
- **VS Code:** The [clangd extension](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd) runs clang-tidy checks inline via the bundled clangd binary — no separate install needed for in-editor diagnostics.

### Version pinning

Different sources ship different versions of clang-tidy, and versions affect which checks are available, how they behave, and what C++ syntax is understood. A check that passes on one version may fire differently or not at all on another; older versions may not parse C++20/23 constructs correctly.

Pin a specific version for the project and document it prominently. Verify the active version with:

```bash
clang-tidy --version
```

On Linux, install a specific version by name (`run-clang-tidy` is included in the same package):

```bash
sudo apt remove clang-tidy # Remove the default one first
sudo apt install clang-tidy-20
```

The default repos may not carry recent versions — the [LLVM apt repository](https://apt.llvm.org/) provides all recent releases for Debian/Ubuntu. On Windows, download a specific release installer from [LLVM GitHub releases](https://github.com/llvm/llvm-project/releases) rather than using a package manager that defaults to latest.

---

## Configuration

Place a `.clang-tidy` file at the repository root.

```yaml
Checks: >
  clang-diagnostic-*,
  clang-analyzer-*,
  modernize-*,
  readability-*,
  -modernize-use-trailing-return-type
WarningsAsErrors: "*"
HeaderFilterRegex: ".*"
```

Common check categories: `clang-analyzer-*`, `modernize-*`, `readability-*`, `performance-*`, `bugprone-*`, `cppcoreguidelines-*`. Prefix any check with `-` to disable it.

### Subdirectory overrides

clang-tidy walks up from a file's directory to find the nearest `.clang-tidy`. A closer file takes precedence over the root config. To disable all checks for a subdirectory (e.g. `third_party/`):

```yaml
# third_party/.clang-tidy
Checks: '-*'
```

Place a full config there instead to apply different check sets to vendored or generated code.

### Inline suppression

Suppress on the same line, the next line, or a region:

```cpp
int x = 42;  // NOLINT(readability-magic-numbers)

// NOLINTNEXTLINE(modernize-use-nullptr)
int* p = NULL;

// NOLINTBEGIN(readability-magic-numbers)
int a = 1, b = 2, c = 3;
// NOLINTEND(readability-magic-numbers)
```

`NOLINT` and `NOLINTNEXTLINE` without a check name suppress all checks on that line — avoid this, as it silently masks new warnings. `NOLINTBEGIN`/`NOLINTEND` requires clang-tidy 14 or later.

clang-tidy can produce false positives, especially from `clang-analyzer-*` and `cppcoreguidelines-*`. Prefer fixing the underlying issue or disabling a specific over-eager check in `.clang-tidy` over scattering `NOLINT` comments.

---

## Usage

Run on a single file using the compilation database in `build/`:

```bash
clang-tidy -p build/ file.cpp
```

Apply auto-fixes:

```bash
clang-tidy -p build/ --fix file.cpp
```

Run only a specific check, ignoring `.clang-tidy`:

```bash
clang-tidy -p build/ -checks="-*,modernize-use-override" file.cpp
```

Run on all files in the project using `run-clang-tidy` (ships with LLVM, parallelizes automatically):

```bash
run-clang-tidy -p build/
```

Apply fixes to all files:

```bash
run-clang-tidy -p build/ -fix
```

Run only a specific check across all files:

```bash
run-clang-tidy -p build/ -checks="-*,bugprone-*"
```

---

## Integration

clang-tidy integrates with editors via the clangd language server and can run as a CI step or part of the CMake build.

**Editor:** The `clangd` language server reads `.clang-tidy` and surfaces diagnostics inline. VS Code via the `clangd` extension picks this up automatically; CLion supports it natively.

**CI:**

```bash
run-clang-tidy -p build/ -warnings-as-errors="*"
```

**CMake integration** — runs clang-tidy as part of the build:

```cmake
find_program(CLANG_TIDY clang-tidy)
if(CLANG_TIDY)
    set(CMAKE_CXX_CLANG_TIDY "${CLANG_TIDY}")
endif()
```

This adds overhead to every compilation. Prefer `run-clang-tidy` as a separate CI step for large projects.
