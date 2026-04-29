# zen-dots

> Keyboard-first. Visually silent. Contextually aware.

Dotfiles para um ambiente de desenvolvimento minimalista e reproduzível.
Testado em **Arch Linux (WSL2)** e **macOS (ARM)**.

---

## Filosofia

Cada ferramenta tem uma responsabilidade única — sem sobreposição:

| Camada | Responsabilidade |
|---|---|
| **ZSH** | Ambiente: aliases, pagers, editor, keybinds |
| **Starship** | Contexto em tempo real: git, runtimes, exit codes, vi-mode |
| **Mise** | Runtimes: node, python, rust, go |
| **VS Code** | Editor: LSP, diagnósticos, formatters, Vim motions |
| **LazyVim** | Editor de terminal: edição rápida, git, arquivos remotos |
| **Topgrade** | Manutenção: atualiza todo o stack com um comando |

---

## Estrutura do Repositório

```
zen-dots/
├── config/
│   ├── zshrc          → ~/.zshrc
│   ├── starship.toml  → ~/.config/starship.toml
│   ├── gitconfig      → ~/.gitconfig
│   ├── mise.toml      → ~/.config/mise/config.toml
│   └── topgrade.toml  → ~/.config/topgrade.toml
└── vscode/
    └── settings.json  → ~/.config/Code/User/settings.json       (Linux/WSL)
                       → ~/Library/Application Support/Code/User/ (macOS)
```

---

## Instalação — Arch Linux (WSL2)

### 0. Pré-requisitos

- Windows 11 (WSL 2.4.4+)
- PowerShell com privilégios de administrador
- Virtualização habilitada na BIOS

---

### 1. Instalação do Arch Linux

```powershell
# PowerShell (Admin)
wsl --install archlinux
```

A distribuição abre como root. Defina a senha do root:

```bash
passwd
```

---

### 2. Pacotes base e usuário

```bash
# Atualizar sistema
pacman -Syu --noconfirm

# Pacotes base + ferramentas via pacman
pacman -S --noconfirm --needed \
  base-devel git curl wget unzip zsh fzf man-db less \
  eza bat ripgrep fd zoxide git-delta starship lazygit \
  wl-clipboard neovim
```

Criar usuário com ZSH como shell padrão desde o início:

```bash
useradd -m -G wheel -s /bin/zsh seu_usuario
passwd seu_usuario
echo "%wheel ALL=(ALL:ALL) NOPASSWD: ALL" > /etc/sudoers.d/wheel
chmod 440 /etc/sudoers.d/wheel
```

---

### 3. Locale e `/etc/wsl.conf`

```bash
# Locale
sed -i 's/#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# WSL config
cat <<EOF > /etc/wsl.conf
[automount]
enabled = true
options = "metadata,umask=22,fmask=11"

[network]
generateResolvConf = true
hostname = arch-dev

[interop]
enabled = true
appendWindowsPath = false

[user]
default = seu_usuario

[boot]
systemd = true
EOF
```

Reiniciar para aplicar:

```powershell
# PowerShell
wsl --terminate archlinux
wsl -d archlinux
```

> A partir daqui, todos os comandos são executados como `seu_usuario`.

---

### 4. Mise (runtimes)

```bash
curl https://mise.run | sh

mkdir -p ~/.config/mise
cp ~/dotfiles/config/mise.toml ~/.config/mise/config.toml

# Ativa mise temporariamente para instalar runtimes
~/.local/bin/mise install
```

---

### 5. Tooling via Cargo

Mise já instalou o Rust. Ative o cargo do mise antes de continuar:

```bash
eval "$(~/.local/bin/mise activate bash)"
source "$HOME/.cargo/env"

# Ferramentas não disponíveis no pacman oficial
cargo install yazi-fm topgrade
```

Dependências do yazi:

```bash
sudo pacman -S --noconfirm --needed \
  file unzip jq ffmpegthumbnailer imagemagick
```

