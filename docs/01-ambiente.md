# 01 - Ambiente

Descrição do sistema e do notebook usados como referência neste guia.

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

**Debian *stable*:** os pacotes são testados por muito tempo e quase não mudam durante a vida
da versão, exceto correções de segurança. Os programas não são os mais novos, mas o sistema
é estável. Para uma máquina de trabalho, é uma boa troca.

**trixie:** nome-código da versão 13. Aparece nos arquivos de repositório do APT.

**13.7:** *point release*, um conjunto acumulado de correções dentro da mesma versão.

**Wayland:** sistema de exibição moderno, padrão no GNOME. Alguns programas antigos, como
compartilhamento de tela e acesso remoto, podem se comportar de forma diferente do X11.

**Repositórios APT** (`/etc/apt/sources.list`): `trixie`, `trixie-updates` e `trixie-security`,
com os componentes `main non-free-firmware`. Ainda sem `contrib` e `non-free`.

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
| SSD NVMe de 238 GB | Sistema: EFI (976 MB, vfat), raiz `/` (225 GB, ext4) e swap (12 GB) |
| SSD SATA de 240 GB | Armazenamento extra (jogos, programas e arquivos). btrfs, montado em `/mnt/ssd` pelo `fstab` |

`/home` fica dentro da raiz, sem partição separada.

## Energia, firmware e drivers

- **Energia:** `power-profiles-daemon` ativo (é o que o menu do GNOME usa). `TLP` e `thermald` não são usados (o `thermald` não roda em ThinkPads com controle térmico pelo firmware).
- **Firmware presente:** `firmware-amd-graphics`, `firmware-intel-graphics`, `firmware-iwlwifi`, `firmware-realtek`, `firmware-sof-signed`.
- **Microcódigo:** `intel-microcode`.
- **Atualização de firmware:** `fwupd` disponível.

## Software base

| Item | Estado |
|---|---|
| GNOME (`gnome-core`), Ajustes (`gnome-tweaks`), NetworkManager | instalados |
| Firefox ESR, LibreOffice Writer | instalados |
| `curl`, `wget` | instalados |
| Google Chrome | instalado, com o repositório oficial do Google (atualiza pelo `apt`) |
| `openfortivpn` | instalado (cliente de VPN Fortinet por linha de comando, usado como reserva) |
| `network-manager-openconnect-gnome` | instalado (VPN Fortinet pela interface gráfica) |
| Claude Code | instalado em `~/.local/bin` |
| `git`, `gh` | instalados |
| `btrfs-progs`, `smartmontools` | instalados |
| `mesa-utils`, `vulkan-tools` | instalados (testes de vídeo) |
| `unattended-upgrades`, `powermgmt-base` | instalados, com atualizações automáticas ativas (Debian e Google Chrome) |
| `timeshift` | ausente |
| Flatpak, extensões do GNOME | ausentes |

## Decisões a tomar

1. **BIOS.** É da época do lançamento (2020). O notebook funciona bem, então a atualização é opcional.
2. **Repositórios `contrib` e `non-free`.** Necessários para alguns pacotes (fontes, drivers, codecs).
3. **Secure Boot.** Desativado. O Debian suporta Secure Boot, então dá para ativar se necessário.
