# system setup

this file details information about my linux mint install

## names of things

- the taskbar is a "panel"
- pulseaudio output devices are called "sinks"
- pulseaudio input devices are called "sources"

## desktop wallpaper

- this is managed by nitrogen instead of the default linux mint thing in order to have different wallpapers on each monitor.
- `nitrogen --restore` runs on login (via UI app "Startup Applications" in system preferences)

## audio

### pipewire

- pulseaudio server is replaced with pipewire, compiled from source
  - source lives in ~/Code/pipewire
  - compiled output is in /opt/pipewire (used PREFIX)
  - systemd files for pipewire are in ~/.config/systemd/user/
    - they're copied there from /opt/pipewire/lib/systemd/user. I tried to symlink them instead, but it confused systemd.
  - /etc/default/pulse.pa renames some devices, but that doesn't actually get run anymore because the pipewire pulseaudio server doesn't real that file (it doesn't support pacmd, and every line in that file gets passed to pacmd, when using a normal pulseaudio server)
  - instead, audio devices are renamed in `~/.config/pipewire/media-session.d/alsa-monitor.conf`
- there's an audio filter device that takes "USB Audio 2", and passes it through RNNoise. This works like Krisp (AI/neural net/machine learning voice noise filter). It's configured in `~/.config/pipewire/custom/source-rnnoise.conf`, and gets launched by user systemd on login via `~/.config/systemd/user/pipewire-input-filter-chain.service`.

### pulseaudio UI apps

- pasystray, pavucontrol, and pavumeter are installed because they're more full-featured than the default mint volume stuff

### kxstudio patchbay/JACK stuff

- there's audio stuff from the kxstudio-debian/* ppas installed. there's a patchbay/audio routing app called catia, and catarina exists too.
  - the ppas themselves were added via a .deb file that installed a package called kxstudio-repos

## shell stuff

- modifications are in ~/.bashrc
- globstar is enabled
- ~/.local/bin is added to PATH, to make meson work
- /opt/pipewire/bin is added to PATH, for pipewire CLI stuff
- there is a function "findtext" that searches files for text. it's defined as `rg "$1" -g '**/*' --iglob '/{media,var,proc,sys,run}/**/*' "${@:2}"`

## packages

- there are two package systems: apt and flatpak.
  - "Software Manager" shows stuff from both. non-flatpak stuff is more convenient when available.
  - there was also snapd at one point but I didn't need it anymore.

### apt
- use `dpkg --get-selections | grep whatever` to search through installed apt packages
- use `apt-cache search whatever` to search through apt repos
- use this to show all ppas: `egrep -v '^#|^ *$' /etc/apt/sources.list /etc/apt/sources.list.d/*`
- use this to show where a package came from: `apt-cache policy <package-name>`

### flatpak
- flatpak packages run in a little sandbox
- all their filesystem stuff is in ~/.var/app/<com.app.identifier>/
- use `flatpak list` to show installed flatpaks

## kernel modules

- managed through dkms
- use `dkms status` to list them
- use `dpkg --get-selections | grep dkms` to show what apt package they came from (if any)
- whenever apt updates the kernel, dkms rebuilds them for the new kernel version
- prefer apt packages for dpkg stuff oover manually doing it, so that when apt updates the kernel, it can tell dkms to rebuild stuff
- nvidia module comes from official sources; package name is nvidia-dkms-470 (version number might change)
- realtek-r8125 (for ethernet) module comes from the ppa ; package name is realtek-r8125-dkms
- we won't need r8125 after we're on kernel 5.10.

## external volumes

- The windows C:/ drive gets automounted at /media/suchipi/Konata
- the windows steam data drive gets automounted at /media/suchipi/Kagami
- the automount stuff is in /etc/fstab, but I use the cinammon "Disks" app to edit it.
- to make steam libraries work on linux, you have to mount with these options: `nosuid,nodev,nofail,x-gvfs-show,exec,uid=1000,gid=1000`
  - (but change uid and gid if a different user uses stuff; suchipi is 1000:1000 but a new user would be 1001:1001, etc)

## folder conventions

- ~/Code is github repos
- ~/Apps is portable apps
- ~/Games is lutris's wine prefixes
- ~/ROMs is game roms and isos; no subfolders, just dump everything in there

## random installed apps

- there's a video editor called Olive in ~/Apps
- there's an ips patcher called flips in ~/Apps
- the shadow client (https://shadow.tech) is in ~/Apps/Shadow

## game stuff

- lutris is installed as a game launcher that creates and sets up wine prefixes for games automatically
- it puts its wine prefixes in ~/Games
- there's also a default wineprefix in ~/.wine that runs stuff when you use `wine whatever.exe` from the command line. It's got a bunch of random stuff installed in it, to make stuff work.
- instead of game launcher binaries, lutris uses `lutris://` URLs to launch games. they work in .desktop files; idk about the terminal, maybe with xdg-open

### FFXIV

- FFXIV is in lutris; there's two copies, so I can multibox using my alt account
- FFXIV uses data from the windows steam data drive; it's shared with the windows install of FFXIV
- this shared data is in /media/suchipi/Kagami/SteamLibrary/steamapps/common/FINAL FANTASY XIV Online
- but it uses a GShade install symlinked in from ~/.local/share/GShade. When running FFXIV on windows or linux, you have to change two symlinks:
  - d3d11.dll
  - gshade-shaders
- client config and xlplugins stuff for each ffxiv install live inside the wineprefixes in ~/Games
  - client config is in Documents/My Games/Final Fantasy XIV
  - xivplugins stuff is in appdata somewhere
- you have to connect your controller before launching the game for it to show up in-game.

- there's a `dxvk.conf` in the FFXIV folder that has options needed to make the game run well. those options are (file content):
```
dxvk.shrinkNvidiaHvvHeap = True
dxgi.maxFrameLatency = 1
d3d9.maxFrameLatency = 1
```

`dvvk.shrinkNvidiaHvvHeap` works well for me but not for Gen. Maybe we should only use the `maxFrameLatency` options?
in a future dxvk release, `dxvk.shrinkNvidiaHvvHeap = True` will be default for FFXIV (it detects game based on exe name)

### Modded minecraft

- I use multimc to manage minecraft installs
  - but sometimes I have to copy stuff over from windows CurseForge installs because multimc doesn't download all the libraries (idk why). but the libraries are just jars, so they're cross-platform
- Had to install oracle java to make this work. minecraft 1.16.5 only works on java 8
- You can use the cinnamon menu editor to make .desktop files that launch instances directly without having to open the multimc UI window, by passing a flag to multimc
  - eg `/opt/multimc/run.sh -l "Comfy.Creator-1.9.0-public"`, where the argument to -l is the name of the instance folder somewhere in the dotfiles in ~ (I think there's a button in MultiMC to open an instance's folder in the file browser)

### SRB2 (Sonic Robo Blast 2)

- installed via flatpak
- ~/.srb2 is a symlink to the flatpak's stuff; you can put addons in there, there's screenshots in there, etc
- it's an SDL2 game so PS4 controller works out of the box

### Gunfire Reborn

- installed via Steam Proton (steam's wine/dxvk thing)
- don't run it in fullscreen ever because if you alt-tab, you won't be able to alt-tab back in. only run in borderless or windowed

### Metroid Prime mod (PrimeHack)

- runs via default wine prefix; game is in ~/Apps/PrimeHack
- windows dolphin runs surprisingly well in wine; just make sure to use Vulkan or OpenGL graphics backend instead of DirectX







