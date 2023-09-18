译文：[Learn How To Use NeoVim As an IDE](https://programmingpercy.tech/blog/learn-how-to-use-neovim-as-ide/)

Many years ago I tried NeoVim, I was horrified by the amount of setup needed and I ran.

I ran to the comfor of my regular VS Code. An IDE that worked, looked good and had the most features I wanted.

Now, I thought I was effective in VS Code, but then I decided to give NeoVim a try again. 

Some might ask, why spend a few hours setting up an IDE when you can have a complete working one in a minute, I agree. But let's face it if we are going to work daily in our IDE, a few hours to configure is not really a bad thing.

The reason to use NeoVim is that it is super customizable, you can really make this editor look the way you want, and make it behave the way you want.


The con of NeoVIm is also the customization abilities, it requires some time to set up and modify in the fure. But once you know how to, it isn't really that hard or time-consuming, for the ability to add ANY feature that you want.

You can also watch this artcile on YouTube.

This tutorial won't hand you a perfect IDE configuration for NeoVim, for that you can google after people's dotfiles. This tutorial aims to help you understand NeoVim and learn how to manage it, allowing you to build whatever you want after. I won't expect any previous knowledge in either Vi or NeoVim. You want what they way.

Hand a delevoper an IDE, and he will code.

Hand a developer the ability to customize in his IDE, and he will be a god.

## Installing NeoVim And Packer The Plugin Manager

Begin by installing NeoVim by following the instructions on GitHub. I don't recommend using apt-get install if you are using ubuntu as it is an old version.

Move the unpacked folder somewhere you store your software, or add the binary found at `./nvim-linx64/bin/nvim` to your $PATH.

You can then open it up by running `nvim`.

```bash
nvim
```

Doing this will open up NeoVim and a splash screen. Remember, NeoVim is more of a terminal instead of a regular editor with a fancy UI.

You're terminal should look like the following:

NeoVim is nothing special at all if we don't start adding a few plugins. This is the part that scares most people away when they first using NeoVim.

You can now NeoVim by pressing `SHIFT+:` when opens the command prompt and then write q in the tab that appears at the bottom, and press enter.

Press `SHIFT+:` will open up the command tab, which allows us to pass in commands.

We read a Plugin Manager, a popular one that will use is `Packer`.

We can install it using a one-liner in bash, it will fetch the repository and place it at the `~/.local/share/nvim` path that is the second parameter. Don't modify that path unless you know what you're doing, it is using default path to something named PackPath in Vim/NeoVim.

Now that we have Packer installed we need to make a frist visit to the Plugin configuration file.

ALl configurations go into an LUA file which is by default stored at `~./config/nvim/lua/`. At that location, we can specify configuration files.


## Write The First Configuration & Importing Plugins & Navigating

Everything in NeoVim is almost handled by a Plugin, for instance, for instance, if we are going to use NeoVim as an IDE we will need Language Servers to understand code.

We will go head and create the first configuration file, which will be `~/.config/nvim/lua/plugins.lua`.

If the folder does not exist then create it and create the file by opening it up with NeoVim.

```bash
mkdir -p ~/.config/nvim/lua
nvim ~/.config/nvim/lua/plugins.lua
```

Creating the configuration folder.

Packer can be imported int he Lua file by passing the name to the `require` function. We will then run a function on that called `startup` which will be performed on startup on NeoVim, which means we add the plugins and capabilities whenever NeoVim is started.

Enter Insert Mode by pressing `Insert` or i and then add the following:

```lua
return require('packer').startup(function(use))
  -- configurations will go here soon.
  use 'wbthomason/packer.nvim'
end
```

Adding packer as a Plugin.

What we do is that we are making Packer run on start, we then import all the plugins we want to use, and we include the packer plugin also. This allows Packer to self-manage itself and update etc.

Press `ESC` to leave Insert Mode, Save the file by pressing opening the command tab `SHIFT+:` and passing in `w` which is short for write.

If your restart NeoVim now, nothing will have changed, bummer huh?

We need to also tell NeoVim to find this configuration file and load it.

This can be done inside `~/.config/nvim/init.lua` which is a second configuration file, this file is executed by NeoVim when started up, we will require our configuration scripts inside Lua from here.

Open the `plugins.lua` file again, and let us learn how to navigate (for now) inside NeoVim so we don't need to jump out all the time.

Once NeoVim is opened, type in the command `Explore` which will open up a text-based explorer for us. It is called Netrw, and I won't go into details, but it can even fetch remote file, etc.

From here, you can see the files in the current directory, and also the default Linux indicators `./` and `../`. A single dot is a current director, two dots is the parent directory.

We want to create the `init.lua` file in the parent directory, so press enter once you select `../` and then press `SHIFT+%`. Shift+% will open an input bar at the bottom asking for a file name, enter `init.lua`. It will open the file after creating it.

We will add a single line to it, or create the file if it does not exist. That line will make sure plugins are added.

```
require("plugins")
```

Loading in the `plugins.lua` configuration.


Save using the `w` command and restart NeoVim and we should have `Packer` installed, which we can control by running commands.

If `Packer` was installed correctly we should be able to use its commands, one of them is `PackerStatus` which prints a list of all used Plugins.

In Short what we have done so far.

- Learned how we can use `Packer` for importing Plugins, `~/.config/nvim/lua/plugins.lua` whichi is related to that.
- `~/.config/nvim/init.lua` allowed use to tell NeoVim to respect our Packer file upon startup.
- Running commands using `SHIFT+:` such as `:PackerStatus`
- Learned how to navigate using the `Explore` command.


