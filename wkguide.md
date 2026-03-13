# AstroNvim which-key: switch to the LazyVim-style floating popup

If you want AstroNvim's `which-key` menu to look like LazyVim's horizontal floating popup, the setting you want is `preset = "helix"`.

`helix` is the correct preset name in `folke/which-key.nvim`. LazyVim uses that preset by default.

## What to change

Create a new file at `lua/plugins/which-key.lua` with this content:

```lua
return {
  {
    "folke/which-key.nvim",
    opts = {
      preset = "helix",
    },
  },
}
```

## Why this works

AstroNvim lets you override plugin options by adding plugin spec files under `lua/plugins/`.

`which-key.nvim` supports these presets:

- `classic`
- `modern`
- `helix`

The `helix` preset is the one that gives the floating popup layout similar to LazyVim.

## Apply the change

1. Save `lua/plugins/which-key.lua`.
2. Restart Neovim.
3. If prompted to sync plugins, run `:Lazy sync`.

## Verify

Press `<leader>` in normal mode.

If the change worked, `which-key` should open as a floating popup instead of the older bottom-attached horizontal display.

## If you already have a which-key override

If you already define `folke/which-key.nvim` somewhere under `lua/plugins/`, add this inside that existing spec instead of creating a second file:

```lua
opts = {
  preset = "helix",
}
```

## Does this override only `preset`?

Yes. In this case, you are only overriding the `preset` field.

You do not need to fully reconfigure `which-key` just to change this one option.

With this plugin spec:

```lua
return {
  {
    "folke/which-key.nvim",
    opts = {
      preset = "helix",
    },
  },
}
```

AstroNvim and `lazy.nvim` keep the rest of the existing `which-key` configuration and merge in your new `preset` value.

## How this config architecture works

AstroNvim is built on top of `lazy.nvim`, and plugin configuration is defined as Lua plugin specs.

When you add a file under `lua/plugins/`, AstroNvim loads that spec and combines it with the base spec for the same plugin. For an existing plugin like `folke/which-key.nvim`, that means:

- AstroNvim provides a default plugin spec.
- Your local spec is matched to the same plugin.
- `lazy.nvim` merges your `opts` table into the existing options table.
- Any option you do not mention keeps its previous value.

So `preset = "helix"` replaces only that one setting, while everything else from AstroNvim's default `which-key` setup stays in place.

You only need a full manual config if you want to replace larger parts of the plugin setup, such as custom window behavior, sorting, icons, delays, or mappings metadata.

## Where to find more options

The best reference is the upstream `which-key.nvim` documentation:

- README: <https://github.com/folke/which-key.nvim?tab=readme-ov-file#%EF%B8%8F-configuration>
- Current defaults in source: <https://github.com/folke/which-key.nvim/blob/main/lua/which-key/config.lua>

If you want to sanity-check your setup inside Neovim, `:checkhealth which-key` is also useful.

## AstroNvim note

Because this is AstroNvim, the safest pattern is still to override only the specific `which-key` options you care about.

This works well:

```lua
return {
  {
    "folke/which-key.nvim",
    opts = {
      preset = "helix",
    },
  },
}
```

That preserves AstroNvim's existing `which-key` setup and changes only the display preset.

The larger example below is useful as a reference for the available options, but if you paste the whole thing into AstroNvim you may also replace some of AstroNvim's own opinionated defaults for `which-key` such as icon-related settings. It should still work, but it is more override than most AstroNvim users need.

## Annotated example `lua/plugins/which-key.lua`

This is the same config shape `which-key.nvim` expects, but with comments aimed at someone who already writes software and just has not internalized Lua or Neovim plugin config yet.

Treat this as a reference template, not something you should blindly paste in full. In AstroNvim, the normal move is still to override only the fields you actually want to change.

