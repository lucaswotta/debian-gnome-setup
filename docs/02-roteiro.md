# 02 - Roteiro

Legenda: `[x]` feito · `[ ]` pendente · `[~]` em discussão.

Regras:

- Uma fase por vez, explicada antes de executar.
- Comando com `sudo` só roda depois de confirmação.
- Só vira script (`scripts/`) o que já foi feito à mão e entendido.
- Cada fase termina com uma verificação e um registro neste arquivo.

## Visão geral

| Fase | Tema | Status |
|---|---|---|
| 0 | Base mínima | Concluída |
| 1 | Git e GitHub | Concluída |
| 2 | Base do sistema, com disco SATA extra | Concluída |
| 3 | Atualizações | Concluída |
| 4 | Notebook | Concluída |
| 5 | Aplicativos, com VPN | Concluída |
| 6 | GNOME | Em andamento |
| 7 | Backup | Planejada |
| 8 | Automação | Planejada |

---

## Fase 0 - Base mínima (concluída)

- [x] Instalar o Debian 13 com GNOME (UEFI, raiz em ext4, swap de 12 GB)
- [x] Configurar idioma pt_BR, fuso America/Sao_Paulo e teclado ABNT2
- [x] Colocar o usuário no grupo `sudo`
- [x] Instalar o `curl`
- [x] Instalar o Claude Code

```bash
sudo apt update && sudo apt install -y curl
curl -fsSL https://claude.ai/install.sh | bash
```

| Comando | O que faz |
|---|---|
| `sudo` | Executa o comando seguinte como administrador. |
| `apt update` | Baixa a lista atualizada de pacotes. Não instala nem atualiza nada. |
| `apt install -y curl` | Instala o `curl`. O `-y` responde "sim" às confirmações. |
| `&&` | Executa o próximo comando só se o anterior der certo. |
| `curl -fsSL URL` | Baixa o conteúdo de uma URL. `-f` falha em erro HTTP, `-s` fica em silêncio, `-S` mostra erros, `-L` segue redirecionamentos. |
| `\| bash` | Entrega o conteúdo baixado ao `bash` para executar. |

Observações:

- O `apt` precisa de administrador. Sem `sudo`, o `apt update` falha por permissão.
- Se o `sudo` disser que o usuário não pode usá-lo, adicione-o ao grupo `sudo` como `root` (`su -`).
- `curl ... | bash` executa o que o site enviar. Use só com fontes confiáveis e leia o script nos outros casos.
- O Claude Code foi instalado só para o usuário (`~/.local/bin`), sem `sudo`.

---

## Fase 1 - Git e GitHub (concluída)

Objetivo: atualizar o sistema e versionar este guia.

- [x] Atualizar o sistema
- [x] Instalar `git` e `gh` (CLI do GitHub)
- [x] Configurar a identidade do git
- [x] Autenticar no GitHub com chave SSH
- [x] Ligar a pasta local ao repositório remoto e publicar o primeiro commit

```bash
sudo apt update && sudo apt full-upgrade -y && sudo apt install -y git gh

git config --global user.name "Seu Nome"
git config --global user.email "ID+usuario@users.noreply.github.com"
git config --global init.defaultBranch main

gh auth login --hostname github.com --git-protocol ssh --web
gh api meta --jq '.ssh_keys[] | "github.com " + .' >> ~/.ssh/known_hosts
ssh -T git@github.com

git init
git remote add origin git@github.com:USUARIO/REPOSITORIO.git
git add -A && git commit
git push -u origin main
```

| Comando | O que faz |
|---|---|
| `apt full-upgrade` | Atualiza todos os pacotes, instalando ou removendo dependências quando preciso. |
| `git config --global` | Grava a configuração do git para o usuário, em `~/.gitconfig`. |
| `gh auth login` | Faz login no GitHub e cria a chave SSH. O `--web` autoriza pelo navegador. |
| `gh api meta` | Lista as chaves oficiais do servidor do GitHub. Gravadas em `known_hosts`, evitam confirmar a chave "às cegas" na primeira conexão. |
| `ssh -T git@github.com` | Testa a autenticação. A resposta esperada é `Hi USUARIO! You've successfully authenticated`. |
| `git init` e `git remote add origin` | Iniciam o repositório local e apontam para o remoto. Usados no lugar de `git clone` porque a pasta já tem conteúdo. |

Como conferir:

```bash
apt list --upgradable      # sem pacotes pendentes
git config --global -l     # nome, e-mail e branch padrão
gh auth status             # logado, protocolo ssh
git status -sb             # main acompanhando origin/main
```

Observações:

- **Repositório público:** o e-mail de cada commit fica visível. Use o endereço `noreply` do GitHub
  (`ID+usuario@users.noreply.github.com`). O ID sai de `gh api user --jq .id`.
- **Chave SSH:** proteja com senha (passphrase).
- **Trabalho direto na `main`:** neste repositório de uso individual não há branches nem PRs.

---

## Fase 2 - Base do sistema (concluída)

- [x] **TRIM do SSD:** já vem ativo, com o `fstrim.timer` rodando toda semana.
- [x] **Catálogo de firmware:** atualizado com `fwupd`. `fwupdmgr get-updates` lista as atualizações disponíveis.
- [x] **BIOS:** é a da época do lançamento do modelo, e o notebook funciona bem com ela.
  A atualização é opcional.
- [x] **`contrib`:** habilitado na fase 5, para as fontes da Microsoft. O `non-free` segue desativado até um pacote exigir.
- [x] **Disco SATA extra:** verificado, mantido em btrfs e montado em `/mnt/ssd` (ver abaixo).

```bash
sudo fwupdmgr refresh --force
fwupdmgr get-updates
```

| Comando | O que faz |
|---|---|
| `fwupdmgr refresh --force` | Baixa do LVFS a lista atualizada de firmwares. Só atualiza o cache e não instala nada. |
| `fwupdmgr get-updates` | Mostra, por dispositivo, se há atualização de firmware disponível. |

Como conferir:

```bash
systemctl is-enabled fstrim.timer      # enabled
lsblk -D -o NAME,DISC-GRAN,DISC-MAX    # discos com suporte a TRIM
cat /sys/class/dmi/id/bios_version     # versão da BIOS instalada
```

Observações:

- **Firmware:** o LVFS não tem todos os modelos. "Sem atualização" no `fwupd` não garante que a BIOS seja a última.
- **BIOS:** atualizar é opcional quando o equipamento funciona bem. Se decidir atualizar, siga o procedimento
  oficial do fabricante, com o notebook na tomada e sem interromper o processo.
- **Repositórios:** adicione `contrib` ou `non-free` somente quando um pacote exigir.

### Disco SATA extra

Decisões: manter o btrfs de origem, sem formatar, e montar em `/mnt/ssd`.

- [x] Conferir o conteúdo antes de reaproveitar o disco.
- [x] Conferir a saúde (SMART).
- [x] Montar de forma permanente pelo `/etc/fstab`, identificando o disco por **UUID**.
- [x] Criar a estrutura de pastas e ajustar dono e permissões.
- [x] Confirmar que o disco monta sozinho depois de reiniciar.
- [ ] Apontar para ele o que ocupa espaço, como a biblioteca de jogos (fase 5).
- [ ] Incluí-lo no backup (fase 7). Um disco só não é backup.

```bash
sudo apt install -y btrfs-progs smartmontools
sudo smartctl -H -A /dev/sda

udisksctl unmount -b /dev/sda1
sudo cp -a /etc/fstab /etc/fstab.bak-$(date +%F)
sudo mkdir -p /mnt/ssd
echo "UUID=$(lsblk -no UUID /dev/sda1) /mnt/ssd btrfs defaults,noatime,compress=zstd:1,nofail,x-systemd.device-timeout=5s,x-gvfs-show 0 0" | sudo tee -a /etc/fstab > /dev/null

sudo systemctl daemon-reload && sudo mount -a
sudo chown "$USER:$USER" /mnt/ssd && chmod 755 /mnt/ssd
mkdir -p /mnt/ssd/{Jogos,Programas,Arquivos}
```

