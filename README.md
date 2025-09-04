# gsoc25
All work done in the project is a part of [Google Summer of Code 2025](https://summerofcode.withgoogle.com/).
project link -- https://summerofcode.withgoogle.com/programs/2025/projects/mV0VlVwv

## Project Description
Regolith is a productivity-focused Ubuntu-based desktop environment that combines tiling window managers (Sway, i3) with GNOME components for system management and GUI features. While GNOME depends on Mutter and custom Wayland protocols unsupported by wlroots-based compositors like Sway, this limits modularity and integration. To improve flexibility, this project aim to integrate the new Cosmic desktop components (cosmic-epoch), which, despite being in early alpha, offers a more modular and integration-friendly design33.

## Outcome
Core system settings, input, display, power management, session integration, config bridging, and installation steps are fully implemented with COSMIC components replacing GNOME-based ones. The forked cosmic-settings replacing regolith-control-center applies live sway commands and persists configs. Installation and session launching are documented and working. The complete checklist can be found below.

### `regolith-control-center` -> `cosmic-settings` 
`cosmic-settings` was proposed to replace exsting system of (`gnome-control-center` + `regolith-inputd` + `regolith-powerd` + `regolith-displayd`) which manages changes in settings for input (keyboard, mouse, touchpad), display and power. During the development I tried 2 methods to do the same job and selected `generated-config`

#### Method 1: generated-config
https://github.com/pop-os/cosmic-settings/pull/1337
[This fork](https://github.com/sandptel/cosmic-settings) of cosmic-settings that runs a equivalent sway command for `input` settings : `mouse` + `keyboard` + `touchpad` instead of running the change for the compositor. 
This is used as a replacement to the existing gnome-settings application. 

##### workflow
- Settings changes trigger equivalent sway commands through swayipc for instantaneous changes.
- Settings are simultaneously written to persistent config files at `~/.config/regolith3/sway/cosmic-settings/generated-config.d/*`
- Changes persist when Sway restarts by loading the generated configuration files
- Conflicting settings will be reported by sway. 

##### example: 
If we increase or decrease the speed of say touchpad or mouse, the cosmic-settings daemon on any change in the settings values will equivalently apply the state in sway using swayipc rust and in `~/.config/regolith3/sway/cosmic-settings/generated-config.d`

https://github.com/user-attachments/assets/4f0c103c-a678-42c5-aeb7-330fb7963462

#### Method 2: Background Daemon Approach

##### Workflow
1. This will run a persistent background daemon at startup.
2. Daemon watches for changes in cosmic-config files (~/.config/cosmic/)
3. On config changes, daemon immediately applies equivalent sway commands via swayipc
4. Daemon maintains mapping between cosmic-config values and sway IPC commands and runs those when starting to persist through boot.

#### Comparison
| Feature                  | Generated Configs                               | Background Daemon                                 |
|--------------------------------|--------------------------------------------------------|---------------------------------------------------------|
| **Modularity**                 | High - Every Single page and option can be removed customized and replaced according to regolith and sway needs. | Lower - Can not change/add UI/Options for settings; less modular due to only background watcher dependency |
| **Performance**                | High - Uses file I/O to write and read generated configs; changes applied instantly via swayipc-rs and persistant on reload | High - Applies settings live instantly via ipc without writing files |
| **Conflicts**| Low - All conflict will be marked as config errors and will | Low - For every reload and restart rerunning the  |
| **Persistence / State Saving**| High - Configuration saved in generated config is loaded by sway every reload and boot automatically | Medium - Daemon must continuously run to maintain state; needs to reapply the whole list of changes every reload or boot |

### `configuration` 
https://github.com/regolith-linux/regolith-wm-config/pull/56

This setup replaces GNOME components with COSMIC ones and adds a live bridge from COSMIC Settings to Sway: `regolith-control-center` is swapped for `cosmic-settings` + `cosmic-settings-daemon`, `swaybg` for `cosmic-bg`, `nautilus` for `cosmic-files`, and `gnome-session-quit` actions are routed through `systemctl`/`loginctl`, while `cosmic-idle` provides idle/blank/lock timing; a forked cosmic-settings emits changes via swayipc-rs so adjustments apply immediately through Sway IPC, and simultaneously writes equivalent configuration into `$HOME/.config/regolith3/sway/cosmic-settings/generated-config.d/*`, which must be included in the main Sway config to persist those changes across sessions

### `session` files
## How to boot into regolith + cosmic session ?
We need to install the COSMIC components 
1. using the OpenSUSE Build Service first update the 
```bash
echo 'deb http://download.opensuse.org/repositories/home:/nomispaz:/debian:/cosmic-desktop/Debian_13/ /' | sudo tee /etc/apt/sources.list.d/home:nomispaz:debian:cosmic-desktop.list
curl -fsSL https://download.opensuse.org/repositories/home:nomispaz:/debian:cosmic-desktop/Debian_13/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/home_nomispaz_debian_cosmic-desktop.gpg > /dev/null
sudo apt update
```
2. Leave `cosmic-settings` application and install other components
```bash
sudo apt install cosmic-app-library cosmic-applets cosmic-bg cosmic-comp cosmic-edit cosmic-ext-alternative-startup cosmic-ext-extra-sessions cosmic-files cosmic-greeter cosmic-icons cosmic-idle cosmic-launcher cosmic-notifications cosmic-osd cosmic-panel cosmic-player cosmic-randr cosmic-screenshot cosmic-session cosmic-settings-daemon cosmic-store cosmic-term cosmic-wallpapers cosmic-workspaces xdg-desktop-portal-cosmic
```
3. clone and run/install the fork of `cosmic-settings`
```bash
git clone https://github.com/sandptel/cosmic-settings
cd cosmic-settings
just install
```
4. fetch the updated `regolith-wm-config` and `regolith-session` files using git clone
```bash
git clone https://github.com/sandptel/regolith-session/tree/cosmic
git clone https://github.com/sandptel/regolith-wm-config/tree/cosmic
```
and move these files to /usr and /etc respectively.

5. Run the `cosmic-session` along with sway 
```bash
cosmic-session sway --config <path to cloned regolith-wm-config dir>/etc/regolith/sway/config -d
```

## References 
https://github.com/ErikReider/SwaySettings
https://github.com/Drakulix/cosmic-ext-extra-sessions/tree/main/sway
