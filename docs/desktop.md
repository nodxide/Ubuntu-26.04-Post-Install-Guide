# Desktop

Ubuntu 26.04 Desktop uses **GNOME 50** with **Wayland by default**.

GNOME should remain the primary desktop configuration layer. Prefer its built-in settings and components before adding third-party extensions or customization tools.

## GNOME Tools

Optional tools for configuration that is not exposed through the standard Settings application:

```bash
sudo apt install \
  gnome-tweaks \
  gnome-shell-extension-manager
```

Use **GNOME Settings** for normal desktop configuration.

Use **GNOME Tweaks** for additional options such as font behavior, interface details, and application appearance.

Use **Extension Manager** to discover, install, disable, and remove GNOME Shell extensions.

Avoid installing multiple tools that provide overlapping functionality.

## GNOME Extensions

GNOME Shell extensions modify core desktop behavior and may become incompatible after GNOME or extension API changes.

Use as few extensions as practical and install only extensions that provide functionality not already available through GNOME.

Before installing an extension, consider:

- whether GNOME already provides the functionality;
- whether the extension is actively maintained;
- whether it supports the current GNOME version;
- whether it introduces unnecessary dependencies or system modifications.

When troubleshooting GNOME Shell problems:

1. Disable all extensions.
2. Log out and log back in.
3. Reproduce the problem.
4. Re-enable extensions individually.
5. Identify and remove or replace the problematic extension.

> [!Warning]
> Do not assume that an extension working on one GNOME release will continue working after a system upgrade. Extensions should be treated as version-sensitive components.

## Fonts

Install user fonts under:

```text
~/.local/share/fonts/
```

Refresh the font cache:

```bash
fc-cache -fv
```

Verify installed fonts:

```bash
fc-list | head
```

Common programming fonts include:

- JetBrains Mono;
- Iosevka;
- Fira Code;
- IBM Plex Mono.

Prefer user-level font installation unless a font is required system-wide.

## Audio

Ubuntu's modern desktop audio stack uses **PipeWire** with **WirePlumber**.

Check the user services:

```bash
systemctl --user status pipewire
systemctl --user status wireplumber
```

For advanced audio-device and mixer management:

```bash
sudo apt install pavucontrol
```

Use GNOME's built-in audio controls for normal volume and device management. Use `pavucontrol` when more detailed routing or device configuration is required.

## Bluetooth

Check the Bluetooth service:

```bash
systemctl status bluetooth
```

For additional graphical Bluetooth management:

```bash
sudo apt install blueman
```

Use GNOME's built-in Bluetooth settings for normal device pairing and management.

## Shell

**Bash is the default shell**. Zsh and Fish are valid alternatives.

Shell configuration should remain independent from GNOME configuration.

Before changing the default shell, understand the existing shell initialization, environment variables, startup files, and development tooling that depend on it.

Do not change the system shell merely to customize the terminal experience. Configure the terminal emulator and shell independently when possible.

## Appearance

GNOME customization commonly includes:

- fonts;
- GTK and application themes;
- icons;
- cursor;
- wallpaper;
- terminal;
- keyboard shortcuts;
- display scaling;
- GNOME Shell behavior.

Prefer GNOME's built-in configuration wherever possible.

Use **GNOME Tweaks** for settings that are not exposed through GNOME Settings, and use extensions only when native functionality is insufficient.

Keep appearance-related customization consistent. Avoid combining multiple theme engines, shell modifications, and extension-based customization systems unless there is a specific reason to do so.

## Configuration

Prefer user-level configuration for desktop customization:

```text
~/.config/
~/.local/share/
```

Avoid modifying system files for preferences that can be configured at the user level.

Where practical, keep important configuration reproducible and version-controlled. Do not commit machine-specific paths, credentials, or other sensitive information.

## Baseline

Establish a stable GNOME installation before applying extensive customization.

A sensible order is:

```text
GNOME
  ↓
System updates
  ↓
Basic configuration
  ↓
Fonts and applications
  ↓
Required extensions
  ↓
Optional customization
```

> [!Important]
> Avoid installing large collections of extensions, themes, and shell modifications immediately after installation. Add customizations incrementally so that compatibility problems can be isolated, diagnosed, and reverted without destabilizing the desktop environment.