| Parte | O que faz |
|---|---|
| `smartctl -H -A` | Lê a saúde do SSD (`-H`) e os contadores (`-A`). Só leitura. |
| `udisksctl unmount` | Desmonta a montagem temporária que o GNOME fez sozinho. |
| `cp -a /etc/fstab ...` | Faz backup do `fstab` antes de editar. |
| `UUID=$(lsblk -no UUID ...)` | Identifica o disco pelo UUID, que não muda. O nome `sda` pode mudar. |
| `noatime` | Não grava o horário de cada leitura. Poupa escrita no SSD. |
| `compress=zstd:1` | Compressão transparente e rápida, que economiza espaço. |
| `nofail` | Se o disco faltar, o sistema ainda inicia. |
| `x-systemd.device-timeout=5s` | Espera no máximo 5 s pelo disco, em vez dos 90 s padrão. |
| `x-gvfs-show` | Mostra o disco na barra lateral do gerenciador de arquivos. |
| `0 0` | Sem `dump` e sem verificação na inicialização (o btrfs não usa esse recurso). |
| `daemon-reload` e `mount -a` | O systemd relê o `fstab` e monta tudo. Testa a linha sem reiniciar. |
| `chown` e `chmod 755` | O disco passa a ser do usuário, e os demais só leem. |

Como conferir:

```bash
findmnt /mnt/ssd                   # montado, com noatime e compress=zstd:1
grep /mnt/ssd /etc/fstab           # a linha existe
ls -ld /mnt/ssd                    # dono: o usuário, permissão drwxr-xr-x
df -h /mnt/ssd                     # tamanho e espaço livre
sudo btrfs filesystem usage /mnt/ssd
```

Como desfazer:

```bash
sudo umount /mnt/ssd
sudo cp -a /etc/fstab.bak-AAAA-MM-DD /etc/fstab
```

Observações:

- **Automontagem do GNOME:** monta o disco em `/media/USUARIO/UUID`, de forma temporária e com a raiz do disco
  pertencendo ao `root`. Para uso contínuo, configure o `fstab`.
- **UUID no `fstab`:** o nome `sda` pode mudar. O UUID não muda. Não publique o UUID no repositório.
- **`nofail`:** sem ele, a falta do disco pode travar a inicialização.
- **Saúde do SSD:** olhe o resultado geral (`PASSED`), as horas ligado, a vida útil restante e os contadores de
  setores realocados e erros de CRC. Acompanhe também o contador de desligamentos abruptos.
- **`findmnt --verify` sem `sudo`:** mostra avisos de permissão negada. Não são erros do `fstab`.
- **Contadores do SMART depois de reiniciar:** o de desligamentos abruptos (`Unsafe_Shutdown_Count`) não muda num
  reinício limpo. O de ciclos de energia (`Power_Cycle_Count`) também não muda, porque num reinício o SSD continua
  alimentado. Ele só sobe quando o notebook é desligado por completo.
- **Programas:** o APT instala em `/usr` e não permite escolher outro disco. Neste disco vão jogos, arquivos,
  Flatpaks e ferramentas portáteis. Os pacotes do sistema ficam no NVMe.

---

## Fase 3 - Atualizações (concluída)

Objetivo: manter o sistema atualizado com pouco esforço.

- [x] Instalar o `unattended-upgrades` (atualizações automáticas)
- [x] Ativar a execução diária
- [x] Incluir o Google Chrome nas atualizações automáticas
- [x] Validar com uma simulação
- [x] Confirmar a execução automática pelo log em `/var/log/unattended-upgrades/`

```bash
sudo apt install -y unattended-upgrades powermgmt-base

printf 'APT::Periodic::Update-Package-Lists "1";\nAPT::Periodic::Unattended-Upgrade "1";\n' \
  | sudo tee /etc/apt/apt.conf.d/20auto-upgrades > /dev/null

printf '// Inclui as atualizações do Google Chrome (repositório do Google)\nUnattended-Upgrade::Origins-Pattern:: "origin=Google LLC,codename=stable";\n' \
  | sudo tee /etc/apt/apt.conf.d/52unattended-upgrades-local > /dev/null

sudo unattended-upgrade --dry-run --debug
```

| Parte | O que faz |
|---|---|
| `powermgmt-base` | Permite ao sistema saber se o notebook está na bateria. As atualizações automáticas esperam a tomada. |
| `20auto-upgrades` | Liga a rotina diária: `Update-Package-Lists` atualiza a lista de pacotes e `Unattended-Upgrade` instala as atualizações permitidas. |
| `52unattended-upgrades-local` | Arquivo próprio, que acrescenta uma origem à lista (o `::` adiciona sem apagar as do Debian). |
| `origin=Google LLC,codename=stable` | Origem do repositório do Chrome. Sem ela, o Chrome não é atualizado automaticamente. |
| `--dry-run --debug` | Simula e explica o que faria, sem instalar nada. |

Atualização manual, para o que não é automático:

| Comando | O que faz |
|---|---|
| `sudo apt update` | Só baixa a lista atualizada de pacotes. Não instala nada. |
| `sudo apt upgrade` | Instala as atualizações, sem remover nem instalar pacotes novos. |
| `sudo apt full-upgrade` | Igual ao anterior, mas também resolve dependências, instalando ou removendo o que for preciso. **Recomendado.** |
| `sudo apt autoremove` | Remove dependências que ninguém mais usa. |

Como conferir:

```bash
systemctl list-timers apt-daily.timer apt-daily-upgrade.timer   # próximas execuções
apt-config dump | grep -E "APT::Periodic|Origins-Pattern"       # valores efetivos
sudo unattended-upgrade --dry-run --debug                       # "origens permitidas"
sudo tail /var/log/unattended-upgrades/unattended-upgrades.log   # histórico das execuções
```

Como desfazer:

```bash
sudo rm /etc/apt/apt.conf.d/20auto-upgrades              # desliga tudo
sudo rm /etc/apt/apt.conf.d/52unattended-upgrades-local  # tira só o Chrome
```

Observações:

- **Instalar não ativa:** o pacote sozinho não cria o `20auto-upgrades`. Sem ele, nada roda.
- **Arquivo separado:** o `50unattended-upgrades` pertence ao pacote e pode ser sobrescrito. Personalize num arquivo próprio.
- **Padrão do Debian:** atualiza o arquivo principal da versão e as correções de segurança, sem reiniciar sozinho.
- **Kernel novo:** só passa a valer depois de reiniciar. Se o arquivo `/var/run/reboot-required` existir, há reinício pendente.
- **Repositórios externos:** ficam de fora por padrão. Cada um precisa de uma origem na lista.
- **Notebook:** sem o `powermgmt-base`, o `unattended-upgrades` não sabe se está na bateria e pode atualizar sem tomada.
- **Log:** a pasta `/var/log/unattended-upgrades/` só é legível por `root` e pelo grupo `adm`. Sem estar no grupo, use `sudo tail`.
  O `ls` sem `sudo` falha com "Permissão negada". No log, os repositórios que não estão na lista (como o do Docker) aparecem
  como "Marking not allowed", e uma execução sem novidades registra `No packages found that can be upgraded unattended`.
- **Hábito semanal:** o `sudo apt update && sudo apt full-upgrade` cobre o que o automático não pega.

---

## Fase 4 - Notebook (concluída)

Objetivo: ajustar energia, vídeo, suspensão e bateria do ThinkPad E14 Gen 1.

- [x] **Energia e temperatura:** o `power-profiles-daemon` (perfil `balanced`) basta. Em uso leve, a temperatura fica baixa e a ventoinha desligada.
- [x] **GPU AMD sob demanda:** testada com OpenGL e Vulkan. Dorme sozinha depois do uso.
- [x] **Suspensão:** tela apaga em 30 min, suspende em 60 min. Fechar a tampa suspende na hora.
- [x] **Limite de carga da bateria:** opcional e desativado. O procedimento está documentado.
- [x] **Teclas Fn:** funcionam.
- [x] **Leitor de digital:** sem suporte no Linux (ver observações).

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
- **Limite de carga:** o firmware vem com início em 95% e fim em 100%. Depois de ligar o limite, o UPower informa
  75% e 80%. A leitura em `sysfs` mostra `start=80 end=75`, invertida, mas no boot o kernel registra
  `start 75, stop 80` (`journalctl -k -b | grep "battery 1 registered"`). Os valores gravados estão certos e
  persistem depois de reiniciar. O limite não afeta o desempenho e reduz a autonomia por carga em cerca de 20%.
  Com o limite ligado, a carga para em cerca de 80% da capacidade atual da bateria, com o estado `Not charging` e potência de 0 W.
  O painel calcula a porcentagem sobre a capacidade atual, e não sobre a de projeto, e arredonda para baixo, então mostra 79%.
- **Equipamento cedido ou gerenciado por terceiros:** prefira deixar a bateria no comportamento de fábrica. O limite de carga é
  opcional e reversível. Depois de desligá-lo, a carga volta a passar de 80% e o firmware retoma os valores de fábrica.
- **Leitor de digital:** o Goodix `27c6:55a4` está na seção "Known unsupported devices" da libfprint, e o `fprintd-list`
  responde `No devices available`. Não há o que configurar.
