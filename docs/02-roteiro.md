# 02 - Roteiro

Legenda: `[x]` feito · `[ ]` pendente · `[~]` em discussão.

Regras:

- Uma fase por vez, explicada antes de executar.
- Comando com `sudo` só roda depois de confirmação.
- Só vira script (`scripts/`) o que já foi feito à mão e entendido.
- Cada fase termina com uma verificação e um registro neste arquivo.

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

Aprendizados:

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

Aprendizados:

- **Repositório público:** o e-mail de cada commit fica visível. Use o endereço `noreply` do GitHub
  (`ID+usuario@users.noreply.github.com`). O ID sai de `gh api user --jq .id`.
- **Chave SSH:** proteja com senha (passphrase).
- **Trabalho direto na `main`:** neste repositório de uso individual não há branches nem PRs.

---

## Fase 2 - Base do sistema (em andamento)

- [x] **TRIM do SSD:** já vem ativo, com o `fstrim.timer` rodando toda semana.
- [x] **Catálogo de firmware:** atualizado com `fwupd`. Nenhum componente tem atualização no LVFS.
- [ ] **BIOS:** comparar a versão instalada com a mais recente no site de suporte da Lenovo.
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

Aprendizados:

- **Firmware:** o LVFS não tem todos os modelos. "Sem atualização" no `fwupd` não garante que a BIOS seja a última.
- **Atualização de BIOS:** só com o notebook na tomada e sem desligar durante o processo.
- **Escada:** um repositório novo só entra quando há um pacote que precise dele.

---

## VPN Fortinet com interface gráfica (adiantada da fase 6)

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

Aprendizados:

- **Plugin:** o Debian 13 não tem plugin do NetworkManager para o `openfortivpn`. O OpenConnect fala o
  protocolo Fortinet e resolve. O `openfortivpn` continua instalado como reserva.
- **Onde ficam os dados:** endereço e credenciais ficam no NetworkManager, fora do repositório.
- **Túnel dividido:** só as redes internas passam pela VPN. A internet segue pela rota normal.
- **DNS:** sem `systemd-resolved`, o NetworkManager grava os DNS da VPN em `/etc/resolv.conf`.
  Se um nome interno não resolver, comece por esse arquivo.
- **Certificado:** aceite o certificado do servidor só se reconhecer o servidor.

---

## Fases seguintes (proposta)

| Fase | Tema | Itens |
|---|---|---|
| 2 | Base do sistema | Em andamento (ver acima) |
| 3 | Segurança | Firewall (`ufw`), atualizações automáticas de segurança, criptografia, Secure Boot |
| 4 | Backup | Snapshots do sistema (Timeshift) e backup dos dados |
| 5 | Notebook | Bateria e temperatura, GPU AMD, leitor de digital, teclas Fn |
| 6 | Aplicativos | Flatpak e Flathub, comunicação, ferramentas de trabalho (navegador e VPN já feitos) |
| 7 | GNOME | Extensões, atalhos, gestos do touchpad, tema e fontes |
| 8 | Automação | Transformar o que foi validado em `scripts/` |

### Fase 2: disco SATA extra (concluída)

Decisões: manter o btrfs (o disco veio vazio, então nada foi apagado) e montar em `/mnt/ssd`.

- [x] Conferir o conteúdo. O disco estava vazio.
- [x] Conferir a saúde (SMART). Aprovado.
- [x] Montar de forma permanente pelo `/etc/fstab`, identificando o disco por **UUID**.
- [x] Criar a estrutura de pastas e ajustar dono e permissões.
- [ ] Apontar para ele o que ocupa espaço, como a biblioteca de jogos (fase 6).
- [ ] Incluí-lo no backup (fase 4). Um disco só não é backup.

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

Aprendizados:

- **Automontagem do GNOME:** monta o disco em `/media/USUARIO/UUID`, de forma temporária e com a raiz do disco
  pertencendo ao `root`. Para uso contínuo, configure o `fstab`.
- **UUID no `fstab`:** o nome `sda` pode mudar. O UUID não muda. Não publique o UUID no repositório.
- **`nofail`:** sem ele, a falta do disco pode travar a inicialização.
- **Saúde do SSD:** olhe o resultado geral (`PASSED`), as horas ligado, a vida útil restante e os contadores de
  setores realocados e erros de CRC. Acompanhe também o contador de desligamentos abruptos.
- **`findmnt --verify` sem `sudo`:** mostra avisos de permissão negada. Não são erros do `fstab`.
- **Programas:** o APT instala em `/usr` e não permite escolher outro disco. Neste disco vão jogos, arquivos,
  Flatpaks e ferramentas portáteis. Os pacotes do sistema ficam no NVMe.

---

## Decisões em aberto

1. Há política de TI que exija criptografia de disco, antivírus, VPN ou software específico?
2. Quais ferramentas de trabalho são necessárias (banco de dados, modelagem, Office, videoconferência, acesso remoto)?
   Alguma só existe para Windows?
3. Será preciso rodar Windows em máquina virtual para algum sistema legado?
4. Dados pessoais e de trabalho ficarão no mesmo perfil de usuário?
