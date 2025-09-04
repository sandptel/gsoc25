# gsoc25
All work done in the project is a part of [Google Summer of Code 2025](https://summerofcode.withgoogle.com/)

- organization : [CCExtractor](https://ccextractor.org/)
- contributor : Sandeep Patel ([sandptel](https://github.com/sandptel))
- mentor : Soumya Ranjan Patnaik ([SoumyaRanjanPatnaik](https://github.com/SoumyaRanjanPatnaik))
- project:   [Build a COSMIC based wayland session for Regolith](https://summerofcode.withgoogle.com/programs/2025/projects/mV0VlVwv)

## Project Description
[Regolith](https://regolith-desktop.com/) is a productivity-focused Ubuntu-based desktop environment that combines tiling window managers (Sway, i3) with GNOME components for system management and GUI features. Regolith currently relies on a mix of GNOME components (like regolith-control-center) and Sway/i3 tiling window manager for its session and settings management. GNOME’s dependencies (Mutter compositor, custom Wayland protocols) limit flexibility and modularity, especially for Wayland environments based on wlroots like Sway.

[COSMIC](https://system76.com/cosmic/?srsltid=AfmBOorpuyn88diqJVLAWtDfzj8nbNDT1MEGNQqowEia67oghLiOw5Av), a new Rust-based, modular, and efficient desktop environment by System76, offers a more integrated and modern alternative. Therefore, We are transforming Regolith from a complex, GNOME-dependent hybrid into a modern, modular, and cohesive Wayland session environment by leveraging the COSMIC ecosystem. It enhances usability, maintainability, and aligns Regolith with the future direction of Linux desktop development. 

In the outcomes we will be discussing how we replaced individual dependencies either directly or by adapting (fork) it to work with regolith (sway).

## Outcomes
Core system settings, input, display, power management, session integration, config bridging (partial) , and installation steps are fully implemented with COSMIC components replacing GNOME-based ones. The forked cosmic-settings replacing regolith-control-center applies live sway commands and persists configs. Installation and session launching are documented and working. The core parts of the work are as follows: 

### `regolith-control-center` -> `cosmic-settings` 
`cosmic-settings` was proposed to replace exsting system of (`gnome-control-center` + `regolith-inputd` + `regolith-powerd` + `regolith-displayd`) which manages changes in settings for input (keyboard, mouse, touchpad), display and power. During the development I tried 2 methods to do the same job and selected `generated-config`

#### Method 1: generated-config
https://github.com/sandptel/cosmic-settings/pull/2
[This fork](https://github.com/sandptel/cosmic-settings) of cosmic-settings that runs a equivalent sway command for `input` settings : `mouse` + `keyboard` + `touchpad` instead of running the change for the compositor. 

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

With these pros and cons, the chaning `cosmic-settings` to add sway (ipc+ generated-config) is selected and will be added to the release. More pages (configuration) are planned to be added to this.

### `configuration` 
https://github.com/regolith-linux/regolith-wm-config/pull/56

This setup replaces GNOME components with COSMIC ones and adds a live bridge from COSMIC Settings to Sway: `regolith-control-center` is swapped for `cosmic-settings` + `cosmic-settings-daemon`, `swaybg` for `cosmic-bg`, `nautilus` for `cosmic-files`, and `gnome-session-quit` actions are routed through `systemctl`/`loginctl`, while `cosmic-idle` provides idle/blank/lock timing; a forked cosmic-settings emits changes via swayipc-rs so adjustments apply immediately through Sway IPC, and simultaneously writes equivalent configuration into `$HOME/.config/regolith3/sway/cosmic-settings/generated-config.d/*`, which must be included in the main Sway config to persist those changes across sessions

### `session` files
https://github.com/regolith-linux/regolith-session/pull/54
Minimally replaces direct sway launch with exec `/usr/bin/dbus-run-session -- cosmic-session sway -c "$SWAY_CONFIG_FILE"` to launch cosmic session alongside sway. The rest of the components are launched using sway configuration for startup.

## Completed Goals / Future Plans Checklist
I was able to create working session that runs with cosmic and its settings application, Post GSoC I will be updating on these planned goals to create a beta release of this regolith-session : 
- [ ] regolith-control-center ->   cosmic-settings
  - [x] Display Settings
  - [x] Input Settings
  - [x] Power Settings
  - [x] Appearance Page ( Wallpaper/ Cosmic-UI changes)
  - [ ] Workspace Management ( Planned )
  - [ ] Keyboard Shortcuts ( Ongoing/Partial )
  - [ ] Custom Settings Pages ( Planned )
    - [ ] Startup Applications
    - [ ] Window Rules
    - [ ] Default Applications
- [x] `regolith-wm-config`
  - [x] Includes Generated Configs
  - [x] cosmic-osd
  - [x] swaybg -> cosmic-bg (Working with `cosmic-settings`)
  - [x] sway-idle -> cosmic-idle
  - [x] xdg-desktop-portal -> xdg-desktop-portal-cosmic 
  - [ ] Optional (full-cosmic version) ( These are already working cosmic components that can be user-specific/optional whether to replace then with existing ones). { Planned }
    - [ ] ilia -> cosmic-launcher
    - [ ] swaybar -> cosmic-panel
    - [ ] use cosmic-dock
    - [ ] use cosmic-applets
- [x] Updated `regolith-session` to launch a cosmic-session parallely with sway. 
- [x] Updated Build Files 
- [ ] Verify Installation / Testing
  - [x] Ubuntu
  - [ ] Debian
- [ ] Beta Release

> **Note:** Since COSMIC is still in early development (pre-alpha), this integration project is effectively in a beta/testing phase, which means the work is pioneering and in progress.

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
## Other Contributions during timeline
- https://github.com/pop-os/cosmic-settings/pull/1227
- https://github.com/pop-os/cosmic-applets/pull/948

## References 
- https://github.com/pop-os/cosmic-epoch
- https://github.com/ErikReider/SwaySettings
- https://github.com/Drakulix/cosmic-ext-extra-sessions/tree/main/sway
