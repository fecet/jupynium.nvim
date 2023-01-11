# Jupynium: Control Jupyter Notebook on Neovim with ZERO Compromise

[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
<a href="https://github.com/kiyoon/jupynium.nvim/actions/workflows/tests.yml">
    <img src="https://github.com/kiyoon/jupynium.nvim/workflows/Tests/badge.svg?style=flat" />
</a>

Jupynium uses Selenium to automate Jupyter Notebook, synchronising everything you type on Neovim.  
Never leave Neovim. Switch tabs on the browser as you switch files on Neovim.

<img src=https://user-images.githubusercontent.com/12980409/211894627-73037e83-4730-4387-827c-98ed2522740d.gif width=100% />

Note that it doesn't sync from Notebook to Neovim so be careful.

### How does it work?

<img src=https://user-images.githubusercontent.com/12980409/211933889-e31e844c-1cf3-4d1a-acdc-70cb6e244ee4.png width=50% />

The Jupynium server will receive events from Neovim, keep the copy of the buffer and apply that to the Jupyter Notebook by using Selenium browser automation. It interacts only through the front end so it doesn't require installing extensions on the kernel etc., which makes it possible to

- Develop locally, run remotely (or vice versa)
- Use university-hosted Jupyter Notebook

Only supports Jupyter Notebook (Jupyter Lab disables front-end interaction)

## 🛠️ Installation

### Requirements

- 💻 Linux, macOS and Windows WSL2
- 🦊 Firefox (Other browsers are not supported due to their limitation with Selenium)
- ✌️ Neovim >= v0.8
- 🐍 Python >= 3.10

Don't have system python 3.10? You can use [Conda](https://docs.conda.io/en/latest/miniconda.html).

```bash
conda create -n jupynium
conda activate jupynium && conda install -c conda-forge python=3.11
```

Install with vim-plug:

```vim
Plug 'kiyoon/jupynium.nvim', { 'do': 'pip3 install --user .' }
" Plug 'kiyoon/jupynium.nvim', { 'do': '~/miniconda3/bin/envs/jupynium/bin/pip install .' }
```

Install with packer.nvim:

```lua
use { "kiyoon/jupynium.nvim", run = "pip3 install --user ." }
-- use { "kiyoon/jupynium.nvim", run = "~/miniconda3/bin/envs/jupynium/bin/pip install ." }
```

Setup is optional for system python users and here are the defaults. Conda users need to change the `python_host`.
<details>
<summary>
See setup defaults
</summary>

This comes after `call plug#end()` for vim-plug users.

```lua
require("jupynium").setup({
  -- Conda users:
  -- python_host = "~/miniconda3/envs/jupynium/bin/python",
  python_host = vim.g.python3_host_prog or "python3",

  default_notebook_URL = "localhost:8888",

  -- Open the Jupynium server if it is not already running
  -- which means that it will open the Selenium browser when you open this file.
  -- Related command :JupyniumStartAndAttachToServer
  auto_start_server = {
    enable = false,
    file_pattern = { "*.ju.py" },
  },

  -- Attach current nvim to the Jupynium server
  -- Without this step, you can't use :JupyniumStartSync
  -- Related command :JupyniumAttachToServer
  auto_attach_to_server = {
    enable = true,
    file_pattern = { "*.ju.py", "*.md" },
  },

  -- Automatically open an Untitled.ipynb file on Notebook
  -- when you open a .ju.py file on nvim.
  -- Related command :JupyniumStartSync
  auto_start_sync = {
    enable = false,
    file_pattern = { "*.ju.py", "*.md" },
  },

  -- Automatically keep filename.ipynb copy of filename.ju.py
  -- by downloading from the Jupyter Notebook server.
  -- WARNING: this will overwrite the file without asking
  -- Related command :JupyniumDownloadIpynb
  auto_download_ipynb = true,

  -- Always scroll to the current cell.
  -- Related command :JupyniumScrollToCell
  autoscroll = {
    enable = true,
    mode = "always", -- "always" or "invisible"
    cell = {
      top_margin_percent = 20,
    },
  },

  scroll = {
    page = { step = 0.5 },
    cell = {
      top_margin_percent = 20,
    },
  },

  use_default_keybindings = true,
  textobjects = {
    use_default_keybindings = true,
  },

  -- Dim all cells except the current one
  -- Related command :JupyniumShortsightedToggle
  shortsighted = false,
})

```

</details>

## 🚦 Usage

First, I recommend setting password on your notebook (rather than using tokens)

```console
$ jupyter notebook password
Enter password: 🔒

$ jupyter notebook 			# leave notebook opened
```

**Jupynium server stays alive as long as the browser is alive.**  
So you can see them as the same thing in this doc.  
For example:

- Starting Jupynium server = opening a Selenium browser
- Manually closing the browser = closing the Jupynium server

### Open and attach to a Jupynium server

Running `:JupyniumStartAndAttachToServer` will open the notebook.  
Type password and once **you need to be on the main page (file browser) for the next steps**.

### Sync current buffer to the Jupynium server

You attached your nvim instance to the server, but it won't automatically start syncing.  
Run `:JupyniumStartSync` to create a new ipynb file and start syncing from vim to notebook.

You can also:

1. `:JupyniumStartSync filename` to give name (`filename.ipynb`) instead of `Untitled.ipynb`.

- This will not open a file if it exists. Instead filename will be ignored.

2. Manually open the file from the browser, and `:JupyniumStartSync 2` to sync using the 2nd tab (count from 1).

### Use multiple buffers

Run `:JupyniumStartSync` again with a new buffer you want to sync.

### Use multiple neovim

You can run `:JupyniumStartSync` on a new neovim instance.  
If you have `auto_attach_to_server = false` during setup, you need to run `:JupyniumAttachToServer` and `:JupyniumStartSync`.

## 📝 .ju.py file format

The file format is designed to be LSP friendly even with markdown code injected in it. The markdown cells will be part of python string `"""%%` ... `%%"""`.

**Code cell separators:**  
i.e. Any code below this line (and before the next separator) will be a code cell.

- `# %%`: recommended
- `%%"""`: use when you want to close markdown cell
- `%%'''`

**Markdown cell separators:**

- `"""%%`: recommended
- `'''%%`
- `# %%%`

**Explicitly specify the first cell separator to use it like a notebook.**

- If there is one or more cells, it works as a notebook mode.
  - Contents before the first cell are ignored, so use it as a heading (shebang etc.)
- If there is no cell, it works as a markdown preview mode.
  - It will still open ipynb file but will one have one markdown cell.

## ⌨️ Keybindings

- `<space>x`: Execute selected cells
- `<space>c`: Clear selected cells
- `<space>S`: Scroll to cell (if autoscroll is disabled)
- `<PageUp>`, `<PageDown>`: Scroll notebook

If you want custom keymaps, add `use_default_keybindings = false` and follow `M.default_keybindings()` in [lua/jupynium/init.lua](lua/jupynium/init.lua).

### Textobjects

- `[j`, `]j`: go to previous / next cell separator
- `<space>j`: go to current cell separator
- `vaj`,`vij`, `vaJ`, `viJ`: select current cell
  - `a`: include separator, `i`: exclude separator
  - `j`: exclude next separator, `J`: include next separator

If you want custom keymaps, add `textobjects = { use_default_keybindings = false }` and follow `M.default_keybindings()` in [lua/jupynium/textobj.lua](lua/jupynium/textobj.lua).

## 📡 Available Vim Commands

```vim
" Server and sync
:JupyniumStartAndAttachToServer [notebook_URL]
:JupyniumAttachToServer [notebook_URL]
:JupyniumStartSync [filename / tab_index]
:JupyniumStopSync

" Notebook
:JupyniumSaveIpynb
:JupyniumDownloadIpynb [filename]
:JupyniumAutoDownloadIpynbToggle

:JupyniumScrollToCell
:JupyniumScrollUp
:JupyniumScrollDown
:JupyniumAutoscrollToggle

:JupyniumExecuteSelectedCells
:JupyniumClearSelectedCellsOutputs
:JupyniumToggleSelectedCellsOutputsScroll

" Highlight
:JupyniumShortsightedToggle
```

## 👨‍💻️ Command-Line Usage

**You don't need to install the vim plugin to use Jupynium.**  
**The plugin is responsible of adding `:JupyniumStartAndAttachToServer` etc. that just calls the command line program,**  
**plus it has textobjects and shortsighted support.**

Install Jupynium if you haven't.

```bash
$ pip3 install jupynium
```

Open a python/markdown file with nvim and see `:echo v:servername`.  
Run Jupynium like:

```bash
$ jupynium --nvim_listen_addr /tmp/your_socket_path
```

Or, you can run Neovim like

```bash
$ nvim --listen localhost:18898 notebook.ju.py
```

Then start Jupynium as well as attaching the neovim to it.

```bash
$ jupynium --nvim_listen_addr localhost:18898
```

Note that you can attach to remote neovim by changing `localhost` to `servername.com` or using SSH port forwarding.

This will open Firefox with Selenium, defaulting to `http://localhost:8888`.

Additionally,

- You can run `jupynium` command multiple times to attach more than one Neovim instance.
- `jupynium --notebook_URL localhost:18888` to view different notebook.

### Converting ipynb to Jupynium file

```bash
ipynb2jupy [-h] [--stdout] file.ipynb [file.ju.py]
```

## ⚠️ Caution

The program is in an alpha stage. If it crashes it's likely that the whole browser turns off without saving!

Rules:

1. Always leave the home page accessible. Jupynium interacts with it to open files. Do not close, or move to another website.

- It's okay to move between directories.

2. It's OK to close the notebook pages. If you do so, it will stop syncing that buffer.
3. Changing tab ordering or making it to a separate window is OK.

## 📰 Fun Facts

- I spent my whole Christmas and New Year holidays (and more) just making this plugin.