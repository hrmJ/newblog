---
title: "A basic Neovim + local llm setup"
datePublished: Fri Nov 15 2024 09:06:30 GMT+0000 (Coordinated Universal Time)
cuid: cm3iilmau000509lg8t54hlzc
slug: a-basic-neovim-local-llm-setup
tags: neovim, ai-tools

---

Ok, here we go: wanted to play around with AI editing a bit even though I am not using ai-powered editing in any serious work. Here's a quick working neovim setup with [avante.nvim](https://github.com/yetone/avante.nvim) and [ollama](https://ollama.com/). So this is a local llm which could potentially make it more likely to be able to actually use it even in proprietary work.

I have a first generation M1 mac and it seems to be quite enough for this kind of setup.

Here's a sneak peak into the avante.nvim-powered editor: I selected my current line in visual mode, pressed `leader+ae` and asked for an edit

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731661204356/426cd1fc-e8d1-48ce-87b4-086a81238a40.png align="center")

pressed `C-s` and my code was edited in a second or so:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731661281367/b5833bbf-0c52-4ed0-a661-1af2b68fb2ba.png align="center")

Here’s another task, pressing `leader + aa` and asking for unit tests:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731660812930/f10dfdee-9aef-4d56-bfc8-694eba88e08a.png align="center")

Now here’s a quick walkthrough of the basic setup.

## Setting up ollama and downloading models

Nothing complicated here: just download the installer from ollama.com. Download and test a model by running `ollama run codegemma` or any other model name.

## Setting up Neovim

I actually tried a couple of ai-integrating options for nvim. The ones that worked where

1. https://github.com/David-Kunz/gen.nvim (although I occasianlly had a hard time getting the right amount of actual code without too many markdown explanations)
    
2. https://github.com/bernardo-bruning/ollama-copilot along with https://github.com/github/copilot.vim . A bit hacky: had to dig into copilot.nvim's source to disable the check for openapi credentials and force it to proxy all requests to my local ai instead of trying to reach for the cloud. However: this setup *did* also work and I used copilot.vim with ollama instead of the actual copilot
    
3. https://github.com/yetone/avante.nvim - an actually useful experience. Although this one moves rather rapidly and the docs aren't completely up to date. So, again, had to do somer source-code-digging to [find out that a parameter name for custom models has changed](https://github.com/yetone/avante.nvim/issues/811).
    

Here's my avante.nvim config. Note, that this is using [lazy.nvim](https://github.com/folke/lazy.nvim) as the plugin manager.

```lua
  {
    'yetone/avante.nvim',
    event = 'VeryLazy',
    lazy = false,
    version = false, -- set this if you want to always pull the latest change
    opts = {
      provider = 'ollama',
      vendors = {
        ---@type AvanteProvider
        ollama = {
          ['local'] = true,
          endpoint = '127.0.0.1:11434/v1',
          model = 'codegemma',
          parse_response_data = function(data_stream, event_state, opts)
            require('avante.providers').copilot.parse_response(data_stream, event_state, opts)
          end,
          parse_curl_args = function(opts, code_opts)
            return {
              url = opts.endpoint .. '/chat/completions',
              headers = {
                ['Accept'] = 'application/json',
                ['Content-Type'] = 'application/json',
              },
              body = {
                model = opts.model,
                messages = require('avante.providers').copilot.parse_messages(code_opts), -- you can make your own message, but this is very advanced
                max_tokens = 2048,
                stream = true,
              },
            }
          end,
        },
      },
    },
```

I changed `parse_response` to `parse_response_data` in contrast to the example in the project's wiki to get this to work. Might need to come back to that if this part of the project's initialization logic changes in the future.

Here's the rest of the config, just to be complete, although this is just copied from the project's README:

```lua
    -- if you want to build from source then do `make BUILD_FROM_SOURCE=true`
    build = 'make',
    -- build = "powershell -ExecutionPolicy Bypass -File Build.ps1 -BuildFromSource false" -- for windows
    dependencies = {
      'nvim-treesitter/nvim-treesitter',
      'stevearc/dressing.nvim',
      'nvim-lua/plenary.nvim',
      'MunifTanjim/nui.nvim',
      --- The below dependencies are optional,
      'nvim-tree/nvim-web-devicons', -- or echasnovski/mini.icons
      'zbirenbaum/copilot.lua', -- for providers='copilot'
      {
        -- support for image pasting
        'HakonHarnes/img-clip.nvim',
        event = 'VeryLazy',
        opts = {
          -- recommended settings
          default = {
            embed_image_as_base64 = false,
            prompt_for_file_name = false,
            drag_and_drop = {
              insert_mode = true,
            },
            -- required for Windows users
            use_absolute_path = true,
          },
        },
      },
      {
        -- Make sure to set this up properly if you have lazy=true
        'MeanderingProgrammer/render-markdown.nvim',
        opts = {
          file_types = { 'markdown', 'Avante' },
        },
        ft = { 'markdown', 'Avante' },
      },
    },
```

## Conclusions

As I said in the beginning, I have not yet tried this with anything besides toy projects, but the bottom line for me is that it seems to be perfectly possible to have a very usable ai-enhanced nvim experience - even with local models and an M1 mac!