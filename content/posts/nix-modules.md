+++
title = "Some Neat Nix Modules"
date = "2025-06-28T20:23:52-05:00"
author = "Taylor Font"
authorTwitter = ""
cover = ""
coverCaption = ""
tags = ["nix", "neovim", "dotfiles", "home-manager", "nvf", "hjem"]
keywords = ["nix", "neovim", "dotfiles", "modules", "nvf", "hjem"]
description = ""
showFullContent = false
readingTime = true
hideComments = false
color = ""
+++

I have been using NixOS for about a year now. My daily driver is a Mac Mini M4,
which I manage with `nix` (the package manager). I also have several NixOS
machines that host my homelab, and a NixOS cirtual machine that I have started
to use as my own fully-customized development environment.

All that said, the majority of my time using Nix & NixOS has involved minimal
exploration of modules. I relied heavily on the use of the usual suspects:
`home-manager` for manaing my user packages and dotfiles and `nix-darwin` for
declarative management of my MacOS environment. I never found an ideal
replacement for my Neovim configurations, so I relied on the trusty
[`mkOutOfStoreSymlink`](https://github.com/nix-community/home-manager/blob/76d0c31fce2aa0c71409de953e2f9113acd5b656/modules/files.nix#L76-L119).

More recently, I have been exploring more modules and alternative approaches to
managing my nix environments, and have been pleasantly surprised by what I have
found out in the community.

## For the Neovim User - `nvf`

### The problem

First, let's address the ugly duckling in my configurations - Neovim. While I
could have opted for management via other modules like
[`nixvim`](https://github.com/nix-community/nixvim), I always found them to be
lacking for one reason or another.

Here's a few of the common headaches I always ran into when attempting to use
other modules:

- **Lack of flexibility.** It can be quite tedious (and sometimes downright
  frustrating) to hunt down if certain options that were easily accessible in
  traditional environments via Lua were exposed in Nix module options. Writing
  overlays of the module just to enable a single feature was just not for me. If
  that's your truth, I respect your dedication.
- **Multiple configuration languages.** Most modules that attept to avoid the
  flexibility issue above opt for management of configuration in Lua directly.
  This is certainly more flexible, but then my configurations were all written
  in Nix language _except_ Neovim, which I managed in Lua seperately. I found
  the context switching was quite annoying, and simply wrapping Lua files felt
  far less "magical" than the Nix experience is with other configurations.
- **Dependencies and portability.** Many of the modules also required the use of
  `home-manager` or other global binaries, which was not what I was looking for.
  I wanted to be able to configure my Neovim without needing to manually pull in
  different modules just to expose options or have dependencies available.

### The solution

Enter [`nvf`](https://github.com/NotAShelf/nvf) by the amazingly talented raf
(aka NotAShelf). `nvf` manages your Neovim configurations in Nix language, but
allows for the use of raw Lua when it is needed via the
[`mkLuaInline`](https://nixos.org/manual/nixpkgs/unstable/#function-library-lib.generators.mkLuaInline)
generator. It is available as a standalone Nix module (or as a Home-Manager
module, for those who wish to stick with Home-Manager; though be sure to read
[the next section](#for-the-home-manager-hater---hjem) for an up-and-coming
module which may change your mind!).

There's also a very comprehnsive list of existing support for Neovim plugins
from maintainers as well as the community - with more plugins and options being
added all the time. However, `nvf` also provides the ability to bring your own
plugins and modules from npins or nixpkgs with ease, meaning you are never
dependent on the option being exposed in the module or needing to write overlays
for that _one_ perfect configuration.

Documentation is another pervasive problem in the Nix community; paradigms are
changing frequently, and often documentation is not caught up with what is
proper that week. I was pleasantly surprised how well-documented the module was,
and in just an hour I had a fully functional Neovim configuration up and
running! The documentation is built out of the module directly, so there's never
problems with things being out of date or broken when trying to RTFM.

finally - let's talk about contributions. I found myself in a very niche
situation one day where I needed to configure the placement of my diagnostic
icons in neotree, but found the option did not exist. The contribution
guidelines for `nvf` are also excellent, and I was able to quickly get new
customizations implemented into the module for others to use.

### Try it out

If my little plug has you curious, you can run `nvf` without needing to modify
your configurations at all with the help of the `nix run` command:

```bash
nix run github:notashelf/nvf
```

To see just how much `nvf` can do, try running the maximal version. It'll take a
little longer to run, but you'll see many more features for a super-powered
Neovim:

```bash
nix run github:notashelf/nvf#maximal
```

My personal configurations fall somewhere between the default and maximal
examples. You can view my `nvf` configurations in my
[dotfiles](https://github.com/taylrfnt/dotfiles/tree/main/modules/nvf).

## For the Home-Manager Hater - `hjem`

Let's talk about Home-Manager. It is one of the largest and most popular modules
in the Nix ecosystem. If you are just starting to use Nix or NixOS, one of the
first recommendations you will probably come across is to use Home-Manager.

I want to first start off by saying that Home-Manager is fantastic for
beginners, and exposes a massive number of options for various packages that
people use every single day. There is a point in time in which every single host
I configured in my flake used Home-Manager. I do not think Home-Manager is a bad
module, but I think it is a bad module for me and my preferred configurations.

As your experience and comfortability with your Nix configuration improve,
you'll begin to notice how much your system evaluation & build times seem to be
slower and slower. If you've tried to contribute to Home-Manager (which I
admittedly have not, but I do chat regularly with folks that do), you might have
found their methods bloated and more convuluted than necessary to manage
something like a simple configuration file.

Between the bloated module, slower rebuild times, and difficult to customize
options, I simply had enough. I saw a new module called `hjem` mentioned in a
Nix thread on the Ghostty Discord server, and I immediately dove in.

`hjem` is a far cry from Home-Manager - it uses native syntax for your
configuration files (whatever that is for your package - TOML, YAML, or any
other language) and links them with `systemd-tmpfiles`. There's work ongoing to
use new linkers for better non-NixOS support (such as
[`smfh`](https://github.com/feel-co/smfh)).

I know I just got finished singing the praises of `nvf` for not requiring Lua,
the effectively "native" configuration language for Neovim, and now I am singing
the opposite tune - using native configuration syntax for _all_ other packages.
I'm not sure if it is simply my own exhaustion with trying to hunt for specific
options exposed in Home-Manager for years every time I want to change my
configuration, my personal dislike of Lua, or just simply rose-colored glasses,
but I find it refreshing to have what is effectively a Nix-ified `stow` system
for my configurations.

I can read directly from the documentation for any package, update my
configuration file and rebuild, and that file is immediately in my store. No
more needing to consult Home-Manager documentation or thinking about writing
overlays. Speaking of rebuilds, I noticed my system rebuild time dropped
dramatically (almost ~2x faster) when I removed Home-Manager.

I will caveat my overwhelmingly positive review of `hjem` by saying I do still
daily drive a macOS system, and `hjem` does not currently support macOS (there's
no `systemd` to use as a linker). I am currently stuck using Home-Manager for my
macOS system until the new linker(s) make their way into the light of day.

If you're curious about `hjem`, you can check it out
[here](https://github.com/feel-co/hjem).

::::::::::

That's all I have for now! Hopefully my experience on these modules have at
least given you something to think about for your own Nix configurations.
Cheers!
