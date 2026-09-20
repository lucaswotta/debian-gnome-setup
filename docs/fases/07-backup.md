# Fase 7 - Backup

**Status:** concluída · **Escopo:** geral. Adapte o disco de destino e as pastas excluídas

[← Fase 6](06-gnome.md) · [Roteiro](../02-roteiro.md)

Objetivo: poder voltar o sistema a um estado conhecido depois de uma atualização ou de uma configuração que quebre algo.

- [x] **Ferramenta:** Timeshift, em modo `rsync`, porque a raiz é `ext4`.
- [x] **Destino:** o SSD extra (`/mnt/ssd`, `btrfs`), que é outro disco do notebook.
- [x] **Conteúdo:** o sistema e as configurações da pasta pessoal (arquivos ocultos), sem cache.
- [x] **Agenda:** mensal, com as 2 últimas cópias.
- [x] **Snapshot manual:** "Configuracao base", criado depois de concluída a configuração do sistema.

```bash
# 1. Instalar (o rsync vem como dependência)
sudo apt-get install -y timeshift

# 2. Configuração: /etc/timeshift/timeshift.json, criada a partir do modelo /etc/timeshift/default.json
#    "backup_device_uuid": UUID do disco de destino (findmnt -no UUID /mnt/ssd)
#    "btrfs_mode": "false", "schedule_monthly": "true", "count_monthly": "2", "do_first_run": "false"
#    "exclude": as regras da lista abaixo, nesta ordem, com o nome do usuário no lugar de <usuario>

# 3. Snapshot manual (a etiqueta padrão é O, que a agenda não apaga)
sudo timeshift --rsync --create --comments "Configuracao base" --scripted
```

Regras de `exclude`, na ordem em que valem (o `rsync` usa a primeira que casa):

```
/home/<usuario>/.cache/**
/home/<usuario>/.config/google-chrome/OptGuideOnDeviceModel/**
/home/<usuario>/.var/app/com.valvesoftware.Steam/**
/home/<usuario>/.local/share/Trash/**
/var/lib/docker/**
/var/lib/containerd/**
+ /home/<usuario>/.**
/home/<usuario>/**
/root/**
```

Antes de usar as regras, teste-as como usuário comum, sem gravar nada:

```bash
R=(--exclude='/<usuario>/.cache/**' --include='/<usuario>/.**' --exclude='/<usuario>/**')       # caminhos relativos a /home
rsync -an --info=stats2 "${R[@]}" /home/ "$(mktemp -d)/" | grep -E 'files transferred|Total transferred'
```

Como conferir:

```bash
sudo timeshift --list                       # snapshots e o dispositivo
cat /etc/cron.d/timeshift-hourly            # 0 * * * * root timeshift --check --scripted
sudo du -sh /mnt/ssd/timeshift              # espaço ocupado
```

Como desfazer:

```bash
sudo rm /etc/cron.d/timeshift-hourly /etc/timeshift/timeshift.json     # desliga a agenda e a configuração
sudo timeshift --delete-all                                            # apaga todos os snapshots
sudo apt remove --autoremove timeshift
```

Observações:

- **Alcance:** o snapshot fica no mesmo notebook. Protege contra atualização ou configuração que quebrem o sistema e contra falha do disco do sistema, mas não contra perda,
  roubo ou falha total do equipamento. Uma cópia em disco externo é decisão à parte.
- **Agenda:** ao ler uma configuração com agenda ligada, o Timeshift cria `/etc/cron.d/timeshift-hourly`. O `cron` confere de hora em hora e cria o snapshot mensal quando ele vence.
  Só roda com o notebook ligado, e a checagem seguinte recupera o que ficou para trás.
- **Snapshot manual:** a etiqueta `O` não entra na rotação. Ele fica até ser removido com `sudo timeshift --delete --snapshot '<nome>'`. Não passe `--tags O`, porque essa versão
  recusa o valor, e `O` já é o padrão.
- **Espaço:** o `du` mostra o tamanho aparente da pasta `timeshift`, e o `df` mostra o espaço realmente ocupado, menor por causa da compressão do `btrfs`. Só o primeiro snapshot é completo. Os seguintes guardam apenas o que mudou, porque os arquivos iguais são compartilhados entre as cópias (*hard links*).
- **O que fica de fora:** os dados do Docker (recriáveis e inconsistentes com o serviço rodando), o cache, a lixeira, os dados da Steam e o modelo de IA local do Chrome
  (`OptGuideOnDeviceModel`, que o Chrome baixa de novo). Os Flatpaks do sistema (`/var/lib/flatpak`) entram.
- **Chaves na cópia:** a pasta `.ssh` faz parte das configurações da pasta pessoal. O snapshot só é legível pelo `root`.
- **Restaurar:** `sudo timeshift --restore` escolhe o snapshot e o destino de forma interativa e exige reiniciar *(proposta, ainda não executada)*.
  Como a pasta pessoal entra só com os arquivos ocultos, a restauração também volta as configurações a esse estado.
