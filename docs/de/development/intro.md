# Development Environments on NixOS

NixOS's reproducibility makes it ideal for building development environments. However, if you're used to other distros, you may encounter problems because NixOS has its own logic. We'll briefly explain this below.

In den folgenden Abschnitten werden wir darauf eingehen, wie die Entwicklungsumgebung in NixOS funktioniert

## Erstellen einer benutzerdefinierten Shell-Umgebung mit `nix shell`

Der einfachste Weg eine Entwicklungsumgebung zu erstellen ist es, `nix shell` zu verwenden. `nix shell` erstellt eine Shell-Umgebung, in der die gewünschten Nix-Paketen installiert sind.

Hier ein Beispiel:

```shell
# hello ist nicht verfügbar
› hello
hello: command not found

# Erstellen einer Umgebung mit den Paketen 'hello' and `cowsay`
› nix shell nixpkgs#hello nixpkgs#cowsay

# hello ist nun verfügbar
› hello
Hello, world!

# cowsay ist ebenfalls verfügbar
› cowsay "Hello, world!"
 _______________
< Hello, world! >
 ---------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

`nix shell` ist sehr hilfreich, wenn man ein paar Pakete nur mal ausprobieren oder schnell eine saubere Umgebung erstellen möchte.

## Erstellen einer Entwicklungsumgebung

Die `nix shell` ist einfach zu benutzen, aber sie ist nicht sehr flexibel. Für eine komplexere Entwicklungsumgebung müssen wir auf `pkgs.mkShell` und `nix develop` zurückgreifen.

Wir können eine Entwicklungsumgebung erstellen, indem wir `pkgs.mkshell { ... }` benutzen und eine interaktive Bash-Shell für diese Entwicklungsumgebung öffnen, indem wir `nix develop` benutzen.

Um zu sehen, wie `pkgs.mkShell` funktioniert, werfen wir einen Blick in dessen [Quellcode](https://github.com/NixOS/nixpkgs/blob/nixos-23.05/pkgs/build-support/mkshell/default.nix).

```nix
{ lib, stdenv, buildEnv }:

# A special kind of derivation that is only meant to be consumed by the
# nix-shell.
{ name ? "nix-shell"
, # a list of packages to add to the shell environment
  packages ? [ ]
, # propagate all the inputs from the given derivations
  inputsFrom ? [ ]
, buildInputs ? [ ]
, nativeBuildInputs ? [ ]
, propagatedBuildInputs ? [ ]
, propagatedNativeBuildInputs ? [ ]
, ...
}@attrs:
let
  mergeInputs = name:
    (attrs.${name} or [ ]) ++
    (lib.subtractLists inputsFrom (lib.flatten (lib.catAttrs name inputsFrom)));

  rest = builtins.removeAttrs attrs [
    "name"
    "packages"
    "inputsFrom"
    "buildInputs"
    "nativeBuildInputs"
    "propagatedBuildInputs"
    "propagatedNativeBuildInputs"
    "shellHook"
  ];
in

stdenv.mkDerivation ({
  inherit name;

  buildInputs = mergeInputs "buildInputs";
  nativeBuildInputs = packages ++ (mergeInputs "nativeBuildInputs");
  propagatedBuildInputs = mergeInputs "propagatedBuildInputs";
  propagatedNativeBuildInputs = mergeInputs "propagatedNativeBuildInputs";

  shellHook = lib.concatStringsSep "\n" (lib.catAttrs "shellHook"
    (lib.reverseList inputsFrom ++ [ attrs ]));

  phases = [ "buildPhase" ];

  # ......

  # when distributed building is enabled, prefer to build locally
  preferLocalBuild = true;
} // rest)
```

`pkgs.mkShell { ... }` ist eine spezielle Variation (Nix-Paket). Sein `name`, `buildInputs` und andere Parameter sind anpassbar, und `shellHook` ist ein spezieller Parameter, der ausgeführt wird, wenn `nix develop` die Umgebung betritt.

Hier ist eine `flake.nix`, die eine Entwicklungsumgebung mit installiertem Node.js 18 definiert:

```nix
{
  description = "A Nix-flake-based Node.js development environment";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-24.11";
  };

  outputs = { self , nixpkgs ,... }: let
    # system should match the system you are running on
    # system = "x86_64-linux";
    system = "x86_64-darwin";
  in {
    devShells."${system}".default = let
      pkgs = import nixpkgs {
        inherit system;
      };
    in pkgs.mkShell {
      # create an environment with nodejs_18, pnpm, and yarn
      packages = with pkgs; [
        nodejs_18
        nodePackages.pnpm
        (yarn.override { nodejs = nodejs_18; })
      ];

      shellHook = ''
        echo "node `${pkgs.nodejs}/bin/node --version`"
      '';
    };
  };
}
```

Erstelle einen leeren Ordner, speichere die Konfiguration oben als `flake.nix` und führe dann `nix develop` (oder genauer gesagt `nix develop .#default`) aus. Die aktuelle Version Node.js wird ausgegeben und man kann `node`, `pnpm` und `yarn` sofort verwenden.

