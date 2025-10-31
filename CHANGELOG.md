# Changelog

All notable changes to the rholang-nvim plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0] - 2025-10-31

### Added
- Standard LSP keybindings following Neovim conventions
  - `K` for hover documentation
  - `<leader>rn` for rename (replaces `e`)
  - `<leader>ds` for document symbols
  - `<leader>ws` for workspace symbols
  - `<C-k>` for signature help in insert and normal modes
  - Commented bindings for future features (code actions, formatting)
- Unified `validator_backend` configuration option
- Installation instructions for pckr.nvim plugin manager
- Comprehensive LSP keybindings documentation section in README
- Migration guide from v0.3.0 to v0.4.0
- Performance information (4-8x faster with rholang-language-server v0.1.0)
- CHANGELOG.md file

### Changed
- **BREAKING**: Replaced `use_rnode`, `grpc_host`, `grpc_port` config options with unified `validator_backend`
  - Old: `use_rnode = true, grpc_host = 'localhost', grpc_port = 40402`
  - New: `validator_backend = 'grpc:localhost:40402'`
- **BREAKING**: Changed rename keybinding from `e` to `<leader>rn` (standard LSP convention)
- **BREAKING**: Removed custom `*`, `n`, `N` keybindings for reference navigation
- Updated LSP command generation to use `--validator-backend` flag
- Updated README with complete installation instructions for all major plugin managers
- Enhanced features list to highlight rholang-language-server v0.1.0 compatibility
- Updated health check documentation to reflect validator backend configuration

### Removed
- Custom reference highlighting and navigation implementation
- Non-standard `e` keybinding for rename
- Legacy configuration options: `use_rnode`, `grpc_host`, `grpc_port`

### Fixed
- Improved consistency with standard Neovim LSP workflows

## [0.3.0] - 2025-06-10

### Added
- Tree-sitter query files for comprehensive language support
  - `highlights.scm` - Comprehensive syntax highlighting rules (174 lines)
  - `folds.scm` - Code folding rules for contracts, blocks, and other structures (56 lines)
  - `indents.scm` - Smart indentation rules (60 lines)
  - `locals.scm` - Scope and variable tracking (68 lines)
  - `textobjects.scm` - Text object definitions for code navigation (91 lines)
- Health check system (`:checkhealth rholang`)
  - Verifies Neovim version, dependencies, Tree-sitter setup
  - Reports LSP status and RNode mode configuration
  - Validates filetype detection
- Smart delimiter handling
  - Auto-closing for `{`, `(`, `[`, `"` with matching pairs
  - Smart deletion with `<BS>`, `<DEL>`, `x`, `X` for empty pairs
  - Intelligent quote handling inside string literals
- Enhanced indentation behavior
  - Custom `<CR>`, `o`, `O` keymaps for smart indentation
  - Context-aware indentation within delimited nodes
- LSP features
  - Custom reference highlighting with `*` keybinding
  - Reference navigation with `n`/`N` keybindings
  - Location list integration for cross-file references
  - Standard LSP keybindings: `gd`, `gD`, `gr`, `e`
- RNode integration options
  - Embedded Rust parser/interpreter mode (default)
  - Legacy RNode via gRPC mode
  - Configuration flags for gRPC host and port

### Changed
- Migrated to Tree-sitter-based parser from legacy Vim syntax
- Enhanced filetype detection and configuration
- Improved delimiter auto-pairing logic
- Updated LSP client initialization with PID passing

### Fixed
- Tree-sitter parser compatibility with Neovim v0.9.0+
- Indentation edge cases in nested structures
- Empty pair deletion behavior

## [0.2.0] - 2025-04-15

### Added
- LSP integration with rholang-language-server
  - Basic autocompletion support
  - Diagnostics from language server
  - Go-to-definition functionality
  - Basic LSP command generation
- Tree-sitter foundation
  - `grammar.js` - Tree-sitter grammar definition
  - Basic parsing infrastructure
  - Cross-platform build system (Linux, macOS, Windows)
- Build automation
  - `Makefile` for compilation
  - `build.sh` cross-platform build script
  - `CMakeLists.txt` for CMake builds
- Plugin manager support
  - lazy.nvim configuration examples
  - packer.nvim integration
  - vim-plug support
- LSP root directory detection
  - Searches for `.git` and `rholang.toml`
  - Fallback to single-file mode

### Changed
- Improved filetype detection for `.rho` files
- Enhanced configuration structure
- Updated installation documentation

## [0.1.0] - 2025-02-01

### Added
- Initial release of rholang-nvim
- Basic Rholang language support for Neovim
- Legacy Vim syntax highlighting (`syntax/rholang.vim`)
  - Keywords: `contract`, `new`, `for`, `match`, etc.
  - String and URI highlighting
  - Number and operator support
  - Comment highlighting (line and block)
- Filetype detection for `.rho` files
- Basic plugin structure
  - `lua/rholang.lua` - Main plugin module
  - `ftplugin/rholang.lua` - Filetype-specific settings
- Configuration system
  - `setup()` function for plugin initialization
  - LSP enable/disable toggle
  - Tree-sitter enable/disable toggle
- Documentation
  - README with installation instructions
  - Basic usage examples
  - Manual installation guide
- License (LICENSE.TXT)

### Notes
- This version provides fallback syntax highlighting for environments without Tree-sitter
- LSP features require separate installation of rholang-language-server

---

## Version Compatibility

- **v0.4.0+**: Requires Neovim v0.9.0+, rholang-language-server v0.1.0+
- **v0.3.0**: Requires Neovim v0.9.0+, rholang-language-server v0.0.x
- **v0.2.0**: Requires Neovim v0.8.0+, rholang-language-server v0.0.x
- **v0.1.0**: Requires Neovim v0.7.0+

## Migration Guides

### Migrating from v0.3.0 to v0.4.0

**Configuration Changes:**
```lua
-- Old (v0.3.0)
require('rholang').setup({
  lsp = {
    use_rnode = true,
    grpc_host = 'localhost',
    grpc_port = 40402,
  },
})

-- New (v0.4.0)
require('rholang').setup({
  lsp = {
    validator_backend = 'grpc:localhost:40402',
  },
})
```

**Keybinding Changes:**
- Rename: `e` → `<leader>rn`
- Reference highlighting: `*` → Use LSP's built-in document highlighting
- Reference navigation: `n`/`N` → No longer bound (use location list with `:lnext`/`:lprev`)

**New Keybindings:**
- `K` - Hover documentation
- `<leader>ds` - Document symbols
- `<leader>ws` - Workspace symbols
- `<C-k>` - Signature help

[0.4.0]: https://github.com/F1R3FLY-io/rholang-nvim/compare/v0.3.0...v0.4.0
[0.3.0]: https://github.com/F1R3FLY-io/rholang-nvim/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/F1R3FLY-io/rholang-nvim/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/F1R3FLY-io/rholang-nvim/releases/tag/v0.1.0
