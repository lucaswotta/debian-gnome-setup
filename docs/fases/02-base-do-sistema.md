# Fase 2 - Base do sistema

**Status:** concluída · **Escopo:** geral, com uma seção específica do disco extra

[← Fase 1](01-git-e-github.md) · [Roteiro](../02-roteiro.md) · [Fase 3 →](03-atualizacoes.md)

Objetivo: deixar o armazenamento e o firmware prontos para o uso diário.

- [x] **TRIM do SSD:** já vem ativo, com o `fstrim.timer` rodando toda semana.
- [x] **Catálogo de firmware:** atualizado com `fwupd`. `fwupdmgr get-updates` lista as atualizações disponíveis.
- [x] **BIOS:** a versão de fábrica funciona bem, e a atualização é opcional.
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

## Disco SATA extra

O modelo de referência tem um segundo disco, SATA, já formatado em `btrfs`. As decisões foram manter o sistema de arquivos, sem formatar, e montá-lo em `/mnt/ssd`.
Adapte o dispositivo (`/dev/sda1` abaixo) ao seu disco, conferindo com `lsblk` antes de qualquer comando.

- [x] Conferir o conteúdo antes de reaproveitar o disco.
- [x] Conferir a saúde (SMART).
- [x] Montar de forma permanente pelo `/etc/fstab`, identificando o disco por **UUID**.
- [x] Criar a estrutura de pastas e ajustar dono e permissões.
- [x] Confirmar que o disco monta sozinho depois de reiniciar.
- [x] **Uso:** guarda a biblioteca de jogos, arquivos e os snapshots do backup ([fase 7](07-backup.md)).
- Um disco só não é backup. As limitações estão descritas na fase 7.

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
- **Contadores do SMART:** `Unsafe_Shutdown_Count` e `Power_Cycle_Count` não mudam num reinício limpo, porque o SSD continua alimentado. Só sobem quando o notebook desliga por completo.
- **Programas:** o APT instala em `/usr` e não permite escolher outro disco. Os pacotes do sistema e os Flatpaks ficam no disco do sistema. Neste disco vão jogos, arquivos e os snapshots.
