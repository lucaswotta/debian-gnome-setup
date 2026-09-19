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
| Vídeo | Intel UHD Graphics (`i915`) e AMD Radeon série R7 M / 500 (`amdgpu`) |
| Wi-Fi e Bluetooth | Intel AX201 |
| Ethernet | Realtek RTL8111/8168 Gigabit |
| Outros | Webcam integrada e leitor de digital Goodix |
| Boot | UEFI, Secure Boot desativado |
| Bateria | Li-poly, 45,7 Wh (capacidade de projeto) |

## Armazenamento

| Disco | Uso |
|---|---|
| SSD NVMe de 238 GB | Sistema: EFI (976 MB, vfat), raiz `/` (225 GB, ext4, sem criptografia) e swap (12 GB) |
| SSD SATA de 240 GB | Armazenamento extra (programas, jogos e arquivos). Vem em btrfs e ainda não está montado |

`/home` fica dentro da raiz, sem partição separada.

## Energia, firmware e drivers

- **Energia:** `power-profiles-daemon` ativo (é o que o menu do GNOME usa). `TLP` e `thermald` não estão configurados.
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
| `openfortivpn` | instalado (cliente de VPN Fortinet, por linha de comando) |
| Claude Code | instalado em `~/.local/bin` |
| `git`, `gh` | instalados |
| `ufw`, `timeshift` | ausentes |
| Flatpak, extensões do GNOME | ausentes |

## Decisões a tomar

1. **Criptografia de disco (LUKS).** Em notebook, protege os dados em caso de perda ou furto.
   Ativar depois da instalação exige reinstalar. Decidir antes de guardar dados importantes.
2. **BIOS.** A do ThinkPad E14 Gen 1 é da época do lançamento (2020). Verificar atualizações
   pelo `fwupd` ou pelo site de suporte da Lenovo.
3. **Disco SATA extra.** Vem formatado em btrfs. Antes de montar, conferir o conteúdo em modo
   somente leitura. Só reformatar com certeza de que não há nada a preservar.
4. **Repositórios `contrib` e `non-free`.** Necessários para alguns pacotes (fontes, drivers, codecs).
5. **GPU AMD.** Conferir qual GPU o sistema usa e o efeito na bateria e na temperatura.
6. **Leitor de digital.** Confirmar se é suportado pelo `fprintd` antes de depender dele.
7. **Secure Boot.** Desativado. O Debian suporta Secure Boot, então dá para ativar se necessário.
