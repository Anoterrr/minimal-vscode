# zen-dots

> Keyboard-first. Visually silent. Contextually aware.

Dotfiles for a minimal and reproducible development environment.
Tested on **Arch Linux (WSL2)** and **macOS (ARM)**.

---

## Philosophy

Each tool has a single responsibility — no overlap:

| Layer | Responsibility |
|---|---|
| **ZSH** | Environment: aliases, pagers, editor, keybinds |
| **Starship** | Real-time context: git, runtimes, exit codes, vi-mode |
| **Mise** | Runtime manager: node, python, rust, go, java |
| **VS Code** | Editor: LSP, diagnostics, formatters, Vim motions |
| **LazyVim** | Terminal editor: quick edits, git, remote files |
| **Topgrade** | Maintenance: updates the full stack in one command |

---

## Repository Structure

```
zen-dots/
├── config/
│   ├── zshrc          → ~/.zshrc
│   ├── gitconfig      → ~/.gitconfig
│   ├── mise.toml      → ~/.config/mise/config.toml
│   └── topgrade.toml  → ~/.config/topgrade.toml
└── vscode/
    └── settings.json  → ~/.config/Code/User/settings.json       (Linux/WSL)
                       → ~/Library/Application Support/Code/User/ (macOS)
```

---

## Installation — Arch Linux (WSL2)

### 0. Prerequisites

- Windows 11 (WSL 2.4.4+)
- PowerShell with administrator privileges
- Virtualization enabled in BIOS

---

### 1. Install Arch Linux

```powershell
# PowerShell (Admin)
wsl --install archlinux
```

The distro opens as root. Set the root password:

```bash
passwd
```

---

### 2. Base packages and user

```bash
# Sync and update
pacman -Syu --noconfirm

# Base packages + tooling via pacman
pacman -S --noconfirm --needed \
  base-devel git curl wget unzip zsh fzf man-db less openssh \
  eza bat ripgrep fd zoxide git-delta starship lazygit \
  wl-clipboard neovim
```

Create user with ZSH as the default shell from the start:

```bash
useradd -m -G wheel -s /bin/zsh your_username
passwd your_username
echo "%wheel ALL=(ALL:ALL) NOPASSWD: ALL" > /etc/sudoers.d/wheel
chmod 440 /etc/sudoers.d/wheel
```

---

### 3. Locale and `/etc/wsl.conf`

```bash
# Locale
sed -i 's/#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# WSL config
cat <<EOF > /etc/wsl.conf
[automount]
enabled = true
options = "metadata,uid=1000,gid=1000,umask=022,fmask=011"

[network]
generateResolvConf = true
hostname = arch-dev

[interop]
enabled = true
appendWindowsPath = true

[user]
default = your_username

[boot]
systemd = true
EOF
```

Restart to apply:

```powershell
# PowerShell
wsl --terminate archlinux
wsl -d archlinux
```

> From here, all commands run as `your_username`.

---

### 4. Mise (runtimes)

```bash
curl https://mise.run | sh

mkdir -p ~/.config/mise
cp ~/dotfiles/config/mise.toml ~/.config/mise/config.toml

~/.local/bin/mise install
```

---

### 5. Tooling via Cargo

Mise has already installed Rust. Activate cargo before continuing:

```bash
eval "$(~/.local/bin/mise activate bash)"
source "$HOME/.cargo/env"

# Tools not available in the official Arch repos
cargo install yazi-fm topgrade cargo-update cargo-cache
```

Yazi optional dependencies:

```bash
sudo pacman -S --noconfirm --needed \
  file unzip jq ffmpegthumbnailer imagemagick
```

---

### 6. Bat — Tokyo Night theme

```bash
mkdir -p ~/.config/bat/themes
curl -o ~/.config/bat/themes/tokyonight_night.tmTheme \
  https://raw.githubusercontent.com/folke/tokyonight.nvim/main/extras/sublime/tokyonight_night.tmTheme
bat cache --build
```

---

### 7. ZSH — Plugins

```bash
mkdir -p ~/.zsh/plugins

git clone --depth=1 https://github.com/Aloxaf/fzf-tab \
  ~/.zsh/plugins/fzf-tab

git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions \
  ~/.zsh/plugins/zsh-autosuggestions

git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting \
  ~/.zsh/plugins/zsh-syntax-highlighting
```