## Using zsh/fish/... instead of bash

`pkgs.mkShell` benutzt standardmäßig `bash`, man kann aber auch `zsh` oder `fish` durch den Zusatz `exec <your-shell>` in `shellHook` benutzen.

Hier ein Beispiel:

```nix
{
  description = "A Nix-flake-based Node.js development environment";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-24.11";
  };

  outputs = { self , nixpkgs ,... }: let
    # system should match the system you are running on
    # system = "x86_64-linux";
    system = "x86_64-darwin";
  in {
    devShells."${system}".default = let
      pkgs = import nixpkgs {
        inherit system;
      };
    in pkgs.mkShell {
      # create an environment with nodejs_18, pnpm, and yarn
      packages = with pkgs; [
        nodejs_18
        nodePackages.pnpm
        (yarn.override { nodejs = nodejs_18; })
        nushell
      ];

      shellHook = ''
        echo "node `${pkgs.nodejs}/bin/node --version`"
        exec nu
      '';
    };
  };
}
```

Mit der obigen Konfiguration wird `nix develop` die REPL-Umgebung von nushell betreten.

## Eine Entwicklungsumgebung mit `pkgs.runCommand` erstellen

Die mit `pkgs.mkshell` erstellte Abwandlung kann nicht direkt benutzt werden, sie muss mittels `nix develop` betreten werden.

It is actually possible to create a shell wrapper containing the required packages via `pkgs.stdenv.mkDerivation`, which can then be run directly into the environment by
executing the wrapper.

Using `mkDerivation` directly is a bit cumbersome, and Nixpkgs provides some simpler functions to help us create such wrappers, such as `pkgs.runCommand`.

Beispiel:

```nix
{
  description = "A Nix-flake-based Node.js development environment";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-24.11";
  };

  outputs = { self , nixpkgs ,... }: let
    # system should match the system you are running on
    # system = "x86_64-linux";
    system = "x86_64-darwin";
  in {
    packages."${system}".dev = let
      pkgs = import nixpkgs {
        inherit system;
      };
      packages = with pkgs; [
          nodejs_20
          nodePackages.pnpm
          nushell
      ];
    in pkgs.runCommand "dev-shell" {
      # Dependencies that should exist in the runtime environment
      buildInputs = packages;
      # Dependencies that should only exist in the build environment
      nativeBuildInputs = [ pkgs.makeWrapper ];
    } ''
      mkdir -p $out/bin/
      ln -s ${pkgs.nushell}/bin/nu $out/bin/dev-shell
      wrapProgram $out/bin/dev-shell --prefix PATH : ${pkgs.lib.makeBinPath packages}
    '';
  };
}
```

Then execute `nix run .#dev` or `nix shell .#dev --command 'dev-shell'`, you will enter a nushell session, where you can use the `node` `pnpm` command normally, and the node version is 20.

The wrapper generated in this way is an executable file, which does not actually depend on the `nix run` or `nix shell` command.

For example, we can directly install this wrapper through NixOS's `environment.systemPackages`, and then execute it directly:

```nix
{pkgs, lib, ...}:{

  environment.systemPackages = [
    # Install the wrapper into the system
    (let
      packages = with pkgs; [
          nodejs_20
          nodePackages.pnpm
          nushell
      ];
    in pkgs.runCommand "dev-shell" {
      # Dependencies that should exist in the runtime environment
      buildInputs = packages;
      # Dependencies that should only exist in the build environment
      nativeBuildInputs = [ pkgs.makeWrapper ];
    } ''
      mkdir -p $out/bin/
      ln -s ${pkgs.nushell}/bin/nu $out/bin/dev-shell
      wrapProgram $out/bin/dev-shell --prefix PATH : ${pkgs.lib.makeBinPath packages}
    '')
  ];
}
```