---

### 6. Bat — tema Tokyo Night

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

### 8. Aplicar dotfiles

```bash
# ZSH
cp ~/dotfiles/config/zshrc ~/.zshrc

# Starship
mkdir -p ~/.config
cp ~/dotfiles/config/starship.toml ~/.config/starship.toml

# Git
cp ~/dotfiles/config/gitconfig ~/.gitconfig

# Topgrade
cp ~/dotfiles/config/topgrade.toml ~/.config/topgrade.toml

# VS Code (WSL/Linux)
mkdir -p ~/.config/Code/User
cp ~/dotfiles/vscode/settings.json ~/.config/Code/User/settings.json
```

Recarregar shell:

```bash
exec zsh
```

---

### 9. LazyVim

```bash
# Backup de config existente (se houver)
mv ~/.config/nvim ~/.config/nvim.bak 2>/dev/null || true

# Instalar LazyVim
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git

# Abrir nvim — plugins instalam automaticamente na primeira abertura
nvim
```

Aplicar tema Tokyo Night em `~/.config/nvim/lua/plugins/colorscheme.lua`:

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

### 10. Verificação do stack

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
echo $WAYLAND_DISPLAY   # deve retornar wayland-0 (WSLg injeta automaticamente)
wl-copy --version       # deve retornar sem erro
```

---

## Instalação — macOS (ARM)

```bash
# Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Pacotes base
brew install git curl fzf zsh neovim \
  eza bat ripgrep fd zoxide git-delta starship lazygit

# Mise
curl https://mise.run | sh

mkdir -p ~/.config/mise
cp ~/dotfiles/config/mise.toml ~/.config/mise/config.toml
~/.local/bin/mise install

# Cargo (via mise rust)
eval "$(~/.local/bin/mise activate bash)"
cargo install yazi-fm topgrade

# Bat tema
mkdir -p ~/.config/bat/themes
curl -o ~/.config/bat/themes/tokyonight_night.tmTheme \
  https://raw.githubusercontent.com/folke/tokyonight.nvim/main/extras/sublime/tokyonight_night.tmTheme
bat cache --build

# Plugins ZSH (mesmos da seção 7)

# Dotfiles
cp ~/dotfiles/config/zshrc ~/.zshrc
cp ~/dotfiles/config/starship.toml ~/.config/starship.toml
cp ~/dotfiles/config/gitconfig ~/.gitconfig
cp ~/dotfiles/config/topgrade.toml ~/.config/topgrade.toml

mkdir -p ~/Library/Application\ Support/Code/User
cp ~/dotfiles/vscode/settings.json \
  ~/Library/Application\ Support/Code/User/settings.json

# LazyVim (mesmos passos da seção 9)
exec zsh
```

---

## Manutenção

```bash
topgrade              # atualiza tudo: pacman + cargo + mise + plugins ZSH
topgrade --only cargo
topgrade --only mise
```

---

## VS Code — Extensões

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

| Binding | Ação |
|---|---|
| `<leader>e` | Toggle sidebar |
| `<leader>ff` | Fuzzy find arquivo |
| `<leader>fg` | Find in files |
| `<leader>fr` | Arquivos recentes |
| `<leader>bd/bn/bp` | Fechar / próximo / anterior buffer |
| `gd` / `gD` | Go to / Peek definition |
| `gr` / `gi` / `gh` | References / Implementation / Hover |
| `<leader>ca/cr/cs` | Code action / Rename / Symbol |
| `<leader>cd` | Problems panel |
| `]d` / `[d` | Próximo / anterior diagnóstico |
| `<leader>wv/ws` | Split vertical / horizontal |
| `<leader>wh/l/k/j` | Navegar entre splits |
| `<leader>tt/tn` | Toggle / novo terminal |
| `<leader>mf` | Format document |
| `<leader>gs` | Source Control view |
| `<leader>h` | Clear search highlight |
