---
title: "You’ve Been Using the WRONG LSP for C# in Neovim"
date: 2025-10-17
categories: 
  - "net"
  - "programming"
  - "neovim"
  - "vim"
tags: 
  - "lsp"
  - "neovim"
  - "vim"
  - "csharp"
---

More and more .NET developers are starting to notice the advantages of using Neovim as their main code editor. But let’s be honest most of us don’t want to waste time configuring endless settings. We want the things and we wanted now. I mean, we just want a great editor, an enjoyable coding experience, and maximum efficiency.

![](images/samurai.jpg)

That’s why most developers choose the most straightforward way to set up their LSP inside Neovim: Omnisharp LSP.

Although the OmniSharp LSP has been good enough for developing inside Neovim, it has never quite matched the experience of the LSP used by VS Code or Visual Studio.
However, things have changed, now you can use Roslyn LSP, which provides the same core functionality offered by VS Code and Visual Studio.

In this article I show you how to install it in just four steps. We’ll need the Lazy package manager and a basic understanding of how to install a plugin, which you can find here:

https://lazy.folke.io/installation

## 1. Install Mason

Mason is a plugin that allows you to easily install the LSPs you need.
To install it, you just need to add the plugin:

``` lua
{
    "mason-org/mason.nvim",
     opts = {}
}
```

Unfortunately, there is still no official mason register for Roslyn, but there are few non oficial registers that we can use.

We need to modify the previous code and set it up like this:

``` lua 
{
    "mason-org/mason.nvim",
    config = function()
        require('mason').setup({
            registries = {
                'github:Crashdummyy/mason-registry', -- this contains the register for Roslyn
                'github:mason-org/mason-registry', 
            },
        })
    end
}
```
  
## 2. Install Roslyn LSP

Inside Neovim, you just need to execute this command in normal mode:

``` bash
:MasonInstall Roslyn
```

## 3. Install seblyng/roslyn.nvim plugin

Right now seblying plugin is the best alternative to avoid writting configuration. You just need to add this plugin

``` lua
{
    "seblyng/roslyn.nvim",
    enabled = true,
    ft = "cs",
    config = function()
        vim.lsp.config("roslyn", {
            on_attach = function()
                print("This will run when the server attaches!")
            end,
            settings = {
              -- Check settings in https://github.com/seblyng/roslyn.nvim
            },

        })
        local roslyn = require("roslyn");
        roslyn.setup();
    end
}
```

## 4. Check your configuration

Lets create a proyect to test the configuration.

``` bash
dotnet new console --name roslyn
```

Now enter enter to neovim

``` bash
nvim roslyn/Program.cs
``` 

Check if lsp is enabled running this command

``` bash
:LspInfo
``` 

You should see something like 

``` bash
vim.lsp: Active Clients ~
- roslyn (id: 1)
  - Version: ? (no serverInfo.version response)
  - Root directory: ~/roslyn
  - Command: { "roslyn", "--logLevel=Information", "--extensionLogDirectory=/home/ricardo/.local/state/nvim", "--stdio" }
  - Settings: {}
  - Attached buffers: 1

``` 

**Links**

- Github: [https://github.com/FractalCodeRicardo](https://github.com/FractalCodeRicardo)

- Medium: [https://medium.com/@nosilverbullet](https://medium.com/@nosilverbullet)

- Web page: [https://programmingheadache.com](https://programmingheadache.com/)

- Youtube: [https://www.youtube.com/@ProgrammingHeadache](https://www.youtube.com/@ProgrammingHeadache)

- Buy me a Coffee: [https://buymeacoffee.com/programmingheadache](https://buymeacoffee.com/programmingheadache)