Add the above configuration to any NixOS Module, then deploy it with
`sudo nixos-rebuild switch`, and you can enter the development environment directly with the `dev-shell` command, which is the special feature of `pkgs.runCommand` compared to `pkgs.mkShell`.

Related source code:

- [pkgs/build-support/trivial-builders/default.nix - runCommand](https://github.com/NixOS/nixpkgs/blob/nixos-24.11/pkgs/build-support/trivial-builders/default.nix#L21-L49)
- [pkgs/build-support/setup-hooks/make-wrapper.sh](https://github.com/NixOS/nixpkgs/blob/nixos-24.11/pkgs/build-support/setup-hooks/make-wrapper.sh)

## Enter the build environment of any Nix package

Now let's take a look at `nix develop`, first read the help document output by `nix develop --help`:

```
Name
    nix develop - run a bash shell that provides the build environment of a derivation

Synopsis
    nix develop [option...] installable
# ......
```

It tells us that `nix develop` accepts a parameter `installable`, which means that we can enter the development environment of any installable Nix package through it, not just the environment created by `pkgs.mkShell`.

By default, `nix develop` will try to use the following attributes in the flake outputs:

- `devShells.<system>.default`
- `packages.<system>.default`

If we use `nix develop /path/to/flake#<name>` to specify the flake package address and flake output name, then `nix develop` will try the following attributes in the flake outputs:

- `devShells.<system>.<name>`
- `packages.<system>.<name>`
- `legacyPackages.<system>.<name>`

Now let's try it out. First, test it to confirm that We don't have `c++` `g++` and other compilation-related commands in the current environment:

```shell
ryan in 🌐 aquamarine in ~
› c++
c++: command not found

ryan in 🌐 aquamarine in ~
› g++
g++: command not found
```

Then use `nix develop` to enter the build environment of the `hello` package in `nixpkgs`:

```shell
# login to the build environment of the package `hello`
ryan in 🌐 aquamarine in ~
› nix develop nixpkgs#hello

ryan in 🌐 aquamarine in ~ via ❄️  impure (hello-2.12.1-env)
› env | grep CXX
CXX=g++

ryan in 🌐 aquamarine in ~ via ❄️  impure (hello-2.12.1-env)
› c++ --version
g++ (GCC) 12.3.0
Copyright (C) 2022 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

ryan in 🌐 aquamarine in ~ via ❄️  impure (hello-2.12.1-env)
› g++ --version
g++ (GCC) 12.3.0
Copyright (C) 2022 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

We can see that the `CXX` environment variable have been set, and the `c++` `g++` and other commands can be used normally now.

In addition, we can also call every build phase of the `hello` package normally:

> The default execution order of all build phases of a Nix package is:
> `$prePhases unpackPhase patchPhase $preConfigurePhases configurePhase $preBuildPhases buildPhase checkPhase $preInstallPhases installPhase fixupPhase installCheckPhase $preDistPhases distPhase $postPhases`

```shell
# unpack source code
ryan in 🌐 aquamarine in /tmp/xxx via ❄️  impure (hello-2.12.1-env)
› unpackPhase
unpacking source archive /nix/store/pa10z4ngm0g83kx9mssrqzz30s84vq7k-hello-2.12.1.tar.gz
source root is hello-2.12.1
setting SOURCE_DATE_EPOCH to timestamp 1653865426 of file hello-2.12.1/ChangeLog

ryan in 🌐 aquamarine in /tmp/xxx via ❄️  impure (hello-2.12.1-env)
› ls
hello-2.12.1

ryan in 🌐 aquamarine in /tmp/xxx via ❄️  impure (hello-2.12.1-env)
› cd hello-2.12.1/

# generate Makefile
ryan in 🌐 aquamarine in /tmp/xxx/hello-2.12.1 via ❄️  impure (hello-2.12.1-env)
› configurePhase
configure flags: --prefix=/tmp/xxx/outputs/out --prefix=/tmp/xxx/outputs/out
checking for a BSD-compatible install... /nix/store/02dr9ymdqpkb75vf0v1z2l91z2q3izy9-coreutils-9.3/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /nix/store/02dr9ymdqpkb75vf0v1z2l91z2q3izy9-coreutils-9.3/bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for gcc... gcc
# ......
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating po/Makefile.in
config.status: creating config.h
config.status: config.h is unchanged
config.status: executing depfiles commands
config.status: executing po-directories commands
config.status: creating po/POTFILES
config.status: creating po/Makefile

# build the package
ryan in 🌐 aquamarine in /tmp/xxx/hello-2.12.1 via C v12.3.0-gcc via ❄️  impure (hello-2.12.1-env) took 2s
› buildPhase
build flags: SHELL=/run/current-system/sw/bin/bash
make  all-recursive
make[1]: Entering directory '/tmp/xxx/hello-2.12.1'
# ......
ranlib lib/libhello.a
gcc  -g -O2   -o hello src/hello.o  ./lib/libhello.a
make[2]: Leaving directory '/tmp/xxx/hello-2.12.1'
make[1]: Leaving directory '/tmp/xxx/hello-2.12.1'

# run the built program
ryan in 🌐 aquamarine in /tmp/xxx/hello-2.12.1 via C v12.3.0-gcc via ❄️  impure (hello-2.12.1-env)
› ./hello
Hello, world!
```

This usage is mainly used to debug the build process of a Nix package, or to execute some commands in the build environment of a Nix package.

## `nix build`

The `nix build` command is used to build a software package and creates a symbolic link named `result` in the current directory, which points to the build result.

Here's an example:

```bash
# Build the package 'ponysay' from the 'nixpkgs' flake
nix build "nixpkgs#ponysay"
# Use the built 'ponysay' command
› ./result/bin/ponysay 'hey buddy!'
 ____________
< hey buddy! >
 ------------
     \
      \
       \
       ▄▄  ▄▄ ▄ ▄
    ▀▄▄▄█▄▄▄▄▄█▄▄▄
   ▀▄███▄▄██▄██▄▄██
  ▄██▄███▄▄██▄▄▄█▄██
 █▄█▄██▄█████████▄██
  ▄▄█▄█▄▄▄▄▄████████
 ▀▀▀▄█▄█▄█▄▄▄▄▄█████         ▄   ▄
    ▀▄████▄▄▄█▄█▄▄██       ▄▄▄▄▄█▄▄▄
    █▄██▄▄▄▄███▄▄▄██    ▄▄▄▄▄▄▄▄▄█▄▄
    ▀▄▄██████▄▄▄████    █████████████
       ▀▀▀▀▀█████▄▄ ▄▄▄▄▄▄▄▄▄▄██▄█▄▄▀
            ██▄███▄▄▄▄█▄▄▀  ███▄█▄▄▄█▀
            █▄██▄▄▄▄▄████   ███████▄██
            █▄███▄▄█████    ▀███▄█████▄
            ██████▀▄▄▄█▄█    █▄██▄▄█▄█▄
           ███████ ███████   ▀████▄████
           ▀▀█▄▄▄▀ ▀▀█▄▄▄▀     ▀██▄▄██▀█
                                ▀  ▀▀█
```

## Using `nix profile` to manage development environments and entertainment environments

`nix develop` is a tool for creating and managing multiple user environments, and switch to different environments when needed.

Unlike `nix develop`, `nix profile` manages the user's system environment, instead of creating a temporary shell environment. So it's more compatible with Jetbrains IDE / VSCode and other IDEs, and won't have the problem of not being able to use the configured development environment in the IDE.

TODO

## Other Commands

There are other commands like `nix flake init`, which you can explore in [New Nix
Commands][New Nix Commands]. For more detailed information, please refer to the
documentation.

## References

- [pkgs.mkShell - nixpkgs manual](https://nixos.org/manual/nixpkgs/stable/#sec-pkgs-mkShell)
- [A minimal nix-shell](https://fzakaria.com/2021/08/02/a-minimal-nix-shell.html)
- [Wrapping packages - NixOS Cookbook](https://wiki.nixos.org/wiki/Nix_Cookbook#Wrapping_packages)
- [One too many shell, Clearing up with nix' shells nix shell and nix-shell - Yannik Sander](https://blog.ysndr.de/posts/guides/2021-12-01-nix-shells/)
- [Shell Scripts - NixOS Wiki](https://wiki.nixos.org/wiki/Shell_Scripts)

[New Nix Commands]: https://nixos.org/manual/nix/stable/command-ref/new-cli/nix.html
