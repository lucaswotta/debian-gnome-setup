# 01 - Ambiente

Sistema e notebook usados como referência neste guia. Se o seu hardware ou a sua versão forem diferentes, adapte os passos das fases com escopo **Modelo de referência** (ver o [roteiro](02-roteiro.md)).

## Sistema

| Item | Valor |
|---|---|
| Distribuição | Debian GNU/Linux 13 "trixie" (13.7) |
| Ramo | *stable* |
| Kernel | Linux 6.12 LTS |
| Arquitetura | amd64 |
| Ambiente gráfico | GNOME 48 |
| Servidor gráfico | Wayland |
| Idioma e fuso | pt_BR.UTF-8, America/Sao_Paulo (hora sincronizada por NTP) |
| Teclado | ABNT2 |

- **Debian *stable*:** os pacotes são testados por muito tempo e quase não mudam durante a vida da versão, exceto por correções de segurança. Os programas não são os mais novos, mas o sistema é estável, o que favorece uma máquina de trabalho.
- **trixie:** nome-código da versão 13. Aparece nos arquivos de repositório do APT.
- **13.7:** *point release*, um conjunto acumulado de correções dentro da mesma versão.
- **Wayland:** sistema de exibição moderno, padrão no GNOME. Alguns programas antigos, como compartilhamento de tela e acesso remoto, podem se comportar de forma diferente do X11.
- **Repositórios APT** (`/etc/apt/sources.list.d/`, formato `.sources`): `trixie`, `trixie-updates` e `trixie-security`, com os componentes `main contrib non-free-firmware`. O `non-free` fica desativado. Há também os repositórios do Google Chrome, do Docker e dos fabricantes do VS Code, do AnyDesk e do DBeaver.

## Notebook

| Item | Valor |
|---|---|
| Modelo | Lenovo ThinkPad E14 Gen 1, família 20RB |
| Processador | Intel Core i7-10510U (4 núcleos, 8 threads) |
| Memória | 16 GB |
| Tela | 14", 1920x1080 |
| Vídeo | Intel UHD Graphics (`i915`) e AMD Radeon 530 (`amdgpu`), esta última sob demanda |
| Wi-Fi e Bluetooth | Intel AX201 |
| Ethernet | Realtek RTL8111/8168 Gigabit |
| Outros | Webcam integrada e leitor de digital Goodix (sem suporte no Linux) |
| Boot | UEFI, Secure Boot desativado |
| Bateria | Li-poly, 45,7 Wh (capacidade de projeto) |

## Armazenamento

| Disco | Uso |
|---|---|
| SSD NVMe de 238 GB | Sistema: EFI (976 MB, `vfat`), raiz `/` (225 GB, `ext4`) e swap (12 GB) |
| SSD SATA de 240 GB | Armazenamento extra: jogos, arquivos, discos de máquinas virtuais e os snapshots do backup. `btrfs`, montado em `/mnt/ssd` pelo `fstab` |

`/home` fica dentro da raiz, sem partição separada.

## Energia, firmware e drivers

- **Energia:** `power-profiles-daemon` (perfil `balanced`), usado pelo menu do GNOME. O `TLP` não é usado, porque conflita com ele. O `thermald` não roda em ThinkPads com controle térmico pelo firmware.
- **Suspensão:** a tela apaga após 30 minutos de inatividade e o sistema suspende após 60 minutos. Fechar a tampa suspende na hora.
- **Bateria:** sem limite de carga. O recurso é opcional ([fase 4](fases/04-notebook.md)).
- **GPU:** a Intel atende o uso comum. A AMD fica suspensa e é acionada sob demanda.
- **Firmware presente:** `firmware-amd-graphics`, `firmware-intel-graphics`, `firmware-iwlwifi`, `firmware-realtek` e `firmware-sof-signed`.
- **Microcódigo:** `intel-microcode`.
- **Atualização de firmware:** `fwupd` disponível.
- **Boot:** o GRUB não mostra o menu e inicia o sistema direto.

## Software instalado

### Base do sistema