- **Logs:** sem estar no grupo `systemd-journal`, o `journalctl` mostra "sem entradas". Isso **não** significa "sem erros".
- **Erros do boot:** ver a subseção abaixo.

### Erros do boot

`journalctl -b -p err` lista os erros do boot atual. As linhas de erro deste notebook vêm de quatro causas, mais o relatório do Wi-Fi
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

---

## Fase 5 - Aplicativos (concluída)

Objetivo: instalar apenas o que será usado, nesta ordem de preferência: pacote do Debian, recurso nativo do GNOME, Flatpak.

- [x] **Navegador:** Google Chrome, instalado pelo pacote `.deb` oficial. O pacote configura o repositório do Google, e as atualizações chegam pelo `apt`.
- [x] **VPN corporativa:** ver abaixo.
- [x] **Perfil de uso:** desenvolvimento como foco, com uso geral.
- [x] **Lote 1:** Flatpak, base de desenvolvimento, fontes, Wireshark, Meld e Docker.
- [x] **Lote 2:** Go 1.27, Node 24 LTS, TypeScript 7, Python 3.14 e Java 25 LTS.
- [x] **Lote 3:** VS Code, DBeaver e AnyDesk (repositórios dos fabricantes), Postman, SoapUI e Discord (Flatpak).
- [x] **LibreOffice:** configurado para se parecer com o Office.
- [x] **Lote 4:** VLC, utilitários de diagnóstico e de rede (Debian) e Steam (Flatpak).
- Aplicativos adicionais, como GIMP, OBS Studio e Inkscape, entram sob demanda, pelo Debian ou pelo Flatpak.

### Lote 1: base de desenvolvimento, fontes e Docker

```bash
# Flatpak e Flathub
sudo apt install -y flatpak gnome-software-plugin-flatpak
sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Compilador, Meld, fontes e português do LibreOffice
sudo apt install -y build-essential meld fonts-crosextra-carlito fonts-crosextra-caladea fonts-noto-core fonts-firacode \
  libreoffice-help-pt-br hyphen-pt-br mythes-pt-br

# Wireshark, com captura permitida ao grupo wireshark
echo "wireshark-common wireshark-common/install-setuid boolean true" | sudo debconf-set-selections
sudo apt install -y wireshark && sudo usermod -aG wireshark "$USER"

# Habilitar o contrib no formato moderno de repositórios
sudo cp -a /etc/apt/sources.list /etc/apt/sources.list.bak-$(date +%F)
sudo apt modernize-sources -y
sudo sed -i 's/^Components: main non-free-firmware$/Components: main contrib non-free-firmware/' /etc/apt/sources.list.d/debian.sources
sudo sed -i 's/^Types: deb deb-src$/Types: deb/' /etc/apt/sources.list.d/debian.sources
sudo apt update

# Fontes da Microsoft (aceita o EULA delas)
echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | sudo debconf-set-selections
sudo apt install -y ttf-mscorefonts-installer && sudo fc-cache -f

# Repositório oficial do Docker e instalação
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
printf 'Types: deb\nURIs: https://download.docker.com/linux/debian\nSuites: trixie\nComponents: stable\nArchitectures: amd64\nSigned-By: /etc/apt/keyrings/docker.asc\n' \
  | sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Docker sem sudo e rede fora das faixas da VPN
sudo usermod -aG docker "$USER"
printf '{\n  "bip": "100.64.0.1/24",\n  "default-address-pools": [\n    {"base": "100.65.0.0/16", "size": 24}\n  ]\n}\n' \
  | sudo tee /etc/docker/daemon.json > /dev/null
sudo systemctl restart docker
```

| Parte | O que faz |
|---|---|
| `flatpak remote-add ... flathub` | Adiciona a loja Flathub. Os aplicativos Flatpak ficam no disco do sistema. |
| `fonts-crosextra-carlito` e `-caladea` | Substituem o Calibri e o Cambria com as mesmas métricas, então o layout dos documentos se mantém. |
| `install-setuid true` | Permite capturar pacotes sem ser administrador, para quem estiver no grupo `wireshark`. |
| `apt modernize-sources` | Converte `/etc/apt/sources.list` para o formato `.sources`, recomendado no Debian 13. |
| `Components: main contrib ...` | Habilita o `contrib`, que contém o `ttf-mscorefonts-installer`. |
| `Types: deb` | Remove o código-fonte (`deb-src`), que o `apt update` baixaria sem necessidade. |
| `ttf-mscorefonts-installer` | Baixa as fontes da Microsoft (Arial, Times New Roman, Verdana e outras) e as instala. |
| `Signed-By: ...docker.asc` | Só aceita pacotes assinados pela chave do Docker. Confira o fingerprint antes de usar. |
| `usermod -aG docker` | Permite usar o Docker sem `sudo`. Vale a partir do próximo login. |
| `bip` e `default-address-pools` | Fixam as redes do Docker na faixa `100.64.0.0/10`, fora das faixas da VPN. |

Como conferir:

```bash
fc-match Calibri; fc-match Cambria; fc-match Arial        # Carlito, Caladea e Arial
gpg --show-keys --with-fingerprint /etc/apt/keyrings/docker.asc   # 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
docker --version && docker compose version
docker run --rm hello-world                               # sem sudo, depois de um novo login
ip -br addr show docker0                                  # 100.64.0.1/24
ip route get 172.17.5.5                                   # com a VPN ativa: dev do túnel, não docker0
```

Como desfazer:

```bash
sudo apt remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo rm /etc/apt/sources.list.d/docker.sources /etc/apt/keyrings/docker.asc /etc/docker/daemon.json
sudo apt remove ttf-mscorefonts-installer wireshark meld flatpak
sudo rm /etc/apt/sources.list.d/debian.sources && sudo cp -a /etc/apt/sources.list.bak-AAAA-MM-DD /etc/apt/sources.list && sudo apt update
```

Observações:

- **Fontes:** o Calibri e o Cambria originais não têm fonte gratuita legal. O Carlito e o Caladea são clones métricos e
  mantêm o layout. Arial, Times New Roman e as demais fontes da Microsoft vêm do `ttf-mscorefonts-installer`,
  que exige aceitar o EULA delas.
- **Docker e VPN:** uma VPN corporativa costuma rotear as faixas privadas inteiras (`10.0.0.0/8`, `172.16.0.0/12` e
  `192.168.0.0/16`), e o Docker usa `172.17.0.0/16` até `172.31.0.0/16` e `192.168.0.0/16`. Com a VPN ativa, um pacote para
  `172.17.x.x` vai para o `docker0`, e não para a VPN. Confira com `ip route get 172.17.5.5`. A correção é fixar a rede do
  Docker numa faixa livre (`bip` e `default-address-pools`). Escolha a faixa comparando com as rotas reais da VPN.
- **Grupo `docker`:** equivale a ser administrador, porque quem controla o Docker pode montar o disco inteiro num contêiner.
  O modo *rootless* é mais seguro, mas limita a rede dos contêineres.
- **Grupos e sessão:** `docker` e `wireshark` só valem depois de sair da sessão gráfica e entrar de novo. Fechar o terminal não basta.
  Um novo login também faz os aplicativos Flatpak aparecerem no menu.
- **Docker e atualizações automáticas:** o Docker não entra no `unattended-upgrades`, porque atualizar o serviço o reinicia e para os contêineres.
  Atualize com o `sudo apt full-upgrade` semanal.
- **`deb-src`:** removido de `debian.sources` (`Types: deb`). Sem necessidade de código-fonte, o `apt update` baixa menos índices.
- **Repositórios externos:** o `debian.sources` e o `docker.sources` usam `Signed-By`, que limita cada chave ao seu repositório.

### Lote 2: linguagens

```bash
# Go: arquivo oficial, com conferência do SHA-256 publicado em go.dev/dl
curl -fsSL https://go.dev/dl/go1.27.1.linux-amd64.tar.gz -o go.tar.gz
echo "63d339f0da5ab53635a56f2490a7984dfe12dfcff22ad749f63edaf590168445  go.tar.gz" | sha256sum -c -
mkdir -p ~/.local && tar -C ~/.local -xzf go.tar.gz

# fnm, Node 24 LTS e TypeScript 7 (o instalador é lido antes de ser executado)
curl -fsSL https://fnm.vercel.app/install -o fnm-install.sh
bash fnm-install.sh --skip-shell
export PATH="$HOME/.local/share/fnm:$PATH" && eval "$(fnm env --shell bash)"
fnm install 24 && fnm default 24
npm install -g typescript@7

# uv e Python 3.14
curl -LsSf https://astral.sh/uv/install.sh -o uv-install.sh
UV_NO_MODIFY_PATH=1 sh uv-install.sh
uv python install 3.14

# SDKMAN e Java 25 LTS (Temurin). O SDKMAN exige zip e unzip
sudo apt install -y zip
curl -fsSL https://get.sdkman.io -o sdkman-install.sh
bash sdkman-install.sh
source ~/.sdkman/bin/sdkman-init.sh
sdk install java 25.0.4-tem

# PATH e integração do fnm no shell
cp -a ~/.bashrc ~/.bashrc.bak-$(date +%F)
cat >> ~/.bashrc <<'EOF'
export PATH="$HOME/.local/go/bin:$HOME/go/bin:$HOME/.local/share/fnm:$PATH"
eval "$(fnm env --use-on-cd --shell bash)"
EOF
```

