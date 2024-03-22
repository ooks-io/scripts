> [!WARNING]\
> Still in a prototype stage. Script is currently hacked together and requires
> cleanup. Feedback welcome.

[Dependencies](#dependencies) | [Installation](#installation) | [Usage](#usage)
| [Waybar](#waybar-integration) | [Planned](#features-planned-for-the-future)

# Hyprrecord

A simple screen recording script for
[hyprland](https://github.com/hyprwm/hyprland), based on
[grimblast](https://github.com/hyprwm/contrib/tree/main/grimblast).

## Dependencies

- [wl-screenrec](https://github.com/russelltg/wl-screenrec): A Wayland screen
  recording utility
- [wl-clipboard](https://github.com/bugaevc/wl-clipboard): A Wayland clipboard
  utility.
- [slurp](https://github.com/emersion/slurp): For selecting a region in wayland.
- [ffmpeg](https://ffmpeg.org/): Tool used to convert, stream and manipulate
  media.
- [pactl](https://manpages.ubuntu.com/manpages/jammy/en/man1/pactl.1.html):
  Command-line utility for controlling a PulseAudio server.
- [jq](https://github.com/jqlang/jq): Json parsing.

## Installation

### Manual

```shell
$ git clone https://github.com/ooks-io/hyprrecord.git
$ cd hyprrecord
$ make
$ sudo make install

# Remove repo if you want.
cd ..
rm -rf hyprrecord
```

### Nix

Add flake as input:

```nix
# flake.nix
{
  inputs = {
    # your inputs ...
    hyprrecord = {
      url = "github:ooks-io/hyprrecord";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };
}

# ... rest of flake
```

Ensure inputs are being passed into your home-manager/nixos config:

```nix
# flake.nix
homeConfiguration = {
  "YOURHOMEHERE" = home-manager.lib.homeManagerConfiguration {
    #...config
    extraSpecialArgs = {inherit inputs;};
  };
};
```

```nix
#flake.nix
nixosConfiguration = {
  SYSTEMHERE = nixpkgs.lib.nixosSystem {
    #...config
    specialArgs = {inherit inputs;};
  };
};
```

Add the package to configuration:

```nix
#home.nix
{ pkgs, inputs, ... }:
{
  home.packages = [ inputs.hyprrecord.packages.${pkgs.system}.hyprrecord ];
}
```

```nix
#configuration.nix
{ pkgs, inputs, ... }:
{
  environment.systemPackages = [ inputs.hyprrecord.packages.${pkgs.system}.hyprrecord ];
}
```

## Usage
``` bash
$ hyprrecord [OPTION]... [TYPE] [SUBJECT] [ACTION]
```
### Options

| Command                            | Result                                    |
| ---------------------------------- | ----------------------------------------- |
| `-a`, `--audio`  | Enable audio recording, uses [pactl](https://manpages.ubuntu.com/manpages/jammy/en/man1/pactl.1.html) to get the current default sink |
| `-w`, `--waybar` | Enable Waybar integration. See [Waybar Integration](#waybar-integration) for more infomation. |

### Types

| Command                            | Result                                    |
| ---------------------------------- | ----------------------------------------- |
| `video` [**default**] | Record video as mp4                                    |
| `gif`                 | Record video as mp4, then convert to gif with ffmpeg.  |

### Subjects

| Command                            | Result                                    |
| ---------------------------------- | ----------------------------------------- |
| `screen`[**default**]  | Record currently active screen |
| `area`                 |Record a selected area |
| `active`               |Record the currently focused window |

### Actions

| Command                            | Result                                    |
| ---------------------------------- | ----------------------------------------- |
| `usage`[**default**] | Print help information. |
| `check`              | Tests to see if dependencies are available. |
| `save`               | Save the recording to **$XDG_RECORDINGS_DIR** if it exists, otherwise save to **$XDG_VIDEOS_DIR**. If neither exists, save to **$HOME**. |
| `copy`               | Save recording to /tmp and put file into clipboard. |
| `copysave`           | Save recording and put into clipboard. |

### Examples

| Command                            | Result                                    |
| ---------------------------------- | ----------------------------------------- |
| `hyprrecord video screen save`     | Record the entire screen and save to recording directory |  
|`hyprrecord -a video area copy`     | Record a video with audio of a selected area and copy into clipboard |
| `hyprrecord video active copysave` | Record video of currently focused window, save to recording directory & copy into clipboard |
| `hyprrecord gif area copy`         | Record video of a selected area, convert into gif and copy into clipboard. |

## Hyprland Integration

To add keybinding to hyprland you can add these lines to your _hyprland.conf_

```bash
# ~/.config/hypr/hyprland.conf

# Record entire screen with sound, save and copy into clipboard
bind = SUPER, R, exec, hyprrecord -a screen copysave video

# Record area, convert to gif, put recording into clipboard
bind = SUPER CTRL, R, exec, hyprrecord area copy gif

# Record area with sound, save to recording dir and integrate with waybar  
bind = SUPER SHIFT, R, exec, hyprrecord -a area copysave video
```

## Waybar Integration

Waybar integration is achieved by sending a signal to waybar when recording
starts and stops.

To start, add the `--waybar` flag to your command and configure with a custom
moddule in waybar's configuration:

```jsonc
// ~/.config/waybar/config.jsonc

"modules-right": ["custom/hyprrecord"],

"custom/hyprrecord": {
  "format": "{}",
  "interval": "once",
  "exec": "echo ",
  "tooltip": "false",
  "exec-if": "pgrep 'wl-screenrec'",
  "signal": 12
}
```

```css
/* ~/.config/waybar/style.css */

#custom-hyprrecord {
  color: red;
}
```

> [!NOTE]\
> The custom module must use signal 12 to integrate with the script.

## Features planned for the future

- Microphone option (_requires pipewire creating a virtual output with multiple
  audio streams_)
- Convert to WebP (_will require ffmpeg with libwebp_)
- AGS support