---

### 8. Apply dotfiles

```bash
cp ~/dotfiles/config/zshrc ~/.zshrc
cp ~/dotfiles/config/gitconfig ~/.gitconfig
cp ~/dotfiles/config/topgrade.toml ~/.config/topgrade.toml

mkdir -p ~/.config/Code/User
cp ~/dotfiles/vscode/settings.json ~/.config/Code/User/settings.json

# Apply the official Starship tokyo-night preset
# This replaces the custom starship.toml — no manual config needed
starship preset tokyo-night -o ~/.config/starship.toml
```

Reload shell:

```bash
exec zsh
```

---

### 9. LazyVim

```bash
# Backup existing config if any
mv ~/.config/nvim ~/.config/nvim.bak 2>/dev/null || true

# Install LazyVim starter
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git

# Open nvim — plugins install automatically on first launch
nvim
```

Apply Tokyo Night theme at `~/.config/nvim/lua/plugins/colorscheme.lua`:

```lua
return {
  {
    "folke/tokyonight.nvim",
    opts = { style = "night" },
  },
  {
    "LazyVim/LazyVim",
    opts = { colorscheme = "tokyonight-night" },
  },
}
```

---

### 10. Stack verification

```bash
mise --version
starship --version
nvim --version
lazygit --version
eza --version
bat --version
rg --version
fd --version
delta --version
topgrade --version
echo $WAYLAND_DISPLAY   # should return wayland-0 (injected by WSLg automatically)
wl-copy --version       # should return without error
```

---

## Installation — macOS (ARM)

### 0. Prerequisites

- macOS 13 Ventura or later (Apple Silicon)
- Xcode Command Line Tools

```bash
xcode-select --install
```

---

### 1. Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

On Apple Silicon, Homebrew installs to `/opt/homebrew`. Add to PATH for the current session:

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```

This will be permanent once the dotfiles are applied via `~/.zshrc`.

---

### 2. Base packages

```bash
brew update

brew install git curl fzf zsh openssh \
  eza bat ripgrep fd zoxide git-delta \
  starship lazygit neovim
```

> macOS clipboard works natively via `pbcopy`/`pbpaste`. Neovim and VSCodeVim detect this automatically — `wl-clipboard` is not needed.

---

### 3. Default shell

macOS ships with ZSH as default since Catalina. Verify:

```bash
echo $SHELL
# Expected: /bin/zsh or /opt/homebrew/bin/zsh
```

To use the Homebrew ZSH (more up-to-date than the system one):

```bash
echo "/opt/homebrew/bin/zsh" | sudo tee -a /etc/shells
chsh -s /opt/homebrew/bin/zsh
```

Restart the terminal to apply.

---

### 4. Mise (runtimes)

```bash
curl https://mise.run | sh

mkdir -p ~/.config/mise
cp ~/dotfiles/config/mise.toml ~/.config/mise/config.toml

~/.local/bin/mise install
```

---

### 5. Tooling via Brew

Everything with a Homebrew formula goes via brew — faster and integrated with topgrade:

```bash
# Yazi + optional dependencies (brew resolves everything together)
brew install yazi ffmpeg sevenzip jq poppler resvg imagemagick

# Update management
brew install topgrade cargo-update
```

`cargo-cache` has no brew formula — install via cargo after activating mise:

```bash
eval "$(~/.local/bin/mise activate bash)"
cargo install cargo-cache
```

---

### 6. Bat — Tokyo Night theme

```bash
mkdir -p ~/.config/bat/themes
curl -o ~/.config/bat/themes/tokyonight_night.tmTheme \
  https://raw.githubusercontent.com/folke/tokyonight.nvim/main/extras/sublime/tokyonight_night.tmTheme
bat cache --build
```

---

### 7. ZSH — Plugins

Same commands as Arch:

```bash
mkdir -p ~/.zsh/plugins

git clone --depth=1 https://github.com/Aloxaf/fzf-tab \
  ~/.zsh/plugins/fzf-tab

git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions \
  ~/.zsh/plugins/zsh-autosuggestions

