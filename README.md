# nixos_my_voyage_to_rust_on_nixos

**Tutorial how to start using NixOS on Windows with WSL**  
***version: 1.0.0 date: 2024-09-15 author: [bestia.dev](https://bestia.dev) repository: [GitHub](https://github.com/bestia-dev/nixos_my_voyage_to_nixos)***  

It sound like NixOS is an interesting Linux OS for easy declarative management to recreate the same OS many times. The main point is reproducibility.  
Then the Nix package manager is very interesting for ephemeral installations of programs that are easily removed. Good for experimenting and testing.

Let's try it out!

## Install NixOS on Windows11 inside WSL

<https://nix-community.github.io/NixOS-WSL/install.html>

- download the 0,5GB tar  

Run in git-bash on windows:  

```bash
wsl --import NixOS --version 2 $USERPROFILE/NixOS/ nixos-wsl.tar.gz
# enter NixOS shell: 
wsl -d NixOS
```

Run in NixOS shell:

```bash
# update Nix channel:
sudo nix-channel --update
# after a declarative change rebuild the OS: 
sudo nixos-rebuild switch
#check version: 
nixos-version
```

## Config the WSL

I don't want NixOS (guest) to see the filesystem on windows (host).  
I need to make changes to NixOS `/etc/wsl.conf`, but this file is readonly. I have to tinker with `/etc/nixos/configuration.nix` and then rebuild the system. This is the NixOS way of doing things. Makes sense.

```bash
sudo nano /etc/nixos/configuration.nix
```

Here I added these lines:

```ini
  wsl.enable = true;
  wsl.defaultUser = "nixos";

  # disable auto-mounting of win drives
  wsl.wslConf.automount.enabled = false;
  # process fstab entries to allow some folders to be mounted
  wsl.wslConf.automount.mountFsTab = true;
  # disable launching windows exe files
  wsl.wslConf.interop.enabled = false;
  # disable appending thw windows path
  wsl.wslConf.interop.appendWindowsPath = false;
```

Finally rebuild the system and switch:

```bash
sudo nixos-rebuild switch
# now the wsl.conf is changed, but not yet used
# exit from WSL
exit
```

Shutdown wsl from git-bash in windows and reopen WSL for the change to be used:

```bash
wsl --shutdown
wsl -d nixos
```

TODO: maybe these changes can be added to the image before creating the WSL distro.

## Container for Rust development with systemd-nspawn

It is preferable to run NixOS containers with systemd-nspawn instead of Docker or Podman.

<https://nixcademy.com/posts/nixos-nspawn/>  
Container image: <https://github.com/tfc/nspawn-nixos>  

```bash
machinectl pull-tar https://github.com/tfc/nspawn-nixos/releases/download/v1.0/nixos-system-x86_64-linux.tar.xz nixos --verify=no
```

I get the error: Access denied.  
Probably Github wants some access control?  
OK, I download the 139 MB with the browser in Windows and then copy it to WSL/NixOS:  
`\\wsl.localhost\nixos\home\nixos\`   
The default user of this WSL image is called `nixos`.  
The `pull-tar` command does just one thing: it copies the files into  
`/var/lib/machines/{container_name}`.

I can do that manually here in the NixOS shell.

```bash
cd ~
# [nixos@nixos:~]$
sudo mkdir /var/lib/machines/nixos
sudo tar -xvf nixos-system-x86_64-linux.tar.xz -C /var/lib/machines/nixos/
sudo ls -la /var/lib/machines/nixos
# run it in the background
sudo machinectl start nixos
```

## Set root password

```bash
sudo machinectl shell nixos /usr/bin/env passwd
```

TODO: How to create a new user? The default is `nixos` and that is defined in the `/etc/nixos/configuration.nix` file.



# check status of the machine that runs in the background
machinectl status nixos

machinectl login nixos
- Press ^] three times within 1s to exit session
On my keyboard layout I have to use Ctrl+5 three times in one second.
nixos login: root
password : ***

nixos-version
23.11.20230826.5237477 (Tapir)

A simple `exit` command in the container calls `logout` and then continues to ask
nixos login:

Use three times Ctrl+5 to really exit the container.

## Network access

enable internet access is to share the host’s network
On the host:
create /etc/systemd/nspawn/nixos.nspawn
with content
[Network]
VirtualEthernet=no

After changing the file, reboot the container:
machinectl reboot nixos

## NixOS Configuration

Now, we can edit the NixOS configuration file 
/etc/nixos/configuration.nix
 in the container’s file system. We can do that either from inside the container or from the host, as the container paths are all below 
 /var/lib/machines/<machine name>. 
 For the configuration file, the full host path is 
 /var/lib/machines/nixos/etc/nixos/configuration.nix.

```bash
nixos-rebuild switch
```

## Open-source and free as a beer

My open-source projects are free as a beer (MIT license).  
I just love programming.  
But I need also to drink. If you find my projects and tutorials helpful, please buy me a beer by donating to my [PayPal](https://paypal.me/LucianoBestia).  
You know the price of a beer in your local bar ;-)  
So I can drink a free beer for your health :-)  
[Na zdravje!](https://translate.google.com/?hl=en&sl=sl&tl=en&text=Na%20zdravje&op=translate) [Alla salute!](https://dictionary.cambridge.org/dictionary/italian-english/alla-salute) [Prost!](https://dictionary.cambridge.org/dictionary/german-english/prost) [Nazdravlje!](https://matadornetwork.com/nights/how-to-say-cheers-in-50-languages/) 🍻

[//bestia.dev](https://bestia.dev)  
[//github.com/bestia-dev](https://github.com/bestia-dev)  
[//bestiadev.substack.com](https://bestiadev.substack.com)  
[//youtube.com/@bestia-dev-tutorials](https://youtube.com/@bestia-dev-tutorials)  

[//]: # (auto_md_to_doc_comments segment end A)
