# Fase 6 - GNOME

**Status:** concluída · **Escopo:** depende do perfil de uso. Extensões, atalhos e visual são preferências

[← Fase 5](05-aplicativos.md) · [Roteiro](../02-roteiro.md) · [Fase 7 →](07-backup.md)

Objetivo: deixar o GNOME 48 confortável para quem vem do Windows, com poucas extensões e sem trocar o visual padrão.

- [x] **Extensões:** AppIndicator (ícones de bandeja), Dash to Dock (barra de aplicativos) e Caffeine (impede a suspensão), pelos pacotes do Debian.
- [x] **Janelas:** botões de minimizar e maximizar ao lado do fechar.
- [x] **Relógio:** bateria em porcentagem e dia da semana.
- [x] **Atalhos:** `Super+E` abre o Arquivos, `Super+D` mostra a área de trabalho e `Ctrl+Alt+T` abre o terminal.
- [x] **Arquivos:** visualização em lista. Nas janelas de abrir e salvar, pastas antes dos arquivos.
- [x] **Visual:** tema escuro e destaque verde, com a fonte Inter na interface e nos títulos das janelas e o cursor Bibata.
- [x] **Extensões extras:** GPaste (histórico da área de transferência), Tiling Assistant (encaixe de janelas) e Blur my Shell (desfoque).
- [x] **Atalho do histórico:** `Super+V` abre o GPaste. A lista de notificações fica no `Super+M`.
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

# 6. Fonte e cursor, pelo Debian
sudo apt-get install -y fonts-inter bibata-cursor-theme
gsettings set org.gnome.desktop.interface font-name 'Inter 11'
gsettings set org.gnome.desktop.interface document-font-name 'Inter 11'
gsettings set org.gnome.desktop.wm.preferences titlebar-font 'Inter Bold 11'
gsettings set org.gnome.desktop.interface cursor-theme 'Bibata-Modern-Classic'

# 7. Extensões extras e o atalho do histórico
sudo apt-get install -y gnome-shell-extension-gpaste gnome-shell-extension-tiling-assistant gnome-shell-extension-blur-my-shell
gsettings set org.gnome.shell enabled-extensions \
  "['ubuntu-appindicators@ubuntu.com', 'dash-to-dock@micxgx.gmail.com', 'caffeine@patapon.info', 'GPaste@gnome-shell-extensions.gnome.org', 'tiling-assistant@leleat-on-github', 'blur-my-shell@aunetx']"
gsettings set org.gnome.shell.keybindings toggle-message-tray "['<Super>m']"
gsettings set org.gnome.GPaste show-history '<Super>v'

# 8. Arquivos
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
- **Atalho do GPaste:** a chave `show-history` é do tipo texto (`'<Super>v'`), e não lista. O GNOME já usa `Super+V` na lista de notificações (`toggle-message-tray`),
  então esse atalho é reduzido a `Super+M`. O `Super+N` fica de fora porque já foca a notificação ativa.
- **Cursor nos Flatpaks:** aplicativos Flatpak não enxergam os cursores do sistema e mantêm o padrão, a menos que se libere a pasta de ícones para eles.
- **Terminal:** o `gnome-terminal` é o instalado. O `kgx` (Console) e o `ptyxis` não estão presentes.
- **Visualização do Arquivos:** o Arquivos regrava `default-folder-viewer` ao ser usado. Se a lista voltar a ícones, repita o comando com o Arquivos fechado.
- **Pastas primeiro:** o Nautilus 48 não tem chave para isso. A opção existe só nas janelas de abrir e salvar arquivos (GTK).
- **Touchpad:** toque para clicar, rolagem natural e rolagem com dois dedos já vêm ligados, e o clique é por número de dedos.

## Personalizar por conta própria

O Debian e o GNOME permitem trocar extensões, ícones, cursores e papéis de parede sem `sudo`, com exceção dos pacotes do Debian.

### Extensões

| Jeito | Como | Quando usar |
|---|---|---|
| Gerenciador de extensões | `sudo apt install gnome-shell-extension-manager`. Na aba *Navegar*, procure e instale do site extensions.gnome.org | O mais simples |
| Pacote do Debian | `apt search gnome-shell-extension` e `sudo apt install <pacote>` | O mais seguro: testado para a versão do GNOME e atualizado com o sistema |
| Arquivo `.zip` | `gnome-extensions install arquivo.zip` | Extensão baixada de outro lugar |

- **Listar:** `gnome-extensions list` mostra o nome (UUID) de cada uma, e `gnome-extensions list --enabled` mostra as ativas.
- **Ligar e desligar:** pelo Gerenciador, ou com `gnome-extensions enable <uuid>` e `gnome-extensions disable <uuid>`. Uma extensão nova só carrega em uma sessão nova, por causa do Wayland.
- **Remover:** as instaladas pelo usuário saem pelo Gerenciador ou por `gnome-extensions uninstall <uuid>`. As do Debian saem por `sudo apt remove --autoremove <pacote>`.
- **Cuidado:** uma extensão roda dentro do GNOME Shell, com acesso total à sessão. Prefira as do Debian e as de autores conhecidos. As do site oficial podem parar de funcionar quando o GNOME é atualizado.
- **Desligar todas:** `gsettings set org.gnome.shell enabled-extensions "[]"` *(proposta)*.

### Ícones, cursores e temas

- **Escolher:** o aplicativo Ajustes (GNOME Tweaks), na seção *Aparência*, lista ícones, cursor e tema. O tema do Shell exige a extensão *User Themes*.
- **Pelo Debian:** `apt search icon-theme` lista os conjuntos de ícones, como `papirus-icon-theme`, `numix-icon-theme` e `yaru-theme-icon`. Para cursores, `bibata-cursor-theme`, `breeze-cursor-theme` e `dmz-cursor-theme`.
- **Por conta própria:** descompacte o tema em `~/.local/share/icons/` (ícones e cursores) ou em `~/.local/share/themes/` (temas GTK). Ele aparece no Ajustes.
- **Voltar ao padrão:** `gsettings reset org.gnome.desktop.interface icon-theme` (ou `cursor-theme`, ou `gtk-theme`).
- **Limites:** os aplicativos novos do GNOME ignoram temas GTK de terceiros, e ícones e cursores funcionam. Aplicativos Flatpak só enxergam temas com uma permissão extra.

### Papéis de parede

- **Já no sistema:** o pacote `gnome-backgrounds` traz o conjunto do GNOME em `/usr/share/backgrounds/gnome/`, e o `desktop-base`, as artes do Debian em `/usr/share/desktop-base/`.
- **Outros pacotes:** `mate-backgrounds` e `budgie-backgrounds` trazem os fundos desses ambientes.
- **Na internet:** repositórios de imagens com licença aberta, como Unsplash, Pexels e Pixabay, e as imagens de domínio público da NASA. Confira a licença antes de redistribuir.
- **Definir:** *Configurações > Aparência*, no plano de fundo, com *Adicionar imagem*. Também vale o botão direito na imagem, no aplicativo Arquivos, em *Definir como plano de fundo*.
- **Tamanho:** para a tela de referência, de 1920x1080, use imagens 16:9 com essa resolução ou maior.
- **Pela linha de comando:** `gsettings set org.gnome.desktop.background picture-uri 'file:///caminho/imagem.jpg'`, e `picture-uri-dark` para o tema escuro.
