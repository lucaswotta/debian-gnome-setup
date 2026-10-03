# Fase 6 - GNOME

**Status:** concluída · **Escopo:** depende do perfil de uso. Extensões, atalhos e visual são preferências

[← Fase 5](05-aplicativos.md) · [Roteiro](../02-roteiro.md) · [Fase 7 →](07-backup.md)

Objetivo: deixar o GNOME 48 confortável para quem vem do Windows, com poucas extensões e sem trocar o visual padrão.

- [x] **Extensões:** pacotes do Debian instalados para AppIndicator (bandeja), Dash to Dock (barra de aplicativos), Caffeine (impede a suspensão), Tiling Assistant (encaixe de janelas) e Blur my Shell (desfoque), mais a Copyous, do site de extensões. Só o **Blur my Shell** e a **Copyous** ficam ativos; as demais ficam instaladas e desligadas, à disposição pra religar sob demanda.
- [x] **Área de transferência:** histórico no `Super+V`, como o `Win+V` do Windows, com a extensão Copyous. O que foi copiado continua disponível depois que o programa de origem fecha.
- [x] **Janelas:** botões de minimizar e maximizar ao lado do fechar.
- [x] **Relógio:** bateria em porcentagem e dia da semana.
- [x] **Atalhos:** `Super+E` abre o Arquivos, `Super+D` mostra a área de trabalho e `Ctrl+Alt+T` abre o terminal.
- [x] **Arquivos:** visualização em lista. Nas janelas de abrir e salvar, pastas antes dos arquivos.
- [x] **Visual:** tema escuro e destaque verde, com a fonte Inter na interface e nos títulos das janelas e o cursor Bibata.
- [x] **Ícones:** Papirus, pelo Debian, na variante escura, com as pastas em verde.
- [x] **Favoritos da barra de aplicativos:** Chrome, Arquivos e Terminal.

```bash
# 1. Extensões, pelo Debian (o pacote de preferências vem como dependência)
sudo apt-get install -y gnome-shell-extension-appindicator gnome-shell-extension-dashtodock gnome-shell-extension-caffeine \
  gnome-shell-extension-tiling-assistant gnome-shell-extension-blur-my-shell

# 2. Copyous (histórico da área de transferência), do site de extensões, na pasta do usuário
sudo apt-get install -y gir1.2-gda-5.0 gir1.2-gsound-1.0
curl -fLo copyous.zip 'https://extensions.gnome.org/download-extension/copyous@boerdereinar.dev.shell-extension.zip?shell_version=48'
gnome-extensions install copyous.zip

# 3. Ligar só o Blur my Shell e a Copyous (vale a partir da próxima sessão; as outras ficam desligadas)
gsettings set org.gnome.shell enabled-extensions "['blur-my-shell@aunetx', 'copyous@boerdereinar.dev']"

# 4. Histórico no Super+V, notificações só no Super+M e sem prévia de links
gsettings set org.gnome.shell.keybindings toggle-message-tray "['<Super>m']"
C="gsettings --schemadir $HOME/.local/share/gnome-shell/extensions/copyous@boerdereinar.dev/schemas"
$C set org.gnome.shell.extensions.copyous open-clipboard-dialog-shortcut "['<Super>v']"
$C set org.gnome.shell.extensions.copyous.link-item show-link-preview false
$C set org.gnome.shell.extensions.copyous.link-item show-link-preview-image false

# 5. Janelas e relógio
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'
gsettings set org.gnome.desktop.interface clock-show-weekday true
gsettings set org.gnome.desktop.interface show-battery-percentage true

# 6. Atalhos (o GNOME 48 não traz atalho de terminal: é um atalho personalizado)
gsettings set org.gnome.settings-daemon.plugins.media-keys home "['<Super>e']"
gsettings set org.gnome.desktop.wm.keybindings show-desktop "['<Super>d']"
K=/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['$K']"
S="org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:$K"
gsettings set $S name 'Terminal'
gsettings set $S command 'gnome-terminal'
gsettings set $S binding '<Primary><Alt>t'

# 7. Favoritos da barra de aplicativos
gsettings set org.gnome.shell favorite-apps \
  "['google-chrome.desktop', 'org.gnome.Nautilus.desktop', 'org.gnome.Terminal.desktop']"

# 8. Fonte e cursor, pelo Debian
sudo apt-get install -y fonts-inter bibata-cursor-theme
gsettings set org.gnome.desktop.interface font-name 'Inter 11'
gsettings set org.gnome.desktop.interface document-font-name 'Inter 11'
gsettings set org.gnome.desktop.wm.preferences titlebar-font 'Inter Bold 11'
gsettings set org.gnome.desktop.interface cursor-theme 'Bibata-Modern-Classic'

# 9. Arquivos
gsettings set org.gnome.nautilus.preferences default-folder-viewer 'list-view'
gsettings set org.gtk.Settings.FileChooser sort-directories-first true
gsettings set org.gtk.gtk4.Settings.FileChooser sort-directories-first true

# 10. Ícones, pelo Debian
sudo apt-get install -y papirus-icon-theme
gsettings set org.gnome.desktop.interface icon-theme 'Papirus-Dark'
```

