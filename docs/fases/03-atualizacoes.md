# Fase 3 - Atualizações

**Status:** concluída · **Escopo:** geral

[← Fase 2](02-base-do-sistema.md) · [Roteiro](../02-roteiro.md) · [Fase 4 →](04-notebook.md)

Objetivo: manter o sistema atualizado com pouco esforço.

- [x] Instalar o `unattended-upgrades` (atualizações automáticas)
- [x] Ativar a execução diária
- [x] Incluir o Google Chrome nas atualizações automáticas
- [x] Validar com uma simulação
- [x] Confirmar a execução automática pelo log em `/var/log/unattended-upgrades/`
- [x] Atualizar os aplicativos Flatpak todo dia, por um timer do systemd (ver [Flatpak](#flatpak))

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

## Flatpak

O `unattended-upgrades` cuida só dos pacotes do APT. Os aplicativos Flatpak são atualizados pelo GNOME Software, que precisa ficar aberto em segundo plano. Sem ele no login (ver [fase 5](05-aplicativos.md#processos-em-segundo-plano)), um timer do systemd faz a atualização uma vez por dia.

```bash
printf '%s\n' '[Unit]' 'Description=Atualiza os aplicativos Flatpak do sistema' 'Wants=network-online.target' \
  'After=network-online.target' '' '[Service]' 'Type=oneshot' \
  'ExecStart=/usr/bin/flatpak update --system --noninteractive --assumeyes' 'Nice=10' 'IOSchedulingClass=idle' \
  > flatpak-update.service
printf '%s\n' '[Unit]' 'Description=Atualizacao diaria dos aplicativos Flatpak' '' '[Timer]' 'OnCalendar=daily' \
  'RandomizedDelaySec=1h' 'Persistent=true' '' '[Install]' 'WantedBy=timers.target' > flatpak-update.timer
systemd-analyze verify ./flatpak-update.service ./flatpak-update.timer
sudo install -m 644 flatpak-update.service flatpak-update.timer /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now flatpak-update.timer
```

| Parte | O que faz |
|---|---|
| `--system --noninteractive --assumeyes` | Atualiza a instalação do sistema, onde ficam os Flatpaks deste guia, sem perguntas. |
| `Nice` e `IOSchedulingClass=idle` | A atualização cede processador e disco para o que estiver em uso. |
| `RandomizedDelaySec=1h` | Sorteia um atraso de até 1 hora, para não pesar sempre no mesmo minuto. |
| `Persistent=true` | Se o notebook estiver desligado no horário, a execução acontece no próximo boot. |

Como conferir:

```bash
systemctl list-timers flatpak-update.timer          # próxima execução
sudo systemctl start flatpak-update.service         # roda agora, para testar
journalctl -u flatpak-update.service                # o que foi atualizado
```

Como desfazer: `sudo systemctl disable --now flatpak-update.timer && sudo rm /etc/systemd/system/flatpak-update.*`.

Observações:

- **GNOME Software continua funcionando:** aberto pela grade, ele mostra e instala atualizações normalmente.
- **Instalação do usuário:** Flatpaks instalados com `--user` ficam de fora. Para eles, um timer do usuário com `flatpak update --user` *(proposta)*.

