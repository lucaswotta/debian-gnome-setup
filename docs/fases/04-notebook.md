# Fase 4 - Notebook

**Status:** concluída · **Escopo:** específico do modelo de referência (ThinkPad E14 Gen 1). Energia, suspensão e boot valem para qualquer notebook com GNOME

[← Fase 3](03-atualizacoes.md) · [Roteiro](../02-roteiro.md) · [Fase 5 →](05-aplicativos.md)

Objetivo: ajustar energia, vídeo, suspensão e bateria do ThinkPad E14 Gen 1.

- [x] **Energia e temperatura:** o `power-profiles-daemon` (perfil `balanced`) basta. Em uso leve, a temperatura fica baixa e a ventoinha desligada.
- [x] **GPU AMD sob demanda:** testada com OpenGL e Vulkan. Dorme sozinha depois do uso.
- [x] **Suspensão:** tela apaga em 30 min, suspende em 60 min. Fechar a tampa suspende na hora.
- [x] **Limite de carga da bateria:** opcional e desativado. O procedimento está documentado.
- [x] **Teclas Fn:** funcionam.
- [x] **Tecla da barra (`/ ?`):** remapeada por `hwdb` (ver [Tecla da barra](#tecla-da-barra--)).
- [x] **Leitor de digital:** sem suporte no Linux (ver observações).
- [x] **Boot direto:** o menu do GRUB fica oculto, com 1 segundo de espera invisível (ver [Boot direto](#boot-direto-sem-o-menu-do-grub)).

```bash
# GPU: ferramentas de teste
sudo apt install -y mesa-utils vulkan-tools
glxinfo -B | grep "OpenGL renderer"                 # Intel (padrão)
DRI_PRIME=1 glxinfo -B | grep "OpenGL renderer"     # AMD
vulkaninfo --summary

# Suspensão
gsettings set org.gnome.desktop.session idle-delay 1800
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout 3600
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-timeout 3600

# Limite de carga da bateria (opcional)
BAT=$(upower -e | grep -i BAT | head -1)
busctl call org.freedesktop.UPower "$BAT" org.freedesktop.UPower.Device EnableChargeThreshold b true

# Ler os logs do sistema sem sudo (vale a partir do próximo login)
sudo usermod -aG systemd-journal "$USER"
```

| Parte | O que faz |
|---|---|
| `DRI_PRIME=1` | Executa o comando na GPU dedicada. Sem ela, a Intel é usada. |
| `idle-delay` | Segundos de inatividade até a tela apagar (1800 = 30 min). |
| `sleep-inactive-*-timeout` | Segundos de inatividade até suspender, na tomada e na bateria (3600 = 60 min). |
| `EnableChargeThreshold` | Liga (`b true`) ou desliga (`b false`) o limite de carga do UPower, que vale de 75% a 80%. |
| `usermod -aG systemd-journal` | Permite ao usuário ler o log do sistema (`journalctl`) sem `sudo`. |

Como conferir:

```bash
upower -i "$(upower -e | grep -i BAT)"           # charge-threshold-supported: yes
cat /sys/class/drm/card1/device/power/runtime_status   # suspended = AMD dormindo
gsettings get org.gnome.desktop.session idle-delay
journalctl -k -b -p err                          # erros do kernel neste boot
```

Como desfazer:

```bash
gsettings reset org.gnome.desktop.session idle-delay
gsettings reset org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout
gsettings reset org.gnome.settings-daemon.plugins.power sleep-inactive-battery-timeout
BAT=$(upower -e | grep -i BAT | head -1)
busctl call org.freedesktop.UPower "$BAT" org.freedesktop.UPower.Device EnableChargeThreshold b false
sudo gpasswd -d "$USER" systemd-journal
```

Observações:

- **TLP:** conflita com o `power-profiles-daemon`, porque os dois querem gerenciar a energia. Use só um.
- **`thermald`:** não roda em ThinkPads com controle térmico pelo firmware (DYTC). O log informa
  `Thermald can't run on this platform`. Não instale. Se já estiver instalado, remova com `sudo apt remove thermald`.
  Não use `--ignore-cpuid-check`, que ignora o aviso e conflita com o firmware.
- **GPU híbrida:** o `switcheroo-control` mostra a opção de iniciar com a placa dedicada ao clicar com o botão direito
  no ícone do aplicativo. No terminal e no Steam, use `DRI_PRIME=1`. A AMD é uma placa de entrada (2 GB) e não serve
  para jogos recentes em qualidade alta.
- **Limite de carga:** o firmware de fábrica carrega de 95% a 100%. Com o limite ligado, o UPower informa 75% e 80%, e a carga para em cerca de 80% da capacidade atual, com o estado `Not charging`.
  O painel arredonda para baixo e mostra 79%. A leitura em `sysfs` (`start=80 end=75`) aparece invertida, e o registro do kernel confirma `start 75, stop 80`
  (`journalctl -k -b | grep "battery 1 registered"`). O limite não afeta o desempenho e reduz a autonomia por carga em cerca de 20%.
- **Equipamento cedido ou gerenciado por terceiros:** prefira deixar a bateria no comportamento de fábrica. O limite de carga é
  opcional e reversível. Depois de desligá-lo, a carga volta a passar de 80% e o firmware retoma os valores de fábrica.
- **Leitor de digital:** o Goodix `27c6:55a4` está na seção "Known unsupported devices" da libfprint, e o `fprintd-list`
  responde `No devices available`. Não há o que configurar.
- **Logs:** sem estar no grupo `systemd-journal`, o `journalctl` mostra "sem entradas". Isso **não** significa "sem erros".
- **Erros do boot e boot direto:** ver as seções abaixo.

## Tecla da barra (`/ ?`)

Neste modelo, a tecla `/ ?` do ABNT2, ao lado do `PrtSc`, envia o scancode `0x9d`. O kernel o interpreta como `Ctrl` direito (`KEY_RIGHTCTRL`), então a tecla não digita a barra. O teclado não tem outro `Ctrl` direito, e o layout `br` está correto. A saída é remapear o scancode para `KEY_RO`, a tecla `/ ?` do ABNT2.

```bash
# Identificar o que a tecla envia (10 s; aperte a tecla)
sudo timeout 10 python3 -c "import struct;f=open('/dev/input/by-path/platform-i8042-serio-0-event-kbd','rb');[print(struct.unpack('llHHi',f.read(24))[2:]) for _ in iter(int,1)]"
# Saída com defeito: (4, 4, 157) e (1, 97, ...), ou seja, scancode 0x9d e KEY_RIGHTCTRL

# Remapear (o modelo vem de /sys/class/dmi/id/modalias, campo pvr)
printf 'evdev:atkbd:dmi:bvn*:bvr*:bd*:svnLENOVO*:pn*:pvrThinkPadE14*\n KEYBOARD_KEY_9d=ro\n' > 90-thinkpad-e14-slash.hwdb
sudo install -m 644 90-thinkpad-e14-slash.hwdb /etc/udev/hwdb.d/
sudo systemd-hwdb update && sudo udevadm trigger --subsystem-match=input --action=change
```

| Parte | O que faz |
|---|---|
| `evdev:atkbd:dmi:...` | Aplica a regra só ao teclado interno (`atkbd`) de ThinkPads E14 (`pvr`, versão do produto no DMI). |
| `KEYBOARD_KEY_9d=ro` | Troca o scancode `0x9d` pelo código `KEY_RO`. |
| `systemd-hwdb update` | Recompila o banco de hardware. |
| `udevadm trigger` | Reaplica as regras ao teclado sem reiniciar. |

Como conferir: a tecla digita `/`, e `?` com `Shift`. Se ainda enviar `Ctrl`, faça logout e login. Depois, `udevadm info /dev/input/by-path/platform-i8042-serio-0-event-kbd | grep KEYBOARD_KEY` deve listar `KEYBOARD_KEY_9d=ro`.

Como desfazer: `sudo rm /etc/udev/hwdb.d/90-thinkpad-e14-slash.hwdb && sudo systemd-hwdb update && sudo udevadm trigger --subsystem-match=input --action=change`.

Observações:

- **Outros modelos:** o scancode e o padrão `pvr` valem para o modelo de referência. Em outro notebook, repita a identificação e ajuste os dois.

## Erros do boot

`journalctl -b -p err` lista os erros do boot atual. As linhas de erro do modelo de referência vêm de quatro causas, mais o relatório do Wi-Fi
depois de cada suspensão.

| Causa | Linhas | Situação |
|---|---|---|
| Regra `udev` do ALSA com rótulo ausente (bug do Debian #1093057 e #1104719) | 2 | Corrigido com uma cópia local da regra |
| Comandos de inicialização que terminam antes de o systemd criar o escopo (`PID vanished`) | 4 a 5 | Corrigido escondendo essas inicializações |
| Método ACPI da GPU com defeito no firmware (`AE_AML_BUFFER_LIMIT`) | 4 | Sem correção sem atualizar a BIOS. A GPU funciona |
| Mensagem do PAM no login (`gkr-pam: unable to locate daemon control file`) | 1 | Esperada, por desenho |
| Relatório de falha do `iwlwifi` ao acordar da suspensão | cerca de 75 por suspensão | Bug conhecido do kernel 6.12. O firmware recarrega e a rede volta sozinha |

Com as correções, restam 5 linhas fixas (4 do firmware e 1 do PAM), mais o relatório do Wi-Fi a cada suspensão.

```bash
# 1. Inicializações que terminam antes de o systemd criar o escopo (só para o usuário)
mkdir -p ~/.config/autostart
for f in gnome-keyring-pkcs11 gnome-keyring-secrets gnome-keyring-ssh user-dirs-update-gtk xdg-user-dirs xdg-user-dirs-kde im-launch; do
  NAME=$(grep -m1 '^Name=' /etc/xdg/autostart/$f.desktop | cut -d= -f2-)
  printf '[Desktop Entry]\nType=Application\nName=%s\nHidden=true\nX-GNOME-Autostart-enabled=false\n' "$NAME" > ~/.config/autostart/$f.desktop
done

# 2. Agente SSH: só o do GNOME (gcr) define o SSH_AUTH_SOCK
mkdir -p ~/.config/environment.d
echo 'SSH_AUTH_SOCK=${XDG_RUNTIME_DIR}/gcr/ssh' > ~/.config/environment.d/50-ssh-agent.conf
systemctl --user mask --now ssh-agent.socket ssh-agent.service
systemctl --user set-environment SSH_AUTH_SOCK=${XDG_RUNTIME_DIR}/gcr/ssh

# 3. Regra udev do ALSA com o rótulo corrigido, testada antes de instalar
sed -e '26s/alsa_restore_go/alsa_restore_std/' \
    -e '1i # Copia local de 90-alsa-restore.rules com o rotulo corrigido (Debian #1093057). Remover quando o alsa-utils corrigir.' \
    /usr/lib/udev/rules.d/90-alsa-restore.rules > alsa-fix.rules
udevadm verify alsa-fix.rules
sudo install -m 644 alsa-fix.rules /etc/udev/rules.d/90-alsa-restore.rules && sudo udevadm control --reload
```

Como conferir:

```bash
journalctl -b -p err --no-pager -q | wc -l               # total de linhas de erro neste boot
journalctl -b -p err --no-pager -q | grep -vc iwlwifi    # sem o relatório do Wi-Fi
echo $SSH_AUTH_SOCK && ssh-add -l                        # .../gcr/ssh e a chave carregada
udevadm verify /etc/udev/rules.d/90-alsa-restore.rules   # sem avisos
```

Como desfazer:

```bash
rm ~/.config/autostart/{gnome-keyring-pkcs11,gnome-keyring-secrets,gnome-keyring-ssh,user-dirs-update-gtk,xdg-user-dirs,xdg-user-dirs-kde,im-launch}.desktop
rm ~/.config/environment.d/50-ssh-agent.conf && systemctl --user unmask ssh-agent.socket ssh-agent.service
sudo rm /etc/udev/rules.d/90-alsa-restore.rules
```

Observações:

- **Corrida no systemd:** o `gnome-session` move cada aplicativo de inicialização para um escopo do systemd. Um comando que termina
  antes disso gera `Failed to start ... scope` e `PID vanished`. Qual comando perde a corrida muda de um boot para outro. A saída é
  esconder o que é redundante ou de execução instantânea.
- **Chaveiro:** esconder as três inicializações dele é seguro, porque o serviço do chaveiro já sobe pelo systemd com os componentes
  `pkcs11` e `secrets`, e o agente SSH vem do `gcr`.
- **`im-launch`:** o comando `im-launch true` só confere o ambiente do método de entrada e termina de imediato. O teclado ABNT2 é configurado pelo `xkb`, então esconder a inicialização não muda a digitação.
- **Três agentes SSH disputam o `SSH_AUTH_SOCK`:** o do GNOME (`gcr-ssh-agent`), o do OpenSSH (`ssh-agent.socket`) e o do GnuPG
  (só se o `enable-ssh-support` estiver ativo). Cada um executa `systemctl --user set-environment SSH_AUTH_SOCK=...` no login, e vence o
  último. Um arquivo em `environment.d` **não** vence essa disputa, porque é lido antes. Sem escolher um, o agente vira o do OpenSSH,
  que não guarda chaves nem usa o chaveiro. A saída é mascarar os que não interessam.
- **Regra do ALSA:** a cópia em `/etc/udev/rules.d/` tem prioridade sobre a do pacote. Apague-a quando o `alsa-utils` corrigir o bug,
  para voltar a receber as atualizações.
- **Wi-Fi na suspensão:** é uma regressão do kernel 6.12 no `iwlwifi`, uma corrida na retomada, com patch de reversão publicado.
  O relatório tem cerca de 75 linhas de nível `err` por suspensão. Não há correção segura pelo lado do sistema. As atualizações do
  kernel do Debian trazem a correção.
- **Firmware da GPU:** o erro do ACPI vem da BIOS. O `amdgpu` usa outro caminho e a placa funciona.

### Boot direto (sem o menu do GRUB)

Com um único sistema instalado, o menu do GRUB só atrasa o boot. O GRUB passa a esperar 1 segundo sem mostrar nada, e a tecla `Esc` nessa janela abre o menu.

```bash
sudo cp -a /etc/default/grub /etc/default/grub.bak-antes-boot-direto
sudo sed -i 's/^GRUB_TIMEOUT=.*/GRUB_TIMEOUT=1/' /etc/default/grub
sudo sed -i '/^GRUB_TIMEOUT=/a GRUB_TIMEOUT_STYLE=hidden' /etc/default/grub
sudo update-grub
```

Como conferir: `grep -E '^GRUB_TIMEOUT' /etc/default/grub` mostra `GRUB_TIMEOUT=1` e `GRUB_TIMEOUT_STYLE=hidden`. Depois de reiniciar, `systemd-analyze` informa o tempo do carregador (`loader`).

Como desfazer: `sudo cp /etc/default/grub.bak-antes-boot-direto /etc/default/grub && sudo update-grub`.

Observações:

- **Falha de boot:** o `grub.cfg` gerado mostra o menu por 30 segundos depois de um boot que falhou (`recordfail`), então o menu reaparece quando é necessário.
- **Tela de login:** continua a do GDM. O login automático não é usado, porque o desbloqueio do chaveiro e do agente SSH depende da senha no login.
- **Menu do GRUB e `os-prober`:** o `update-grub` avisa que não procura outros sistemas. Com um só sistema, o aviso não tem efeito.