| Parte | O que faz |
|---|---|
| `sha256sum -c` | Confere que o arquivo baixado é o publicado pelo Go. |
| `--skip-shell` | Impede que o instalador do fnm edite o `~/.bashrc`. A integração é feita à parte, de forma visível. |
| `fnm default 24` | Define o Node 24 como padrão em todo terminal novo. |
| `UV_NO_MODIFY_PATH=1` | Impede que o uv altere o `PATH`. O `~/.local/bin` já está nele. |
| `uv python install 3.14` | Instala o Python 3.14 na pasta pessoal, sem tocar no Python do sistema. |
| `fnm env --use-on-cd` | Troca a versão do Node ao entrar numa pasta com `.node-version` ou `.nvmrc`. |
| `sdk install java 25.0.4-tem` | Instala o Temurin 25 LTS e o define como padrão, com `JAVA_HOME` apontando para ele. |

Como conferir:

```bash
go version && node --version && npm --version && tsc --version
uv --version && python3.14 --version
java -version && javac -version
python3 --version        # continua sendo o Python do sistema
```

Como desfazer:

```bash
rm -rf ~/.local/go ~/go
rm -rf ~/.local/share/fnm ~/.local/state/fnm
rm -f ~/.local/bin/uv ~/.local/bin/uvx ~/.local/bin/python3.14 && rm -rf ~/.local/share/uv
rm -rf ~/.sdkman
cp -a ~/.bashrc.bak-AAAA-MM-DD ~/.bashrc
```

Observações:

- **Sem `sudo`:** tudo fica na pasta pessoal, e o Python do sistema não é tocado. O `python3.14` é um comando à parte.
- **Ler antes de executar:** o instalador do fnm edita o `~/.bashrc` se não receber `--skip-shell`, o do uv altera o `PATH`
  sem `UV_NO_MODIFY_PATH=1`, e o do SDKMAN sempre acrescenta um bloco ao `~/.bashrc`. O do fnm não confere checksum.
- **TypeScript global:** fica dentro da versão do Node. Ao instalar outro Node, reinstale com `npm install -g typescript@7`,
  ou use o TypeScript de cada projeto.
- **Node 24 LTS:** quando o Node 26 virar LTS, instale-o com `fnm install 26` e `fnm default 26`.
- **Go:** o `GOTOOLCHAIN=auto` baixa sozinho o toolchain que o `go.mod` de um projeto pedir.
- **Python:** não use `pip install` no Python do sistema, que é protegido (PEP 668). Use `uv venv`, `uv run` ou `pipx`.
- **SDKMAN:** o instalador sempre acrescenta um bloco ao `~/.bashrc`, que deve ficar no **fim** do arquivo. Só confere a integridade do zip,
  e não a autenticidade. Exige `zip` e `unzip`.
- **Java:** outras versões se instalam com `sdk install java <identificador>` e se alternam com `sdk use` ou `sdk default`.
  `sdk list java` mostra os identificadores. Maven e Gradle também vêm pelo SDKMAN (`sdk install maven`).

### Lote 3: aplicativos

Repositórios dos fabricantes (VS Code, AnyDesk e DBeaver) e Flatpak (Postman, SoapUI e Discord).

```bash
# 1. Baixar as chaves dos fabricantes e conferir o fingerprint
mkdir ~/lote3 && cd ~/lote3
curl -fsSL https://packages.microsoft.com/keys/microsoft.asc -o microsoft.asc
curl -fsSL https://keys.anydesk.com/repos/DEB-GPG-KEY -o anydesk.asc
curl -fsSL https://dbeaver.io/debs/dbeaver.gpg.key -o dbeaver.asc
gpg --show-keys --with-fingerprint microsoft.asc anydesk.asc dbeaver.asc

# 2. Provar que cada chave assina o repositório do fabricante
G=$(mktemp -d); chmod 700 $G
gpg --homedir $G --import microsoft.asc
curl -fsSL https://packages.microsoft.com/repos/code/dists/stable/InRelease -o $G/InRelease
gpg --homedir $G --verify $G/InRelease        # "Good signature"; repita para AnyDesk e DBeaver

# 3. Arquivos de repositório (formato .sources), por exemplo vscode.sources
#    Types: deb
#    URIs: https://packages.microsoft.com/repos/code
#    Suites: stable
#    Components: main
#    Architectures: amd64
#    Signed-By: /etc/apt/keyrings/microsoft.asc
#    (AnyDesk: URIs https://deb.anydesk.com, Suites all, Components main)
#    (DBeaver: URIs https://dbeaver.io/debs/dbeaver-ce, Suites /, sem Components)

# 4. Instalar chaves e repositórios
sudo install -d -m 755 /etc/apt/keyrings
sudo install -m 644 microsoft.asc anydesk.asc dbeaver.asc /etc/apt/keyrings/
sudo install -m 644 vscode.sources anydesk.sources dbeaver.sources /etc/apt/sources.list.d/
sudo apt update

# 5. Instalar (o debconf impede o pacote do VS Code de registrar o repositório por conta própria)
echo "code code/add-microsoft-repo boolean false" | sudo debconf-set-selections
sudo apt install -y code anydesk dbeaver-ce

# 6. Postman, SoapUI e Discord
sudo flatpak install -y flathub com.getpostman.Postman org.soapui.SoapUI com.discordapp.Discord

# 7. Incluir os repositórios nas atualizações automáticas: o arquivo 52unattended-upgrades-local passa a ter as regras
#    do Chrome (já existente) e estas três, uma por linha, no formato Unattended-Upgrade::Origins-Pattern:: "..."
#    origin=code stable,codename=stable
#    origin=philandro Software GmbH,codename=all
#    site=dbeaver.io                     (o repositório do DBeaver não informa Origin nem Label)
sudo install -m 644 52unattended-upgrades-local /etc/apt/apt.conf.d/52unattended-upgrades-local
sudo unattended-upgrade --dry-run --debug
```

| Fabricante | Fingerprint da chave | Repositório |
|---|---|---|
| Microsoft (VS Code) | `BC52 8686 B50D 79E3 39D3 721C EB3E 94AD BE12 29CF` | `packages.microsoft.com/repos/code` |
| AnyDesk | `06B5 EA2F AE20 8E7C DA97 61DC A2FB 21D5 A877 2835` | `deb.anydesk.com` |
| DBeaver | `BDFB 19F6 8151 4B43 875D 16FA 132C 13A8 A330 F403` | `dbeaver.io/debs/dbeaver-ce` |

Como conferir:

```bash
code --version && dbeaver --help >/dev/null && dpkg -s anydesk | grep Status
flatpak list --app
ls /etc/apt/sources.list.d/                      # sem vscode.list nem dbeaver.list duplicados
sudo unattended-upgrade --dry-run --debug        # as origens novas aparecem em "origens permitidas"
```

Como desfazer:

```bash
sudo apt remove code anydesk dbeaver-ce
sudo rm /etc/apt/sources.list.d/{vscode,anydesk,dbeaver}.sources /etc/apt/keyrings/{microsoft,anydesk,dbeaver}.asc
sudo flatpak uninstall com.getpostman.Postman org.soapui.SoapUI com.discordapp.Discord
```

Observações:

- **Conferência das chaves:** só a Microsoft publica o fingerprint de forma amplamente conhecida. Para os outros, a garantia vem do
  download por HTTPS do domínio do fabricante e de a chave **assinar de fato** os metadados do repositório (`gpg --verify`).
- **Teste em área isolada:** o `apt-get` aceita `-o Dir::State::Lists=... -o Dir::Etc::sourceparts=...` e roda sem `sudo`, o que permite
  testar um repositório novo sem tocar em `/etc`.
- **Duplicidade de repositório:** o pacote `code` registra o repositório da Microsoft sozinho, o que duplicaria o `vscode.sources`.
  O `debconf-set-selections` acima impede isso. Confira com `apt update`, que avisa de destinos configurados duas vezes.
