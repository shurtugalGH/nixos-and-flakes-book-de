![](/nixos-and-flakes-book.webp)

# Einführung in Nix und NixOS

Nix ist ein deklarativer Paketmanager, der es dem Benutzer ermöglicht den 
gewünschten Systemzustand in Konfigurationsdateien zu deklarieren (deklarative 
Konfiguration), und übernimmt die Verantwortung diesen Zustand zu erreichen.

>Einfach gesagt meint "deklarative Konfiguration", dass der Benutzer nur den 
gewünschten Zustand des Systems beschreibt.  
Wenn du zum Beispiel eklarierst, dass du den i3-Fenstermanager durch sway 
ersetzen möchtest, wird Nix dich dabei unterstüzen diese Ziel zu erreichen. Du 
musst dir keine Gedanken über tiefliegende Details machen, wie zum Beispiel 
welche Pakete musst du installieren, welche i3-Pakete müssen deinstalliert 
werden, oder die nötigen Anpassungen in der Systemkontiguration und 
Umgebungsvariablen für sway.  
Nix kümmert sich automatisch um diese Details für den Benutzer (vorausgesetzt, 
die Nix-Pakete für sway und i3 sind richtig beschrieben).

NixOS, eine Linuxdistribution, welche auf dem Nix-Paketmanager aufbaut, kann als 
"Betriebssystem als Code" beschrieben werden. Es enthält deklarative 
Nix-Konfigurationsdateien, um den gesammten Zustand des Betriebssystems zu 
beschreiben.

Ein Betriebssystem besteht aus verschiedenen Softwarepaketen, Konfigurations-, 
Text- und Binärdateien, welche alle den aktuellen Zustand des Systems darstellen. 
Die deklarative Konfiguration kann nur den statischen Teil dieses Zustands 
verwalten.  
Dynamische Daten (wie PostgreSQL-, MySQL- oder MongoDB-Daten) können nicht 
effektiv durch deklarative Konfiguration verwaltet werden (es ist nicht möglich, 
alle neuen PostgreSQL-Daten, die nicht in der Konfiguration deklariert sind, bei 
jeder Bereitstellung zu löschen). **Deshalb konzentriert sich NixOS darauf den 
statischen Teil des Systemszustands deklarativ zu verwalten**. Dynamische Daten 
sowie die Inhalte im Home-Verzeichnis des Benutzers bleiben von NixOS beim 
Rollback auf eine frühere Generation unberührt.

Although we cannot achieve complete system reproducibility, the `/home` directory, being an important user directory, contains many necessary configuration files - [Dotfiles](https://wiki.archlinux.org/title/Dotfiles). A significant community project called [home-manager](https://github.com/nix-community/home-manager) is designed to manage user-level packages and configuration files within the user's home directory.

Due to Nix's features, such as being declarative and reproducible, Nix is not limited to managing desktop environments but is also extensively used for managing development environments, compilation environments, cloud virtual machines, and container image construction. [NixOps](https://github.com/NixOS/nixops) (an official Nix project) and [colmena](https://github.com/zhaofengli/colmena) (a community project) are both operational tools based on Nix.

## Warum NixOS?

I first learned about the Nix package manager several years ago. It utilizes the Nix language to describe system configuration. NixOS, the Linux distribution built on top of it, allows for rolling back the system to any previous state (although only the state declared in Nix configuration files can be rolled back). While it sounded impressive, I found it troublesome to learn a new language and write code to install packages, so I didn't pursue it at the time.

However, I recently encountered numerous environmental issues while using EndeavourOS, and resolving them consumed a significant amount of my energy, leaving me exhausted. Upon careful consideration, I realized that the lack of version control and rollback mechanisms in EndeavourOS prevented me from restoring the system when problems arose.

Das war der Moment, in dem ich mich dazu entschied zu NixOS zu wechseln.

To my delight, NixOS has exceeded my expectations. The most astonishing aspect is that I can now restore my entire i3 environment and all my commonly used packages on a fresh NixOS host with just one command `sudo nixos-rebuild switch --flake .`. It's truly fantastic!

The rollback capability and reproducibility of NixOS has instilled a great deal of
confidence in me—I no longer fear breaking the system. I've even ventured into experimenting with new things on NixOS, such as the hyprland compositor. Previously, on EndeavourOS, I wouldn't have dared to tinker with such novel compositors, as any system mishaps would have entailed significant manual troubleshooting using various workarounds.

As I get more and more involved with NixOS and Nix, I find it also very suitable for synchronously managing the configuration of multiple hosts. Currently my personal [nix-config](https://github.com/ryan4yin/nix-config) synchronously manages the configuration of many hosts:

- Desktop computers
  - 1 Macbook Pro 2020 (Intel amd64).
  - 1 Macbook Pro 2022 (M2 aarch64).
  - 1 NixOS desktop PC (amd64).
- Servers
  - 3 NixOS virtual machines (amd64).
  - Several development boards for aarch64 and riscv64.

The development environment of three desktop computers is managed by Home Manager, the main configuration is completely shared, and the configuration modified on any host can be seamlessly synchronized to other hosts through Git.

Nix almost completely shielded me from the differences between OS and architecture at the bottom of the three machines, and the experience was very smooth!