```lua
return {
  {
    -- This string identifies the plugin we are overriding.
    -- AstroNvim/lazy.nvim match this against the existing plugin spec.
    "folke/which-key.nvim",

    -- `opts` is "the table of options passed into which-key.setup(...)"
    -- Think of it like a JSON/YAML object, except Lua tables can also hold functions.
    opts = {
      -- Pick a built-in layout preset.
      -- `false` means "do not apply any preset".
      -- `classic` is the older default layout.
      -- `modern` is a newer style.
      -- `helix` is the LazyVim-style floating horizontal layout.
      preset = "helix",

      -- How long which-key waits before it pops up, in milliseconds.
      -- This can be a number, or a function if you want dynamic behavior.
      -- Here: show immediately for plugin-generated popups, otherwise wait 200ms.
      delay = function(ctx)
        return ctx.plugin and 0 or 200
      end,

      -- Filter decides which mappings should be shown.
      -- Return true to keep a mapping, false to hide it.
      -- `mapping` is just a Lua table describing one keybinding.
      filter = function(mapping)
        return true
      end,

      -- Extra mapping metadata you want to register manually.
      -- Leave this empty unless you want to add group names, descriptions,
      -- or synthetic entries that are not already defined elsewhere.
      spec = {},

      -- If true, which-key warns you about config problems or mapping issues.
      notify = true,

      -- `triggers` tells which-key what key prefixes should cause the popup to appear.
      -- `<auto>` means "infer the prefixes from existing mappings".
      -- `mode = "nxso"` means normal, visual, select, and operator-pending modes.
      triggers = {
        { "<auto>", mode = "nxso" },
      },

      -- `defer` is for cases where which-key should wait one more keypress
      -- before opening. This mostly matters in operator-pending / visual flows.
      -- Here it defers in linewise visual mode and block visual mode.
      defer = function(ctx)
        return ctx.mode == "V" or ctx.mode == "<C-V>"
      end,

      plugins = {
        -- Show marks after pressing `'` or `` ` ``.
        marks = true,

        -- Show register contents after pressing `"`.
        registers = true,

        -- Spelling suggestions for `z=`.
        spelling = {
          enabled = true,
          suggestions = 20,
        },

        -- These are not plugins you install separately.
        -- They are built-in helper groups that teach default Vim key prefixes.
        presets = {
          -- Operators like `d`, `y`, `c`.
          operators = true,

          -- Motion help.
          motions = true,

          -- Text objects like `iw`, `ap`, etc.
          text_objects = true,

          -- `<C-w>` window commands.
          windows = true,

          -- Misc navigation-related helpers.
          nav = true,

          -- `z`-prefixed commands.
          z = true,

          -- `g`-prefixed commands.
          g = true,
        },
      },

      win = {
        -- Keep the popup from covering the cursor when possible.
        no_overlap = true,

        -- Fixed width in columns. Usually you leave this unset.
        -- width = 1,

        -- Height can be a number or a table like { min = 4, max = 25 }.
        -- height = { min = 4, max = 25 },

        -- Exact floating window position. If unset, which-key computes it.
        -- col = 0,
        -- row = math.huge,

        -- Floating border style.
        -- Common values are "none", "single", "double", "rounded", "solid", "shadow".
        -- border = "none", -- see :h nvim_open_win() for valid border values

        -- Extra padding inside the popup.
        -- This is { vertical_padding, horizontal_padding }.
        padding = { 1, 2 },

        -- Show a window title.
        title = true,

        -- Title alignment. In practice this is usually "left", "center", or "right".
        title_pos = "center",

        -- Draw order. Higher means "more on top".
        zindex = 1000,

        -- Extra buffer-local options for the floating buffer.
        -- Think of this as "low-level buffer config".
        bo = {},

        -- Extra window-local options for the floating window.
        -- Think of this as "low-level window config".
        wo = {
          -- Transparency. 0 = opaque, 100 = very transparent.
          -- winblend = 10, -- 0-100
        },
      },

      layout = {
        -- Minimum popup column width.
        width = { min = 20 },

        -- Space between columns.
        spacing = 3,
      },

      keys = {
        -- Scroll keybindings while the popup is open.
        scroll_down = "<c-d>",
        scroll_up = "<c-u>",
      },

      -- Order in which which-key sorts entries.
      -- Available built-ins:
      -- `local`   -> buffer-local mappings first
      -- `order`   -> preserve plugin-defined order when relevant
      -- `group`   -> groups later in the list
      -- `alphanum`-> normal alphanumeric sort
      -- `mod`     -> modifier-heavy keys later
      -- `manual`  -> preserve manual insertion order
      -- `case`    -> lowercase before uppercase
      sort = { "local", "order", "group", "alphanum", "mod" },

      -- Auto-expand groups when they are small enough.
      -- `0` means "only expand groups with 0 or fewer children", i.e. effectively off.
      -- You can also provide a function if you want custom logic.
      expand = 0,

      replace = {
        key = {
          -- Format keys before display.
          -- This calls which-key's built-in pretty-printer.
          function(key)
            return require("which-key.view").format(key)
          end,

          -- Example of a manual replacement rule:
          -- turn `<Space>` into `SPC` in the UI.
          -- { "<Space>", "SPC" },
        },
        desc = {
          -- These are Lua pattern replacements applied to descriptions.
          -- In plain terms: strip noisy boilerplate from labels before display.
          { "<Plug>%(?(.*)%)?", "%1" },
          { "^%+", "" },
          { "<[cC]md>", "" },
          { "<[cC][rR]>", "" },
          { "<[sS]ilent>", "" },
          { "^lua%s+", "" },
          { "^call%s+", "" },
          { "^:%s*", "" },
        },
      },

      icons = {
        -- Breadcrumb shown in the command area for the current prefix chain.
        breadcrumb = "»",

        -- Separator between key and description.
        separator = "➜",

        -- Prefix shown for a group heading.
        group = "+",

        -- Used when content is truncated.
        ellipsis = "…",

        -- Master switch for showing mapping icons.
        mappings = true,

        -- Rule-based icons.
        -- `false` disables them entirely.
        -- `{}` means enabled but with the default rule set.
        rules = {},

        -- If true, try to use icon highlight colors from mini.icons.
        colors = true,

        -- Text/icon replacements used when formatting keys.
        keys = {
          Up = " ",
          Down = " ",
          Left = " ",
          Right = " ",
          C = "󰘴 ",
          M = "󰘵 ",
          D = "󰘳 ",
          S = "󰘶 ",
          CR = "󰌑 ",
          Esc = "󱊷 ",
          ScrollWheelDown = "󱕐 ",
          ScrollWheelUp = "󱕑 ",
          NL = "󰌑 ",
          BS = "󰁮",
          Space = "󱁐 ",
          Tab = "󰌒 ",
          F1 = "󱊫",
          F2 = "󱊬",
          F3 = "󱊭",
          F4 = "󱊮",
          F5 = "󱊯",
          F6 = "󱊰",
          F7 = "󱊱",
          F8 = "󱊲",
          F9 = "󱊳",
          F10 = "󱊴",
          F11 = "󱊵",
          F12 = "󱊶",
        },
      },

      -- Show a short help hint in the command line.
      show_help = true,

      -- Show the currently pressed key sequence in the command line.
      show_keys = true,

      disable = {
        -- Filetypes where which-key should not run.
        ft = {},

        -- Buffer types where which-key should not run.
        bt = {},
      },

      -- Write debug logs in the current directory.
      -- Only enable this if you are actively debugging which-key itself.
      debug = false,
    },
  },
}
```

## Plain-language walkthrough of the Lua shape

If you are coming from TypeScript, Go, C, Python, or Zig, the fastest useful mental model is:

- Lua has one real aggregate data structure: the table.
- A table is doing the work of an object, a dict/map, an array, and sometimes a namespace.
- Neovim config is mostly "build a table, pass it to some code".

So this:

```lua
opts = {
  preset = "helix",
  notify = true,
}
```

is basically the same idea as:

```ts
opts: {
  preset: "helix",
  notify: true,
}
```

or:

```python
opts = {
    "preset": "helix",
    "notify": True,
}
```

with one big difference: Lua uses the same `{}` syntax for both arrays and maps. The language decides which "shape" you mean based on how you fill it.

Examples:

```lua
{ "a", "b", "c" }            -- array-like table
{ x = 1, y = 2 }             -- map/object-like table
{ { "<auto>", mode = "nxso" } } -- nested tables; first element is an array item,
                                -- inside it, `mode = ...` is a named field