git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting \
  ~/.zsh/plugins/zsh-syntax-highlighting
```

---

### 8. Apply dotfiles

```bash
cp ~/dotfiles/config/zshrc ~/.zshrc
cp ~/dotfiles/config/gitconfig ~/.gitconfig
cp ~/dotfiles/config/topgrade.toml ~/.config/topgrade.toml

mkdir -p ~/Library/Application\ Support/Code/User
cp ~/dotfiles/vscode/settings.json \
  ~/Library/Application\ Support/Code/User/settings.json

# Apply the official Starship tokyo-night preset
# This replaces the custom starship.toml — no manual config needed
starship preset tokyo-night -o ~/.config/starship.toml
```

Reload shell:

```bash
exec zsh
```

---

### 9. LazyVim

```bash
mv ~/.config/nvim ~/.config/nvim.bak 2>/dev/null || true

git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git

nvim
```

Apply Tokyo Night theme at `~/.config/nvim/lua/plugins/colorscheme.lua`:

```lua
return {
  {
    "folke/tokyonight.nvim",
    opts = { style = "night" },
  },
  {
    "LazyVim/LazyVim",
    opts = { colorscheme = "tokyonight-night" },
  },
}
```

---

### 10. VS Code

```bash
brew install --cask visual-studio-code

code --install-extension rust-lang.rust-analyzer
code --install-extension charliermarsh.ruff
code --install-extension esbenp.prettier-vscode
code --install-extension vscodevim.vim
code --install-extension PKief.material-icon-theme
code --install-extension enkia.tokyo-night
```

---

### 11. Font — JetBrains Mono Nerd Font

```bash
brew install --cask font-jetbrains-mono-nerd-font
```

After installing, configure your terminal (iTerm2, Ghostty, or Terminal.app) to use `JetBrainsMono Nerd Font`.

---

### 12. Stack verification

```bash
brew doctor
mise --version
starship --version
nvim --version
lazygit --version
eza --version
bat --version
rg --version
fd --version
delta --version
topgrade --version
echo $SHELL
```

---

### macOS vs Arch Linux (WSL2) differences

| Item | Arch (WSL2) | macOS (ARM) |
|---|---|---|
| Package manager | `pacman` | `brew` |
| Clipboard | `wl-clipboard` via WSLg | `pbcopy`/`pbpaste` native |
| Default shell | Set via `chsh` | ZSH already default |
| Systemd / wsl.conf | Required | Not applicable |
| Nerd Font | Manual install | `brew install --cask` |
| VS Code | WSL symlink | `brew install --cask` |
| `yazi` deps | `pacman -S ffmpegthumbnailer...` | `brew install ffmpeg sevenzip...` |
| `topgrade` | Via `cargo` | Via `brew` |
| `cargo-update` | Via `cargo` | Via `brew` |

---

## Maintenance

```bash
topgrade              # updates everything: pacman/brew + cargo + mise + ZSH plugins
topgrade --only cargo
topgrade --only mise
```

---

## VS Code — Extensions

```
rust-lang.rust-analyzer
charliermarsh.ruff
esbenp.prettier-vscode
vscodevim.vim
PKief.material-icon-theme
enkia.tokyo-night
```

---

## Keybindings (VSCodeVim)

| Binding | Action |
|---|---|
| `<leader>e` | Toggle sidebar |
| `<leader>ff` | Fuzzy find file |
| `<leader>fg` | Find in files |
| `<leader>fr` | Recent files |
| `<leader>bd/bn/bp` | Close / next / previous buffer |
| `gd` / `gD` | Go to / Peek definition |
| `gr` / `gi` / `gh` | References / Implementation / Hover |
| `<leader>ca/cr/cs` | Code action / Rename / Symbol |
| `<leader>cd` | Problems panel |
| `]d` / `[d` | Next / previous diagnostic |
| `<leader>wv/ws` | Split vertical / horizontal |
| `<leader>wh/l/k/j` | Navigate between splits |
| `<leader>tt/tn` | Toggle / new terminal |
| `<leader>mf` | Format document |
| `<leader>gs` | Source Control view |
| `<leader>h` | Clear search highlight |