- **Erro `Unit anydesk.service not loaded`:** aparece na instalação, porque o pacote tenta parar um serviço que ainda não existe.
  É inofensivo.
- **AnyDesk:** o pacote instala e habilita um serviço que escuta portas de entrada. Ver "AnyDesk sob demanda" abaixo.
- **DBeaver:** instala em `/usr/share/dbeaver-ce`, com um Java embutido (OpenJDK 25), e independe do SDKMAN.
- **Flatpak:** cada aplicativo pede uma versão diferente da base (24.08, 25.08 e 26.08), e cada uma ocupa cerca de 700 MB, além do Mesa.
  Os três aplicativos, com as bases, ocupam cerca de 5 GB. O Postman baixa o binário do fabricante na instalação (*extra-data*).
- **Permissões dos Flatpak:** o SoapUI só acessa Documentos. O Postman acessa a pasta pessoal inteira. O Discord acessa Downloads e
  **todos os dispositivos** (câmera e microfone). Dá para restringir com `flatpak override`.
- **Docker fora das atualizações automáticas:** os quatro repositórios externos restantes (Chrome, VS Code, AnyDesk e DBeaver) entram;
  o Docker não, porque atualizar o serviço reinicia os contêineres.
- **VS Code e a Fira Code:** em `~/.config/Code/User/settings.json`, use `editor.fontFamily`, `editor.fontLigatures: true` e
  `terminal.integrated.fontFamily`.

#### AnyDesk sob demanda

O pacote instala e habilita um serviço (`anydesk --service`, como `root`) que escuta em `0.0.0.0:7070` (TCP) e `50001` (UDP), para
conexões diretas de entrada, e um autostart que abre o ícone da bandeja em todo login. Para usar o AnyDesk só de saída e só com o
aplicativo aberto:

```bash
sudo systemctl disable --now anydesk       # desliga o serviço e impede que suba no boot
mkdir -p ~/.config/autostart
printf '[Desktop Entry]\nType=Application\nName=AnyDesk Tray\nHidden=true\nX-GNOME-Autostart-enabled=false\n' \
  > ~/.config/autostart/anydesk_global_tray.desktop     # a bandeja deixa de abrir no login
```

Sem o serviço do sistema, o aplicativo abre um serviço local do próprio usuário (`--local-service`, sem `root`), registra-se na rede
do AnyDesk e o encerra sozinho ao fechar (`Initiating auto-shutdown` no registro `~/.anydesk/anydesk.trace`).

Como conferir:

```bash
systemctl is-active anydesk               # inactive
pgrep -c -x anydesk                       # 0 com o aplicativo fechado
ss -ltn | grep -c ':7070 '                # 0 com o aplicativo fechado
```

Como desfazer: `sudo systemctl enable --now anydesk` e `rm ~/.config/autostart/anydesk_global_tray.desktop`.

Observações:

- O registro do aplicativo mostra que ele se registra na rede do AnyDesk e se encerra sozinho ao fechar. A conexão de saída depende de uma
  máquina de destino e não foi validada neste guia.
- Se algo exigir o serviço do sistema (acesso sem supervisão, por exemplo), reative-o com o `enable --now` acima.

#### Serviços e portas em escuta

Um serviço instalado junto com o sistema ou com um aplicativo pode abrir portas sem que isso fique evidente. Para listar o que está em escuta:

```bash
ss -ltn      # portas TCP em escuta
ss -lun      # portas UDP em escuta
```

`0.0.0.0` e `[::]` aceitam conexões de qualquer interface. `127.0.0.1` e `[::1]` aceitam só da própria máquina. O `ss -p`, sem `sudo`,
só mostra os processos do próprio usuário, então serviços do `root` aparecem sem nome.

O instalador do Debian pode ativar um servidor SSH. Para mantê-lo instalado, mas desligado, e ligá-lo só quando precisar:

```bash
sudo systemctl disable --now ssh ssh.socket     # desliga agora e impede que suba no boot
sudo systemctl start ssh                        # liga sob demanda (não sobrevive ao reinício)
sudo systemctl stop ssh                         # desliga ao terminar
```

Com isso, a única porta em escuta fica sendo a do serviço de impressão, restrita à própria máquina.

#### LibreOffice parecido com o Office

Objetivo: faixa de opções em abas, ícones no estilo do Office e gravação em `.docx`, `.xlsx` e `.pptx`, mantendo o layout dos documentos.
Os pacotes de idioma, ajuda, dicionário e as fontes compatíveis já vêm do lote 1.

Pela interface:

- *Exibir > Interface do usuário...*, escolher *Em abas* e clicar em *Aplicar a todos*.
- *Ferramentas > Opções > LibreOffice > Exibir > Estilo dos ícones*: Colibre (SVG).
- *Ferramentas > Opções > Carregar/Salvar > Geral*: em "Sempre salvar como", o formato do Office para cada tipo de documento, e
  desmarcar o aviso ao gravar fora do formato ODF.

Pelo perfil, com o LibreOffice **fechado** (ele reescreve o arquivo ao sair):

```bash
pgrep -cx soffice.bin                                # 0: LibreOffice fechado
soffice --headless --norestore --terminate_after_init   # cria o perfil, se ainda não existir
cp -a ~/.config/libreoffice/4/user/registrymodifications.xcu{,.bak}
```

Itens acrescentados ao `registrymodifications.xcu` (uma linha `<item>` para cada um, antes do `</oor:items>`):

| Ajuste | Caminho e propriedade | Valor |
|---|---|---|
| Faixa em abas | `/org.openoffice.Office.UI.ToolbarMode/Applications/Writer`, `Calc`, `Impress` e `Draw`, propriedade `Active` | `notebookbar.ui` |
| Layout da faixa | `/org.openoffice.Office.UI.ToolbarMode`, propriedades `ActiveWriter`, `ActiveCalc`, `ActiveImpress` e `ActiveDraw` | `notebookbar.ui` |
| Ícones | `/org.openoffice.Office.Common/Misc`, propriedade `SymbolStyle` | `colibre_svg` |
| Formato do Word | `/org.openoffice.Setup/Office/Factories/com.sun.star.text.TextDocument`, propriedade `ooSetupFactoryDefaultFilter` | `Office Open XML Text` |
| Formato do Excel | `.../com.sun.star.sheet.SpreadsheetDocument`, mesma propriedade | `Calc MS Excel 2007 XML` |
| Formato do PowerPoint | `.../com.sun.star.presentation.PresentationDocument`, mesma propriedade | `Impress MS PowerPoint 2007 XML` |
| Sem aviso de formato | `/org.openoffice.Office.Common/Save/Document`, propriedade `WarnAlienFormat` | `false` |
| Sem linhas de margem na folha | `/org.openoffice.Office.Writer/Content/Display`, propriedade `TextBoundaries` | `false` |
| Régua vertical | `/org.openoffice.Office.Writer/Layout/Window`, propriedade `VerticalRuler` | `true` |

Os três formatos padrão de gravação só persistem quando gravados pela API de configuração (`ConfigurationUpdateAccess`, como no exemplo do
modelo padrão mais abaixo). O mesmo valor escrito à mão no `registrymodifications.xcu` pode ser descartado e o *Salvar como* volta a sugerir ODF.

Cada item segue este formato:

```xml
<item oor:path="/org.openoffice.Office.UI.ToolbarMode/Applications/Writer"><prop oor:name="Active" oor:op="fuse"><value>notebookbar.ui</value></prop></item>
```

Como conferir:

```bash
# Fidelidade de fontes: gera um documento com Calibri, Cambria e Arial e confere o resultado
soffice --headless --convert-to docx teste.fodt && soffice --headless --convert-to pdf teste.fodt
unzip -p teste.docx word/document.xml | grep -o 'w:ascii="[^"]*"' | sort -u   # nomes da Microsoft
pdffonts teste.pdf                                                             # Carlito, Caladea e Arial
```

Ao abrir o Writer, a faixa deve mostrar abas (Arquivo, Página Inicial, Inserir, Layout...), e o *Salvar como* deve sugerir `.docx`.

Como desfazer: com o LibreOffice fechado, `cp -a ~/.config/libreoffice/4/user/registrymodifications.xcu.bak ~/.config/libreoffice/4/user/registrymodifications.xcu`.

Observações:

- **Nomes de fonte no arquivo:** o `.docx` grava `Calibri` e `Cambria`, e o LibreOffice as exibe com o Carlito e o Caladea, de mesmas métricas.
  O Word de quem receber usa as originais, e o layout se mantém.
