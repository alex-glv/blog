+++
date = "2016-03-18"
draft = false
title = "Developing Golang in NixOS"
description = "A short guide on develop Golang on NixOS"

+++


# Why NixOS?

The idea of fully reproducible system fascinates me a lot.
Part of Getting Things Done philosophy is not to contaminate your head with the little things you have to do, you put them down on a list.

That's why we use password managers, because you don't want to keep all those password from a bazillion websites in your head - you trust something else to keep them.
With you development environment it's the same - you build your custom machine and when your drive goes bust, you go: "Oh, shit.. When's my latest backup, again?"

That's why you check in your OS configuration.
NixOS eliminates this issue 99% so you have even less to worry about.

Take a look, [this is my OS](https://github.com/alex-glv/nixos-conf) - all the software I use, my user user account, services - everything.
The only thing that I need to do to fully replicate my system on a machine is pull the config, and run ```nixos-install```.

This is great and all, but where's the catch?

# The Catch
NixOS comes with the Nix package manager and their its flavored language to describe those packages.
You most likely have never heard of Nix, it's not arch or OSX with Homebrew and all the packages imaginable.
So, you'll have to write some packages yourself.

# Golang - dependencies

Nix package manager is also capable of handling go dependencies, which is great, because it's a decent replacement for ```go get```. [1]
However, if you want to quickly hack on a CLI tool, it gets a bit hairy: try to figure if the packages are present in the repository, if not then you have to write them yourself. 
It's a bit too slow and I am not very experienced with that yet.
So, I prefer to use godep instead. Luckily it's easy to obtain with the usual ```nix-env -i godep```

# GOROOT and GOPATH
I store my sources in a home directory, which means I also need to have $GOROOT exported to allow go tools like gofmt and gorcode to locate the standard library sources.
Luckily, adding extra variables to your environment is very easy in nix:

Add go to your systemPackages list under your configuration.nix [2] file:

```
environment.systemPackages = with pkgs; [
  go
  // other pkgs
];
```

Then, add the extra environment variable:

```
environment.variables = { GOROOT = [ "${pkgs.go.out}/share/go" ]; };
```

This ensures your GOROOT will always point to standard library sources directory.
Adding GOPATH is very similar, however I tend to store it in my bashrc somewher under dotfiles.

# Balance
I found a good compromise between reproducible system and convenience to use it.
You can have godep and gocode fully working with the necessary environment variables set, and also have these settings stored under nix configuration file.

# Footnotes
* [Developing in golang with nix package](http://lethalman.blogspot.nl/2015/02/developing-in-golang-with-nix-package.html)
* [NixOS manual: changing configuration](https://nixos.org/nixos/manual/index.html#sec-changing-config)
