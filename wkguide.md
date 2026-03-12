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

## References

- which-key README: <https://github.com/folke/which-key.nvim>
- which-key defaults (`preset = "classic"`): <https://github.com/folke/which-key.nvim/blob/main/lua/which-key/config.lua>
- LazyVim's which-key config (`preset = "helix"`): <https://github.com/LazyVim/LazyVim/blob/main/lua/lazyvim/plugins/editor.lua>
- AstroNvim plugin customization docs: <https://docs.astronvim.com/configuration/customizing_plugins/>
