# RFC 0001: Reorganize Repository Folder Structure

**Status:** Draft

## Summary

**English**
Reorganize the repository’s directory layout to comply with the *projectify* skill conventions. The change will:

- Separate executables, libraries, examples, tests, and vendor packages.
- Preserve full compatibility with all existing Zig, CMake, and Nix build pipelines.
- Improve discoverability and maintenance for future contributors.

**中文**
按照 *projectify* 技能规范重组仓库目录结构，提升模块划分、可维护性，且保持现有 Zig、CMake 与 Nix 构建系统的完整兼容。

## Problem

The current tree mixes executables, source code, examples, tests, vendored libraries, and header files into a flat structure. This makes:

- **Discoverability** – New contributors cannot easily locate binaries or library sources.
- **Maintainability** – Files are scattered, raising the risk of accidental removal or duplication.
- **Build Fragility** – Hard‑coded paths in build scripts and CI pipelines become fragile under such a layout.

## Goals

1. Place all executable products under `apps/<delivery‑form>/` (e.g., `apps/desktop/`).
2. Store reusable libraries in `libs/<language>/` (e.g., `libs/ghostty/`, `libs/c/`).
3. Retain existing build commands (`zig build`, `cmake`, `nix build`) with no changes to logic.
4. Keep all functional binaries, tests, and examples operational after the move.
5. Follow the projectify layout preferences without forcing a single workspace format.

## Non‑goals
- Redesigning or rewriting build system logic.
- Adding new programming languages or dependencies.
- Forcing a uniform workspace style across all languages.
- Modifying any existing APIs or source content.

## Design

### Current Structure Analysis
```
macos/   – macOS app (executable product)
src/     – Shared Zig core (reusable library)
example/ – Demo / usage example
test/    – Test suites
pkg/     – Vendored external packages
include/ – Public C/C++ headers
```

### Proposed Directory Layout
```
/apps/                # Executable products
├── desktop/          # macOS app (src: macos/)
└── cli/              # Future CLI tools (placeholder)
/libs/                # Reusable libraries
├── ghostty/          # Zig core (src: src/)
└── c/                # C/C++ headers (src: include/)
/examples/             # Usage examples (src: example/)
/tests/                # Test suites (src: test/)
/packages/             # Vendored packages (src: pkg/)
/docs/                # Documentation
/build/                # Build artifacts & helpers
/config/               # Build/development configuration
```

### Detailed Implementation Plan
| Step | Action | Script / Command | Expected Result |
|------|--------|------------------|-----------------|
| 1 | Create directory skeleton | ```bash
mkdir -p apps/desktop apps/cli libs/ghostty libs/c examples tests packages docs build config
``` | Empty directories created |
| 2 | Move existing artefacts | ```bash
mv macos/* apps/desktop/
mv src/* libs/ghostty/
mv example/* examples/
mv test/* tests/
mv pkg/* packages/
mv include/* libs/c/
``` | Files relocated; old dirs empty |
| 3 | Update build scripts & CI | * `build.zig` → replace `src/` and `include/` with `libs/ghostty/` & `libs/c/`.
* `CMakeLists.txt` → `add_executable` / `target_include_directories` use `apps/desktop/` & `libs/ghostty/`.
* `flake.nix` → `src = ./libs/ghostty;  include = ./libs/c;`.
* CI YAMLs → paths updated to new layout. | Scripts refer to new paths, compile succeeds |
| 4 | Local verification | ```bash
# Zig build
zig build
# CMake build
mkdir -p build && cd build && cmake .. && make
# Nix build
nix build .#default
# Run Zig tests
zig test
# Verify macOS app
apps/desktop/Matrix0AI.app
``` | No build or test failures, binary runs |
| 5 | Commit & push | ```bash
git add .
git commit -m "Implement RFC 0001: Reorganize repo structure"
git push
``` | Changes reflected on `main`; no branches used |
| 6 | Verify CI | Trigger run on `main`. All jobs must pass. | CI status green |
| 7 | Archive RFC | ```bash
node scripts/specify.mjs archive 0001
``` | RFC moved to `.spec/rfc/pending/` or `.spec/rfc/completed/` |

## Acceptance Criteria
1. Executables live under `apps/<delivery‑form>/` (e.g., `apps/desktop`).
2. Reusable code is in `libs/<language>/` folders.
3. All build scripts reference the new paths and compile without errors.
4. All tests pass after the move.
5. Runtime behaviour unchanged.
6. CI pipeline runs to completion with the new layout.
7. Documentation paths reflect the new structure.

## Implementation Tasks (Detailed)
### 1️⃣ Directory Creation
```bash
mkdir -p apps/desktop apps/cli
mkdir -p libs/ghostty libs/c
mkdir -p examples tests packages docs build config
```

### 2️⃣ Moving Source Files
```bash
# macOS app
mv macos/* apps/desktop/
# Zig core library
mv src/* libs/ghostty/
# Demo / example usage
mv example/* examples/
# Test suites
mv test/* tests/
# Vendored packages
mv pkg/* packages/
# C/C++ headers
mv include/* libs/c/
```

### 3️⃣ Updating Build Scripts
- **build.zig** – Find any line importing from `src/` or `include/` and replace with `libs/ghostty/` or `libs/c/`. Example:
  ```zig
  const GhostTy = @import("libs/ghostty");
  ```
- **CMakeLists.txt** – Adjust `add_executable` to point to `apps/desktop/Matrix0AI.cpp` (or equivalent) and set include directories:
  ```cmake
  add_executable(Matrix0AI apps/desktop/Matrix0AI.cpp)
  target_include_directories(Matrix0AI PRIVATE libs/ghostty libs/c)
  ```
- **flake.nix** – Update the `src` and `include` attributes:
  ```nix
  src = ./libs/ghostty;
  include = ./libs/c;
  ```
- **CI YAMLs** – Search `${{ github.workspace }}` paths that still refer to `macos/`, `src/`, `example/`, etc., and replace with the new layout.

### 4️⃣ Local Verification
```bash
# Zig build
zig build
# CMake pipeline
mkdir -p build && cd build && cmake .. && make
# Nix build
nix build .#default
# Run Zig tests
zig test
# Manual macOS app test
open apps/desktop/Matrix0AI.app
```

### 5️⃣ Commit to Main
```bash
git add .
git commit -m "Implement RFC 0001: Reorganize repository structure"
git push
```

### 6️⃣ Verify CI
Push triggers the CI workflow on `main`. Confirm all jobs finish successfully.

### 7️⃣ Archive RFC
```bash
node scripts/specify.mjs archive 0001
```

## Contingency & Rollback
- **Backup** before moving: `tar -zcf backup-0001-$(date +%Y%m%d).tar.gz .spec/rfc/0001-reorg-repo-folder.md`.
- **Rollback** if a build fails: `git reset --hard HEAD~1` followed by re‑examination.
- **Re‑attempt**: Re‑run steps from 1 whenever changes are needed.
- **Documentation**: Update `CHANGELOG.md` and README to reflect new paths.

## Note
All actions are performed directly on the `main` branch; no worktree, branch, or separate environment is used unless explicitly requested. The RFC now contains the complete step‑by‑step plan and all required details.

---

*Reference: projectify skill, RFC 0001 – Reorganize Repository Folder Structure*
