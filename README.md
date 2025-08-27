# gsoc25
project link -- https://summerofcode.withgoogle.com/programs/2025/projects/mV0VlVwv

## Project Description
Regolith is a productivity-focused Ubuntu-based desktop environment that combines tiling window managers (Sway, i3) with GNOME components for system management and GUI features. While GNOME depends on Mutter and custom Wayland protocols unsupported by wlroots-based compositors like Sway, this limits modularity and integration. To improve flexibility, this project aim to integrate the new Cosmic desktop components (cosmic-epoch), which, despite being in early alpha, offers a more modular and integration-friendly design.

## Outcome
### `regolith-wm-config` 

### `cosmic-settings` adapted for regolith
[This fork](https://github.com/sandptel/cosmic-settings) of cosmic-settings that runs a equivalent sway command for `input` settings : `mouse` + `keyboard` + `touchpad` instead of running the change for the compositor. 
This is used as a replacement to the existing gnome-settings application. 

https://github.com/user-attachments/assets/4f0c103c-a678-42c5-aeb7-330fb7963462

## How to replicate yourself?


