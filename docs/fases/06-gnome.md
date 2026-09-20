# Fase 6 - GNOME

**Status:** concluída · **Escopo:** depende do perfil de uso. Extensões, atalhos e visual são preferências

[← Fase 5](05-aplicativos.md) · [Roteiro](../02-roteiro.md) · [Fase 7 →](07-backup.md)

Objetivo: deixar o GNOME 48 confortável para quem vem do Windows, com poucas extensões e sem trocar o visual padrão.

- [x] **Extensões:** AppIndicator (ícones de bandeja), Dash to Dock (barra de aplicativos) e Caffeine (impede a suspensão), pelos pacotes do Debian.
- [x] **Janelas:** botões de minimizar e maximizar ao lado do fechar.
- [x] **Relógio:** bateria em porcentagem e dia da semana.
- [x] **Atalhos:** `Super+E` abre o Arquivos, `Super+D` mostra a área de trabalho e `Ctrl+Alt+T` abre o terminal.
- [x] **Arquivos:** visualização em lista. Nas janelas de abrir e salvar, pastas antes dos arquivos.
- [x] **Visual:** mantido o tema escuro, o destaque verde e a fonte Cantarell.
- [x] **Extensões carregadas:** as três ficam com o estado `ACTIVE` em uma sessão nova.
- [x] **Favoritos da barra de aplicativos:** Chrome, Arquivos, Terminal, VS Code, DBeaver, Postman e Discord.

```bash
# 1. Extensões, pelo Debian (o pacote de preferências vem como dependência)
sudo apt-get install -y gnome-shell-extension-appindicator gnome-shell-extension-dashtodock gnome-shell-extension-caffeine

# 2. Ligar as três (vale a partir da próxima sessão)
gsettings set org.gnome.shell enabled-extensions \
  "['ubuntu-appindicators@ubuntu.com', 'dash-to-dock@micxgx.gmail.com', 'caffeine@patapon.info']"

# 3. Janelas e relógio
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'
gsettings set org.gnome.desktop.interface clock-show-weekday true
gsettings set org.gnome.desktop.interface show-battery-percentage true

# 4. Atalhos (o GNOME 48 não traz atalho de terminal: é um atalho personalizado)
gsettings set org.gnome.settings-daemon.plugins.media-keys home "['<Super>e']"
gsettings set org.gnome.desktop.wm.keybindings show-desktop "['<Super>d']"
K=/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['$K']"
S="org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:$K"
gsettings set $S name 'Terminal'
gsettings set $S command 'gnome-terminal'
gsettings set $S binding '<Primary><Alt>t'

# 5. Favoritos da barra de aplicativos
gsettings set org.gnome.shell favorite-apps \
  "['google-chrome.desktop', 'org.gnome.Nautilus.desktop', 'org.gnome.Terminal.desktop', 'code.desktop', 'dbeaver-ce.desktop', 'com.getpostman.Postman.desktop', 'com.discordapp.Discord.desktop']"

# 6. Arquivos
gsettings set org.gnome.nautilus.preferences default-folder-viewer 'list-view'
gsettings set org.gtk.Settings.FileChooser sort-directories-first true
gsettings set org.gtk.gtk4.Settings.FileChooser sort-directories-first true
```

Como conferir:

```bash
gsettings get org.gnome.shell enabled-extensions
gnome-extensions list --enabled                    # depois de uma sessão nova
gsettings get org.gnome.shell favorite-apps
gsettings get org.gnome.desktop.wm.preferences button-layout
gsettings list-recursively | grep -i '<Super>e'    # o atalho não pode aparecer em outra ação
```

Como desfazer: `gsettings reset <esquema> <chave>` para cada chave acima, `gsettings reset org.gnome.shell enabled-extensions` para desligar as extensões e
`sudo apt remove gnome-shell-extension-appindicator gnome-shell-extension-dashtodock gnome-shell-extension-caffeine` para removê-las.

Observações:

- **Extensões no Wayland:** o GNOME só carrega extensões novas em uma sessão nova. Saia e entre de novo.
- **Pacotes do Debian:** as três declaram suporte ao GNOME 48. No Debian, o AppIndicator se chama `ubuntu-appindicators@ubuntu.com`.
- **Dash to Dock:** os padrões já lembram a barra do Windows (embaixo, clique alterna as janelas do aplicativo, ícone da lixeira e dos discos).
  A barra se esconde quando uma janela a cobre (`intellihide`). Para deixá-la sempre visível: `gsettings set org.gnome.shell.extensions.dash-to-dock dock-fixed true`.
- **Terminal:** o `gnome-terminal` é o instalado. O `kgx` (Console) e o `ptyxis` não estão presentes.
- **Visualização do Arquivos:** o Arquivos regrava `default-folder-viewer` ao ser usado. Se a lista voltar a ícones, repita o comando com o Arquivos fechado.
- **Pastas primeiro:** o Nautilus 48 não tem chave para isso. A opção existe só nas janelas de abrir e salvar arquivos (GTK).
- **Touchpad:** toque para clicar, rolagem natural e rolagem com dois dedos já vêm ligados, e o clique é por número de dedos.
