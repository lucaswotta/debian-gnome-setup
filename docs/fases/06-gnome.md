# Fase 6 - GNOME

**Status:** concluída · **Escopo:** depende do perfil de uso. Extensões, atalhos e visual são preferências

[← Fase 5](05-aplicativos.md) · [Roteiro](../02-roteiro.md) · [Fase 7 →](07-backup.md)

Objetivo: deixar o GNOME 48 confortável para quem vem do Windows, com poucas extensões e sem trocar o visual padrão.

- [x] **Extensões:** AppIndicator (bandeja) e Blur my Shell (desfoque), pelo Debian, mais Copyous (área de transferência), Tiling Shell (encaixe de janelas), Just Perfection (ajustes do Shell), Custom Hot Corners - Extended (cantos ativos), Vertical App Grid (grade com rolagem vertical) e a extensão complementar do Smile (emojis), do site de extensões. As oito ficam ativas.
- [x] **Área de transferência:** histórico no `Super+V`, como o `Win+V` do Windows, com a extensão Copyous. O que foi copiado continua disponível depois que o programa de origem fecha.
- [x] **Emojis:** `Super+.` abre o Smile, que cola o emoji escolhido no campo em uso, como o `Win+.` do Windows.
- [x] **Cantos ativos:** o canto superior esquerdo abre a visão geral, e o inferior esquerdo, a grade de aplicativos.
- [x] **Janelas:** botões de minimizar e maximizar ao lado do fechar. Encaixe no estilo do Windows 11: arrastar até o topo maximiza ou abre o seletor de layouts, e arrastar até as bordas encaixa em metades e quartos.
- [x] **Relógio:** bateria em porcentagem e dia da semana.
- [x] **Atalhos:** `Super+E` abre o Arquivos, `Super+D` mostra a área de trabalho e `Ctrl+Alt+T` abre o terminal.
- [x] **Arquivos:** visualização em lista. Nas janelas de abrir e salvar, pastas antes dos arquivos.
- [x] **Visual:** tema escuro e destaque verde, com a fonte Inter na interface e nos títulos das janelas e o cursor padrão do GNOME (Adwaita). Depois do login, o GNOME abre na área de trabalho, e não na visão geral.
- [x] **Ícones:** Papirus na variante escura, com as pastas em verde. Aplicativos sem ícone no Papirus recebem um equivalente do próprio Papirus.
- [x] **Grade de aplicativos:** organizada em pastas, com os aplicativos sem uso escondidos (ver [Grade de aplicativos](#grade-de-aplicativos)).
- [x] **Papel de parede e telas:** papel de parede do GNOME. A tela de bloqueio usa o mesmo, e a tela de login acompanha o papel de parede atual (ver [Tela de login](#tela-de-login)).
- [x] **Favoritos da dash:** Chrome, Arquivos, Terminal e Obsidian, na dash da visão geral (`Super`). Não há barra de aplicativos fixa.

```bash
# 1. Extensões, pelo Debian (o pacote de preferências vem como dependência)
sudo apt-get install -y gnome-shell-extension-appindicator gnome-shell-extension-blur-my-shell

# 2. Extensões do site de extensões, na pasta do usuário (leia o código de cada uma antes de ligar)
sudo apt-get install -y gir1.2-gda-5.0 gir1.2-gsound-1.0      # dependências da Copyous
for u in copyous@boerdereinar.dev vertical-app-grid@lublst.github.io just-perfection-desktop@just-perfection \
         tilingshell@ferrarodomenico.com custom-hot-corners-extended@G-dH.github.com smile-extension@mijorus.it; do
  curl -fLo "$u.zip" "https://extensions.gnome.org/download-extension/$u.shell-extension.zip?shell_version=48"
  gnome-extensions install "$u.zip"
done

# 3. Ligar as oito (vale a partir da próxima sessão)
gsettings set org.gnome.shell enabled-extensions "['ubuntu-appindicators@ubuntu.com', 'blur-my-shell@aunetx', \
  'copyous@boerdereinar.dev', 'vertical-app-grid@lublst.github.io', 'just-perfection-desktop@just-perfection', \
  'tilingshell@ferrarodomenico.com', 'custom-hot-corners-extended@G-dH.github.com', 'smile-extension@mijorus.it']"

# As extensões do site guardam o esquema de configuração na própria pasta
X() { gsettings --schemadir "$HOME/.local/share/gnome-shell/extensions/$1/schemas" "${@:2}"; }

# 4. Copyous: histórico no Super+V, sem ícone no painel e sem prévia de links; notificações só no Super+M
gsettings set org.gnome.shell.keybindings toggle-message-tray "['<Super>m']"
X copyous@boerdereinar.dev set org.gnome.shell.extensions.copyous open-clipboard-dialog-shortcut "['<Super>v']"
X copyous@boerdereinar.dev set org.gnome.shell.extensions.copyous show-indicator false
X copyous@boerdereinar.dev set org.gnome.shell.extensions.copyous.link-item show-link-preview false
X copyous@boerdereinar.dev set org.gnome.shell.extensions.copyous.link-item show-link-preview-image false

# 5. Just Perfection: animações rápidas, abrir na área de trabalho e menos avisos
J="X just-perfection-desktop@just-perfection set org.gnome.shell.extensions.just-perfection"
$J animation 5                         # "rápida" (fator 0,8)
$J startup-status 0                    # depois do login, área de trabalho em vez da visão geral
$J workspace-popup false               # sem popup ao trocar de área de trabalho
$J window-demands-attention-focus true # janela que pede atenção ganha o foco, sem notificação "está pronto"
$J weather false                       # menu do relógio sem clima
$J world-clock false                   # e sem relógios mundiais
$J support-notifier-type 0             # sem aviso de doação a cada atualização

# 6. Tiling Shell: encaixe no estilo do Windows 11
T="X tilingshell@ferrarodomenico.com set org.gnome.shell.extensions.tilingshell"
$T show-indicator false
$T inner-gaps 0
$T outer-gaps 0
$T top-edge-maximize true
$T enable-snap-assistant-windows-suggestions true
$T enable-screen-edges-windows-suggestions true
$T override-alt-tab false

# 7. Cantos ativos, nos dois monitores (a extensão substitui o canto nativo do GNOME)
H=org.gnome.shell.extensions.custom-hot-corners-extended
P=/org/gnome/shell/extensions/custom-hot-corners-extended
for m in 0 1; do
  X custom-hot-corners-extended@G-dH.github.com set $H.corner:$P/monitor-$m-top-left-0/ action 'toggle-overview'
  X custom-hot-corners-extended@G-dH.github.com set $H.corner:$P/monitor-$m-bottom-left-0/ action 'show-applications'
done
X custom-hot-corners-extended@G-dH.github.com set $H.misc panel-menu-enable false

# 8. Emojis no Super+. com o Smile (Flatpak); o IBus deixa de usar o Super+.
sudo flatpak install -y flathub it.mijorus.smile
gsettings set org.freedesktop.ibus.panel.emoji hotkey "['<Super>semicolon']"

# 9. Janelas e relógio
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'
gsettings set org.gnome.desktop.interface clock-show-weekday true
gsettings set org.gnome.desktop.interface show-battery-percentage true

# 10. Atalhos (o GNOME 48 não traz atalho de terminal nem de emojis: são atalhos personalizados)
gsettings set org.gnome.settings-daemon.plugins.media-keys home "['<Super>e']"
gsettings set org.gnome.desktop.wm.keybindings show-desktop "['<Super>d']"
K=/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['$K/custom0/', '$K/custom1/']"
S="org.gnome.settings-daemon.plugins.media-keys.custom-keybinding"
gsettings set $S:$K/custom0/ name 'Terminal'
gsettings set $S:$K/custom0/ command 'gnome-terminal'
gsettings set $S:$K/custom0/ binding '<Primary><Alt>t'
gsettings set $S:$K/custom1/ name 'Emojis (Smile)'
gsettings set $S:$K/custom1/ command 'flatpak run it.mijorus.smile'
gsettings set $S:$K/custom1/ binding '<Super>period'

# 11. Favoritos da dash da visão geral
gsettings set org.gnome.shell favorite-apps \
  "['google-chrome.desktop', 'org.gnome.Nautilus.desktop', 'org.gnome.Terminal.desktop', 'md.obsidian.Obsidian.desktop']"

# 12. Fonte, pelo Debian (o cursor fica o padrão do GNOME)
sudo apt-get install -y fonts-inter
gsettings set org.gnome.desktop.interface font-name 'Inter 11'
gsettings set org.gnome.desktop.interface document-font-name 'Inter 11'
gsettings set org.gnome.desktop.wm.preferences titlebar-font 'Inter Bold 11'

# 13. Arquivos
gsettings set org.gnome.nautilus.preferences default-folder-viewer 'list-view'
gsettings set org.gtk.Settings.FileChooser sort-directories-first true
gsettings set org.gtk.gtk4.Settings.FileChooser sort-directories-first true

# 14. Ícones, pelo Debian
sudo apt-get install -y papirus-icon-theme
gsettings set org.gnome.desktop.interface icon-theme 'Papirus-Dark'

# 15. Papel de parede do GNOME, no fundo e na tela de bloqueio
B=/usr/share/backgrounds/gnome
gsettings set org.gnome.desktop.background picture-uri "file://$B/adwaita-l.jpg"
gsettings set org.gnome.desktop.background picture-uri-dark "file://$B/adwaita-d.jpg"
gsettings set org.gnome.desktop.screensaver picture-uri "file://$B/adwaita-d.jpg"
```

Como conferir:

```bash
gsettings get org.gnome.shell enabled-extensions
gnome-extensions list --enabled                    # depois de uma sessão nova: as oito
journalctl --user -b -o cat _COMM=gnome-shell | grep -i 'JS ERROR'   # vazio
gsettings get org.gnome.shell favorite-apps
gsettings list-recursively | grep -i -E "'<Super>(e|period)'"        # cada atalho numa ação só
```

Como desfazer: `gsettings reset <esquema> <chave>` para cada chave acima, `gsettings reset org.gnome.shell enabled-extensions` para desligar todas as extensões,
`gnome-extensions uninstall <uuid>` para as do site (mais `sudo apt remove gir1.2-gda-5.0 gir1.2-gsound-1.0`, das dependências da Copyous),
`sudo apt remove gnome-shell-extension-appindicator gnome-shell-extension-blur-my-shell` para as do Debian e `sudo flatpak uninstall it.mijorus.smile` para o Smile.

Observações:

- **Extensões no Wayland:** o GNOME só carrega extensões novas em uma sessão nova; para *desligar* uma já carregada não precisa. Saia e entre de novo para ativar uma extensão nova.
- **Pacotes do Debian:** declaram suporte ao GNOME 48. No Debian, o AppIndicator se chama `ubuntu-appindicators@ubuntu.com`.
- **Uma extensão por função:** a Tiling Shell substitui o Tiling Assistant, e a Just Perfection cobre a velocidade das animações, papel da Impatience. Duas extensões de encaixe ao mesmo tempo disputam as bordas da tela e os atalhos `Super` + setas.
- **Sem barra de aplicativos fixa:** os favoritos e os aplicativos abertos ficam na dash da visão geral (`Super`). O Dash to Dock e o Caffeine ficam de fora.
- **Listas de extensões:** `enabled-extensions` e `disabled-extensions` guardam também extensões já removidas. Ao remover uma, tire o UUID dela das duas listas.
- **Copyous:** a janela abre no ponteiro do mouse, guarda texto, imagens e arquivos, e os itens fixados não saem do histórico. O ícone no painel superior fica desligado (`show-indicator`), e as preferências abrem por `gnome-extensions prefs copyous@boerdereinar.dev`. A prévia de links fica desligada porque, ligada, a extensão baixa cada endereço copiado.
- **Smile:** o aplicativo copia o emoji, e a extensão complementar o cola com um `Ctrl+V` simulado. No terminal, que cola com `Ctrl+Shift+V`, o emoji fica só na área de transferência. Sem a extensão, o Smile só copia.
- **Super+. e o IBus:** o IBus também reserva o `Super+.` para o seletor de emojis dele, que no GNOME não abre. O atalho dele passa para `Super+;`.
- **Tiling Shell:** o seletor de layouts aparece ao arrastar a janela até o centro do topo da tela. Depois de encaixar uma janela, a extensão sugere as outras para preencher o espaço restante. `Super` + setas move a janela entre as áreas. O `Alt+Tab` fica o padrão do GNOME.
- **Cantos ativos:** a Custom Hot Corners - Extended substitui o canto nativo do GNOME, por isso o canto superior também é configurado nela. O ícone que ela põe no painel fica desligado.
- **Just Perfection:** a opção `theme` da extensão fica desligada, para manter o visual padrão do GNOME.
- **Atalho das notificações:** o GNOME usa `Super+V` e `Super+M` para a lista de notificações (`toggle-message-tray`). Com o `Super+V` na Copyous, ela fica só no `Super+M`. O `Super+N` fica de fora porque já foca a notificação ativa.
- **Seleção e `Ctrl+V`:** selecionar um texto o coloca na seleção primária, colada com o botão do meio, sem mudar o que o `Ctrl+V` cola. O GPaste fica de fora: o serviço dele (`gpaste-daemon`) continua rodando com a extensão desligada e, com `synchronize-clipboards` ligado, faz cada seleção substituir o conteúdo do `Ctrl+V`.
- **Pastas verdes:** no Papirus as pastas são azuis por padrão. A variante com pastas verdes (`papirus-icon-theme-green-folders-dark`) é de terceiros, instalada em `~/.local/share/icons/` pelo Wardrobe (Flatpak), e traz uma cópia completa do Papirus.
- **Papel de parede:** a tela de bloqueio desfoca e escurece o papel de parede. Os papéis do GNOME, em versão clara e escura, ficam em *Configurações > Aparência*.
- **Terminal:** o `gnome-terminal` é o instalado. O `kgx` (Console) e o `ptyxis` não estão presentes.
- **Visualização do Arquivos:** o Arquivos regrava `default-folder-viewer` ao ser usado. Se a lista voltar a ícones, repita o comando com o Arquivos fechado.
- **Pastas primeiro:** o Nautilus 48 não tem chave para isso. A opção existe só nas janelas de abrir e salvar arquivos (GTK).
- **Touchpad:** toque para clicar, rolagem natural e rolagem com dois dedos já vêm ligados, e o clique é por número de dedos.

## Grade de aplicativos

A grade fica com os aplicativos de uso diário na primeira linha e o resto em seis pastas. Os aplicativos sem uso somem da grade sem ser desinstalados.

```bash
# 1. Esconder sem desinstalar: um .desktop do usuário com Hidden=true tem prioridade sobre o do sistema
esconder() {
  printf '[Desktop Entry]\nType=Application\nName=%s\nHidden=true\n' \
    "$(grep -m1 '^Name=' "/usr/share/applications/$1.desktop" | cut -d= -f2-)" > "$HOME/.local/share/applications/$1.desktop"
}
mkdir -p ~/.local/share/applications
for a in vim gnome-system-monitor-kde libreoffice-startcenter org.gnome.Extensions org.gnome.Characters \
         org.freedesktop.MalcontentControl org.gnome.font-viewer org.gnome.Connections; do esconder "$a"; done

# 2. Ícone do Papirus para quem não tem: cópia do .desktop com outro Icon=
icone() {   # $1 = .desktop de origem, $2 = nome do ícone no Papirus
  cp "$1" "$HOME/.local/share/applications/" && sed -i "s|^Icon=.*|Icon=$2|" "$HOME/.local/share/applications/$(basename "$1")"
}
F=/var/lib/flatpak/exports/share/applications
icone /usr/share/applications/dbeaver-ce.desktop dbeaver
icone /usr/share/applications/realvnc-vncviewer.desktop realvnc-vncviewer
icone $F/be.alexandervanhee.gradia.desktop gnome-screenshot
icone $F/io.github.totoshko88.RustConn.desktop preferences-desktop-remote-desktop
icone $F/io.github.swordpuffin.wardrobe.desktop preferences-desktop-theme

# 3. Pastas: nome em português, lista fixa de aplicativos e sem inclusão automática por categoria
pasta() {   # $1 = id, $2 = nome, demais = aplicativos
  local f="org.gnome.desktop.app-folders.folder:/org/gnome/desktop/app-folders/folders/$1/" id=$1 nome=$2; shift 2
  gsettings set "$f" name "$nome"
  gsettings set "$f" translate false
  gsettings set "$f" categories "[]"
  gsettings set "$f" apps "[$(printf "'%s', " "$@" | sed 's/, $//')]"
}
pasta Dev 'Desenvolvimento' dbeaver-ce.desktop com.getpostman.Postman.desktop org.soapui.SoapUI.desktop \
  org.gnome.Meld.desktop org.wireshark.Wireshark.desktop com.usebottles.bottles.desktop
pasta Office 'Escritório' libreoffice-writer.desktop libreoffice-calc.desktop libreoffice-impress.desktop \
  libreoffice-draw.desktop libreoffice-math.desktop org.gnome.Evince.desktop org.gnome.Calendar.desktop \
  org.gnome.Contacts.desktop simple-scan.desktop
pasta Remote 'Remoto e VMs' anydesk.desktop io.github.totoshko88.RustConn.desktop realvnc-vncviewer.desktop \
  virt-manager.desktop
pasta Media 'Mídia' vlc.desktop org.gnome.Totem.desktop org.gnome.Music.desktop org.gnome.SoundRecorder.desktop \
  org.gnome.Snapshot.desktop org.gnome.Loupe.desktop org.gnome.Shotwell.desktop be.alexandervanhee.gradia.desktop
pasta Utilities 'Utilitários' org.gnome.Calculator.desktop org.gnome.TextEditor.desktop org.gnome.FileRoller.desktop \
  org.gnome.seahorse.Application.desktop org.gnome.clocks.desktop org.gnome.Weather.desktop org.gnome.Maps.desktop \
  firefox-esr.desktop com.mattjakeman.ExtensionManager.desktop io.github.swordpuffin.wardrobe.desktop \
  org.gnome.Tour.desktop yelp.desktop
pasta System 'Sistema' org.gnome.tweaks.desktop org.gnome.SystemMonitor.desktop org.gnome.DiskUtility.desktop \
  org.gnome.baobab.desktop org.gnome.Logs.desktop timeshift-gtk.desktop org.gnome.Software.desktop \
  nm-connection-editor.desktop im-config.desktop org.freedesktop.IBus.Setup.desktop
gsettings set org.gnome.desktop.app-folders folder-children "['Dev', 'Office', 'Remote', 'Media', 'Utilities', 'System']"

# 4. Ordem: aplicativos de uso diário e depois as pastas, numa página só
ordem=(com.microsoft.VSCode.desktop com.anthropic.Claude.desktop com.discordapp.Discord.desktop \
       com.valvesoftware.Steam.desktop org.gnome.Settings.desktop Dev Office Remote Media Utilities System)
layout=""; for i in "${!ordem[@]}"; do layout+="'${ordem[$i]}': <{'position': <$i>}>, "; done
gsettings set org.gnome.shell app-picker-layout "[{${layout%, }}]"
```

Como conferir:

```bash
gsettings get org.gnome.desktop.app-folders folder-children
gsettings get org.gnome.shell app-picker-layout                      # uma página só, na ordem acima
ls ~/.local/share/applications/                                      # os escondidos e os de ícone trocado
```

Como desfazer: apagar o `.desktop` correspondente em `~/.local/share/applications/` devolve o aplicativo e o ícone originais. `gsettings reset org.gnome.desktop.app-folders folder-children` e `gsettings reset org.gnome.shell app-picker-layout` voltam às pastas e à ordem padrão.

Observações:

- **Favoritos fora da grade:** no GNOME, um aplicativo fixado na dash não aparece na grade. Chrome, Arquivos, Terminal e Obsidian ficam só na dash.
- **Ordem regravada:** o GNOME Shell regrava `app-picker-layout` quando as pastas mudam. Defina as pastas antes e a ordem por último; se a ordem sair embaralhada, aplique-a de novo.
- **Pastas vazias do Debian:** `YaST` e `Pardus` vêm na lista padrão, sem aplicativos. Ficam de fora de `folder-children`.
- **Ícones trocados:** o Papirus tem o ícone do DBeaver (`dbeaver`) e do VNC Viewer (`realvnc-vncviewer`). Para os outros, o ícone escolhido é o mais próximo da função: captura de tela no Gradia, área de trabalho remota no RustConn, temas no Wardrobe e o golfinho do MySQL (`mysql-workbench`) num cliente MySQL. O Claude mantém o ícone próprio. Um `.desktop` que aponta para um arquivo (`Icon=/caminho/icone.png`) também ganha o ícone do Papirus assim.
- **Cópia congelada:** a cópia do `.desktop` não acompanha as mudanças que o pacote fizer no original. Se um aplicativo mudar o comando de abertura, apague a cópia e repita o passo 2.

## Tela de login

A tela de login é do GDM, e o GNOME não tem opção para mudar o fundo dela: o cinza vem de uma regra do tema (`#lockDialogGroup`), dentro de `/usr/share/gnome-shell/gnome-shell-theme.gresource`, do pacote `gnome-shell-common`. Três peças trocam esse cinza pelo papel de parede atual, desfocado e escurecido como na tela de bloqueio:

| Peça | Onde | O que faz |
|---|---|---|
| `gdm-fundo` | `/usr/local/sbin/` | Acrescenta ao tema uma regra que aponta para `/usr/local/share/gdm-fundo/fundo.jpg`. Roda uma vez, como `root` |
| Gancho do APT | `/etc/apt/apt.conf.d/99gdm-fundo` | Roda o `gdm-fundo` depois de cada `dpkg`. Uma atualização do `gnome-shell-common` devolve o cinza, e o gancho reaplica a regra |
| `gdm-fundo-sync` | `~/.local/bin/` e duas unidades do systemd do usuário | Gera o `fundo.jpg` a partir do papel de parede, no login e sempre que uma configuração do GNOME muda. Só refaz a imagem quando o papel de parede mudou |

```bash
# 1. Compilador de recursos do GLib, usado para montar o tema de novo
sudo apt install --no-install-recommends libgio-2.0-dev-bin

# 2. Script do sistema
cat > gdm-fundo <<'EOF'
#!/bin/sh
# Troca o fundo cinza da tela de login do GDM por uma imagem.
# O GNOME nao tem opcao para isso: o fundo vem do CSS do tema, dentro de
# gnome-shell-theme.gresource. O script acrescenta ao CSS uma regra que aponta
# para IMG, fora do tema. Trocar a imagem nao exige rodar o script de novo.
# Atualizar o gnome-shell-common sobrescreve o tema; o hook do APT
# (/etc/apt/apt.conf.d/99gdm-fundo) roda este script de novo.
# Desfazer: apagar o hook e rodar apt install --reinstall gnome-shell-common
set -eu
ALVO=${ALVO:-/usr/share/gnome-shell/gnome-shell-theme.gresource}
IMG=${IMG:-/usr/local/share/gdm-fundo/fundo.jpg}
COMPILAR=${COMPILAR:-glib-compile-resources}
PREFIXO=/org/gnome/shell/theme
MARCA=gdm-fundo

command -v "$COMPILAR" >/dev/null || { echo "gdm-fundo: $COMPILAR ausente" >&2; exit 0; }
REGRA="#lockDialogGroup { background: #222226 url(\"file://$IMG\"); background-size: cover; background-repeat: no-repeat; background-position: center; } /* $MARCA */"
gresource extract "$ALVO" "$PREFIXO/gnome-shell-dark.css" | grep -qxF "$REGRA" && exit 0

TMP=$(mktemp -d)
trap 'rm -rf "$TMP"' EXIT
for r in $(gresource list "$ALVO"); do
	f=${r#"$PREFIXO"/}
	case "$f" in "$MARCA".*) continue ;; esac   # imagem embutida por versoes antigas
	mkdir -p "$TMP/src/$(dirname "$f")"
	gresource extract "$ALVO" "$r" > "$TMP/src/$f"
done
for css in gnome-shell-dark.css gnome-shell-light.css; do
	[ -f "$TMP/src/$css" ] || continue
	grep -v "$MARCA" "$TMP/src/$css" > "$TMP/css" || true
	printf '%s\n' "$REGRA" >> "$TMP/css"
	mv "$TMP/css" "$TMP/src/$css"
done
{
	echo '<?xml version="1.0" encoding="UTF-8"?>'
	echo "<gresources><gresource prefix=\"$PREFIXO\">"
	(cd "$TMP/src" && find . -type f | sed 's|^\./||' | sort | sed 's|.*|<file>&</file>|')
	echo '</gresource></gresources>'
} > "$TMP/tema.xml"
"$COMPILAR" --sourcedir="$TMP/src" --target="$TMP/novo.gresource" "$TMP/tema.xml"
gresource extract "$TMP/novo.gresource" "$PREFIXO/gnome-shell-dark.css" | grep -qxF "$REGRA"
install -m 644 "$TMP/novo.gresource" "$ALVO"
echo "gdm-fundo: regra aplicada em $ALVO"
EOF
sudo install -m 755 gdm-fundo /usr/local/sbin/

# 3. Gancho do APT
cat > 99gdm-fundo <<'EOF'
// Reaplica o fundo da tela de login do GDM depois que o dpkg instala pacotes
// (uma atualizacao do gnome-shell-common sobrescreve o tema). Ver /usr/local/sbin/gdm-fundo.
DPkg::Post-Invoke { "if [ -x /usr/local/sbin/gdm-fundo ]; then /usr/local/sbin/gdm-fundo || true; fi"; };
EOF
sudo install -m 644 99gdm-fundo /etc/apt/apt.conf.d/

# 4. Imagem do fundo, de propriedade do usuário (ele a atualiza sem sudo)
sudo install -d -m 755 /usr/local/share/gdm-fundo
sudo install -m 644 -o "$USER" /dev/null /usr/local/share/gdm-fundo/fundo.jpg
sudo gdm-fundo

# 5. Gerador, do usuário
mkdir -p ~/.local/bin ~/.config/systemd/user
cat > ~/.local/bin/gdm-fundo-sync <<'EOF'
#!/usr/bin/python3
"""Gera o fundo da tela de login (GDM) a partir do papel de parede atual.

Desfoca e escurece o papel de parede, como a tela de bloqueio do GNOME, e grava
em DESTINO, que o tema do GDM usa (ver /usr/local/sbin/gdm-fundo). So refaz a
imagem quando o papel de parede muda. Roda pela unidade gdm-fundo-sync.path.
"""
import os
import sys
import xml.etree.ElementTree as ET

import gi
gi.require_version('GdkPixbuf', '2.0')
from gi.repository import Gio, GdkPixbuf

DESTINO = os.environ.get('DESTINO', '/usr/local/share/gdm-fundo/fundo.jpg')
CACHE = os.path.expanduser('~/.cache/gdm-fundo-sync')
LARGURA, ALTURA = 1920, 1080
BRILHO = 166  # 0-255; a tela de bloqueio do GNOME usa 65% de brilho


def papel_de_parede():
    bg = Gio.Settings.new('org.gnome.desktop.background')
    escuro = Gio.Settings.new('org.gnome.desktop.interface').get_string('color-scheme') == 'prefer-dark'
    uri = bg.get_string('picture-uri-dark' if escuro else 'picture-uri')
    caminho = Gio.File.new_for_uri(uri).get_path() if uri else None
    if caminho and caminho.endswith('.xml'):
        caminho = arquivo_do_xml(caminho)
    return caminho


def arquivo_do_xml(caminho):
    """Primeira imagem de um papel de parede em XML (os que mudam com a hora)."""
    raiz = ET.parse(caminho).getroot()
    arq = raiz.find('.//file')
    if arq is None:
        return None
    tamanhos = arq.findall('size')
    if tamanhos:
        maior = max(tamanhos, key=lambda s: int(s.get('width', 0)) * int(s.get('height', 0)))
        return maior.text.strip()
    return arq.text.strip()


def gerar(origem):
    _, w, h = GdkPixbuf.Pixbuf.get_file_info(origem)
    escala = max(LARGURA / w, ALTURA / h)
    lw, lh = max(LARGURA, round(w * escala)), max(ALTURA, round(h * escala))
    img = GdkPixbuf.Pixbuf.new_from_file_at_scale(origem, lw, lh, False)
    img = img.new_subpixbuf((lw - LARGURA) // 2, (lh - ALTURA) // 2, LARGURA, ALTURA)
    # Desfoque: reduz em etapas ate poucos pixels e amplia de volta
    for tw, th in [(480, 270), (120, 68), (40, 23), (24, 14), (48, 27), (120, 68), (480, 270), (LARGURA, ALTURA)]:
        img = img.scale_simple(tw, th, GdkPixbuf.InterpType.HYPER)
    saida = GdkPixbuf.Pixbuf.new(GdkPixbuf.Colorspace.RGB, False, 8, LARGURA, ALTURA)
    saida.fill(0x000000ff)
    img.composite(saida, 0, 0, LARGURA, ALTURA, 0, 0, 1, 1, GdkPixbuf.InterpType.BILINEAR, BRILHO)
    ok, dados = saida.save_to_bufferv('jpeg', ['quality'], ['90'])
    with open(DESTINO, 'wb') as f:  # grava no lugar: o arquivo pertence ao usuario
        f.write(dados)


def main():
    origem = papel_de_parede()
    if not origem or not os.path.isfile(origem):
        return 0
    chave = f'{origem}|{os.path.getmtime(origem)}|{DESTINO}'
    try:
        if open(CACHE).read() == chave and os.path.getsize(DESTINO) > 0:
            return 0
    except OSError:
        pass
    gerar(origem)
    os.makedirs(os.path.dirname(CACHE), exist_ok=True)
    with open(CACHE, 'w') as f:
        f.write(chave)
    print(f'gdm-fundo-sync: fundo gerado de {origem}')
    return 0


if __name__ == '__main__':
    sys.exit(main())
EOF
chmod 755 ~/.local/bin/gdm-fundo-sync

cat > ~/.config/systemd/user/gdm-fundo-sync.service <<'EOF'
[Unit]
Description=Fundo da tela de login igual ao papel de parede

[Service]
Type=oneshot
ExecStart=%h/.local/bin/gdm-fundo-sync

[Install]
WantedBy=default.target
EOF
cat > ~/.config/systemd/user/gdm-fundo-sync.path <<'EOF'
[Unit]
Description=Observa as configuracoes do GNOME para atualizar o fundo da tela de login

[Path]
PathChanged=%h/.config/dconf/user

[Install]
WantedBy=default.target
EOF
systemctl --user daemon-reload
systemctl --user enable --now gdm-fundo-sync.path gdm-fundo-sync.service
```

Como conferir:

```bash
gresource extract /usr/share/gnome-shell/gnome-shell-theme.gresource \
  /org/gnome/shell/theme/gnome-shell-dark.css | tail -1      # a regra com /* gdm-fundo */
apt-config dump | grep gdm-fundo                             # o gancho está ativo
systemctl --user is-active gdm-fundo-sync.path               # active
cat ~/.cache/gdm-fundo-sync                                  # papel de parede usado na última imagem
ls -l /usr/local/share/gdm-fundo/fundo.jpg                   # do usuário, atualizada ao trocar o papel
```

Depois, saia da sessão: a tela de login mostra o papel de parede desfocado.

Como desfazer:

```bash
systemctl --user disable --now gdm-fundo-sync.path gdm-fundo-sync.service
rm ~/.local/bin/gdm-fundo-sync ~/.config/systemd/user/gdm-fundo-sync.* ~/.cache/gdm-fundo-sync
sudo rm /etc/apt/apt.conf.d/99gdm-fundo /usr/local/sbin/gdm-fundo
sudo rm -r /usr/local/share/gdm-fundo
sudo apt install --reinstall gnome-shell-common             # devolve o tema original
```

Observações:

- **Teste sem risco:** o `gdm-fundo` aceita outro alvo pela variável `ALVO`. Numa cópia, `cp /usr/share/gnome-shell/gnome-shell-theme.gresource /tmp/t.gresource && ALVO=/tmp/t.gresource ./gdm-fundo` aplica a regra sem tocar no sistema, e `gresource list` e `gresource extract` mostram o resultado.
- **Se a tela de login não abrir:** `Ctrl+Alt+F3` abre um terminal de texto. Lá, os comandos de desfazer do sistema devolvem o tema original.
- **Cinza na transição:** um cinza escuro (`#282828`) aparece por cerca de um segundo antes da tela de login e logo depois da senha. É uma cor fixa no código do GNOME Shell (`SystemBackground`), mostrada enquanto ele inicia. Não há opção para mudá-la, e as extensões só carregam depois desse momento.
- **Imagem do usuário:** a tela de login exibe uma imagem que o usuário pode trocar. Numa máquina de um usuário só, que já é o administrador, isso não abre acesso novo.
- **Quando a imagem muda:** a unidade `.path` observa `~/.config/dconf/user`, que muda a cada configuração gravada. Na maioria das vezes o gerador só confere o papel de parede e sai. O GDM lê a imagem quando a tela de login abre, então a troca aparece no próximo logout ou boot.
- **Papel de parede em XML:** os que mudam com a hora do dia apontam para um `.xml`. O gerador usa a primeira imagem dele.
- **Tema escuro e claro:** o gerador usa `picture-uri-dark` com o tema escuro e `picture-uri` com o claro, o mesmo que a área de trabalho mostra.

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