- **Valor da chave `Active`:** é o valor do modo (`notebookbar.ui`), e não o nome exibido no menu (`Tabbed`). Um texto que não seja o
  valor de um modo é aceito pela API, mas ignorado, e o LibreOffice abre no modo clássico.
- **Valores dos modos:** `Default` (menus clássicos), `Single`, `Sidebar`, `notebookbar.ui` (em abas), `notebookbar_compact.ui` (em abas,
  mais baixa), `notebookbar_groupedbar_compact.ui`, `notebookbar_groupedbar_full.ui`, `notebookbar_single.ui` e `notebookbar_groups.ui`.
- **Descobrir a chave certa:** faça a escolha pela interface e compare o `registrymodifications.xcu` antes e depois. O LibreOffice grava
  exatamente o que ele lê. Ler a chave de volta pela API (UNO) confirma que o valor existe, mas não que o programa o reconhece.
- **Ícones:** o Colibre é o tema do LibreOffice no Windows e o mais próximo do Office. A variante escura é a `colibre_dark_svg`.
- **Fonte e estilos de documentos novos:** ver "LibreOffice com os padrões do Microsoft 365" abaixo.
- **Processo aberto:** `pgrep -f soffice.bin` casa com a própria linha de comando do `pgrep`. Use `pgrep -x soffice.bin`.
- **Documentos complexos:** se a fidelidade em documentos muito formatados não bastar, o ONLYOFFICE (Flatpak) reproduz melhor o Office, com a
  interface em faixa de opções.

#### LibreOffice com os padrões do Microsoft 365

Objetivo: aproximar a aparência do que se produz no LibreOffice do que o Microsoft 365 novo produz, mantendo o layout dos documentos.

**Formato padrão do Word.** O filtro `Word 2007–365` (`MS Word 2007 XML`) grava `compatibilityMode=12`, e o Word abre o arquivo em modo de
compatibilidade. O filtro `Word 2010–365` (`Office Open XML Text`) grava `15` e abre normalmente. Por isso o padrão é o segundo:

```bash
soffice --headless --convert-to 'docx:MS Word 2007 XML'   --outdir a arquivo.fodt
soffice --headless --convert-to 'docx:Office Open XML Text' --outdir b arquivo.fodt
unzip -p a/arquivo.docx word/settings.xml | grep -o 'compatibilityMode"[^>]*'   # w:val="12"
unzip -p b/arquivo.docx word/settings.xml | grep -o 'compatibilityMode"[^>]*'   # w:val="15"
```

**Fundo do documento em modo escuro.** Com o tema escuro, a folha também fica escura. Para manter a folha branca:
*Ferramentas > Opções > LibreOffice > Cores do aplicativo > Fundo do documento*, em branco. A escolha vale para todos os aplicativos e é gravada em
`/org.openoffice.Office.UI/ColorScheme/ColorSchemes/...['COLOR_SCHEME_LIBREOFFICE_AUTOMATIC']/DocColor`, na propriedade `Dark` (`16777215`).

**Fonte Aptos.** A Aptos, padrão do Microsoft 365 novo, é proprietária e não existe para Linux. Sem ela, o sistema a troca por Noto Sans, bem mais
larga, e os documentos recebidos mudam de paginação. A regra abaixo escolhe a fonte instalada de largura mais próxima.
Largura do mesmo parágrafo, com a Calibri (Carlito) em 100%, lida dos arquivos de fonte:

| Fonte | Largura |
|---|---|
| Liberation Sans Narrow | 90,3% |
| Carlito (Calibri) | 100% |
| Liberation Sans (Arial) | 110,1% |
| Noto Sans | 115,5% |
| DejaVu Sans | 124,7% |

```bash
mkdir -p ~/.config/fontconfig/conf.d
cat > ~/.config/fontconfig/conf.d/60-aptos-substituta.conf <<'EOF'
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
  <alias binding="same"><family>Aptos</family><prefer><family>Liberation Sans</family></prefer></alias>
  <alias binding="same"><family>Aptos Display</family><prefer><family>Liberation Sans</family></prefer></alias>
  <alias binding="same"><family>Aptos Narrow</family><prefer><family>Liberation Sans Narrow</family></prefer></alias>
  <alias binding="same"><family>Aptos Mono</family><prefer><family>Liberation Mono</family></prefer></alias>
</fontconfig>
EOF
fc-cache -f && fc-match Aptos          # Liberation Sans
```

O arquivo `.docx` continua gravando o nome `Aptos`, então o Word de quem receber usa a fonte original. A substituta só vale na tela e no PDF gerado aqui.
É uma aproximação: sem a Aptos instalada, não dá para medir a diferença exata.

**Idioma.** Interface, formatos e ortografia em português do Brasil. Pacotes: `libreoffice-l10n-pt-br`, `libreoffice-help-pt-br`,
`hunspell-pt-br`, `hyphen-pt-br` e `mythes-pt-br`. Chaves gravadas pela API de configuração:

| Ajuste | Caminho e propriedade | Valor |
|---|---|---|
| Idioma da interface | `/org.openoffice.Setup/L10N`, propriedade `ooLocale` | `pt-BR` |
| Formatos de data, número e moeda | `/org.openoffice.Setup/L10N`, propriedade `ooSetupSystemLocale` | `pt-BR` |
| Idioma padrão dos documentos (ortografia) | `/org.openoffice.Office.Linguistic/General`, propriedade `DefaultLocale` | `pt-BR` |

Os modelos abaixo também gravam `pt-BR` no estilo padrão de cada aplicativo. Scripts UNO devem rodar com o idioma do usuário (`LANG=pt_BR.UTF-8`).
Com `LC_ALL=C`, o LibreOffice trata a sessão como inglesa e os nomes de planilha, de estilo e da interface saem em inglês.

**Grade do Calc e paleta de cores.** A grade fica em cinza claro (`#D4D4D4`), como no Excel, nas propriedades `Light` e `Dark` do nó `CalcGrid`
do esquema de cores. A paleta *Office* (arquivo `~/.config/libreoffice/4/user/config/Office.soc`) traz as cores do tema do Office 2023 em diante, com as cinco
variações de cada uma (mais claro 80%, 60% e 40%, mais escuro 25% e 50%), calculadas em HSL, e as dez cores padrão. Está selecionada em
`/org.openoffice.Office.Common/UserColors`, propriedade `PaletteName` (`Office`):

| Cor do tema | Valor |
|---|---|
| Texto 2 | `#0E2841` |
| Destaque 1 a 6 | `#156082`, `#E97132`, `#196B24`, `#0F9ED5`, `#A02B93`, `#4EA72E` |
| Hiperlink e hiperlink visitado | `#467886` e `#96607D` |

**Modelos padrão.** Os modelos abaixo aproximam os padrões do Word, do Excel e do PowerPoint do Microsoft 365 novo. São criados por um script UNO num
perfil temporário e definidos como padrão pela API de configuração:

| Estilo | Valor |
|---|---|
| Padrão (Word) | Aptos 11 pt, 8 pt depois do parágrafo, entrelinha 1,15 |
| Título 1, 2 e 3 | Aptos Display 20 pt, Aptos Display 16 pt e Aptos 14 pt, cor `#0F4761`, sem negrito |
| Título e Subtítulo | Aptos Display 28 pt, e Aptos 14 pt na cor `#595959` |
| Página | A4, margens de 2,54 cm nos quatro lados (predefinição *Normal* do Word) |

Modelo do Excel (`Excel365.ots`):

| Item | Valor |
|---|---|
| Célula padrão | Aptos Narrow 11 pt, `pt-BR` |
| Planilha | uma, chamada `Planilha1` |
| Coluna | 1,693 cm (8,43 caracteres, 64 px) |
| Página | A4, margens de 1,91 cm em cima e embaixo e de 1,78 cm nas laterais (predefinição *Normal* do Excel), sem cabeçalho e sem rodapé |

Modelo do PowerPoint (`Impress365.otp`):

| Item | Valor |
|---|---|
| Slide | 33,867 x 19,05 cm (16:9, o tamanho padrão do PowerPoint) |
| Título | Aptos Display 44 pt, preto, centralizado na vertical, em uma caixa de 29,21 x 3,68 cm a 2,33 cm da borda esquerda |
| Texto | Aptos 28, 24, 20 e 18 pt (níveis 1 a 5 e seguintes), entrelinha 90%, 10 pt antes do parágrafo |
| Marcadores | `•` em Arial, tamanho 100%, recuo de 0,635 cm e mais 1,27 cm por nível |
| Data, rodapé e número | Aptos 12 pt, cinza `#898989` |
| Slide 1 | *Slide de título*: título de 60 pt centralizado e ancorado embaixo, subtítulo de 24 pt centralizado |
| Formas e caixas de texto | Aptos 18 pt |