```

The other thing that looks weird at first is the amount of inline functions:

```lua
delay = function(ctx)
  return ctx.plugin and 0 or 200
end
```

That is just an anonymous function assigned to the `delay` field. You can read it as:

"when which-key wants the delay, call this function with `ctx`, and return either `0` or `200`".

The `a and b or c` expression is Lua's compact conditional idiom. If you hate it, that is reasonable. Read it like:

```text
ctx.plugin ? 0 : 200
```

in JS/C-style languages.

## Lua for people who can write a lexer in C but do not understand what the FUCK is the deal with this Lua shit

The main thing that makes Lua config feel slippery is that it is not trying to look like a "configuration language". It is just actual code, but with a tiny syntax and one overworked data structure. That means plugin configs often look like somebody jammed JSON, callbacks, and object literals into the same bag.

The good news is that Neovim config is usually much less magical than it first appears. Most of the time, a plugin author has defined "here is a table of options", and your job is just to fill in a few fields. You are not supposed to infer hidden semantics from punctuation. If you see:

```lua
return {
  {
    "folke/which-key.nvim",
    opts = {
      preset = "helix",
    },
  },
}
```

the useful reading is:

1. Return a list of plugin specs.
2. This spec targets `folke/which-key.nvim`.
3. Inside that, pass an options object with `preset = "helix"`.

That is it. The rest is framework convention from `lazy.nvim` and AstroNvim, not some deep Lua ceremony.

If you want one rule that will save you time: when a Neovim Lua config looks bizarre, reduce it to "what table is being built, and who consumes it?" Once you identify that, most of the confusion disappears.

For normal use, the much smaller version is still the right one:

```lua
return {
  {
    "folke/which-key.nvim",
    opts = {
      preset = "helix",
    },
  },
}
```

## References

- which-key README: <https://github.com/folke/which-key.nvim>
- which-key defaults (`preset = "classic"`): <https://github.com/folke/which-key.nvim/blob/main/lua/which-key/config.lua>
- LazyVim's which-key config (`preset = "helix"`): <https://github.com/LazyVim/LazyVim/blob/main/lua/lazyvim/plugins/editor.lua>
- AstroNvim plugin customization docs: <https://docs.astronvim.com/configuration/customizing_plugins/>
