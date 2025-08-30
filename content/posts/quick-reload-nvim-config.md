---
title: Quickly Reload Your Neovim Config
date: 2024-11-18
---

Tired of quitting and restarting Neovim every time you tweak your config? There's a better way!

By separating your plugin declarations and configurations, you can quickly reload changes without the hassle.

The key is to put your plugin declarations (everything needed to load the plugin) in one file, and the actual configuration in another.

For example, let's look at the [zen-mode](https://github.com/folke/zen-mode.nvim) plugin:

Your `lua/custom/plugin/zen-mode.lua` file would contain the plugin declaration:

```lua
return {
  "folke/zen-mode.nvim",
  config = function()
    -- Load in the config for this plugin
    -- from a different file
    require "custom.zen-mode"
  end,
}
```

Then, the configuration itself goes in `lua/custom/zen-mode.lua`:

```lua
local zen = require("zen-mode")
zen.setup({
    window = {
        width = 0.65,
    },
})

vim.keymap.set("n", "<leader>zm", function()
    zen.toggle()
end, {})
```

Now, whenever you want to tweak your zen-mode settings, you can simply run `:source %` or `:so` to re-execute the config file, without having to quit Neovim.

This setup is a real time-saver, especially when you're dialing in your plugin preferences. No more opening and closing Neovim over and over!

## Links

https://www.youtube.com/watch?v=kJVqxFnhIuw

[[neovim]] [[lua]] [[vim]]