Os valores de posição do PowerPoint vêm do tema padrão do Office (`12192000 x 6858000` EMU, com 1 EMU = 1/360 de centésimo de milímetro).

```python
# soffice --headless --accept="socket,host=127.0.0.1,port=2005;urp;" &   (com um perfil temporário, se preferir)
import uno, os
from com.sun.star.beans import PropertyValue
def pv(n, v): p = PropertyValue(); p.Name = n; p.Value = v; return p
ctx = uno.getComponentContext().ServiceManager.createInstanceWithContext("com.sun.star.bridge.UnoUrlResolver", uno.getComponentContext()) \
        .resolve("uno:socket,host=127.0.0.1,port=2005;urp;StarOffice.ComponentContext")
desktop = ctx.ServiceManager.createInstanceWithContext("com.sun.star.frame.Desktop", ctx)
def espacamento(pct):
    ls = uno.createUnoStruct("com.sun.star.style.LineSpacing"); ls.Mode = 0; ls.Height = pct; return ls
doc = desktop.loadComponentFromURL("private:factory/swriter", "_blank", 0, (pv("Hidden", True),))
estilos = doc.StyleFamilies.getByName("ParagraphStyles")
estilos.getByName("Standard").setPropertyValue("CharFontName", "Aptos")
estilos.getByName("Standard").setPropertyValue("CharHeight", 11.0)
estilos.getByName("Standard").setPropertyValue("ParaBottomMargin", 282)          # 8 pt, em centésimos de mm
estilos.getByName("Standard").setPropertyValue("ParaLineSpacing", espacamento(115))
# ... Título 1 a 3, Título, Subtítulo e página, como na tabela acima ...
doc.storeToURL("file://" + os.path.expanduser("~/.config/libreoffice/4/user/template/Word365.ott"), (pv("FilterName", "writer8_template"),))
doc.close(True)
```

Definir os modelos como padrão, com o LibreOffice fechado ou por uma sessão UNO:

```python
cp = ctx.ServiceManager.createInstanceWithContext("com.sun.star.configuration.ConfigurationProvider", ctx)
modelos = {"com.sun.star.text.TextDocument": "Word365.ott",
           "com.sun.star.sheet.SpreadsheetDocument": "Excel365.ots",
           "com.sun.star.presentation.PresentationDocument": "Impress365.otp"}
for fabrica, arquivo in modelos.items():
    no = cp.createInstanceWithArguments("com.sun.star.configuration.ConfigurationUpdateAccess",
            (pv("nodepath", "/org.openoffice.Setup/Office/Factories/" + fabrica),))
    no.setPropertyValue("ooSetupFactoryTemplateFile", "file://" + os.path.expanduser("~/.config/libreoffice/4/user/template/" + arquivo))
    no.commitChanges()
```

Como conferir, abrindo um documento novo de cada tipo:

```python
d = desktop.loadComponentFromURL("private:factory/swriter", "_blank", 0, (pv("Hidden", True),))
e = d.StyleFamilies.getByName("ParagraphStyles").getByName("Standard")
print(e.CharFontName, e.CharHeight)      # Aptos 11.0
# Calc: "scalc", estilo de célula "Default", estilo de página "Default"
# Impress: "simpress", d.StyleFamilies.getByName(d.MasterPages.getByIndex(0).Name).getByName("title")
```

Como desfazer: apague a chave `ooSetupFactoryTemplateFile` de cada fábrica pela mesma API (valor vazio), remova os arquivos de
`~/.config/libreoffice/4/user/template/` e o `Office.soc`, e restaure o perfil com o backup `registrymodifications.xcu.bak`.

Observações:

- **Editar o perfil à mão nem sempre funciona:** o LibreOffice descartou o valor de `ooSetupFactoryTemplateFile` escrito direto no
  `registrymodifications.xcu`. Pela API de atualização de configuração, o mesmo valor persistiu. Confirme sempre abrindo um documento novo.
- **Validar pela API:** ler uma chave de volta não prova que o programa a usa. Abra um documento novo e leia os estilos.
- **Estilos derivados:** os estilos que não foram ajustados (lista, legenda, índice) continuam em Liberation. Só os estilos da tabela acima seguem o Word.
- **Margens e valores dos estilos:** as margens seguem a predefinição *Normal* documentada pela Microsoft (2,54 cm no Word). Um documento em branco criado
  no Word do trabalho é a referência exata: a tag `w:pgMar` do `document.xml` traz as margens, e dá para importar os estilos dele em vez de recriá-los.
  O mesmo vale para `<sheetFormatPr>` e `<pageMargins>` do `.xlsx` e para `sldSz` do `.pptx`.
- **Régua vertical:** o menu *Exibir > Régua* liga só a horizontal. A vertical fica em *Ferramentas > Opções > LibreOffice Writer > Exibir > Régua vertical*
  e vem desligada, então as margens de cima e de baixo só aparecem ao ligá-la.
- **Linhas de margem:** os cantos que o Writer desenha na folha são os *limites do texto*. Ficam desligados pela chave `TextBoundaries`
  (*Exibir > Limites do texto* faz o mesmo pela interface).
- **Altura das linhas do Calc:** o LibreOffice a calcula pela fonte e resulta em 0,487 cm, contra 15 pt (0,529 cm) do Excel. Não foi fixada porque
  uma altura fixa desliga o ajuste automático das linhas com quebra de texto.
- **Impress, família de estilos:** os estilos de apresentação (`title`, `outline1` a `outline9`, `subtitle`) ficam numa família com o nome do slide mestre
  (`Padrão` em português).
- **Impress, marcadores:** `replaceByIndex` em `NumberingRules` só aceita a sequência com `uno.invoke(regras, "replaceByIndex", (i, uno.Any("[]com.sun.star.beans.PropertyValue", valores)))`
  e os campos curtos (`NumberingType`, `Adjust`, `StartWith`, `BulletRelSize`, `SymbolTextDistance`) como `uno.Any("short", valor)`.
- **Layouts do PowerPoint:** o Impress tem um só título e um só corpo por slide mestre. O *Slide de título* fica aplicado só ao primeiro slide do modelo.

### Lote 4: multimídia, utilitários e Steam

| Bloco | Programas | Origem |
|---|---|---|
| Multimídia | VLC | Debian |
| Diagnóstico | `htop` (processos), `ncdu` (uso do disco), `tree` (árvore de pastas) e `dnsutils` (`dig` e `nslookup`) | Debian |
| Rede | `nmap` | Debian |
| Jogos | Steam | Flatpak |

O VLC entra pelo Debian, sem repositório externo. O Steam usa o Flatpak porque a base do Flatpak já traz as bibliotecas de 32 bits que os jogos pedem,
sem ativar a arquitetura `i386` no sistema.

```bash
apt-get -s install vlc htop ncdu tree dnsutils nmap        # simulação, sem root: só pacotes novos e nenhuma remoção
sudo apt-get install -y vlc htop ncdu tree dnsutils nmap
sudo flatpak install -y flathub com.valvesoftware.Steam
flatpak override --user --filesystem=/mnt/ssd/Jogos com.valvesoftware.Steam      # a Steam passa a enxergar a pasta de jogos
```

Como conferir:

```bash
vlc --version | head -1
dig -v && nmap --version | head -1
flatpak list --app --columns=application | grep Steam
flatpak override --user --show com.valvesoftware.Steam                           # filesystems=/mnt/ssd/Jogos;
```

Como desfazer: `sudo apt remove vlc htop ncdu tree dnsutils nmap` e `sudo flatpak uninstall com.valvesoftware.Steam`.
O `flatpak override --user --reset com.valvesoftware.Steam` remove a permissão da pasta.

Observações:

- **Biblioteca no SSD extra:** na Steam, em *Configurações > Armazenamento*, adicione `/mnt/ssd/Jogos` como pasta de biblioteca. O disco é `btrfs`
  e fica fora do NVMe do sistema.
- **Vídeo:** a Intel atende o uso comum. Para acionar a AMD num jogo, a opção de inicialização `DRI_PRIME=1 %command%` *(proposta)* segue a mesma lógica
  descrita em "GPU" da fase 4. A placa é de entrada e roda só jogos leves ou antigos.
- **`dnsutils`:** é um pacote de transição. Quem instala o `dig` e o `nslookup` é o `bind9-dnsutils`.
- **`nmap`:** escaneie só equipamentos próprios ou com autorização. Uma varredura na rede corporativa pode disparar alertas da TI.
- **VLC como `root`:** o VLC se recusa a rodar com `sudo`. Confira a versão como usuário comum.

#### Acesso a servidores por SFTP (no lugar do WinSCP)

