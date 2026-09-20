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
| 3 | Atualizações | Concluída, com uma confirmação pendente |
| 4 | Notebook | Concluída, com uma confirmação pendente |
| 5 | Aplicativos, com VPN | Em andamento |
| 6 | GNOME | Planejada |
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
| `git init` e `git remote add origin` | Iniciam o repositório local e apontam para o remoto. Usados no lugar de `git clone` porque a pasta já tinha conteúdo. |

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
- [x] **Catálogo de firmware:** atualizado com `fwupd`. Nenhum componente tem atualização no LVFS.
- [x] **BIOS:** é da época do lançamento (2020) e o notebook funciona muito bem com ela.
  A atualização é opcional e não é necessária agora.
- [~] **`contrib` e `non-free`:** adiados. Só entram quando um pacote exigir (fontes da Microsoft, alguns codecs).
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

Decisões: manter o btrfs (o disco veio vazio, então nada foi apagado) e montar em `/mnt/ssd`.

- [x] Conferir o conteúdo. O disco estava vazio.
- [x] Conferir a saúde (SMART). Aprovado.
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
- [ ] Confirmar a primeira execução automática, pelo log em `/var/log/unattended-upgrades/`

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
ls /var/log/unattended-upgrades/                                # histórico das execuções
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
- **Hábito semanal:** o `sudo apt update && sudo apt full-upgrade` cobre o que o automático não pega.

---

## Fase 4 - Notebook (concluída)

Objetivo: ajustar energia, vídeo, suspensão e bateria do ThinkPad E14 Gen 1.

- [x] **Energia e temperatura:** o `power-profiles-daemon` (perfil `balanced`) basta. Temperaturas de 38 a 50 °C, ventoinha desligada.
- [x] **GPU AMD sob demanda:** testada com OpenGL e Vulkan. Dorme sozinha depois do uso.
- [x] **Suspensão:** tela apaga em 30 min, suspende em 60 min. Fechar a tampa suspende na hora.
- [x] **Limite de carga da bateria:** ativado entre 75% e 80%.
- [x] **Teclas Fn:** funcionam.
- [x] **Leitor de digital:** sem suporte no Linux (ver observações).
- [ ] Confirmar o limite de carga na prática: descarregar abaixo de 75%, carregar e observar onde a carga para.

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

# Limite de carga da bateria
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
| `EnableChargeThreshold` | Liga o limite de carga do UPower, que vale de 75% a 80%. |
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
- **Leitor de digital:** o Goodix `27c6:55a4` está na seção "Known unsupported devices" da libfprint, e o `fprintd-list`
  responde `No devices available`. Não há o que configurar.
- **Chaveiro no login:** os avisos `gkr-pam: unable to locate daemon control file` e `Failed to start ...keyring...scope`
  aparecem em todo login e são inofensivos. O serviço já sobe pelo systemd e o chaveiro funciona (o `gh` lê o token dele).
- **Logs:** sem estar no grupo `systemd-journal`, o `journalctl` mostra "sem entradas". Isso **não** significa "sem erros".
- **Avisos comuns no boot que não indicam problema:** erro ACPI da GPU (`ATRM`, o `amdgpu` funciona normalmente),
  uma regra `udev` do ALSA sem rótulo, e o erro do `iwlwifi` em cada suspensão. O Wi-Fi recarrega o firmware ao retomar
  e reconecta sozinho.

---

## Fase 5 - Aplicativos (em andamento)

Objetivo: instalar apenas o que será usado, nesta ordem de preferência: pacote do Debian, recurso nativo do GNOME, Flatpak.

- [x] **Navegador:** Google Chrome, instalado pelo pacote `.deb` oficial. O pacote configura o repositório do Google, e as atualizações chegam pelo `apt`.
- [x] **VPN corporativa:** ver abaixo.
- [x] **Perfil de uso:** desenvolvimento como foco, com uso geral.
- [ ] Listar as ferramentas de trabalho e o equivalente de cada uma no Linux
- [ ] Configurar o Flatpak e o Flathub
- [ ] Instalar os aplicativos definidos
- [ ] Apontar a biblioteca de jogos para o SSD extra, se for usada

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

### Organização

| Bloco | Conteúdo | Origem preferida |
|---|---|---|
| Base de desenvolvimento | Compilador, utilitários de terminal, Python | Debian |
| Editor de código ou IDE | A definir | Fora do Debian (Flatpak ou repositório do fabricante) |
| Bancos de dados | Cliente gráfico e servidores de desenvolvimento | Cliente: Flatpak ou fabricante. Servidores: contêineres ou Debian |
| Linguagens e runtimes | Conforme os projetos | Debian. Gerenciador de versões quando a versão do Debian não servir |
| Contêineres e máquinas virtuais | Podman ou Docker. QEMU/KVM para sistemas legados | Debian |
| Comunicação | Videoconferência e mensageria | Navegador ou Flatpak |
| Escritório | LibreOffice (já instalado) e leitor de PDF | Debian |

O Flatpak e o Flathub entram porque alguns aplicativos não estão no repositório do Debian.
O processador tem suporte a virtualização, o que permite máquinas virtuais se necessário.

Fora do repositório oficial do Debian 13: Visual Studio Code, VSCodium, DBeaver e o SDK do .NET 8.
Disponíveis no repositório: Python, Java (OpenJDK), Node.js, Go, Rust, PHP, Ruby, Maven, PostgreSQL, MariaDB,
SQLite, Docker e Podman.

### Pontos em aberto

1. Quais ferramentas de trabalho são necessárias (bancos de dados, editor de código, modelagem, videoconferência, acesso remoto)? Alguma só existe para Windows?
2. Há política de TI que exija antivírus ou software específico?
3. Será preciso rodar Windows em máquina virtual para algum sistema legado?
4. Dados pessoais e de trabalho ficarão no mesmo perfil de usuário?
5. Onde instalar os aplicativos Flatpak: no disco do sistema (padrão) ou no SSD extra?

---

## Fases 6 a 8 (planejadas)

| Fase | Tema | Escopo |
|---|---|---|
| 6 | GNOME | Extensões, atalhos, gestos do touchpad, tema e fontes |
| 7 | Backup | Snapshot do sistema (Timeshift), feito ao terminar a configuração base |
| 8 | Automação | Transformar o que foi validado em `scripts/` |