Como conferir:

```bash
gsettings get org.gnome.shell enabled-extensions
gnome-extensions list --enabled                    # depois de uma sessão nova
gsettings get org.gnome.shell favorite-apps
gsettings get org.gnome.desktop.wm.preferences button-layout
gsettings list-recursively | grep -i '<Super>e'    # o atalho não pode aparecer em outra ação
```

Como desfazer: `gsettings reset <esquema> <chave>` para cada chave acima, `gsettings reset org.gnome.shell enabled-extensions` para desligar todas as extensões,
`gnome-extensions uninstall copyous@boerdereinar.dev` e `sudo apt remove gir1.2-gda-5.0 gir1.2-gsound-1.0` para remover a Copyous e
`sudo apt remove gnome-shell-extension-appindicator gnome-shell-extension-dashtodock gnome-shell-extension-caffeine gnome-shell-extension-tiling-assistant gnome-shell-extension-blur-my-shell` para remover as do Debian.

Observações:

- **Extensões no Wayland:** o GNOME só carrega extensões novas em uma sessão nova; para *desligar* uma já carregada não precisa. Saia e entre de novo para ativar uma extensão nova.
- **Pacotes do Debian:** todas declaram suporte ao GNOME 48. No Debian, o AppIndicator se chama `ubuntu-appindicators@ubuntu.com`.
- **Só o Blur my Shell e a Copyous ficam ligados:** as outras quatro extensões instaladas ficam desativadas por escolha, para manter o GNOME o mais perto do padrão. Religar com `gnome-extensions enable <uuid>`: AppIndicator (`ubuntu-appindicators@ubuntu.com`), Dash to Dock (`dash-to-dock@micxgx.gmail.com`), Caffeine (`caffeine@patapon.info`) e Tiling Assistant (`tiling-assistant@leleat-on-github`).
- **Dash to Dock, se religada:** os padrões já lembram a barra do Windows (embaixo, clique alterna as janelas do aplicativo, ícone da lixeira e dos discos).
  A barra se esconde quando uma janela a cobre (`intellihide`). Para deixá-la sempre visível: `gsettings set org.gnome.shell.extensions.dash-to-dock dock-fixed true`.
- **Copyous:** a janela abre no ponteiro do mouse, guarda texto, imagens e arquivos, e os itens fixados não saem do histórico. Preferências pelo ícone na barra superior ou por `gnome-extensions prefs copyous@boerdereinar.dev`. A prévia de links fica desligada porque, ligada, a extensão baixa cada endereço copiado.
- **Atalho das notificações:** o GNOME usa `Super+V` e `Super+M` para a lista de notificações (`toggle-message-tray`). Com o `Super+V` na Copyous, ela fica só no `Super+M`. O `Super+N` fica de fora porque já foca a notificação ativa.
- **Seleção e `Ctrl+V`:** selecionar um texto o coloca na seleção primária, colada com o botão do meio, sem mudar o que o `Ctrl+V` cola. O GPaste fica de fora: o serviço dele (`gpaste-daemon`) continua rodando com a extensão desligada e, com `synchronize-clipboards` ligado, faz cada seleção substituir o conteúdo do `Ctrl+V`.
- **Cursor nos Flatpaks:** aplicativos Flatpak não enxergam os cursores do sistema e mantêm o padrão, a menos que se libere a pasta de ícones para eles.
- **Pastas verdes:** no Papirus as pastas são azuis por padrão. Uma variante de terceiros com pastas verdes (`papirus-icon-theme-green-folders-dark`, em `~/.local/share/icons/`) herda os ícones do pacote, então o `papirus-icon-theme` precisa continuar instalado.
- **Terminal:** o `gnome-terminal` é o instalado. O `kgx` (Console) e o `ptyxis` não estão presentes.
- **Visualização do Arquivos:** o Arquivos regrava `default-folder-viewer` ao ser usado. Se a lista voltar a ícones, repita o comando com o Arquivos fechado.
- **Pastas primeiro:** o Nautilus 48 não tem chave para isso. A opção existe só nas janelas de abrir e salvar arquivos (GTK).
- **Touchpad:** toque para clicar, rolagem natural e rolagem com dois dedos já vêm ligados, e o clique é por número de dedos.

## Limitação conhecida: rótulos da grade de aplicativos

Ao trocar de página na grade de aplicativos (rodinha do mouse ou gesto), o rótulo de alguns ícones pode ficar maior e em negrito, destoando dos vizinhos, e só volta ao normal numa sessão nova (logout/login). Reproduz com todas as extensões desativadas: não é causado por nenhuma delas.

É uma regressão do GNOME Shell em telas com **escalas diferentes por monitor** com escalonamento fracionário ligado (aqui: notebook a 125%, monitor externo a 100%, `scale-monitor-framebuffer` em `org.gnome.mutter experimental-features`). Sem correção; outras pessoas relatam o mesmo em versões mais novas do GNOME, e os únicos contornos conhecidos exigem igualar a escala dos monitores.

<!-- TODO: colar aqui o link da issue no GitLab do GNOME Shell quando for aberta -->

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
