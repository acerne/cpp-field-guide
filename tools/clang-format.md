[← C++ Field Guide](../README.md)

# clang-format

> Automatically formats C++ source files to a consistent style defined by a `.clang-format` configuration file.

[Documentation](https://clang.llvm.org/docs/ClangFormat.html)

## Setup

No CMake integration required — clang-format operates directly on source files.

### Linux

```bash
sudo apt install clang-format
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
sudo apt remove clang-format
```

**Step 4**: install the binary
```bash
sudo cp bin/clang-format /usr/local/bin/
```

**Step 5**: verify
```bash
clang-format --version
```


### Windows

Windows has no single canonical source; pick the option that fits your workflow:

- **Package manager:** `winget install LLVM.LLVM` (or `choco install llvm` / `scoop install llvm`) — adds `clang-format.exe` to PATH.
- **LLVM installer / binaries:** Pre-built Windows installer or archive of all binaries from [LLVM GitHub releases](https://github.com/llvm/llvm-project/releases); no package manager required.
- **Visual Studio:** Ships its own copy under `VC\Tools\Llvm\x64\bin\clang-format.exe` but does not add it to PATH; accessible only through the IDE.
- **VS Code:** The [C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) bundles clang-format and can format on save without a system install.

### Version pinning

Different sources ship different versions, which can produce different output for the same code and config, leading to spurious diffs and CI rejections. Older versions may also lack the support for newer C++ syntax (concepts, requires clauses) or config options added in later releases.

Pin a specific version for the project and document it prominently. All users on the project should be using the same version. Verify the active version with:

```bash
clang-format --version
```

On Linux, install a specific version by name:

```bash
sudo apt remove clang-format # Remove the default one first
sudo apt install clang-format-20
```

The default repos may not carry recent versions — the [LLVM apt repository](https://apt.llvm.org/) provides all recent releases for Debian/Ubuntu. On Windows, download a specific release installer from [LLVM GitHub releases](https://github.com/llvm/llvm-project/releases) rather than using a package manager that defaults to latest.

---

## Configuration

Place a `.clang-format` file at the repository root; all subdirectories inherit it.

```yaml
BasedOnStyle: Google
IndentWidth: 4
ColumnLimit: 100
AllowShortFunctionsOnASingleLine: None
BreakBeforeBraces: Attach
```

`BasedOnStyle` accepts: `LLVM`, `Google`, `Chromium`, `Mozilla`, `WebKit`, `Microsoft`. Start from the closest match and override individual fields.

Dump a full config from a built-in style as a starting point:

```bash
clang-format --style=LLVM --dump-config > .clang-format
```

### Subdirectory overrides

clang-format walks up from a file's directory to find the nearest `.clang-format`. A closer file takes precedence over the root config. To disable formatting entirely for a subdirectory (e.g. `third_party/`):

```yaml
# third_party/.clang-format
DisableFormat: true
```

Place a full config there instead to apply different style rules to vendored or generated code.

### `.clang-format-ignore`

Available since clang-format 18. A `.clang-format-ignore` file excludes files by glob pattern — one pattern per line, `#` for comments:

```
# vendored code
third_party/**
generated/**/*.pb.h
```

Prefer this over per-directory `DisableFormat: true` when exclusions span multiple directories or don't align with the directory structure.

### Inline suppression

Disable formatting for a region:

```cpp
// clang-format off
Matrix3x3 m = {
    1, 0, 0,
    0, 1, 0,
    0, 0, 1,
};
// clang-format on
```

---

## Usage

Check a file without modifying it — exits non-zero if any change would be made:

```bash
clang-format --dry-run --Werror file.cpp
```

Format a file in place:

```bash
clang-format -i file.cpp
```

Format all C++ files in the tree (bash one-liner):

```bash
find . \( -name "*.cpp" -o -name "*.h" -o -name "*.hpp" \) | xargs clang-format -i
```

Format all C++ files (Python script — easier to extend or exclude paths):

```python
#!/usr/bin/env python3
import subprocess, sys
from pathlib import Path

extensions = {'.cpp', '.h', '.hpp', '.cc', '.cxx'}
root = Path(sys.argv[1]) if len(sys.argv) > 1 else Path('.')

files = [f for f in root.rglob('*') if f.suffix in extensions]
for f in files:
    subprocess.run(['clang-format', '-i', str(f)], check=True)
print(f'Formatted {len(files)} files.')
```

---

## Integration

clang-format integrates with editors, CI pipelines, and CMake targets.

**Editor:** VS Code (via the `clang-format` extension) and CLion natively auto-detect `.clang-format` and can format on save.

**CI** — fail if any file is not formatted:

```bash
find . \( -name "*.cpp" -o -name "*.h" \) | xargs clang-format --dry-run --Werror
```

**CMake custom target:**

```cmake
find_program(CLANG_FORMAT clang-format)
if(CLANG_FORMAT)
    file(GLOB_RECURSE FORMAT_SOURCES src/*.cpp src/*.h)
    add_custom_target(format
        COMMAND ${CLANG_FORMAT} -i ${FORMAT_SOURCES}
        COMMENT "Running clang-format"
    )
endif()
```