| Item | Estado | Fase |
|---|---|---|
| GNOME (`gnome-core`), Ajustes (`gnome-tweaks`) e NetworkManager | Instalados | 0 |
| Extensões do GNOME | AppIndicator, Dash to Dock, Caffeine, GPaste, Tiling Assistant e Blur my Shell, pelo Debian, e o Gerenciador de extensões para instalar outras | [6](fases/06-gnome.md) |
| Fonte e cursor da interface | Inter e Bibata Modern Classic, pelo Debian | [6](fases/06-gnome.md) |
| Ícones | Papirus, pelo Debian, com uma variante de pastas verdes na pasta do usuário | [6](fases/06-gnome.md) |
| Terminal e shell | zsh (aberto pelo perfil do `gnome-terminal`), Starship com prompt em blocos Powerline, `eza`, `bat`, `fzf` e `fastfetch`, pelo Debian, e a FiraCode Nerd Font na pasta do usuário. Configuração em `dotfiles/` | [8](fases/08-terminal.md) |
| `curl`, `wget`, `git` e `gh` | Instalados | [0](fases/00-base-minima.md) e [1](fases/01-git-e-github.md) |
| `btrfs-progs` e `smartmontools` | Instalados | [2](fases/02-base-do-sistema.md) |
| `mesa-utils` e `vulkan-tools` | Instalados, para testar o vídeo | [4](fases/04-notebook.md) |
| `unattended-upgrades` e `powermgmt-base` | Atualizações automáticas ativas para o Debian, Chrome, VS Code, AnyDesk e DBeaver | [3](fases/03-atualizacoes.md) |
| `timeshift` | Modo `rsync`, destino no SSD extra, agenda mensal | [7](fases/07-backup.md) |
| `flatpak` | Instalado, com o Flathub | [5](fases/05-aplicativos.md) |
| Fontes | Liberation, Carlito, Caladea, Noto, Fira Code e as fontes da Microsoft (`ttf-mscorefonts-installer`) | [5](fases/05-aplicativos.md) |
| Fonte Aptos | Indisponível para Linux. Uma regra do `fontconfig` a substitui pela Liberation Sans | [5](fases/05a-libreoffice.md) |

### Desenvolvimento

| Item | Estado | Fase |
|---|---|---|
| `build-essential` e Meld | Instalados | [5](fases/05-aplicativos.md) |
| Docker Engine, Compose e Buildx | Repositório oficial, com a rede fora das faixas da VPN. O serviço inicia sob demanda, pelo `docker.socket` | [5](fases/05-aplicativos.md) |
| Go 1.27, Node 24 LTS, TypeScript 7, Python 3.14 e Java 25 LTS | Na pasta pessoal, com `fnm`, `uv` e SDKMAN. O Python do sistema fica intocado | [5](fases/05-aplicativos.md) |
| Visual Studio Code e DBeaver | Repositórios dos fabricantes, com atualização pelo `apt` | [5](fases/05-aplicativos.md) |
| Postman e SoapUI | Flatpak | [5](fases/05-aplicativos.md) |
| QEMU/KVM, libvirt e virt-manager | Pacotes do Debian, sem as recomendações. Serviços sob demanda, pelos sockets. Rede `default` em `100.66.0.0/24`, fora das faixas da VPN. Discos em `/mnt/ssd/VMs` | [9](fases/09-windows-vm.md) |
| Windows 10 Enterprise LTSC 2019 (VM) | Em português, com UEFI, 4 GB, 4 vCPUs, drivers `virtio`, agentes SPICE e QEMU e Edge | [9](fases/09-windows-vm.md) |
| Claude Code | Em `~/.local/bin` | [0](fases/00-base-minima.md) |

### Rede e acesso remoto

| Item | Estado | Fase |
|---|---|---|
| Google Chrome | Repositório oficial do Google, atualizado pelo `apt` | [5](fases/05-aplicativos.md) |
| `network-manager-openconnect-gnome` | VPN Fortinet pela interface gráfica | [5](fases/05b-vpn-e-servidores.md) |
| `openfortivpn` | VPN Fortinet por linha de comando, como reserva | [5](fases/05b-vpn-e-servidores.md) |
| `putty-tools` | Converte chaves `.ppk` para o formato do OpenSSH | [5](fases/05b-vpn-e-servidores.md) |
| `openssh-server` | Instalado e desativado, para ligar sob demanda | [5](fases/05-aplicativos.md) |
| AnyDesk | Repositório do fabricante, sem serviço ativo: abre sob demanda pelo aplicativo | [5](fases/05-aplicativos.md) |
| Wireshark, `nmap` e `dnsutils` | Instalados | [5](fases/05-aplicativos.md) |

### Escritório, mídia e utilitários

| Item | Estado | Fase |
|---|---|---|
| Firefox ESR | Instalado com o sistema | 0 |
| LibreOffice (Writer, Calc, Impress e Draw) | Faixa em abas, ícones Colibre, folha branca no modo escuro, pt-BR, modelos no estilo do Microsoft 365, gravação em `.docx`, `.xlsx` e `.pptx` | [5](fases/05a-libreoffice.md) |
| VLC | Instalado pelo Debian | [5](fases/05-aplicativos.md) |
| Discord | Flatpak | [5](fases/05-aplicativos.md) |
| Steam | Flatpak, com acesso à pasta `/mnt/ssd/Jogos` | [5](fases/05-aplicativos.md) |
| `htop`, `ncdu` e `tree` | Instalados | [5](fases/05-aplicativos.md) |
