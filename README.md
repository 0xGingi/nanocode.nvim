# nanocode.nvim

Integrate the [nanocode](https://github.com/nnanocode) AI assistant with Neovim — streamline editor-aware research, reviews, and requests.


## ✨ Features

- Auto-connect to _any_ `nanocode` running inside Neovim's CWD, or provide an integrated instance.
- Share editor context (buffer, cursor, selection, diagnostics, etc.).
- Input prompts with completions, highlights, and normal-mode support.
- Select prompts from a library and define your own.
- Execute commands.
- Respond to permission requests.
- Reload edited buffers in real-time.
- Monitor state via statusline component.
- Forward Server-Sent-Events as autocmds for automation.
- Sensible defaults with well-documented, flexible configuration and API to fit your workflow.
- _Vim-y_ — supports ranges and dot-repeat.

## 📦 Setup

### [lazy.nvim](https://github.com/folke/lazy.nvim)

```lua
{
  "nanogpt-community/nanocode.nvim",
  dependencies = {
    -- Recommended for `ask()` and `select()`.
    -- Required for `snacks` provider.
    ---@module 'snacks' <- Loads `snacks.nvim` types for configuration intellisense.
    { "folke/snacks.nvim", opts = { input = {}, picker = {}, terminal = {} } },
  },
  config = function()
    ---@type nanocode.Opts
    vim.g.nanocode = {
      -- Your configuration, if any — see `lua/nanocode/config.lua`, or "goto definition" on the type or field.
    }

    -- Required for `opts.events.reload`.
    vim.o.autoread = true

    -- Recommended/example keymaps.
    vim.keymap.set({ "n", "x" }, "<C-a>", function() require("nanocode").ask("@this: ", { submit = true }) end, { desc = "Ask nanocode…" })
    vim.keymap.set({ "n", "x" }, "<C-x>", function() require("nanocode").select() end,                          { desc = "Execute nanocode action…" })
    vim.keymap.set({ "n", "t" }, "<C-.>", function() require("nanocode").toggle() end,                          { desc = "Toggle nanocode" })

    vim.keymap.set({ "n", "x" }, "go",  function() return require("nanocode").operator("@this ") end,        { desc = "Add range to nanocode", expr = true })
    vim.keymap.set("n",          "goo", function() return require("nanocode").operator("@this ") .. "_" end, { desc = "Add line to nanocode", expr = true })

    vim.keymap.set("n", "<S-C-u>", function() require("nanocode").command("session.half.page.up") end,   { desc = "Scroll nanocode up" })
    vim.keymap.set("n", "<S-C-d>", function() require("nanocode").command("session.half.page.down") end, { desc = "Scroll nanocode down" })

    -- You may want these if you stick with the opinionated "<C-a>" and "<C-x>" above — otherwise consider "<leader>o…".
    vim.keymap.set("n", "+", "<C-a>", { desc = "Increment under cursor", noremap = true })
    vim.keymap.set("n", "-", "<C-x>", { desc = "Decrement under cursor", noremap = true })
  end,
}
```

### [nixvim](https://github.com/nix-community/nixvim)

```nix
programs.nixvim = {
  extraPlugins = [
    pkgs.vimPlugins.nanocode-nvim
  ];
};
```

> [!TIP]
> Run `:checkhealth nanocode` after setup.

## ⚙️ Configuration

`nanocode.nvim` provides a rich and reliable default experience — see all available options and their defaults [here](./lua/nanocode/config.lua).

### Contexts

`nanocode.nvim` replaces placeholders in prompts with the corresponding context:

| Placeholder    | Context                                                       |
| -------------- | ------------------------------------------------------------- |
| `@this`          | Operator range or visual selection if any, else cursor position |
| `@buffer`        | Current buffer                                              |
| `@buffers`       | Open buffers                                                |
| `@visible`       | Visible text                                                |
| `@diagnostics`   | Current buffer diagnostics                                  |
| `@quickfix`      | Quickfix list                                               |
| `@diff`          | Git diff                                                    |
| `@marks`         | Global marks                                                |
| `@grapple`       | [grapple.nvim](https://github.com/cbochs/grapple.nvim) tags |

### Prompts

Select or reference prompts to review, explain, and improve your code:

| Name          | Prompt                                                                 |
| ------------- | ---------------------------------------------------------------------- |
| `diagnostics` | Explain `@diagnostics`                                                 |
| `diff`        | Review the following git diff for correctness and readability: `@diff` |
| `document`    | Add comments documenting `@this`                                       |
| `explain`     | Explain `@this` and its context                                        |
| `fix`         | Fix `@diagnostics`                                                     |
| `implement`   | Implement `@this`                                                      |
| `optimize`    | Optimize `@this` for performance and readability                       |
| `review`      | Review `@this` for correctness and readability                         |
| `test`        | Add tests for `@this`                                                  |

nanocodenanocodenanocodenanocodenanocode<details>
<summary><a href="https://neovim.io/doc/user/terminal.html">Neovim terminal</a></summary>

```lua
vim.g.nanocode_opts = {
  provider = {
    enabled = "terminal",
    terminal = {
      -- ...
    }
  }
}
```

</details>

<details>
<summary><a href="https://github.com/folke/snacks.nvim/blob/main/docs/terminal.md">snacks.terminal</a></summary>

```lua
vim.g.nanocode_opts = {
  provider = {
    enabled = "snacks",
    snacks = {
      -- ...
    }
  }
}
```

</details>

<details>
<summary><a href="https://sw.kovidgoyal.net/kitty/">kitty</a></summary>

```lua
vim.g.nanocode_opts = {
  provider = {
    enabled = "kitty",
    kitty = {
      -- ...
    }
  }
}
```

The kitty provider requires [remote control via a socket](https://sw.kovidgoyal.net/kitty/remote-control/#remote-control-via-a-socket) to be enabled.

You can do this either by running Kitty with the following command:

```bash
# For Linux only:
kitty -o allow_remote_control=yes --single-instance --listen-on unix:@mykitty

# Other UNIX systems:
kitty -o allow_remote_control=yes --single-instance --listen-on unix:/tmp/mykitty
```

OR, by adding the following to your `kitty.conf`:

```
# For Linux only:
allow_remote_control yes
listen_on unix:@mykitty
# Other UNIX systems:
allow_remote_control yes
listen_on unix:/tmp/kitty
```

</details>

<details>
<summary><a href="https://wezterm.org/">wezterm</a></summary>

```lua
vim.g.nanocode_opts = {
  provider = {
    enabled = "wezterm",
    wezterm = {
      -- ...
    }
  }
}
```

</details>

<details>
<summary><a href="https://github.com/tmux/tmux">tmux</a></summary>

```lua
vim.g.nanocode_opts = {
  provider = {
    enabled = "tmux",
    tmux = {
      -- ...
    }
  }
}
```

</details>

<details>
<summary>custom</summary>

Integrate your custom method for convenience!

```lua
vim.g.nanocode_opts = {
  provider = {
    toggle = function(self)
      -- ...
    end,
    start = function(self)
      -- ...
    end,
    stop = function(self)
      -- ...
    end,
  }
}
```

</details>

Please submit PRs adding new providers! 🙂

## 🚀 Usage

### ✍️ Ask — `require("nanocode").ask()`

Input a prompt for `nanocode`.

- Press `<Up>` to browse recent asks.
- Highlights and completes contexts and `nanocode` subagents.
  - Press `<Tab>` to trigger built-in completion.
  - Registers `opts.ask.blink_cmp_sources` when using `snacks.input` and `blink.cmp`.

### 📝 Select — `require("nanocode").select()`

Select from all `nanocode.nvim` functionality.

- Prompts
- Commands
  - Fetches custom commands from `nanocode`
- Provider controls

Highlights and previews items when using `snacks.picker`.

### 🗣️ Prompt — `require("nanocode").prompt()`

Prompt `nanocode`.

- Resolves named references to configured prompts.
- Injects configured contexts.
- `nanocode` will interpret `@` references to files or subagents.

### 🧑‍🔬 Operator — `require("nanocode").operator()`

Wraps `prompt` as an operator, supporting ranges and dot-repeat.

### 🧑‍🏫 Command — `require("nanocode").command()`

Command `nanocode`:

| Command                  | Description                                        |
| ------------------------ | -------------------------------------------------- |
| `session.list`           | List sessions                                      |
| `session.new`            | Start a new session                                |
| `session.select`         | Select a session                                   |
| `session.share`          | Share the current session                          |
| `session.interrupt`      | Interrupt the current session                      |
| `session.compact`        | Compact the current session (reduce context size)  |
| `session.page.up`        | Scroll messages up by one page                     |
| `session.page.down`      | Scroll messages down by one page                   |
| `session.half.page.up`   | Scroll messages up by half a page                  |
| `session.half.page.down` | Scroll messages down by half a page                |
| `session.first`          | Jump to the first message in the session           |
| `session.last`           | Jump to the last message in the session            |
| `session.undo`           | Undo the last action in the current session        |
| `session.redo`           | Redo the last undone action in the current session |
| `prompt.submit`          | Submit the TUI input                               |
| `prompt.clear`           | Clear the TUI input                                |
| `agent.cycle`            | Cycle the selected agent                           |

## 👀 Events

`nanocode.nvim` forwards `nanocode`'s Server-Sent-Events as an `nanocodeEvent` autocmd:

```lua
-- Handle `nanocode` events
vim.api.nvim_create_autocmd("User", {
  pattern = "nanocodeEvent:*", -- Optionally filter event types
  callback = function(args)
    ---@type nanocode.cli.client.Event
    local event = args.data.event
    ---@type number
    local port = args.data.port

    -- See the available event types and their properties
    vim.notify(vim.inspect(event))
    -- Do something useful
    if event.type == "session.idle" then
      vim.notify("`nanocode` finished responding")
    end
  end,
})
```

### Edits

When `nanocode` edits a file, `nanocode.nvim` automatically reloads the corresponding buffer.

### Permissions

When `nanocode` requests a permission, `nanocode.nvim` waits for idle to ask you to approve or deny it.

### Statusline

<details>
<summary><a href="https://github.com/nvim-lualine/lualine.nvim">lualine</a></summary>

```lua
require("lualine").setup({
  sections = {
    lualine_z = {
      {
        require("nanocode").statusline,
      },
    }
  }
})
```

</details>

## 🙏 Acknowledgments

- Inspired by [nvim-aider](https://github.com/GeorgesAlkhouri/nvim-aider), [nenanocode.nvim](https://github.com/loukotal/nenanocode.nvim), and [sidekick.nvim](https://github.com/folke/sidekick.nvim).
- Uses `nanocode`'s TUI for simplicity — see [sudo-tee/nanocode.nvim](https://github.com/sudo-tee/nanocode.nvim) for a Neovim frontend.
- [mcp-neovim-server](https://github.com/bigcodegen/mcp-neovim-server) may better suit you, but it lacks customization and tool calls are slow and unreliable.