O aplicativo **Arquivos** abre servidores por SFTP, sem programa extra. As chaves do PuTTY e do WinSCP (`.ppk`) precisam ser convertidas para o formato do OpenSSH,
e o `puttygen`, do pacote `putty-tools`, faz isso.

```bash
sudo apt-get install -y putty-tools
mv ~/Downloads/<chave>.ppk ~/.ssh/chave-trabalho.ppk && chmod 600 ~/.ssh/chave-trabalho.ppk
puttygen ~/.ssh/chave-trabalho.ppk -O private-openssh-new -o ~/.ssh/chave-trabalho -P     # pede a senha atual e a nova
puttygen ~/.ssh/chave-trabalho.ppk -O public-openssh -o ~/.ssh/chave-trabalho.pub         # parte pública, sem senha
chmod 600 ~/.ssh/chave-trabalho
```

Regra padrão para os servidores da rede interna, em `~/.ssh/config` (permissão `600`):

```
Host 10.* 172.* 192.168.*
    IdentityFile ~/.ssh/chave-trabalho
    IdentitiesOnly yes
    AddKeysToAgent yes
    ServerAliveInterval 30
```

Uso:

- **Arquivos:** `Ctrl+L`, digite `sftp://<usuario>@<ip-do-servidor>/` e confirme. Com `Ctrl+D`, o servidor vai para a barra lateral e abre com um clique.
  O aplicativo também guarda os servidores recentes em *Outros locais > Conectar ao servidor*.
- **Terminal:** `ssh <usuario>@<ip-do-servidor>`.

Como conferir:

```bash
ssh -G <ip-do-servidor> | grep -E '^(identityfile|identitiesonly) '     # a chave de trabalho, e só ela
ssh -G github.com | grep '^identityfile '                                # não inclui a chave de trabalho
ssh-keygen -y -P '' -f ~/.ssh/chave-trabalho > /dev/null; echo $?        # diferente de 0: a chave está protegida por senha
```

Como desfazer: apague o bloco `Host` do `~/.ssh/config`, os arquivos `chave-trabalho*` e, se quiser, `sudo apt remove putty-tools`.

Observações:

- **Um usuário por servidor:** o usuário vai no endereço, então cada servidor pode ter o seu (`opc`, `ubuntu` e outros), com a mesma chave.
- **Alcance da regra:** só as faixas internas (`10.*`, `172.*` e `192.168.*`) usam a chave de trabalho. O GitHub segue com a chave própria, e a chave de trabalho não é oferecida a servidores externos.
  Para servidores acessados por nome, acrescente o padrão à linha `Host` (por exemplo, `*.empresa.interno`).
- **Senha da chave:** o GNOME pergunta a senha na primeira conexão e oferece guardá-la no chaveiro. Com `AddKeysToAgent`, a chave fica no agente até o fim da sessão.
- **Primeira conexão a cada servidor:** o programa mostra a impressão digital do servidor. Confirme com quem o administra e aceite. Ela fica em `~/.ssh/known_hosts`.
- **VPN:** os servidores internos só respondem com a VPN conectada.
- **Chaves fora do repositório:** nunca versione `~/.ssh`. Mantenha o `.ppk` original como cópia.
- **Editar direto no servidor:** um arquivo aberto pelo Arquivos, no Editor de Texto, é salvo no próprio servidor. Para projetos, a extensão *Remote - SSH* do VS Code usa a mesma configuração *(proposta)*.

### VPN Fortinet com interface gráfica

Objetivo: ligar e desligar a VPN pelo menu rápido do GNOME, sem usar o terminal.

- [x] Instalar o plugin do NetworkManager para OpenConnect
- [x] Cadastrar a conexão pela tela de Rede, com o protocolo Fortinet
- [x] Conectar e validar

```bash
sudo apt install -y network-manager-openconnect-gnome
```

Cadastro, pela interface:

1. *Configurações > Rede*, botão **+** ao lado de *VPN*, *Multiprotocol VPN Client (openconnect)*.
2. **Protocolo:** Fortinet SSL VPN.
3. **Gateway:** `ENDERECO:PORTA`.
4. Salvar e conectar. Usuário e senha são pedidos na conexão.

Como conferir:

```bash
nmcli connection show --active     # a VPN aparece com o tipo "vpn"
ip route | grep vpn0               # rotas das redes internas pelo túnel
ip route show default              # a internet segue pela rota normal
```

Observações:

- **Plugin:** o Debian 13 não tem plugin do NetworkManager para o `openfortivpn`. O OpenConnect fala o
  protocolo Fortinet e resolve. O `openfortivpn` continua instalado como reserva.
- **Onde ficam os dados:** endereço e credenciais ficam no NetworkManager, fora do repositório.
- **Túnel dividido:** só as redes internas passam pela VPN. A internet segue pela rota normal.
- **DNS:** sem `systemd-resolved`, o NetworkManager grava os DNS da VPN em `/etc/resolv.conf`.
  Se um nome interno não resolver, comece por esse arquivo.
- **Certificado:** aceite o certificado do servidor só se reconhecer o servidor.

### Organização dos aplicativos

| Bloco | Escolha | Origem |
|---|---|---|
| Base de desenvolvimento | `build-essential` | Debian |
| Editor de código | Visual Studio Code, com a extensão do Antigravity | Repositório da Microsoft |
| Bancos de dados | DBeaver, para Oracle, MySQL e Postgres | Repositório do fabricante |
| Linguagens | Node 24 LTS, Java 25 LTS, Python 3.14, Go 1.27 e TypeScript 7 | Gerenciadores de versão na pasta pessoal |
| Contêineres | Docker Engine | Repositório oficial do Docker |
| APIs | Postman e SoapUI | Flatpak |
| Comunicação | WhatsApp Web, Teams, Meet e Zoom pelo Chrome. Discord | Navegador e Flatpak |
| Rede e acesso remoto | Wireshark, Meld, AnyDesk, `nmap` e `dnsutils` | Debian e repositório do fabricante |
| Diagnóstico | `htop`, `ncdu` e `tree` | Debian |
| Multimídia | VLC | Debian |
| Jogos | Steam, com a biblioteca em `/mnt/ssd/Jogos` | Flatpak |
| Escritório | LibreOffice, configurado para se parecer com o Office | Debian |
| Captura de tela | Recurso nativo do GNOME | GNOME |

O Debian 13 traz versões antigas de algumas linguagens (o Node 20 e o Go 1.24, por exemplo, deixam de receber suporte do projeto de origem), por isso o lote 2 usa
gerenciadores de versão na pasta pessoal: `fnm` para Node, SDKMAN para Java, `uv` para Python e o pacote oficial do Go.
O Python do sistema fica intocado.

### Pontos em aberto

1. Há política de TI que exija antivírus ou software específico?
2. Será preciso rodar Windows em máquina virtual para algum sistema legado?
3. Dados pessoais e de trabalho ficarão no mesmo perfil de usuário?

---

## Fase 6 - GNOME (em andamento)

Objetivo: deixar o GNOME 48 confortável para quem vem do Windows, com poucas extensões e sem trocar o visual padrão.

- [x] **Extensões:** AppIndicator (ícones de bandeja), Dash to Dock (barra de aplicativos) e Caffeine (impede a suspensão), pelos pacotes do Debian.
- [x] **Janelas:** botões de minimizar e maximizar ao lado do fechar.
- [x] **Relógio:** bateria em porcentagem e dia da semana.
- [x] **Atalhos:** `Super+E` abre o Arquivos, `Super+D` mostra a área de trabalho e `Ctrl+Alt+T` abre o terminal.
- [x] **Arquivos:** visualização em lista. Nas janelas de abrir e salvar, pastas antes dos arquivos.
- [x] **Visual:** mantido o tema escuro, o destaque verde e a fonte Cantarell.
- [ ] Conferir as extensões carregadas, os ícones da bandeja e a barra de aplicativos (exige uma sessão nova).
- [ ] Favoritos da barra de aplicativos.

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

# 5. Arquivos
gsettings set org.gnome.nautilus.preferences default-folder-viewer 'list-view'
gsettings set org.gtk.Settings.FileChooser sort-directories-first true
gsettings set org.gtk.gtk4.Settings.FileChooser sort-directories-first true
```

Como conferir:

```bash
gsettings get org.gnome.shell enabled-extensions
gnome-extensions list --enabled                    # depois de uma sessão nova
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

---

## Fases 7 e 8 (planejadas)

| Fase | Tema | Escopo |
|---|---|---|
| 7 | Backup | Snapshot do sistema (Timeshift), feito ao terminar a configuração base |
| 8 | Automação | Transformar o que foi validado em `scripts/` |
