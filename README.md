# Rholang Neovim Plugin

This Neovim plugin provides comprehensive language support for [Rholang](https://github.com/F1R3FLY-io/f1r3fly), a concurrent, process-oriented programming language. It leverages Tree-sitter for syntax highlighting, indentation, folding, and text objects, and integrates with [rholang-language-server](https://github.com/F1R3FLY-io/rholang-language-server) for LSP features in `.rho` files.

## Features

- **Tree-sitter Syntax Highlighting**: Highlights Rholang keywords (`contract`, `new`, `for`, etc.), strings, URIs, numbers, operators, variables, and comments.
- **LSP Integration**: Provides autocompletion, diagnostics, go-to-definition, hover, rename, and more via `rholang-language-server` v0.1.0 (4-8x performance improvement over previous versions).
- **LSP Semantic Highlighting**: Enhanced syntax highlighting using semantic tokens from the language server for more accurate, context-aware coloring (supports embedded MeTTa language).
- **Automatic Indentation**: Smart indentation for blocks (`{}`), lists (`[]`), tuples (`()`), and other constructs, with custom `<CR>`, `o`, and `O` keymaps.
- **Delimiter Handling**: Auto-closes `{`, `(`, `[`, and `"` with matching pairs, skips closing delimiters, and deletes empty pairs or strings (`""`) using `<BS>`, `<DEL>`, `x`, or `X`.
- **Folding**: Tree-sitter-based folding for `contract`, `block`, `input`, `match`, `choice`, `new`, `par`, and `method` nodes.
- **Text Objects**: Navigate and select code blocks, strings, and variables with Tree-sitter text objects.
- **Legacy Syntax Support**: Fallback Vim syntax highlighting (`syntax/rholang.vim`) for environments without Tree-sitter.

## Prerequisites

- Neovim v0.9.0 or later (required for Tree-sitter support)
- [Tree-sitter CLI](https://tree-sitter.github.io/tree-sitter/using-parsers#installation) (`npm install -g tree-sitter-cli`)
- [`rholang-language-server`](https://github.com/F1R3FLY-io/rholang-language-server) installed and in your system PATH
- Compiler tools:
  - Linux: `gcc`, `g++`
  - macOS: `clang` (via Xcode or command line tools)
  - Windows: `cl.exe` (via Visual Studio Build Tools)
- [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) for Tree-sitter parsing

## Installation

### Using a Plugin Manager

#### lazy.nvim

Add to your `lazy.nvim` configuration in `init.lua`:

```lua
{
  'F1R3FLY-io/rholang-nvim',
  config = function()
    require('rholang').setup()
  end,
  build = function()
    local cmd = 'cd ' .. vim.fn.expand('<sfile>:p:h') .. ' && make && make install'
    local result = vim.fn.system(cmd)
    if vim.v.shell_error ~= 0 then
      vim.notify('rholang-nvim build failed: ' .. result, vim.log.levels.ERROR)
    else
      vim.notify('rholang-nvim build successful', vim.log.levels.INFO)
    end
  end,
}
```

#### packer.nvim

Add to your `packer.nvim` configuration:

```lua
use {
  'F1R3FLY-io/rholang-nvim',
  config = function()
    require('rholang').setup()
  end,
  run = ':make && make install',
}
```

#### vim-plug

Add to your `vim-plug` configuration:

```vim
Plug 'F1R3FLY-io/rholang-nvim', { 'do': 'make && make install' }
```

Then, in your `init.lua`:

```lua
require('rholang').setup()
```

#### pckr.nvim

Add to your `pckr.nvim` configuration:

```lua
{
  'F1R3FLY-io/rholang-nvim',
  config = function()
    require('rholang').setup()
  end,
  run = 'make && make install',
}
```

### Manual Installation

1. Clone the repository:

```bash
git clone https://github.com/F1R3FLY-io/rholang-nvim.git ~/.local/share/nvim/site/pack/rholang/start/rholang-nvim
```

2. Compile and install the Tree-sitter parser:

```bash
cd ~/.local/share/nvim/site/pack/rholang/start/rholang-nvim
make && make install
```

Alternatively, use the build script:

```bash
./build.sh
```

3. Install the Rholang Tree-sitter parser in Neovim:

```vim
:TSInstall rholang
```

### Tree-sitter Parser Setup

**Important**: The build scripts automatically set `RHOLANG_NAMED_COMMENTS=1` when generating the parser. This ensures that `line_comment` and `block_comment` nodes are included as named nodes in the parse tree, which is required for LSP features that work with comments. If you manually run `tree-sitter generate`, you must set this environment variable:

```bash
RHOLANG_NAMED_COMMENTS=1 tree-sitter generate grammar.js
```

Ensure `nvim-treesitter` is installed and configured:

```lua
{
  'nvim-treesitter/nvim-treesitter',
  lazy = false,
  build = ':TSUpdate',
  config = function()
    require('nvim-treesitter.configs').setup {
      ensure_installed = { 'rholang' },
      highlight = { enable = true },
      indent = { enable = true },
      fold = { enable = true },
    }
  end,
}
```

## Local Development

For local development, clone the repository to a custom path and use the `dir` parameter. Example for `lazy.nvim`:

```lua
{
  'F1R3FLY-io/rholang-nvim',
  dir = '~/path/to/rholang-nvim',
  config = function()
    require('rholang').setup {
      lsp = {
        enable = true,
        log_level = 'debug',
        language_server_path = 'rholang-language-server',
      },
      treesitter = {
        enable = true,
        highlight = true,
        indent = true,
        fold = true,
      },
    }
  end,
  build = function()
    local cmd = 'cd ~/path/to/rholang-nvim && make && make install'
    local result = vim.fn.system(cmd)
    if vim.v.shell_error ~= 0 then
      vim.notify('rholang-nvim build failed: ' .. result, vim.log.levels.ERROR)
    else
      vim.notify('rholang-nvim build successful', vim.log.levels.INFO)
    end
  end,
}
```

Register the local Tree-sitter parser:

```lua
require('nvim-treesitter.configs').setup {
  ensure_installed = { 'rholang' },
}
vim.api.nvim_create_autocmd('User', {
  pattern = 'TSUpdate',
  callback = function()
    require('nvim-treesitter.parsers').rholang = {
      install_info = {
        path = '~/path/to/rholang-nvim',
        generate = true,
        queries = 'queries/rholang',
      },
      maintainers = { '@your-username' },
      tier = 2, -- Unstable, local parser
    }
  end,
})
```

Build and test locally:

```bash
cd ~/path/to/rholang-nvim
./build.sh  # Automatically sets RHOLANG_NAMED_COMMENTS=1
nvim -u NONE -c 'TSInstall rholang' -c 'q'
```

**Note**: The `build.sh` and `Makefile` automatically set `RHOLANG_NAMED_COMMENTS=1` to enable named comment nodes. If you're building manually with `tree-sitter generate`, ensure you set this variable:

```bash
RHOLANG_NAMED_COMMENTS=1 tree-sitter generate grammar.js
```

## Usage

1. Open a `.rho` file (e.g., `test.rho`):

```rholang
new input, output in {
    for (@message <- input) {
        output!(message)
    }
}
```

2. The plugin automatically:
   - Applies Tree-sitter syntax highlighting, indentation, and folding (if `treesitter.enable = true`).
   - Starts `rholang-language-server` for LSP features (if `lsp.enable = true`).
   - Enables delimiter auto-closing and deletion for `{`, `(`, `[`, and `"`.

3. Key features:
   - **Delimiters**: Type `{`, `(`, `[`, or `"` to auto-close; press `<BS>`, `<DEL>`, `x`, or `X` on empty pairs (e.g., `{}`, `""`) to delete both.
   - **Indentation**: Press `<CR>`, `o`, or `O` to indent new lines within blocks, lists, etc.
   - **Folding**: Use `:set foldmethod=expr foldexpr=v:lua.vim.treesitter.foldexpr()` and `zc`/`zo` to fold/unfold code blocks.
   - **Text Objects**: Select code blocks or strings with `:lua vim.treesitter.textobjects.select('@block.outer')`.
   - **LSP**: See the [LSP Keybindings](#lsp-keybindings) section below.

## LSP Keybindings

The plugin configures standard LSP keybindings when `lsp.enable = true`. These keybindings are only active in Rholang buffers (`.rho` files):

### Navigation

| Keybinding | Command | Description | Status |
|------------|---------|-------------|--------|
| `gd` | `vim.lsp.buf.definition` | Go to definition | ✅ Supported |
| `gD` | `vim.lsp.buf.declaration` | Go to declaration | ✅ Supported |
| `gr` | `vim.lsp.buf.references` | Find all references | ✅ Supported |
| `gi` | `vim.lsp.buf.implementation` | Go to implementation | ❌ Not supported |
| `gy` | `vim.lsp.buf.type_definition` | Go to type definition | ❌ Not supported |

### Documentation and Information

| Keybinding | Command | Description | Status |
|------------|---------|-------------|--------|
| `K` | `vim.lsp.buf.hover` | Show hover documentation | ✅ Supported |
| `<C-k>` | `vim.lsp.buf.signature_help` | Show signature help (insert & normal mode) | ✅ Supported |

### Editing

| Keybinding | Command | Description | Status |
|------------|---------|-------------|--------|
| `<leader>rn` | `vim.lsp.buf.rename` | Rename symbol (workspace-wide) | ✅ Supported |
| `<leader>ca` | `vim.lsp.buf.code_action` | Show code actions | ❌ Not yet implemented |

### Symbols

| Keybinding | Command | Description | Status |
|------------|---------|-------------|--------|
| `<leader>ds` | `vim.lsp.buf.document_symbol` | Show document symbols (outline) | ✅ Supported |
| `<leader>ws` | `vim.lsp.buf.workspace_symbol` | Search workspace symbols | ✅ Supported |

### Formatting

| Keybinding | Command | Description | Status |
|------------|---------|-------------|--------|
| `<leader>f` | `vim.lsp.buf.format` | Format document (normal & visual mode) | ❌ Not yet implemented |

**Note**: Keybindings marked as "Not supported" or "Not yet implemented" are commented out in the plugin code and can be enabled when the language server implements these features.

### Breaking Changes from v0.3.0

If you're upgrading from v0.3.0, note the following keybinding changes:

- **Rename**: Changed from `e` to `<leader>rn` (standard LSP convention)
- **Reference highlighting**: Removed custom `*`, `n`, `N` keybindings for reference navigation (use standard LSP document highlighting instead)

## File Structure

```
rholang-nvim
├── build.sh                     # Build script for Tree-sitter parser
├── CMakeLists.txt               # CMake build configuration
├── ftplugin
│   └── rholang.lua              # Filetype-specific settings
├── grammar.js                   # Tree-sitter grammar for Rholang
├── lua
│   └── rholang.lua              # Plugin setup, LSP, Tree-sitter config
├── Makefile                     # Makefile for parser compilation
├── queries
│   └── rholang
│       ├── folds.scm            # Folding rules
│       ├── highlights.scm       # Highlighting rules
│       ├── indents.scm          # Indentation rules
│       ├── locals.scm           # Variable definitions/references
│       └── textobjects.scm      # Text object definitions
├── syntax
│   └── rholang.vim              # Legacy Vim syntax highlighting
├── LICENSE.TXT                  # License file
└── README.md                    # This file
```

## Configuration

Configure the plugin via `require('rholang').setup(config)`. Default settings:

```lua
{
  lsp = {
    enable = true,
    log_level = 'debug', -- Options: error, warn, info, debug, trace
    language_server_path = 'rholang-language-server',
    validator_backend = 'rust', -- Validator backend: 'rust' or 'grpc:host:port'
    semantic_tokens = true, -- Enable LSP semantic highlighting (default: true)
  },
  treesitter = {
    enable = true,
    highlight = true,
    indent = true,
    fold = true,
  },
}
```

### Validator Backend

The language server supports multiple validator backends for obtaining diagnostics:

1. **Embedded Rust Parser/Interpreter** (default, `validator_backend = 'rust'`):
   - Uses the embedded Rust implementation directly linked into the language server
   - Faster startup, no external dependencies
   - Suitable for most development workflows

2. **Legacy RNode via gRPC** (`validator_backend = 'grpc:host:port'`):
   - Communicates with legacy Scala RNode server via gRPC
   - Stable, production-ready implementation
   - Required for customers using legacy RNode
   - Can run RNode in Docker or locally
   - Example: `'grpc:localhost:40402'`

**Note**: The `validator_backend` option is passed directly to the language server via the `--validator-backend` flag.

### LSP Semantic Highlighting

Semantic tokens provide context-aware syntax highlighting by leveraging the language server's deep understanding of your code. Unlike tree-sitter which uses pattern matching, semantic highlighting understands the semantic meaning of tokens.

**Benefits:**
- **More accurate**: Distinguishes between different uses of the same syntax (e.g., a variable vs. a function)
- **Context-aware**: Knows the type and role of each identifier
- **MeTTa support**: Properly highlights embedded MeTTa language within Rholang strings
- **Semantic categories**: Provides granular token types (parameter, property, method, etc.)

**Configuration:**

Enabled by default. To disable:

```lua
require('rholang').setup({
  lsp = {
    semantic_tokens = false,
  },
})
```

**Note**: Semantic highlighting works alongside tree-sitter highlighting. Tree-sitter provides fast, initial highlighting while semantic tokens add semantic accuracy once the language server has analyzed the file.

### Migration from v0.3.0

If you're upgrading from v0.3.0, the configuration has changed:

**Old configuration (v0.3.0):**
```lua
{
  lsp = {
    use_rnode = true,
    grpc_host = 'localhost',
    grpc_port = 40402,
  },
}
```

**New configuration (v0.4.0):**
```lua
{
  lsp = {
    validator_backend = 'grpc:localhost:40402',
  },
}
```

### Configuration Examples

- **Use RNode for diagnostics** (legacy Scala RNode):

```lua
require('rholang').setup({
  lsp = {
    validator_backend = 'grpc:localhost:40402',
  },
})
```

- **Use RNode in Docker**:

```lua
require('rholang').setup({
  lsp = {
    validator_backend = 'grpc:localhost:40402', -- or use Docker host IP
  },
})
```

- **Disable Tree-sitter** (use legacy syntax):

```lua
require('rholang').setup({
  treesitter = {
    enable = false,
  },
})
```

- **Change LSP log level**:

```lua
require('rholang').setup({
  lsp = {
    log_level = 'info',
  },
})
```

- **Custom LSP command**:

```lua
require('rholang').setup()
vim.api.nvim_create_autocmd('FileType', {
  pattern = 'rholang',
  callback = function()
    vim.lsp.start({
      name = 'rholang',
      cmd = {
        'rholang-language-server',
        '--no-color',
        '--stdio',
        '--log-level', 'trace',
        '--client-process-id', tostring(vim.fn.getpid()),
      },
      root_dir = vim.fs.dirname(
        vim.fs.find({ '.git', 'rholang.toml' }, { upward = true })[1] or '.'
      ),
    })
  end,
})
```

## Troubleshooting

- **Tree-sitter errors**: Run `:TSInstall rholang` and compile the parser (`make && make install` or `./build.sh`).
- **LSP not starting**: Ensure `rholang-language-server` is in your PATH and executable or specify its path in the config via `lsp.language_server_path`.
- **Indentation issues**: Verify `:set indentexpr=v:lua.require'nvim-treesitter'.indentexpr()` and `treesitter.indent = true`.
- **Build failures**: Check for `gcc`/`g++` (Linux), `clang` (macOS), or `cl.exe` (Windows). Run `./build.sh` for diagnostics.
- **Debugging**: Enable LSP logging:
  ```vim
  :lua vim.lsp.set_log_level('debug')
  :LspLog
  ```

## Health Check

Run `:checkhealth rholang` to verify the plugin's setup. The health check ensures that all components are correctly installed and configured, including:

- Neovim version (>= 0.11.0)
- Dependencies: Tree-sitter CLI, `nvim-treesitter` plugin, and appropriate compilers (`gcc`/`g++` for Linux, `clang` for macOS, or `cl.exe` for Windows)
- Tree-sitter configuration: Parser installation, syntax highlighting, indentation, and folding
- LSP configuration: Availability and running status of `rholang-language-server`
- **Validator backend**: Reports which validator backend is configured (Rust or gRPC)
- Filetype detection for `.rho` files

A successful health check output looks like:

```text
rholang-nvim: Plugin Health Check

Checking Neovim version
- OK Neovim version 0.11 meets requirement (>= 0.11.0)

Checking dependencies
- OK Tree-sitter CLI is installed and executable
- OK nvim-treesitter plugin is installed
- OK Compiler gcc is available for linux

Checking Tree-sitter configuration
- OK Tree-sitter highlight is enabled
- OK Tree-sitter indent is enabled
- OK Tree-sitter fold is enabled
- OK Tree-sitter parser for Rholang is installed
- OK Tree-sitter syntax highlighting is functional

Checking LSP configuration
- INFO Validator backend: rust
- INFO LSP semantic tokens: enabled
- OK rholang-language-server is installed and executable
- OK rholang-language-server is running (client ID: 1)

Checking filetype detection
- OK Filetype detection for .rho files is working
```

When using RNode via gRPC, the LSP configuration section will show:

```text
Checking LSP configuration
- INFO Validator backend: grpc:localhost:40402
- INFO LSP semantic tokens: enabled
- OK rholang-language-server is installed and executable
- OK rholang-language-server is running (client ID: 1)
```

If issues are reported, refer to the [Troubleshooting](#troubleshooting) section for guidance on resolving them.

## License

Licensed under the terms in `LICENSE.TXT`.

## Contributing

Submit issues or pull requests to the [rholang-nvim repository](https://github.com/F1R3FLY-io/rholang-nvim).

## Related Projects

- [rholang-language-server](https://github.com/F1R3FLY-io/rholang-language-server)
- [Rholang](https://github.com/F1R3FLY-io/f1r3fly)

## Version

**0.4.0** (October 31, 2025)

### What's New in v0.4.0

- **Standard LSP keybindings**: Migrated to standard Neovim LSP conventions (e.g., `K` for hover, `<leader>rn` for rename)
- **Unified validator backend**: Simplified configuration with `validator_backend` option (replaces `use_rnode`, `grpc_host`, `grpc_port`)
- **Enhanced LSP features**: Added keybindings for document symbols, workspace symbols, and signature help
- **Complete tree-sitter migration**: Improved syntax highlighting, indentation, and code folding
- **Performance improvements**: Compatible with rholang-language-server v0.1.0 (4-8x faster LSP operations)

### Breaking Changes

- Configuration: `use_rnode`, `grpc_host`, `grpc_port` replaced with `validator_backend`
- Keybinding: Rename changed from `e` to `<leader>rn`
- Removed custom `*`, `n`, `N` keybindings for reference navigation

See [CHANGELOG.md](CHANGELOG.md) for full version history.